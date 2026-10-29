---
title: "00 - Fundamentals: The Reference Map"
domain: networking
type: lesson
lesson: 0
status: not-started
---

← Previous: —  |  ↑ Parent: [[Networking]]  |  Next: [[01 - Why Networks Exist]] →

---


> **The core idea:** A *protocol* is just an agreed set of rules for how devices communicate. Every app, attack, and defense in cybersecurity sits on top of these rules — so if you understand the protocols, you understand how data moves and where it can be intercepted, spoofed, or broken.

> 🌍 **The big picture:** This is **Note 00** of the series — the map you keep open while exploring the territory. The numbered notes (01–18) explain *why* everything works the way it does; this page is the *what*: the raw facts, tables, ranges, and ports to look up in seconds. Use it two ways — when a note confuses you, check here for the fact; when a fact here confuses you, the **Go deeper** index at the bottom points you to the note that explains it.

---

## 1. What Is a Network?

A **network** is two or more devices connected so they can exchange data. Each connected device is a **node**; a node that runs applications and services is a **host**.

### Key vocabulary

| Term | Meaning |
| --- | --- |
| **Bandwidth** | The maximum amount of data a link can carry per second (e.g., 1 Gbps) |
| **Throughput** | The amount of data *actually* transferred — usually less than bandwidth |
| **Latency** | The delay for data to travel from source to destination (measured in ms) |
| **Packet** | A small chunk of data sent across a network — big messages are split into many packets |
| **Protocol** | The rules that define how devices format, send, and interpret data |

### Types of networks

| Type | Stands for | Scope | Example |
| --- | --- | --- | --- |
| **PAN** | Personal Area Network | A few meters | Bluetooth between your phone and earbuds |
| **LAN** | Local Area Network | One building or home | Your home Wi-Fi / office Ethernet |
| **MAN** | Metropolitan Area Network | A city | A university campus spanning several blocks |
| **WAN** | Wide Area Network | Countries / the world | The Internet (a network of networks) |

---

## 2. The OSI Model — The 7 Layers

The **OSI model** is the standard reference for how network communication is broken into 7 layers. Each layer has one job and only talks to the layers directly above and below it. When troubleshooting or analyzing an attack, always ask: *which layer is this happening at?*

| # | Layer | Job | Examples | Data unit |
| --- | --- | --- | --- | --- |
| 7 | **Application** | Interfaces directly with the user's app | HTTP, DNS, SMTP, FTP | Data |
| 6 | **Presentation** | Formatting, encryption, compression | TLS, JPEG, ASCII | Data |
| 5 | **Session** | Opens, manages, and closes connections between apps | NetBIOS, RPC | Data |
| 4 | **Transport** | End-to-end delivery, reliability, port numbers | TCP, UDP | Segment / Datagram |
| 3 | **Network** | Logical addressing and routing between networks | IP, ICMP | Packet |
| 2 | **Data Link** | Delivery on the *local* network using MAC addresses | Ethernet, Wi-Fi, ARP | Frame |
| 1 | **Physical** | Actual signals — cables, radio waves, voltages | Copper, fiber, Wi-Fi radio | Bits |

> 🧠 **Mnemonic (top → down):** **A**ll **P**eople **S**eem **T**o **N**eed **D**ata **P**rocessing 🧠 **Mnemonic (bottom → up):** **P**lease **D**o **N**ot **T**hrow **S**ausage **P**izza **A**way

---

## 3. The TCP/IP Model — The One Actually Used

The Internet runs on the [[07 - TCP & UDP - Making Delivery Reliable|TCP]]**/IP model**, which compresses the OSI model into 4 layers. Know both, and how they map:

| TCP/IP layer | Covers OSI layers | Key protocols |
| --- | --- | --- |
| **Application** | 5, 6, 7 (Session, Presentation, Application) | HTTP, DNS, DHCP, SSH |
| **Transport** | 4 (Transport) | TCP, UDP |
| **Internet** | 3 (Network) | IP, ICMP, ARP\* |
| **Network Access** | 1, 2 (Physical, Data Link) | Ethernet, Wi-Fi |

*\*ARP is often placed at Layer 2 or 3 depending on the source — it bridges the two.*

---

## 4. Encapsulation — How Data Travels

As data moves **down** the layers on the sending device, each layer wraps it with its own header (and the Data Link layer adds a trailer). This is **encapsulation**. On the receiving side, each layer strips its header back off (**decapsulation**).

```javascript
Application data                →  "Hello!"
+ Transport header (TCP)        →  Segment
+ Network header (IP)           →  Packet
+ Data Link header + trailer    →  Frame
Converted to electrical/radio   →  Bits
```

**Why it matters:** every header you add is information an attacker can read or forge — source/destination IPs, port numbers, flags, and more. Packet analysis tools like [[16 - Tools of the Trade - Seeing the Network|Wireshark]] simply peel these layers apart one by one.

