# Web App Scanning (DAST)

## DAST Fundamentals

### Core Concept: Black-Box Analysis

DAST is often called "black-box" because the scanner acts exactly like an external attacker.

Field Fact: Why is DAST considered a black-box testing approach compared to SAST? DAST tests the running application from the outside without access to source code, mirroring how an external attacker would probe the application. It interacts with HTTP ports, endpoints, and input fields.

### Technical Overview: Passive vs. Active Scans

A professional DAST scan is executed in stages: Passive first, then Active.

Why passive first?
- Passive scanning inspects headers, cookie flags, and HTML structures without sending attack payloads.
- Takeaway: Passive scanning identifies information disclosure, missing headers, and cookie issues without sending attack payloads that might trigger WAF or IDS alerts. It is a stealthy reconnaissance step.

### Analytical Workflow: DAST Weaknesses

While DAST is great at spotting input validation flaws like SQLi or XSS, it has structural limitations.

Analyst Trap - The Logic Gap: What type of vulnerability is DAST least likely to detect effectively? Business logic flaws that require chaining multiple application actions together, like purchasing items for negative quantities. DAST engines cannot think like humans to understand complex business processes.

## OWASP Top 10 Deep Dive

### Core Concept: Broken Access Control

Broken Access Control occupies the top spot in the OWASP list.

Field Fact: Why is Broken Access Control the most commonly found vulnerability in web application assessments? It encompasses many common patterns like IDOR, privilege escalation, and missing authorization checks that developers frequently overlook during implementation. Unlike SQLi, access control requires manual logical validation at every endpoint.

### Technical Overview: Insecure Design

Insecure Design (A04) is distinct from other OWASP categories because it is not about programming bugs.

Takeaway: It addresses architectural and design flaws that cannot be fixed by changing code, such as a password reset flow that emails passwords in plaintext. If the design itself is flawed, even a perfectly secure coding implementation is vulnerable.

### Analytical Workflow: Parameterized Queries

SQL Injection remains a critical threat, but the remediation is simple.

Analyst Action Plan: Why is parameterized queries the primary defense against SQL injection rather than input validation? Parameterized queries separate the SQL command structure from user data at the database level, while input validation can be bypassed through encoding and edge cases. By binding input parameters, the database engine guarantees user input is never executed as database commands.

## OWASP ZAP Mastery

### Core Concept: Attack Mode Configuration

ZAP has multiple modes of operation (Safe, Protected, Standard, Attack).

Field Fact: When should you use ZAP Attack mode instead of Standard mode? Only against test environments or with explicit permission, because Attack mode removes all restrictions and attempts to exploit vulnerabilities aggressively. Running Attack mode on a production application can result in severe data loss or system downtime.

### Technical Overview: AJAX Spider vs. Traditional Spider

Classic web spiders only parse HTML anchor tags (href). Modern web applications use frameworks like React, Angular, or Vue to build single-page apps (SPAs).

Takeaway: The Traditional Spider only follows HTML links and form actions in static HTML, while the AJAX Spider uses a real browser to execute JavaScript and discover dynamically generated content. Without the AJAX Spider, your DAST tool will miss the majority of SPA routes.

### Analytical Workflow: Verifying Alerts

When ZAP flags a vulnerability, you must investigate the evidence before writing a report.

Analyst Action Plan: What information does ZAP provide in its alert details that helps you verify whether a finding is a false positive? The full HTTP request and response including the exact payloads sent and the server responses, letting you see the actual evidence of the vulnerability. Always inspect the response bytes to confirm the exploit succeeded.

## Advanced Spidering

### Core Concept: Forced Browsing & Wordlists

Forced browsing means requesting URLs that are not linked from the application.
Scenario: Why might automated crawlers miss an admin panel that exists on a web application? Verdict: If no links point to the admin panel from the main application, automated crawlers that follow links will never discover it.

To find hidden directories, you should use directory brute-forcing tools (like ffuf or gobuster) with a wordlist.

Field Fact: What is the benefit of using a tiered approach to forced browsing wordlists? Starting with small common wordlists for quick wins, then using framework-specific lists, then comprehensive lists ensures efficient discovery without wasting time on irrelevant paths.

### Technical Overview: Analyzing Client-Side JS

