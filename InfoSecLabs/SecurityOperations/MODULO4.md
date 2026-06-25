# Vulnerability Scanning

## The Vulnerability Lifecycle

### Core Concept: CVE Identifiers

When a vulnerability is discovered, it is assigned a unique tracking number to avoid confusion between vendors and security companies.

Field Fact: What is the primary purpose of a CVE (Common Vulnerabilities and Exposures) identifier? To provide a universal reference number that uniquely identifies a specific vulnerability across all vendors and databases. This ensures everyone refers to the exact same flaw using a uniform code (e.g., CVE-2021-44228).

### Technical Overview: CVSS Limits

The Common Vulnerability Scoring System (CVSS) provides a score from 0.0 to 10.0 representing the severity of a flaw. However, a CVSS score is not a prioritization score.

Why CVSS is not enough:
- A CVSS 10.0 flaw on an internal database behind firewalls may be less urgent than a CVSS 7.5 flaw on a public-facing web server.
- Takeaway: Context like network exposure, asset value, and available mitigations can make a lower-scored vulnerability more urgent than a higher-scored one.

### Analytical Workflow: Prioritizing with CISA KEV

Faced with thousands of vulnerabilities, SOC teams must focus on what attackers are actually doing in the wild today.

Analyst Action Plan: What is CISA's Known Exploited Vulnerabilities (KEV) catalog and why does it matter for prioritization? It lists vulnerabilities confirmed to be actively exploited by attackers, meaning those should be patched immediately regardless of CVSS score. If an exploit is actively being used in real campaigns, the threat level is maximum, regardless of academic complexity ratings.

## Authenticated vs Unauthenticated

### Core Concept: Unauthenticated Scanner Limitations

An unauthenticated scan scans the target over the network, probing open ports and analyzing service banners.

Field Fact: What is the main limitation of unauthenticated scanning compared to authenticated scanning? It cannot inspect installed packages, patch levels, registry keys, or internal configurations that are only visible after logging in. If a service banner does not leak version information, the unauthenticated scanner will miss internal OS vulnerabilities entirely.

### Technical Overview: Scanning Account Security

To run an authenticated scan, you must save credentials in the vulnerability scanner's configurations.

Analyst Trap - The Admin Vault Hazard: Why should you use a dedicated scanning account instead of domain admin credentials for authenticated scans? If the scanner platform is compromised, domain admin credentials in its configuration become a catastrophic breach vector. Always use a dedicated, restricted service account with read-only permissions on target systems.

## Analytical Workflow: Supplementing with DAST

Authenticated scanners are great at checking if a Windows server has the latest security patch. However, they are blind to custom web application logic.

Takeaway: Authenticated scanning checks the OS and service layer but cannot analyze application logic, input handling, or business logic flaws. To find vulnerabilities like SQL Injection, XSS, or IDORs in custom code, you must supplement infrastructure scans with Dynamic Application Security Testing (DAST).

## Decoding CVSS Scores

### Core Concept: Scope (Changed vs Unchanged)

In CVSS v3.1, the Scope (S) metric indicates whether a vulnerability in one software component can affect resources beyond its own security authority.

Field Fact: What does the Scope:Changed metric in CVSS v3.1 indicate about a vulnerability? The vulnerability can affect resources beyond its original security scope, such as a container escape affecting the host. If a vulnerability in a virtual machine allows an attacker to execute commands directly on the physical hypervisor, the Scope has Changed.

### Technical Overview: Environmental Scores

The CVSS score is divided into three groups: Base (the exploit complexity and impact), Temporal (exploit status and patch availability), and Environmental.

Takeaway: The Environmental Score is the most relevant but least commonly used CVSS metric because it accounts for your specific asset value, network placement, and compensating controls but requires organization-specific analysis to calculate. It is rarely provided by default scanners because it requires manual input regarding your network setup.

### Analytical Workflow: Prioritizing with Asset Criticality

A raw CVSS score does not understand what business function a server performs.

Analyst Trap - The Out-of-Context Score: A vulnerability scores CVSS 6.1 due to required user interaction but it affects your payment processing system. How should you prioritize it? Treat it as higher priority than the score suggests because business context and asset criticality override the base CVSS score. A CVSS 6.1 on a critical transaction database is far more dangerous than a CVSS 9.8 on an isolated test sandbox.

## Nessus Operations

### Core Concept: Verifying Credential Success

When you run a credentialed scan, Nessus logs in to the target system to read registry settings and file versions. You must always verify if the login actually succeeded.

Field Fact: What should you check in Nessus results to verify that a credentialed scan was actually successful on each host? Plugin 19506 shows the Credentialed scan field, which should say yes for reliable patch assessment results. If it says no, Nessus fell back to an unauthenticated scan, making the results incomplete.

### Technical Overview: Authentication Failures

If Nessus cannot log in, it flags it under a specific plugin.

Takeaway: Nessus plugin 21745 (Authentication Failure) indicates the scanner could not authenticate to that target, meaning the scan results for that host are incomplete and unreliable. If you see this plugin in your scan report, you must investigate the saved credentials.

### Analytical Workflow: Scan Policy Customization

Using a default "Basic Network Scan" policy for every target in your enterprise leads to massive noise and potential target crashes.