---

## 5. IP Addressing

### IPv4

- A **32-bit** address written as 4 decimal numbers (**octets**) separated by dots: <code>192.168.1.10</code>
- Each octet ranges 0–255, so there are \~4.3 billion possible addresses — which is why we're running out
- Every address has two parts: the **network portion** and the **host portion**, split by the [[06 - Subnetting - Dividing Networks on Purpose|subnet]]** mask**

### Public vs. private addresses

Private addresses are used inside [[04 - LANs - Many Machines, One Room|LANs]] and are *not routable on the Internet*. Your [[05 - IP & Routing - Connecting Networks Together|router]] translates them to a public address using [[08 - NAT - How One Address Serves a Whole House|NAT]] (Network Address Translation).

| Private range | CIDR | Common use |
| --- | --- | --- |
| 10.0.0.0 – 10.255.255.255 | 10.0.0.0/8 | Large organizations |
| 172.16.0.0 – 172.31.255.255 | 172.16.0.0/12 | Medium networks, cloud |
| 192.168.0.0 – 192.168.255.255 | 192.168.0.0/16 | Home & small office networks |

**Special addresses to know:**

- <code>127.0.0.1</code> — **Loopback** ("[localhost](http://localhost)") — the device talking to itself
- <code>169.254.x.x</code> — **APIPA** — self-assigned when [[10 - DNS, DHCP & the Web - Making It Human|DHCP]] fails (a red flag when troubleshooting)
- <code>0.0.0.0</code> — "all networks" / unspecified

### Subnet masks & CIDR

The subnet mask tells you which part of the address is the network and which part is the host.

- <code>255.255.255.0</code> = **/24** in CIDR notation → 256 addresses, **254 usable hosts**
- The <code>/number</code> counts the network bits: /24 = 24 network bits, 8 host bits → 2⁸ = 256 addresses
- Two addresses in every subnet are reserved: the **network address** (first) and the **broadcast address** (last)

### IPv6

- **128-bit** address written in hexadecimal: <code>2001:0db8:85a3:0000:0000:8a2e:0370:7334</code>
- \~340 undecillion addresses — no NAT needed, no address shortage
- No broadcast; uses **multicast** instead. Loopback is <code>::1</code>

---

## 6. MAC Addresses & ARP

- A [[03 - The First Conversation - Two Computers, One Wire|MAC address]] is a **48-bit** physical address burned into every network card (NIC), written like <code>AA:BB:CC:11:22:33</code>. It only matters on the *local* network.
- [[05 - IP & Routing - Connecting Networks Together|IP addresses]]** are logical** (they change as you move networks); **MAC addresses are physical** (they stay with the device).
- **ARP (Address Resolution Protocol)** bridges the two: when a device knows a destination [[05 - IP & Routing - Connecting Networks Together|IP]] on its LAN but not the MAC, it broadcasts *"Who has 192.168.1.1?"* and the owner replies with its MAC. The result is stored in the **ARP cache**.

> ⚠️ **Security note:** ARP has *no authentication* — any device can answer an ARP request. That's what makes **ARP spoofing/poisoning** (a classic man-in-the-middle attack) possible.

---

## 7. TCP vs. UDP

The two transport protocols make opposite trade-offs: **reliability vs. speed**.

| Feature | TCP | UDP |
| --- | --- | --- |
| Connection | Connection-oriented (handshake first) | Connectionless (just send) |
| Reliability | Guaranteed delivery, retransmits lost data | No guarantee |
| Ordering | Data arrives in order | No ordering |
| Overhead / speed | Higher overhead, slower | Low overhead, fast |
| Used by | Web (HTTP), email, file transfer, SSH | Streaming, gaming, DNS, VoIP, DHCP |

### The TCP three-way handshake

Before any data flows, TCP sets up the connection:

```javascript
Client  ─── SYN ──────────────►  Server   "Let's talk — here's my sequence number"
Client  ◄── SYN-ACK ──────────  Server   "Got it — here's mine"
Client  ─── ACK ──────────────►  Server   "Got yours. Connection open!"
```

The connection closes with FIN/ACK exchanges, or abruptly with an **RST** packet.

> ⚠️ **Security note:** A **SYN flood** attack sends thousands of SYNs and never completes the handshake, exhausting the server's half-open connections. **Port scanners** (like Nmap) abuse these same flags to probe which ports are open.

---

## 8. Ports & Sockets

An IP address gets data to the right *machine*; a **port number** gets it to the right *program* on that machine. IP + port = a **socket** (e.g., <code>192.168.1.10:443</code>).

| Range | Name | Notes |
| --- | --- | --- |
| 0 – 1023 | **Well-known** | Reserved for standard services; need admin rights to bind |
| 1024 – 49151 | **Registered** | Assigned to specific applications |
| 49152 – 65535 | **Ephemeral** | Temporary ports your OS picks for outgoing connections |

