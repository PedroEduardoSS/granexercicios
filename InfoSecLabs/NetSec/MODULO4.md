# Firewalls & IDS/IPS

## Firewalls: The First Line of Defense

A firewall is a network security device that monitors and filters incoming and outgoing network traffic based on an organization's previously established security policies. It acts as the barrier between a trusted internal network and an untrusted external network (like the internet).

### Types of Firewalls

Firewalls have evolved significantly over the years, moving higher up the OSI model to provide better security.

1. Packet-Filtering Firewalls (Stateless)

- Operating Layer: Layer 3 (Network) & Layer 4 (Transport).
- How it works: It looks at each individual packet in isolation. It checks the Source IP, Destination IP, Protocol, Source Port, and Destination Port against a list of rules (Access Control Lists or ACLs).
- Pros: Extremely fast and requires very little CPU power.
- Cons: "Stateless" means it has no memory of the connection. If you allow traffic OUT on port 80, you also have to manually write a rule allowing traffic IN from port 80 so the website can reply. This is tedious and insecure. It also cannot look inside the packet payload, so a malicious command sent over an allowed port (like HTTP 80) will pass right through.

2. Stateful Inspection Firewalls

- Operating Layer: Layer 3 & Layer 4 (with session tracking).
- How it works: This is the standard modern firewall. It tracks the state of active connections. If an internal user initiates an outbound HTTP request, the firewall remembers this "state." When the web server replies, the firewall automatically allows the inbound traffic because it belongs to an established, tracked session.
- Pros: Much more secure than stateless. You only need to write rules for the initiation of traffic.
- Cons: Still primarily focuses on IP addresses and ports, not the actual application data.

3. Next-Generation Firewalls (NGFW)

- Operating Layer: Up to Layer 7 (Application).
- How it works: NGFWs (like Palo Alto, Fortinet, Cisco Firepower) do everything a stateful firewall does, but they also perform Deep Packet Inspection (DPI). They look inside the payload of the packet.
- Pros: They understand applications, not just ports. An NGFW can differentiate between a user browsing Facebook (which might be allowed) and a user trying to play a game on Facebook (which might be blocked), even though both use HTTPS on Port 443. They also include built-in antivirus, intrusion prevention, and SSL decryption capabilities.

### Default Deny Posture

The most fundamental rule of configuring any firewall is Implicit Deny or Default Deny. This means you must explicitly define exactly what traffic is allowed. If traffic does not match an explicit "Allow" rule, the firewall drops it by default.

- Deny vs Reject:
    - Drop/Deny: The firewall silently discards the packet. The sender gets no response, making it look like the IP address doesn't even exist. This is best practice for internet-facing interfaces to frustrate scanners.
    - Reject: The firewall discards the packet but sends a polite "Connection Refused" (ICMP Destination Unreachable or TCP RST) back to the sender. This is sometimes used on internal networks for easier troubleshooting.


## Intrusion Detection vs. Intrusion Prevention

### Intrusion Detection System (IDS)

An IDS is a passive monitoring system. It is the security equivalent of a burglar alarm.

- How it connects: It is normally connected to a "SPAN port" or "Mirror port" on a network switch. The switch sends a copy of all network traffic to the IDS. The original traffic flows uninterrupted to its destination.
- Action: When the IDS detects a threat (e.g., an SQL injection payload inside an HTTP request), it generates an alert and sends it to the SIEM for the SOC team to review.
- Pros: Because it sits out-of-band (analyzing a copy), an IDS can never accidentally block legitimate traffic (no false positive blocks). If the IDS crashes, network traffic continues to flow normally.
- Cons: It cannot stop an attack. It only alerts you that an attack is currently happening or has already succeeded. The attack traffic still reaches the target server.

### Intrusion Prevention System (IPS)

An IPS is an active defense system. It is the security equivalent of an armed guard at the door.

- How it connects: It is deployed in-line with the traffic. All traffic must physically flow through the IPS device before reaching the internal network or servers.
- Action: When the IPS detects a threat, it doesn't just generate an alert; it actively drops the malicious packets, resets the TCP connection, and blocks the attacker's IP address in real-time.
- Pros: It actually stops attacks before they reach the vulnerable server.
- Cons: Because it sits in-line, a false positive is disastrous. If the IPS mistakes a legitimate customer uploading a legitimate document for a malware attack, it drops the connection, causing a business outage. Furthermore, if an in-line IPS crashes, all network traffic stops unless it fails "open" (which brings its own security risks).

### Detection Methods

Both IDS and IPS use two primary methods to find evil:

- Signature-Based: Works exactly like traditional antivirus. It looks for a specific string of bytes (a signature) known to belong to a specific threat, like the "EternalBlue" exploit. It is highly accurate but cannot detect brand new, never-before-seen (Zero-Day) attacks because no signature exists yet.
- Anomaly/Behavior-Based: Uses machine learning and statistical baselines to learn what "normal" network traffic looks like over several weeks. If suddenly the database server starts downloading 50GB of data at 3 AM (which it has never done before), the system flags it as anomalous. This can catch Zero-Day attacks, but suffers from a much higher rate of False Positives because networks are inherently unpredictable.