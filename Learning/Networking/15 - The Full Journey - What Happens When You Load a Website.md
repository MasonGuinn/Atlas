---
title: "15 - The Full Journey: What Happens When You Load a Website"
domain: networking
type: lesson
lesson: 15
status: not-started
---

← Previous: [[14 - The Modern Network - Old Ideas at Planetary Scale]]  |  ↑ Parent: [[Networking]]  |  Next: [[16 - Tools of the Trade - Seeing the Network]] →

---

> **Previously:** every note so far solved one problem in isolation — the last several went deep on each subsystem. Now, the assembly. Real networking is all of them firing in sequence, in milliseconds. Let's follow one act — typing <code>https://example.com</code> into a laptop on a home network — and name every concept from this series as it appears. If you can narrate this page from memory, you understand networking.

> 🌍 **The big picture:** This note *is* the big picture. One ordinary click fires every idea in the series, in a strict order, in milliseconds. If you can narrate these five acts from memory, you don't just know facts about networking — you can *talk* networks: diagnose problems, predict behavior, explain failures.

---

## Act 0: The setup you never see (already happened when you joined Wi-Fi)

Before any browsing, your laptop had to become a citizen of the [[04 - LANs - Many Machines, One Room|LAN]]:

1. **Association:** your [[11 - Wi-Fi Deep Dive - Networking Through Air|Wi-Fi]] card (a NIC) connects to the access point. Everything you send will be radio signals modulated onto Wi-Fi channels, formatted as [[03 - The First Conversation - Two Computers, One Wire|frames]] addressed by MAC.
2. [[10 - DNS, DHCP & the Web - Making It Human|DHCP]]**:** your laptop broadcast *"I'm new here!"* The [[05 - IP & Routing - Connecting Networks Together|router]] offered a lease: say [[05 - IP & Routing - Connecting Networks Together|IP]] <code>192.168.1.50</code>, mask <code>/24</code>, gateway <code>192.168.1.1</code>, [[10 - DNS, DHCP & the Web - Making It Human|DNS]] server <code>192.168.1.1</code> (your router, acting as resolver). **DORA** completed in a blink.
3. Your laptop now knows: *my address, my neighborhood, my exit door, and who to ask for directions-by-name.*

---

## Act 1: "Where is [example.com](http://example.com)?" (DNS)

You type the URL and press Enter. The browser knows names, but the network runs on numbers, so first:

1. Browser checks its **cache** → miss. OS cache → miss.
2. The OS sends a **DNS query** to the resolver (<code>192.168.1.1</code>). Notice the stack this tiny question already uses: a [[07 - TCP & UDP - Making Delivery Reliable|UDP]] packet (destination port 53) inside an IP packet inside a frame riding radio waves.
3. To put that frame on the wire, your laptop needs the **gateway's **[[03 - The First Conversation - Two Computers, One Wire|MAC address]] — the query's destination is "outside," so by the default-gateway rule it goes to the router, not the resolver's own MAC. [[00 - Fundamentals - The Reference Map|ARP]] resolves <code>192.168.1.1</code> → the router's MAC via a LAN **broadcast** that the access point/[[04 - LANs - Many Machines, One Room|switch]] floods.
4. The resolver does the root → TLD → authoritative walk (or answers from *its* cache — TTL in action) and returns: <code>example.com</code> = <code>93.184.216.34</code>, say.

⏱️ *Elapsed: often under 50 ms. Already involved: Wi-Fi, DHCP-state, ARP, switching, *[[05 - IP & Routing - Connecting Networks Together|*routing*]]*, UDP, IP, and the global DNS hierarchy.*

---

## Act 2: "Let's talk" (TCP handshake)

The browser now opens a [[07 - TCP & UDP - Making Delivery Reliable|TCP]] connection to <code>93.184.216.34</code>, port **443**:

1. Your OS picks an **ephemeral port**, e.g. <code>51173</code>. The connection's identity is the socket pair <code>192.168.1.50:51173 ↔ 93.184.216.34:443</code>.
2. **SYN** leaves your laptop: framed to the gateway's MAC (LAN rules), but IP-addressed to the server (Internet rules) — both address systems at once, each doing its own job. It crosses your router, where [[08 - NAT - How One Address Serves a Whole House|NAT]] rewrites the source to your home's single public IP (and remembers the mapping so replies find their way back) — the great [[05 - IP & Routing - Connecting Networks Together|IPv4]] life-extension hack.
3. A dozen or so **routers** forward it hop-by-hop, each consulting its routing table. None of them knows or cares what the packet *means*.
4. **SYN-ACK** returns; your **ACK** completes the handshake. Both sides now have sequence numbers and a shared understanding: *we are connected, and we will notice if anything goes missing.*

---

## Act 3: "Prove it, then whisper" (TLS)

Since this is [[10 - DNS, DHCP & the Web - Making It Human|HTTPS]], before any web data:

