# Introduction to IDS/IPS

## IDS vs IPS: Architecture

### Core Concept: Inline vs. Out-of-band Deployment

Field Fact: Which architectural difference is the defining characteristic between IDS and IPS? IPS is deployed inline in the traffic path while IDS is deployed out-of-band on a span or tap. Because IPS is inline, it can block traffic in real-time. Because IDS is out-of-band, it only analyzes a copy of the traffic and alerts passively.

### Technical Overview: Inline IPS Failure Modes

Since an IPS sits directly inline, all network packets must pass through its processor. If the hardware crashes, the network path is physically broken.

The Solution:

- Modern IPS devices use a hardware bypass pair (fail-open bypass switch).
- If the appliance fails without a bypass pair, the network connection is severed. Takeaway: All traffic through that path is dropped, causing an outage.

### Analytical Workflow: Staged IPS Rollouts

A bad firewall rule drops a single port. A bad IPS rule can trigger on normal traffic and drop all connections across your entire network segment.

Analyst Action Plan: Why should IPS rule changes go through a staged rollout starting in detection mode? A poorly written rule could block legitimate traffic and cause an outage. Always roll out rules in "Alert Only" mode first, review the log rates to eliminate false positives, and only enable "Prevent/Block" mode once the rule is validated.

## Detection Methodologies

### Core Concept: Signature vs. Anomaly Detection

Signature-based detection looks for known patterns (e.g., specific file hashes or text strings). Anomaly-based detection compares current behavior against a baseline of normal behavior.

Field Fact: What is the primary weakness of signature-based detection compared to anomaly-based detection? Signature-based detection cannot identify threats that have no existing signature, such as zero-day exploits. If an attacker crafts a new exploit payload, signature engines will miss it completely, whereas anomaly detection will spot the unusual deviation.

### Technical Overview: Stateful Protocol Analysis

Stateful protocol analysis compares network traffic against a built-in protocol state machine. It understands what is supposed to happen next according to RFC standards.

Takeaway: Stateful protocol analysis uses a protocol state machine to identify violations of expected behavior. For example, it will flag an HTTP request that contains a body when the protocol method forbids a body.

### Analytical Workflow: Spotting DNS Tunneling

DNS tunneling is a technique used to leak data or host remote shells via DNS queries, bypass firewalls, and evade simple detection filters.

Analyst Trap - The Volumetric Spike: In the example described, what behavior triggered the DNS tunneler detection using Zeek anomaly analysis? A workstation querying over 15,000 unique domains in one hour, far exceeding the baseline. Normal systems query a small, repetitive set of domains. DNS tunnels generate a unique subdomain (e.g., [base64-data].evil.com) for every chunk of leaked data.

## Network (NIDS) vs Host (HIDS)

### Core Concept: Host-Based Detection (HIDS)

A Host-based Intrusion Detection System (HIDS) monitors the internals of a specific operating system.

Field Fact: Where does a HIDS agent like Wazuh or OSSEC sit in the detection architecture? Directly on each endpoint it protects, monitoring host-level events such as file changes and process creation. HIDS gathers logs directly from the OS, meaning it has visibility into memory, local users, and file systems.

### Technical Overview: Unique HIDS Capabilities

NIDS can see network traffic, but it has no idea what happens once packets are processed by the OS.

HIDS-only capabilities:

- File Integrity Monitoring (FIM): Detects if system binary configurations have been tampered with.
- Process Monitoring: Flags unauthorized processes launching from temp directories.
- Registry Modifications: Inspects changes to Windows registry persistence keys. Takeaway: Monitoring local file integrity, process creation, and registry modifications on a specific host is capability unique to HIDS that NIDS cannot replicate.

### Analytical Workflow: The Encryption Blind Spot

Modern network traffic is increasingly encrypted (HTTPS/TLS, SSH). This represents the ultimate challenge for network monitoring.

Analyst Trap - The TLS Void: What is the major blind spot of NIDS when inspecting modern network traffic? NIDS cannot inspect the contents of encrypted TLS and SSH traffic payloads. If an attacker executes an exploit payload inside an encrypted channel, NIDS only sees scrambled binary traffic. HIDS, however, can see the command after the host decodes it.

