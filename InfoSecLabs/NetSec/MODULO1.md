# TCP/IP & Protocols

## The OSI & TCP/IP Models

### The OSI Model (Open Systems Interconnection)

- Layer 7: Application - The layer the user interacts with (HTTP, DNS, SMTP, FTP).
- Layer 6: Presentation - Data translation, encryption (SSL/TLS), and compression.
- Layer 5: Session - Establishing, maintaining, and terminating "sessions" between applications.
- Layer 4: Transport - Reliable vs. unreliable delivery. Where TCP and UDP live. Data is packaged into Segments. Ports (like port 80 or 443) operate here.
- Layer 3: Network - IP Addressing and Routing. How data gets from one network to an entirely different network. Data is packaged into Packets. Routers operate here.
- Layer 2: Data Link - MAC Addressing. How data moves between devices on the same local network. Data is packaged into Frames. Switches operate here.
- Layer 1: Physical - The actual cables, radio waves (Wi-Fi), fiber optics, and electrical signals representing 1s and 0s.

### The TCP/IP Model

- Application Layer (Combines OSI Layers 5, 6, 7): HTTP, DNS, SSH.
- Transport Layer (Matches OSI Layer 4): TCP, UDP.
- Internet Layer (Matches OSI Layer 3): IPv4, IPv6, ICMP.
- Network Access Layer (Combines OSI Layers 1, 2): Ethernet, Wi-Fi, MAC addresses.

### Encapsulation

Email example: When you send an email, the data moves down the OSI layers on your computer.

- Your email app (L7) passing data to the Transport layer (L4), which adds a TCP header.
- It goes down to the Network layer (L3), which adds an IP header (Source and Dest IP).
- It goes down to the Data Link layer (L2), which adds a MAC header.
- It finally goes out the Physical layer (L1) as electricity across the wire.

When the receiving server gets the electrical signals, it reverses the process, moving up the layers (De-encapsulation) until the raw email text reaches the Application layer.

## TCP vs UDP

### TCP (Transmission Control Protocol)

TCP is reliable, connection-oriented, and stateful. It guarantees that data will arrive, and in the correct order.

- The 3-Way Handshake: Before ANY data is sent, TCP establishes a strict connection.
    - SYN: Client says "I'd like to talk to you."
    - SYN-ACK: Server says "Okay, I acknowledge your request, let's talk."
    - ACK: Client says "Great, I'm sending data now."
- Reliability: Once connected, every time a computer sends a segment of data, the receiving computer must reply with an acknowledgment (ACK). If the sender doesn't get the ACK (because the packet dropped over Wi-Fi), it will automatically retransmit the data.
- Use Cases: Web Browsing (HTTP/HTTPS), Email (SMTP), File Transfers (FTP/SMB), SSH. If a single byte is missing from a downloaded executable, the file breaks. TCP ensures every byte arrives.

### UDP (User Datagram Protocol)

UDP is unreliable, connectionless, and stateless. It just throws data at the destination and hopes it gets there.

- No Handshake: UDP does not establish a connection first. It just starts sending packets immediately.
- No Acknowledgments: If a packet gets dropped by a router on the internet, it's gone forever. UDP does not resend lost data.
- Why use it? Speed and low overhead. Because there are no handshakes or ACKs, UDP is much faster.
- Use Cases: Live Video Streaming (Zoom, Twitch), VoIP (Phone calls), DNS queries, Online Gaming. If you drop a single frame in a 60fps video game, it doesn't matter, you don't want the game to pause to re-download that old frame. You just proceed to the next current frame.

### Ports

Both TCP and UDP use Ports to direct traffic to the right application on a computer. A computer has 1 IP address, but can have up to 65,535 ports open simultaneously.

- If traffic arrives at IP 192.168.1.10 on TCP Port 80, the OS hands that data to the Web Server software.
- If traffic arrives at TCP Port 22, the OS hands it to the SSH Server software.


## Core Protocols & Services

### DNS (Domain Name System)

- Port: UDP 53 (and sometimes TCP 53)
- Function: The phonebook of the internet. It translates human-readable domain names (infoseclabs.com) into machine IP addresses (104.21.5.12).
- Security Context: Attackers use DNS for Command & Control (C2) beacons, data exfiltration (DNS Tunneling), and redirecting traffic via DNS Spoofing.

### DHCP (Dynamic Host Configuration Protocol)

- Port: UDP 67 (Server) and UDP 68 (Client)
- Function: Automatically assigns IP addresses, Subnet Masks, and Default Gateways to devices when they join a network.
- Security Context: Rogue DHCP servers can assign malicious DNS settings to victim computers, executing Man-In-The-Middle attacks almost invisibly.

### ARP (Address Resolution Protocol)

- Layer: Layer 2 (Data Link) / Layer 3 (Network) boundary.
- Function: Maps an IP address to a physical MAC address on a local network. If computer A (IP 10.0.0.5) wants to talk to the Router (IP 10.0.0.1) on the same Wi-Fi, it shouts "Who has 10.0.0.1?" via ARP. The router replies with its physical MAC address so the switch knows where to send the frame.
- Security Context: ARP Spoofing. An attacker can lie and reply to the ARP request saying they are the router. The victim then sends all their traffic to the hacker instead of the real router.

### ICMP (Internet Control Message Protocol)

- Layer: Layer 3 (Network)
- Function: Used for network diagnostics and error reporting. It is not used for transferring data between systems. The ping command and traceroute commands use ICMP Echo Requests and Echo Replies.
- Security Context: Attackers use ICMP for "Ping Sweeps" to map out live hosts on a network. It can also be abused for ICMP tunneling to bypass firewalls or launch Smurf DDoS attacks.

### HTTP/HTTPS (Hypertext Transfer Protocol)

- Port: TCP 80 (HTTP) and TCP 443 (HTTPS)
- Function: Foundation of the World Wide Web. Facilitates downloading web pages, submitting forms (POST), and API communication. HTTPS adds a layer of encryption (TLS/SSL) to secure the data in transit.
- Security Context: The vast majority of malware C2 traffic hides inside HTTPS, blending in with regular employee web browsing to evade detection by firewalls.

