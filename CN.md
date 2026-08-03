# Computer Networks (CN) — Simple Interview Notes

Format: short definition + example. Keep answers to 1-2 lines in the interview.

---

## ⭐⭐⭐ Must Read

### OSI Model
7 layers, top to bottom: **A**pplication → **P**resentation → **S**ession → **T**ransport → **N**etwork → **D**ata Link → **P**hysical.
Trick to remember: *"All People Seem To Need Data Processing."*
- Each layer has a job: e.g., Network = routing (IP), Transport = reliable delivery (TCP), Data Link = MAC addressing.

### TCP/IP Model
4 (sometimes shown as 5) layers: **Application → Transport → Internet → Network Access (Link)**.
- Practical model actually used on the internet (OSI is more theoretical/teaching model).

### TCP vs UDP
| TCP | UDP |
|---|---|
| Connection-oriented (handshake first) | Connectionless |
| Reliable, ordered delivery | No guarantee of delivery/order |
| Slower (more overhead) | Faster, lightweight |
| Example: web browsing, email, file transfer | Example: video streaming, gaming, DNS |

### Three-Way Handshake
How a TCP connection is **established**:
1. Client → Server: **SYN**
2. Server → Client: **SYN-ACK**
3. Client → Server: **ACK**

Now the connection is open and data transfer begins.

### Four-Way Handshake
How a TCP connection is **terminated**:
1. Client → Server: **FIN**
2. Server → Client: **ACK**
3. Server → Client: **FIN**
4. Client → Server: **ACK**

Both sides close their side of the connection independently (that's why it's 4 steps, not 3).

### HTTP vs HTTPS
- **HTTP** – data sent in plain text, port 80.
- **HTTPS** – HTTP + SSL/TLS encryption, port 443, secure (used for login pages, payments, etc.).

### DNS (Domain Name System)
Translates human-readable domain names into IP addresses.
- Example: `google.com → 142.250.xx.xx`
- Called the "phonebook of the internet." Runs on port 53.

### IP Address
A unique address identifying a device on a network.
- Example: `192.168.1.1`
- Two types: **Public IP** (internet-facing) and **Private IP** (within a local network).

### IPv4 vs IPv6
| IPv4 | IPv6 |
|---|---|
| 32-bit, e.g. `192.168.1.1` | 128-bit, e.g. `2001:0db8::1` |
| ~4.3 billion addresses | Practically unlimited addresses |
| Address running out | Designed to solve that shortage |

### MAC Address vs IP Address
| MAC Address | IP Address |
|---|---|
| Physical address of network card (hardware) | Logical address assigned to device on a network |
| Fixed, doesn't change | Can change (dynamic via DHCP) |
| Used at Data Link layer | Used at Network layer |
| Example: `00:1A:2B:3C:4D:5E` | Example: `192.168.1.5` |

### Router vs Switch
- **Router** – connects **different networks** together (e.g., your home network to the internet); works at Network layer, uses IP addresses.
- **Switch** – connects **devices within the same network** (e.g., computers in one office); works at Data Link layer, uses MAC addresses.

### Hub vs Switch
- **Hub** – sends incoming data to **all** connected devices (broadcasts); dumb device, causes collisions.
- **Switch** – sends data **only to the intended device** using MAC address table; smarter, more efficient, no unnecessary traffic.

### Ports (common ones to remember)
| Port | Service |
|---|---|
| 80 | HTTP |
| 443 | HTTPS |
| 21 | FTP |
| 22 | SSH |
| 53 | DNS |

**One-liner:** "A port identifies which application/service on a device should receive the data — the IP gets you to the device, the port gets you to the right service on it."

---

## ⭐⭐ Important

### DHCP (Dynamic Host Configuration Protocol)
Automatically assigns an IP address to a device when it joins a network — no manual configuration needed.

### Gateway
The device (usually a router) that connects a local network to the outside network/internet — the "exit point" of a network.

### Packet Switching
Data broken into small **packets**, each routed independently, reassembled at destination. Used by the internet — efficient, flexible.

### Circuit Switching
A dedicated communication path is established for the entire session (e.g., traditional telephone calls) before data transfer starts — reserved bandwidth, less flexible.

### Cookies vs Sessions
- **Cookie** – small data stored on the **client (browser)**, used to remember info (login, preferences) across requests.
- **Session** – data stored on the **server**, tied to a client via a session ID (often sent through a cookie).

