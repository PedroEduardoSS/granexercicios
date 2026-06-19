# Traffic Analysis (Wireshark)

## Navigating Wireshark

### The GUI Layout

When you open a packet capture (PCAP) in Wireshark, the interface is divided into three main panes:

- Packet List Pane (Top): This pane displays a summary of each packet captured. By default, it shows the packet number, time, source IP address, destination IP address, protocol, length, and detailed information. Clicking on a packet in this pane controls what is displayed in the other two panes.
- Packet Details Pane (Middle): This pane shows the current packet, broken down layer by layer, closely mirroring the OSI and TCP/IP models. You can expand each layer (e.g., Ethernet II, IPv4, TCP) to see the specific hex values and how Wireshark interprets them.
- Packet Bytes Pane (Bottom): This pane displays the raw, uninterpreted data of the selected packet in hexadecimal and ASCII format. When you select a field in the Details Pane, the corresponding bytes are highlighted in the Bytes Pane.

### Basic Navigation and Sorting

- Time Column: By default, the Time column shows the seconds since the beginning of the capture. You can change this via `View -> Time Display` Format to show the actual Time of Day (e.g., `14:32:05.123`).
- Sorting: You can click on the headers in the Packet List pane to sort by Source IP, Destination IP, Protocol, or Length. This is useful for quickly finding the largest packets or sorting traffic by protocol.
- Finding Packets: You can use Ctrl+F (or Cmd+F) to search for specific strings, hex values, or display filters within the current capture.

### Color Coding

Wireshark uses color-coding to help you quickly identify different types of traffic.

- Green: Typically, HTTP or routing traffic.
- Light Blue: UDP traffic (like DNS lookups).
- Dark Blue/Purple: TCP traffic.
- Black/Red: Packets with errors, warnings, or bad checksums (e.g., a TCP Retransmission or out-of-order packet).

Note: You can customize these coloring rules in `View -> Coloring Rules`.

## Mastering Display Filters

PCAP files often contain hundreds of thousands or even millions of packets. Manually scrolling through them is impossible. Display Filters are your strongest weapon as an analyst to cut through the noise and find the signal.

Important: Do not confuse Display Filters with Capture Filters. Capture filters decide what data gets written to disk. Display filters hide/show data that has already been captured.

### Filter Syntax Basics

- `.==` (Equals): `ip.addr == 192.168.1.100`
- `.!=` (Not Equals): `ip.src != 10.0.0.5`
- `.>` (Greater Than): `frame.len > 1500`

### IP and Protocol Filtering

- By IP Address:
    - ip.src == 8.8.8.8 (Shows only packets originating from 8.8.8.8)
    - ip.dst == 8.8.8.8 (Shows only packets going to 8.8.8.8)
    - ip.addr == 8.8.8.8 (Shows packets where 8.8.8.8 is either the source or destination)
- By Protocol: Just type the protocol name.
    - http
    - dns
    - tcp
- By Port:
    - tcp.port == 443 (Shows HTTPS traffic)
    - udp.port == 53 (Shows DNS traffic)

### Boolean Operators (Combining Filters)

To find highly specific events, you must chain filters together.
- AND (&&): Both conditions must be true.
    - `ip.src == 192.168.1.50 && tcp.port == 80` (Finds HTTP traffic originating from a specific laptop).
- OR (||): Only one condition needs to be true.
    - `tcp.port == 80 || tcp.port == 443` (Shows both HTTP and HTTPS traffic).
- NOT (!): Excludes traffic.
    - `ip.addr == 192.168.1.50 && !(dns)` (Shows all traffic for that IP except DNS queries).

### Advanced Filtering (Contains and Matches)

- Contains: Searches for a specific string payload inside the packets.
    - `http contains "password"` (Searches all unencrypted HTTP packets for the word "password").
- Matches: Uses Regular Expressions (Regex) for powerful pattern matching.
    - `http.host matches ".(cn|ru)$"` (Finds HTTP traffic going to Russian or Chinese top-level domains).

## Following Streams & Object Extraction

### Follow TCP Stream

If you right-click a packet that is part of a larger conversation (like an HTTP GET request) and select Follow -> TCP Stream, Wireshark performs:

- It automatically figures out the source/dest IPs and ports.
- It reassembles all the fragmented packets in the correct order.
- It strips away all the OSI headers (MAC, IP, TCP headers).
- It presents you with a clean text window showing only the application layer data.

Reading the Stream:

- Text colored Red represents data sent from the Client to the Server (e.g., the browser asking "GET /index.html").
- Text colored Blue represents data sent from the Server to the Client (e.g., the server replying "200 OK" and the HTML code).

Note: If you "Follow" a stream of an encrypted protocol like HTTPS or SSH, you will just see gibberish characters, because the payload is encrypted.

### Extracting Objects

If you can see a file (like an image, a PDF, or an .exe) being downloaded over an unencrypted protocol (HTTP, FTP, SMB), Wireshark can carve that file right out of the PCAP and save it to your hard drive so you can analyze it.

- How to Extract (HTTP):
    - Go to File -> Export Objects -> HTTP...
    - Wireshark will open a window listing every single file transferred over HTTP during the entire packet capture.
    - It lists the Packet Number, Hostname, Content Type (e.g., image/png or application/x-msdownload), and the Filename.
    - You can select a suspicious file and click "Save" to put it on your Desktop. You can then submit that file to VirusTotal or analyze it in a sandbox.

This is a critical skill for extracting malware payloads that defense sensors may have missed, but the packet capture recorded.