## Zeek (Bro) Surveillance

### Core Concept: Transactional Logging

Zeek transforms raw packet capture (PCAP) data into high-fidelity, structured logs.

Field Fact: What is the primary output mechanism that makes Zeek valuable for SOC operations? Zeek writes structured, queryable logs (conn.log, dns.log, ssl.log) describing network activity that feed into SIEMs and analytics pipelines. Rather than keeping raw PCAPs, Zeek saves structured text logs for easy searching.

### Technical Overview: conn.log

The conn.log is Zeek's core log, tracking every TCP, UDP, or ICMP session. It logs IP addresses, ports, duration, status, and traffic volume.

Scenario: Which Zeek log file contains the volume of data (bytes) transferred in each connection? Answer: conn.log tracks the byte count in both directions (orig_bytes and resp_bytes), which is critical for finding large database dumps or data exfiltration.

### Analytical Workflow: Encrypted Metadata Analysis

Just because network payloads are encrypted doesn't mean we are blind. Zeek parses the unencrypted TLS handshake to extract metadata.

Analyst Trap - The Encryption Bypass: How can Zeek detect suspicious activity in encrypted traffic without decrypting the payloads? Zeek analyzes TLS handshake metadata such as JA3 fingerprints, certificates, and SNI values in ssl.log. Attackers using Cobalt Strike or other command-and-control software leave unique TLS negotiation footprints (JA3 hashes) that Zeek records perfectly.

## Alert Triage & Investigation

### Core Concept: Success vs. Failure

A signature alert only means an exploit attempt was made. It does not mean the attacker got in.

Field Fact: What is the most important question to answer during alert triage that is frequently skipped? Did the attack or exploit actually succeed, or was it blocked or ineffective? For example, if an attacker executes a Linux exploit against a Windows IIS server, the alert fires but the exploit is completely ineffective. Always verify target context!

### Technical Overview: Escalating to Endpoint Analysis

Network alerts can point you to compromised machines, but the host itself holds the conclusive truth.

Scenario: A Suricata alert fires for Cobalt Strike beacon activity from a workstation to an external IP. The external IP has multiple threat intel hits. What triage step should happen next? Verdict: Check the host via Wazuh or EDR for suspicious process creation, persistence mechanisms, or data staging. You must log in to the endpoint (or use EDR consoles) to verify if the malware has successfully spawned a shell or achieved persistence.

### Analytical Workflow: Proper Incident Documentation

If you did not write it down, it did not happen. Triage decisions must withstand audit and post-incident review.

Analyst Action Plan: What documentation should every triage decision include? The analyst name, timestamp, reasoning, and supporting evidence. Never close an alert with a simple "benign" or "FP." Always document the IP checks, log lines, or user confirmations that led to your decision.

## Handling False Positives

### Core Concept: Scanner Alert Suppression

Vulnerability scanners (like Nessus, Qualys, or OpenVAS) actively probe your environment, triggering hundreds of signature alerts during scheduled scans.

Field Fact: What is the standard solution for a vulnerability scanner like Nessus triggering IDS alerts during scheduled scans? Add a suppression rule in the IDS that suppresses alerts originating from the scanner IP range. Rather than disabling the security signatures entirely, you selectively tell the system to ignore alerts when they come from designated scanner hosts.

### Technical Overview: Deduplication and Rate Limiting

When a misconfigured service fails to authenticate, it may attempt logon 10,000 times in an hour, generating 10,000 identical alerts.

The Solution:
- Alert Deduplication: Group identical alerts targeting a single destination within a specific time window.
- Takeaway: This technique reduces ticket volume when the same signature fires hundreds of times against a single destination.

### Analytical Workflow: Anomaly False Positives

Anomaly-based detection flags anything that deviates from established thresholds. This means benign systems with high traffic rates will naturally trip the sensors.

Analyst Trap - The Monitor Spike: Why might internal network monitoring traffic like SNMP polls or health checks trigger anomaly-based alerts? The monitoring host generates connection volumes that exceed the established baseline for normal host behavior. Internal monitoring servers continuously query thousands of machines, which looks exactly like active reconnaissance to an anomaly engine.