### NAT (Network Address Translation)
Translates private IP addresses (inside a local network) to a public IP address (for internet access) and vice versa — lets multiple devices share one public IP.

### ARP (Address Resolution Protocol)
Finds the **MAC address** of a device when you only know its **IP address** — used within a local network.

### ICMP (basic idea)
Protocol used for network diagnostics/error reporting (not for regular data transfer) — e.g., "destination unreachable" messages. `ping` is built on ICMP.

### Ping
A command that tests if a host is reachable — sends ICMP echo requests and measures response time (latency).

---

## ⭐⭐⭐ High-Frequency Follow-Up Questions

### Connection-oriented vs Connectionless
- **Connection-oriented** – connection established before sending data (e.g., TCP, via three-way handshake).
- **Connectionless** – data sent directly, no connection setup (e.g., UDP).

### Stateful vs Stateless
- **HTTP is stateless** – each request is independent, server doesn't remember previous requests by default.
- **Sessions/cookies** are used to maintain user state across requests.
- **Example:** Login page → server uses a session/cookie to remember you're logged in on the next request.

### Why is TCP reliable?
- **Sequence numbers** – track order of packets, allow correct reassembly.
- **Acknowledgments (ACKs)** – receiver confirms each packet received.
- **Retransmission** – lost/unacknowledged packets are resent.
- **Flow control** – prevents sender from overwhelming a slower receiver.
- **Error checking (checksum)** – detects corrupted data.

*(Very common follow-up right after "TCP vs UDP" — have these 5 points ready.)*

### Why is UDP used for gaming/video calls?
UDP has **lower latency** because it doesn't wait for acknowledgments or retransmissions. For real-time use cases, losing a few packets is better than delaying the stream — a slightly glitchy frame beats a frozen call.

### What happens when you type `google.com` in the browser?
Interview favorite — explain the overall flow, no need for extreme detail:
1. Browser checks its cache (for a cached DNS result/page).
2. **DNS** resolves `google.com` to an IP address.
3. **TCP three-way handshake** with the server.
4. **TLS handshake** (for HTTPS) to set up encryption.
5. Browser sends an **HTTP request**.
6. Server processes it and sends back an **HTTP response**.
7. Browser **renders** the page.

### SSL vs TLS
TLS is the newer, more secure version of SSL — most interviewers accept this one line. (SSL is largely deprecated; "SSL" is still used colloquially to mean TLS, e.g. "SSL certificate.")

### Public IP vs Private IP
- **Private IP** – used within a local network, not reachable from the internet directly. Example ranges: `192.168.x.x`, `10.x.x.x`.
- **Public IP** – assigned by your ISP, visible/reachable on the internet.
- NAT (see above) is what maps private IPs to a shared public IP.

---

## ⭐ Good to Know

### Firewall
A security system (hardware/software) that monitors and filters incoming/outgoing network traffic based on rules — blocks unauthorized access.

### Proxy Server
Acts as a middleman between client and server — forwards requests on behalf of the client, can hide client's IP, cache content, or filter traffic.

### Load Balancer (basic idea)
Distributes incoming traffic across multiple servers to avoid overloading one server — improves availability and performance.

### Socket
An endpoint for sending/receiving data over a network — identified by an **IP address + port number** combination.

### SSL/TLS
Protocols that encrypt data between client and server for secure communication — TLS is the modern, more secure successor to SSL. Powers HTTPS.

### FTP (File Transfer Protocol)
Used to transfer files between client and server over a network. Port 21.

### SMTP, POP3, IMAP (email protocols)
- **SMTP** – used to **send** email.
- **POP3** – downloads email to a device and typically removes it from the server.
- **IMAP** – syncs email with the server, keeps it accessible from multiple devices.

### Bandwidth vs Throughput vs Latency
- **Bandwidth** – maximum data capacity of a connection (theoretical limit), e.g., "100 Mbps."
- **Throughput** – actual amount of data successfully transferred in practice (usually less than bandwidth).
- **Latency** – time delay for data to travel from source to destination (measured in ms).

### Unicast vs Broadcast vs Multicast
- **Unicast** – data sent from one sender to **one specific** receiver.
- **Broadcast** – data sent from one sender to **all** devices on the network.
- **Multicast** – data sent from one sender to a **specific group** of interested receivers.

---

### Quick Interview Tip
Keep every answer to definition + 1 real-world example. If asked "explain in depth," add the *why it's used/why it matters* line — don't over-elaborate unless asked.