### Must-know ports (memorize these — they come up constantly in security)

| Port | Service | Protocol | Notes |
| --- | --- | --- | --- |
| 20 / 21 | FTP | TCP | File transfer — sends credentials in **plaintext** |
| 22 | SSH | TCP | Encrypted remote login — the secure replacement for Telnet |
| 23 | Telnet | TCP | Legacy remote login — **plaintext**, avoid |
| 25 | SMTP | TCP | Sending email |
| 53 | DNS | UDP / TCP | Name resolution (UDP normally, TCP for large responses/zone transfers) |
| 67 / 68 | DHCP | UDP | Automatic IP assignment |
| 80 | HTTP | TCP | Web traffic — **unencrypted** |
| 110 | POP3 | TCP | Receiving email (downloads and deletes) |
| 143 | IMAP | TCP | Receiving email (stays on server) |
| 443 | HTTPS | TCP | Web traffic encrypted with TLS |
| 445 | SMB | TCP | Windows file sharing — frequent attack target |
| 3389 | RDP | TCP | Windows Remote Desktop — frequent attack target |

---

## 9. Core Application Protocols

### DNS — the Internet's phone book

[[10 - DNS, DHCP & the Web - Making It Human|DNS]]** (Domain Name System)** translates human-friendly names (<code>example.com</code>) into IP addresses. What happens when you type a URL:

1. Your browser checks its cache, then asks a **recursive resolver** (often your ISP or 8.8.8.8)
2. The resolver asks a **root server**, which points to the **TLD server** (.com)
3. The TLD server points to the domain's **authoritative server**
4. The authoritative server returns the IP; the resolver caches it (for the record's **TTL**) and hands it to you

**Record types:**

| Record | Purpose |
| --- | --- |
| **A** | Name → IPv4 address |
| **AAAA** | Name → IPv6 address |
| **CNAME** | Alias — one name points to another name |
| **MX** | Mail servers for the domain |
| **NS** | Authoritative name servers for the domain |
| **TXT** | Arbitrary text — used for verification & email security (SPF/DKIM) |
| **PTR** | Reverse lookup — IP → name |

### DHCP — automatic configuration

When a device joins a network, **DHCP** hands it an IP address, subnet mask, default gateway, and DNS server. The 4-step process is **DORA**:

**D**iscover → **O**ffer → **R**equest → **A**cknowledge

### HTTP / HTTPS

- [[10 - DNS, DHCP & the Web - Making It Human|HTTP]] is stateless request/response: your browser sends a request (GET, POST, etc.), the server replies with a status code (200 OK, 404 Not Found, 500 Server Error…)
- [[10 - DNS, DHCP & the Web - Making It Human|HTTPS]] = HTTP inside a [[10 - DNS, DHCP & the Web - Making It Human|TLS]] encrypted tunnel — same conversation, but nobody in the middle can read or modify it

---

## 10. Network Devices

| Device | Layer | What it does |
| --- | --- | --- |
| **Hub** | 1 | Repeats every signal to *all* ports — obsolete, insecure |
| **Switch** | 2 | Learns MAC addresses and forwards frames only to the right port — the backbone of a LAN |
| **Router** | 3 | Connects different networks and forwards packets by IP — your link to the Internet |
| **Modem** | 1–2 | Converts between your ISP's signal (cable/fiber/DSL) and Ethernet |
| **Wireless Access Point (WAP)** | 2 | Bridges Wi-Fi devices onto the wired LAN |
| **Firewall** | 3–7 | Filters traffic by rules — IPs, ports, and (in next-gen firewalls) application content |

**Default gateway:** the router's IP address on your LAN. Any packet destined for a *different* network gets sent to the gateway — it's the exit door of your network.

> 💡 **Switch vs. router, in one line:** a [[04 - LANs - Many Machines, One Room|switch]] delivers *within* one network using MAC addresses; a router delivers *between* networks using IP addresses.

---

## 11. Topologies — How Networks Are Laid Out

| Topology | Layout | Notes |
| --- | --- | --- |
| **Star** | All devices connect to one central switch | Most common today; one cable failure only affects one device |
| **Bus** | All devices share one backbone cable | Legacy; a break can take down the whole network |
| **Ring** | Each device connects to two neighbors in a loop | Legacy (Token Ring); traffic travels the ring |
| **Mesh** | Devices interconnect with many redundant paths | Most resilient; used for critical links and wireless backhaul |

---

## 12. Why This Matters for Cybersecurity

Every layer of the model has known attacks. Defending a network means knowing which layer you're protecting:

| Layer | Example attack | What it abuses |
| --- | --- | --- |
| Application | DNS spoofing, phishing sites | Trust in DNS / HTTP |
| Transport | SYN flood, session hijacking | TCP handshake & state |
| Network | IP spoofing, ICMP tunneling | Lack of source verification |
| Data Link | ARP poisoning, MAC flooding | ARP's missing authentication |
| Physical | Cable tapping, rogue access point | Physical access |

---

## ✅ Self-Check

Use these to test yourself — if you can answer each one out loud, you've got the fundamentals:

- [ ] Can I name all 7 OSI layers in order, both directions?
- [ ] Can I explain TCP vs. [[07 - TCP & UDP - Making Delivery Reliable|UDP]] and give 3 examples of each?
- [ ] Can I walk through the TCP three-way handshake from memory?
- [ ] Do I know the private IP ranges and how to spot them?
- [ ] Can I explain what DNS and DHCP each do, in one sentence each?
- [ ] Can I list the ports for SSH, DNS, HTTPS, RDP, and SMB without looking?
- [ ] Can I explain the difference between a switch and a router?
- [ ] Do I know why ARP spoofing works?

---

## 🤔 Pause & think

*Try answering out loud before peeking at the hints — explaining a thing is how it becomes yours.*

1. **Someone says "the Internet is down" because one website won't load. Using this page's vocabulary, what's wrong with that sentence?**<br />*A website is one service on one host. The Internet — the network of networks — can be perfectly healthy while a single host, service, or DNS record is broken. Naming the layer is the first diagnostic skill.*
2. **Why does a single note need both an OSI model *****and***** a TCP/IP model?**<br />*OSI is the teaching/analysis model — seven clean layers for talking about where a problem lives. TCP/IP is the model the real Internet was actually built on. Pros think in OSI and build in TCP/IP.*
3. **Which single table on this page will you use most in security work?**<br />*The must-know ports table. Every scan result, *[[13 - Firewalls, VPNs & the DMZ - Defending the Network|*firewall*]]* rule, and log line speaks in port numbers — fluency there is non-negotiable.*

## 🔗 Go deeper — the ground-up series

> This page is the *quick reference*. The series below is the *understanding*: each note explains why a concept exists, where it came from, and how it connects to everything else. Read in order, or jump straight to the topic you're stuck on.

**The build (01–05) — the core stack:**

- [[01 - Why Networks Exist|01 — Why Networks Exist]] — ARPANET, packet switching, and the decisions everything else inherited
- [[02 - Bits & Signals - How Computers Physically Talk|02 — Bits & Signals]] — how information physically travels
- [[03 - The First Conversation - Two Computers, One Wire|03 — The First Conversation]] — MAC addresses, frames, Ethernet
- [[04 - LANs - Many Machines, One Room|04 — LANs]] — hubs, switches, VLANs, Wi-Fi basics
- [[05 - IP & Routing - Connecting Networks Together|05 — IP & Routing]] — hierarchical addressing and hop-by-hop delivery

**Going deeper (06–14):**

- [[06 - Subnetting - Dividing Networks on Purpose|06 — Subnetting]] — the math, worked slowly (VLSM included)
- [[07 - TCP & UDP - Making Delivery Reliable|07 — TCP & UDP]] — ports, handshakes, reliability
- [[08 - NAT - How One Address Serves a Whole House|08 — NAT]] — how one address serves a whole house
- [[09 - IPv6 - The Fix That Took 30 Years|09 — IPv6]] — the abundance fix, SLAAC, NDP
- [[10 - DNS, DHCP & the Web - Making It Human|10 — DNS, DHCP & the Web]] — the protocols that made it human-usable
- [[11 - Wi-Fi Deep Dive - Networking Through Air|11 — Wi-Fi Deep Dive]] — bands, channels, WPA2/WPA3
- [[12 - Routing Deep Dive - How the Internet Finds Its Way|12 — Routing Deep Dive]] — OSPF, BGP, and Internet-scale trust
- [[13 - Firewalls, VPNs & the DMZ - Defending the Network|13 — Firewalls, VPNs & the DMZ]] — defending the network
- [[14 - The Modern Network - Old Ideas at Planetary Scale|14 — The Modern Network]] — load balancers, CDNs, and cloud, translated back into fundamentals

**Synthesis (15–18):**

- [[15 - The Full Journey - What Happens When You Load a Website|15 — The Full Journey]] — everything at once: loading a website, act by act
- [[16 - Tools of the Trade - Seeing the Network|16 — Tools of the Trade]] — ping, traceroute, Wireshark, and methodical troubleshooting
- [[17 - Thinking Like an Attacker - Exploiting the Stack|17 — Thinking Like an Attacker]] — exploiting the stack, layer by layer
- [[18 - The Final Exam - Can You Talk Networks|18 — The Final Exam]] — can you talk networks? Prove it

---

← Previous: —  |  ↑ Parent: [[Networking]]  |  Next: [[01 - Why Networks Exist]] →
