# Phishing & Email Analysis

## Email Anatomy: RFC 5322

### Core Concept: Envelope vs. Headers

An email has two components: the envelope (used for server-to-server routing) and the headers (what is displayed to the user).

Field Fact: What is the difference between the SMTP envelope and the RFC 5322 headers? The envelope contains the MAIL FROM used during transmission while the headers contain the From shown to the recipient. The mail server processes the email based on the envelope details, whereas the mail client displays the header From (which can be completely different).

### Technical Overview: The Spoofing Loophole

Because the SMTP protocol treats the headers as raw content, it does not validate that the display headers match the sender.

Why From is untrusted:

- The From header is set by the client.
- Takeaway: The From header is set by the email client and can be trivially spoofed by anyone during SMTP transmission. Never rely on the display name or From header alone to verify legitimacy.

### Analytical Workflow: Tracing the Received Chain

Every mail server that handles an email appends a Received header to the very top of the header section.

Analyst Action Plan: How do you correctly read the chain of Received headers to trace an email path? Read from bottom to top where the bottom-most header was added by the first server handling the email. The bottom-most server indicates the actual source of the message, while the top-most server is the final receiving boundary.

## Authentication: SPF, DKIM, DMARC

### Core Concept: SPF Limitations

Sender Policy Framework (SPF) publishes a DNS TXT record declaring which IPs can send mail on behalf of a domain.

Field Fact: Why is SPF alone insufficient to prevent email spoofing? SPF only validates the envelope MAIL FROM domain, not the header From that recipients actually see. An attacker can send a mail with their own domain in the envelope (passing SPF checks) but display your company domain in the From header, tricking the recipient.

### Technical Overview: DKIM and Forwarding

DomainKeys Identified Mail (DKIM) adds a cryptographic signature to the email headers, verified using a public key published in DNS.

Forwarding Survival:

- When an email is forwarded, the forwarding server's IP address changes, which causes SPF to fail.
- Takeaway: DKIM signatures are part of the email content and survive forwarding while SPF fails because the forwarding server IP is not authorized. As long as the content has not changed, the DKIM signature remains valid.

### Analytical Workflow: DMARC Alignment

MARC ties SPF and DKIM together. It introduces the critical requirement of Alignment.

Analyst Action Plan: What does DMARC alignment ensure and why is it important? It verifies that the domain in the From header matches the domain validated by SPF or DKIM, preventing cross-domain spoofing. Without alignment, an email passing SPF for attacker.com but showing ceo@company.com would be allowed into the inbox.

## Investigation 1: The CEO Fraud

### Core Concept: SPF Bypasses in BEC

In this investigation, a finance director wired $340,000 to fraud accounts because the From header displayed the CEO's email.

Field Fact: Why did the phishing email pass SPF even though it was a spoofed message? The attacker configured SPF for their own domain secure-wire.net, so SPF validated against the attacker domain not the CEO domain. The receiving server checked the envelope domain secure-wire.net (which passed) but did not check or enforce DMARC alignment with the CEO's display domain.

### Technical Overview: Campaign Timing

Threat actors do not launch CEO fraud campaigns at random times. They perform active reconnaissance on the executive's schedule.

Timing Patterns:

- Attackers review social media to find when executives are out of the office or traveling.
- Takeaway: They attack when the executive is traveling or unavailable and often target Fridays or end-of-quarter periods. Fridays are preferred because banking delays make reversals much harder to execute.

### Analytical Workflow: The Ultimate Preventive Control

While email authentication (DMARC) is essential, the final line of defense against financial fraud is an out-of-band verification policy.

Analyst Action Plan: What single control would have prevented the entire financial loss in this case? A policy requiring a phone call to the requesting executive using a known directory number for any financial request. Never call numbers listed in the suspicious email itself; always verify using an internal directory.

## Investigation 2: Malicious Attachment

### Core Concept: Bypassing Filters via Compromised Accounts

If an attacker compromises a legitimate email marketing platform account, the phishing message will carry valid security headers.

Field Fact: Why did the phishing email pass SPF, DKIM, and DMARC despite being malicious? The attacker compromised a legitimate email marketing account so the email was genuinely sent from authorized infrastructure. The headers aligned perfectly because the email was sent by authorized servers, proving that authentication checks cannot verify intent.

### Technical Overview: Office Document Structure

Microsoft Office documents (.docx, .xlsx) are actually ZIP archives containing XML directories.

Verification Technique:

- Rename the file (e.g., invoice.docx to invoice.zip).
- Extract the folder structure.
- Takeaway: Office documents are ZIP archives containing XML files, and extracting reveals embedded macros (such as vbaProject.bin) and content structure.

### Analytical Workflow: Extracting Indicators of Compromise (IOCs)

Once you analyze malware in a sandbox, you must translate the results into shareable indicators.

Analyst Action Plan: What types of IOCs should be extracted from a malicious email attachment analysis? File hashes, download URLs, C2 IP addresses, behavioral patterns, and persistence mechanisms. These are published to security systems to block similar attacks across the organization.

## Investigation 3: The Credential Harvest

### Core Concept: Phish-Resistant Authentication

Standard Multi-Factor Authentication (SMS codes, TOTP app codes) can be intercepted by real-time phishing proxies (like Evilginx).

Field Fact: Why are phishing-resistant authenticators like FIDO2 more effective than TOTP against credential harvesting? FIDO2 cryptographically validates the domain so it will not authenticate on a fake login page while TOTP codes can be captured and relayed. Under FIDO2, the browser refuses to sign the authentication request if the origin URL does not match the registered domain.

### Technical Overview: Attacker Redirections

After entering credentials, victims are frequently redirected to the legitimate target service.

The Reason:

- Redirecting the user to the real service maintains the illusion that the login was a simple technical glitch.
- Takeaway: It makes victims believe the login was a technical glitch rather than a phishing attack, delaying detection. This buys the attacker time to configure backdoors before the password is changed.

### Analytical Workflow: Investigating Post-Compromise Actions

Once an attacker gains credential access, they immediately perform actions to maintain persistence and hide their presence.

Analyst Action Plan: What post-compromise actions should you check for in a credential harvesting incident? Email forwarding rules, inbox rules that hide evidence, OAuth app grants, and active sessions on compromised accounts. Attackers regularly create inbox rules to auto-delete messages containing words like "phish" or "hack" so you do not receive alerts.

