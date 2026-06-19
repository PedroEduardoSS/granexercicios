# Network Attacks (Spoofing, MITM)

## ARP Spoofing & Poisoning

### How Normal ARP Works

As a reminder, ARP operates at Layer 2 (Data Link). When your computer (IP: 10.0.0.5) wants to send a packet to the internet, it must first send it to your Default Gateway (the Router, IP: 10.0.0.1). However, switches don't understand IP addresses; they only understand physical MAC addresses.

- Your computer shouts out to the entire network: "Who has IP 10.0.0.1? Tell 10.0.0.5."
- The Router (10.0.0.1) replies directly to your computer: "I am 10.0.0.1, and my MAC address is AA:BB:CC:DD:EE:FF."
- Your computer saves this mapping in its temporary ARP Cache.

### The Attack: ARP Spoofing

The flaw in ARP is that an attacker can send out an ARP Reply even if no one asked for it (this is called a Gratuitous ARP). Furthermore, devices will gladly accept this unsolicited reply and update their ARP cache without any verification.

- The Scenario: Let's say the Hacker's MAC address is 66:66:66:66:66:66.
    - The attacker sends a forged ARP Reply to the Victim (10.0.0.5) saying: "Hey, the Router (10.0.0.1) is now at MAC 66:66:66:66:66:66."
    - The Victim's computer blindly trusts this and updates its ARP cache.
    - The attacker also sends a forged ARP Reply to the true Router (10.0.0.1) saying: "Hey, the Victim (10.0.0.5) is now at MAC 66:66:66:66:66:66."
    - The Router blindly trusts this and updates its ARP cache.

### The Result: Man-in-the-Middle (MITM)

Because the attacker has "poisoned" the ARP cache of both the Victim and the Router, all traffic flowing between them now routes directly through the attacker's machine.

The attacker can now:

- Packet Sniff: Read unencrypted traffic (HTTP, FTP, Telnet) in real-time, stealing passwords and session cookies.
- Packet Modify: Alter the traffic in transit (e.g., replacing a legitimate software download with a malware payload).
- Denial of Service (DoS): Simply drop all the packets from the victim, disconnecting them from the internet (a "Blackhole" attack).

Note: ARP Spoofing only works on the LOCAL network (LAN). An attacker in Russia cannot ARP Spoof a victim in New York, because MAC addresses do not cross routers.

## DNS Spoofing & Hijacking

### Local DNS Spoofing (via MITM)

This is usually the next step after an attacker has successfully executed an ARP Spoofing Man-in-the-Middle attack.

- The victim types www.bank.com into their browser.
- he victim's computer sends a DNS Query asking for the IP of bank.com.
- Because the attacker is actively performing a MITM attack via ARP Spoofing, that DNS Query flows through the attacker's machine.
- The attacker intercepts the query and sends a fake DNS Reply back to the victim, claiming that bank.com is located at the Attacker's IP address (e.g., 10.0.0.66).
- The victim's browser connects to the Attacker's web server, which is hosting a perfect, pixel-for-pixel fake clone of the bank's login page.

The victim sees www.bank.com in the URL bar, but they are secretly typing their credentials into the attacker's database.

### DNS Cache Poisoning (Server-Side)

This is a much larger scale attack. Instead of attacking a single victim, the attacker targets a recursive DNS server (like the one operated by your ISP, or even a public one like Google's 8.8.8.8).

The attacker exploits vulnerabilities in the DNS server software (like BIND) to inject fake records into the server's cache. If successful, everyone who relies on that DNS server will be redirected to the malicious site when they try to visit bank.com. This affects thousands or millions of users simultaneously.

### Defending Against DNS Attacks

Because classic DNS operates over unencrypted UDP Port 53, it is highly vulnerable to tampering.

- DNSSEC (DNS Security Extensions): Adds cryptographic digital signatures to DNS records. When your computer looks up bank.com, the true owner of bank.com signs the response with a private key. Your computer verifies the signature with a public key. If an attacker modifies the IP address in transit, the signature breaks, and your computer drops the fake response.
- DoH / DoT (DNS over HTTPS / TLS): Encrypts the entire DNS conversation so a local MITM attacker cannot even see what domain you are querying, let alone modify the response.

## Network Denial of Service (DoS)

### Layer 4: SYN Flood

This attack exploits the stateful nature of the TCP 3-Way Handshake.

- The attacker sends a massive flood of SYN requests (Step 1) to a web server.
- The server replies to each with a SYN-ACK (Step 2) and leaves a "half-open" connection waiting in its memory for the final ACK (Step 3).
- The attacker intentionally never sends the final ACK. Sometimes they even spoof the source IP address of the initial SYN, so the server is sending the SYN-ACK to a random, offline IP.
- The Result: The web server's memory fills up with thousands of half-open connections. Eventually, it runs out of resources and drops legitimate new connections from real customers.

### Layer 3 & 4: Amplification/Reflection Attacks

Attackers love "Amplification" attacks because they act as a force multiplier. They send a tiny request and generate a massive response aimed at the victim. This relies on connectionless UDP protocols.

- The Smurf Attack (ICMP):
    - The attacker sends an ICMP Echo Request (Ping) to a network's broadcast address (e.g., 192.168.1.255).
    - However, the attacker spoofs the source IP Address of the ping to be the Victim's IP address.
    - Every single computer on that network (maybe 200 machines) receives the ping, and all 200 simultaneously reply with an ICMP Echo Reply directed at the Victim.
    - One tiny packet from the attacker generated 200 packets hitting the victim.
- DNS/NTP Amplification:
    - The attacker sends a tiny UDP DNS Query (like "give me all records for this domain") to an open DNS server on the internet.
    - They spoof the source IP to be the Victim's IP.
    - The DNS server generates a massive response (sometimes 50x larger than the request) and blasts it at the Victim.
    - Using a botnet, the attacker directs thousands of open DNS servers to reflect massive chunks of traffic at the victim, saturating their internet connection completely.