Analyst Action Plan: Why should you create separate scan policies instead of using a single default policy for all scanning? Different environments and targets require different configurations like port ranges, credentials, plugin families, and assessment types. For instance, scanning sensitive OT/SCADA devices requires disabling invasive plugins that could cause industrial controller crashes, whereas scanning DMZ servers requires aggressive external port testing.

## Analyzing Scan Reports

### Core Concept: The Prioritization Pitfall

Vulnerability management platforms list findings sorted by CVSS score. Fixing them in this exact order is a critical operational mistake.

Field Fact: Why is treating all Critical CVSS findings equally a mistake in vulnerability management? Context like network exposure, asset criticality, and exploitability means a Critical finding on an air-gapped test server poses less risk than a lower-scored finding on a public-facing authentication system. Always evaluate the target's environment before assigning developer hours.

### Technical Overview: Manual Verification

Vulnerability scanners make mistakes. They read file versions or banner numbers and assume a flaw exists, even if a compensating control blocks it.

Takeaway: When a scanner reports a Critical vulnerability but you suspect it might be a false positive, manually verify by checking the actual installed version and configuration on the target system before assigning it for remediation. Don't open tickets for bugs that don't exist.

### Analytical Workflow: Measuring Remediation Performance

Remediation tracking is essential to determine if your security posture is improving.

Analyst Action Plan: What is Mean Time to Remediate (MTTR) and why should it be tracked as a metric? The average time between vulnerability discovery and fix, which measures whether your remediation process is improving or degrading over time. A rising MTTR indicates friction in your patching pipeline.

## Patch Management Strategy

### Core Concept: Rollback Planning

Installing security updates on live production databases carries substantial operational risk.

Field Fact: Why is it important to have a rollback plan before deploying patches to production systems? Patches can break application functionality or cause system instability, and a rollback plan lets you quickly restore the previous state if problems occur. Never patch without a backup or snapshot.

### Technical Overview: Compensating Controls

Sometimes, a critical software vendor has not released a patch, or updating the application requires a week of offline testing. In these scenarios, you deploy compensating controls.

Definition: Security measures that reduce risk when a vulnerability cannot be patched immediately, such as WAF rules or network segmentation, used as temporary measures until the patch can be applied.

### Analytical Workflow: The Backporting Headache

Security scanners read version numbers. On Linux systems, this often leads to massive false positive rates.

Analyst Trap - The Backporting Illusion: Why does Linux backporting of security fixes complicate version-based vulnerability scanning? The package version number stays the same while the vulnerability is fixed, so scanners reading version banners may report false positives. Enterprise Linux maintainers (like Red Hat or Debian) backport security fixes into older package versions without incrementing the main version number, confusing basic scanners.

## Understanding CVEs

### Core Concept: CVE vs. Vendor Advisory

A CVE database entry (from MITRE or NVD) provides a standardized ID and a brief description of the flaw. It rarely tells you how to patch it.

Field Fact: What is the relationship between a CVE entry and a vendor security advisory? The CVE provides the identifier and high-level description while the vendor advisory provides actionable remediation details like specific patch versions and KB numbers. Always go to the vendor's site (like Microsoft Security Update Guide or Red Hat Advisories) for the actual fix.

### Technical Overview: CVSS Variance

A single vulnerability can affect multiple products and configurations.

Takeaway: The same vulnerability can have different impact depending on affected versions and configurations, resulting in separate CVE entries with different scores. For instance, a bug that causes a Denial of Service (DoS) in one version might allow Remote Code Execution (RCE) in another.

### Analytical Workflow: The XZ Backdoor Case Study

In 2024, the security community was shocked by CVE-2024-3094.

Analyst Trap - Supply Chain Compromise: Why was CVE-2024-3094 (XZ Utils Backdoor) significant beyond a typical vulnerability? It was a deliberate supply chain compromise where a malicious contributor inserted a backdoor, highlighting the risk of trusting open-source dependencies without verification. A malicious actor spent years building trust in the open-source project to inject a backdoor into sshd via liblzma.

## Risk Response Strategies

### Core Concept: Risk Acceptance vs. Avoidance

When faced with a vulnerability, you have multiple strategic options.

- Risk Avoidance: You completely eliminate the risk. For example, if a legacy server is vulnerable, you shut it down and decommission it.
- Risk Acceptance: You identify the risk, determine that the cost of patching outweighs the business risk, and choose to live with it. Takeaway: Acceptance acknowledges the risk and chooses to live with it with documented rationale, while avoidance eliminates the risk entirely by removing the threat or asset.

### Technical Overview: The Limits of Risk Transfer

Many organizations buy cyber insurance or outsource their server management to transfer risk.

Field Fact: When is risk transfer an appropriate response, and what limitation does it have? Transfer shifts the financial burden to another party like insurance but does not eliminate the risk itself, so you still deal with operational impact and reputational damage. If your client database is leaked, insurance might pay the fine, but your brand value is still destroyed.

### Analytical Workflow: The Documentation Requirement

Choosing to accept a risk is not an excuse to do nothing.

Analyst Trap - The Unsigned Waiver: Why must risk acceptance be a documented decision rather than simply not patching? Undocumented acceptance looks identical to negligence to auditors, and documentation demonstrates due diligence with rationale, decision maker, and review date. Risk acceptance must be formally signed off by a business owner who owns the budget and understands the consequences.