1. Your browser and the server perform the [[10 - DNS, DHCP & the Web - Making It Human|TLS]]** handshake**: the server presents its **certificate** proving it really is <code>example.com</code> (signed by a Certificate Authority your browser trusts), and the two sides agree on encryption keys — without ever sending the keys themselves across the wire.
2. From now on, everything is encrypted. The dozen routers in the path see only that <code>192.168.1.50</code> (well, your public IP) is exchanging *something* with <code>93.184.216.34:443</code> — metadata, not content.

> 🔐 Pause and appreciate the architecture: TCP/IP's core offers zero confidentiality and zero authentication — and rather than redesign the Internet, we layered trust at the very top, endpoint to endpoint. the end-to-end principle is why your banking works over infrastructure you don't trust.

---

## Act 4: The actual request (HTTP)

Inside the encrypted tunnel, your browser finally sends the thing you asked for seconds ago:

```javascript
GET / HTTP/2
Host: example.com
```

The server responds <code>200 OK</code> with the page's HTML. But TCP makes this orderly:

- The response is a **stream of bytes**, chopped into numbered segments; your machine **ACKs** as they arrive, reassembles them **in order**, and requests retransmission of anything lost. If the network gets congested, both sides' congestion control throttles politely.
- The browser parses the HTML, finds references to CSS, JavaScript, and images — and repeats Acts 1–4 (often in parallel, often reusing the same connection, and with DNS/ARP results now cached) for each.

---

## The whole story on one line

```javascript
You type a name
→ DHCP already gave you an address, gateway, and resolver (Act 0)
→ DNS turns the name into an IP, using ARP + UDP + the hierarchy (Act 1)
→ TCP handshakes across ~12 routers via NAT and routing tables (Act 2)
→ TLS adds proof and privacy, endpoint to endpoint (Act 3)
→ HTTP asks; TCP delivers every byte, in order, verified (Act 4)
→ Frames, switches, and signals carry every single step (always)
```

---

## Why this story is the whole course

- **Troubleshooting** is just asking *"which act is failing?"* — no DHCP lease (Act 0), DNS not resolving (Act 1), connection timing out (Act 2), certificate warnings (Act 3), a 500 error (Act 4). Network+ and Security+ diagnostics are this story, run backwards.
- **Security analysis** is the same story with an adversary inserted: ARP spoofing hijacks Act 1's local step; a rogue DHCP server corrupts Act 0; DNS poisoning forges Act 1; a SYN flood strangles Act 2; only Act 3's TLS stands between all of that and your actual data — which is exactly why attackers go after certificates, and why defenders watch DNS.

---

## 🤔 Pause & think

*Try answering out loud before peeking at the hints — explaining a thing is how it becomes yours.*

1. **In Act 2, the packet's destination *****MAC***** is the router's, but its destination *****IP***** is a server across the country. Why both?**<br />*MAC handles the current hop across the LAN; IP handles the whole journey. The frame is re-addressed at every hop — the packet inside isn't.*
2. **Your address is 169.254.x.x and nothing loads. Which Act failed?**<br />*Act 0. DHCP never issued a real lease, so the journey never even started.*
3. **Which Acts happen once, and which repeat for nearly every file on a page?**<br />*DHCP: once per network. DNS/TCP/TLS: potentially per connection — though caching and connection reuse make the repeats much cheaper.*

## 🧩 Where each act was built

| Act | What fires | Built in |
| --- | --- | --- |
| 0 | Wi-Fi association + DHCP lease | [[11 - Wi-Fi Deep Dive - Networking Through Air|11 — Wi-Fi Deep Dive]], [[10 - DNS, DHCP & the Web - Making It Human|10 — DNS, DHCP & the Web]] |
| 1 | ARP, switching, DNS lookup | [[04 - LANs - Many Machines, One Room|04 — LANs]], [[10 - DNS, DHCP & the Web - Making It Human|10 — DNS, DHCP & the Web]] |
| 2 | Routing, NAT, TCP handshake | [[05 - IP & Routing - Connecting Networks Together|05 — IP & Routing]], [[08 - NAT - How One Address Serves a Whole House|08 — NAT]], [[07 - TCP & UDP - Making Delivery Reliable|07 — TCP & UDP]] |
| 3 | TLS: proof + privacy | [[10 - DNS, DHCP & the Web - Making It Human|10 — DNS, DHCP & the Web]] |
| 4 | HTTP over a reliable stream | [[10 - DNS, DHCP & the Web - Making It Human|10 — DNS, DHCP & the Web]], [[07 - TCP & UDP - Making Delivery Reliable|07 — TCP & UDP]] |

### Final self-check

- [ ] Can I narrate the full journey — DHCP → DNS → ARP → TCP → TLS → [[10 - DNS, DHCP & the Web - Making It Human|HTTP]] — without notes?
- [ ] For each step, can I say *which problem from which note* it's solving?
- [ ] When something breaks, can I isolate which act failed?

**Next →** You can narrate the machine forwards. Now learn to run it backwards — that's troubleshooting: [[16 - Tools of the Trade - Seeing the Network|16 — Tools of the Trade: Seeing the Network]].

---

← Previous: [[14 - The Modern Network - Old Ideas at Planetary Scale]]  |  ↑ Parent: [[Networking]]  |  Next: [[16 - Tools of the Trade - Seeing the Network]] →