Modern Single Page Applications (SPAs) load their entire logic into the user's browser via compressed JavaScript bundles.

Analyst Trap - The JavaScript Goldmine: What should you check in a web application JavaScript bundle that could reveal security-relevant information? Hardcoded API endpoints, API keys, tokens, unreleased features, and developer comments that reveal internal application structure. Always download and format (.js) files to extract hidden API routes.

## Active Scanning Mechanics

### Core Concept: Multi-Payload Testing

A vulnerability scanner does not just send a single test query to a parameter.

Field Fact: Why does a DAST scanner send dozens of different SQL injection payloads instead of just one? Each payload variation tests a different aspect of SQL injection like error-based, union-based, boolean-blind, and time-based blind, so the aggregate of tests determines if the parameter is vulnerable. A single payload might fail due to specific database filters while another succeeds.

### Technical Overview: Active Scanning and WAFs

Running an active scan against a target protected by a Web Application Firewall (WAF) requires careful analysis.

Scenario: What happens to scan accuracy when a WAF starts blocking active scan requests? Verdict: Blocked payloads never reach the application, so the scanner reports false negatives for vulnerabilities that actually exist but were prevented from being detected. You will get a false sense of security unless you scan with the WAF bypassed or temporarily disabled.

### Analytical Workflow: Scan Verification

Once an active scan finishes, you must analyze its operational metrics before relying on the final report.

Analyst Action Plan: Why is scan verification important after an active scan completes? Checking request counts, tested injection points, and authentication status reveals whether the scan actually covered the application or missed portions due to early termination or session expiry.

## Identifying DAST False Positives

### Core Concept: Version-Based False Positives

Vulnerability scanners frequently check application headers and report critical vulnerabilities based on version numbers alone.

Field Fact: Why are version-based false positives particularly common in DAST scanning? Linux distributions often backport security fixes without changing the version number, so the scanner flags a version as vulnerable when the actual installed package has the fix applied. Always perform credentialed or local checks to verify the package patch level.

### Technical Overview: Handling Unverified Findings

Sometimes, you will see a critical alert in the scanner console but you cannot manually reproduce the exploit due to time limits or environmental constraints.

Action Plan: What is the correct approach when a scanner reports a Critical finding that you cannot manually verify? Verdict: Mark it as unverified in your report and explain that manual verification was not possible, rather than including it as a confirmed finding or discarding it entirely. This maintains transparency with the stakeholders.

### Analytical Workflow: Recalibrating Severity

Not all scanner severity ratings are accurate. A vulnerability's threat level depends entirely on its context and exploit requirements.

Analyst Trap - Reflected XSS Ratings: How should you handle a confirmed XSS vulnerability in a search parameter that requires the victim to click a crafted link? Recalibrate the severity to Medium or Low based on the required user interaction and limited impact context, rather than accepting the scanner default High rating. Since the exploit requires active user interaction (clicking a link), the immediate risk is lower than an unauthenticated remote execution vulnerability.

## Effective Remediation Reporting

### Core Concept: Sorting by Remediation Priority

When presenting a list of vulnerabilities to a client's IT team, you should not simply copy the scanner's CVSS severity order.

Field Fact: Why should findings in a remediation report be sorted by remediation priority rather than by CVSS severity? Business context means the most urgent finding might not have the highest CVSS score, so sorting by priority ensures the IT team addresses the most impactful issues first. An internal CVSS 9.8 vulnerability behind three firewalls is less urgent than a CVSS 7.5 vulnerability on the main gateway.

### Technical Overview: Writing for Developers

Developers need precise details to locate and patch bugs.

What to include:

- Management Summary: High-level risk and business impact.
- Developer finding details: Specific endpoint URLs, the exact vulnerable parameter, evidence of the vulnerability, and actionable remediation steps including code-level guidance. Verdict: Provide specific, localized code examples and vulnerable inputs so developers don't have to guess.

### Analytical Workflow: CVSS Vector Strings

A raw numeric score (e.g., 9.8) does not tell the reader how the vulnerability was calculated.

Analyst Action Plan: Why should you include the CVSS vector string in a report rather than just the numeric score? The vector string shows exactly how the score was calculated, allowing readers to understand the specific metrics and context that produced the score. The string (e.g., CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:C/C:H/I:H/A:H) reveals network requirements, privilege levels, and scope changes.