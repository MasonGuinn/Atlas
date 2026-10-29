---
title: "18 - The Final Exam: Can You Talk Networks?"
domain: networking
type: lesson
lesson: 18
status: not-started
---

← Previous: [[17 - Thinking Like an Attacker - Exploiting the Stack]]  |  ↑ Parent: [[Networking]]  |  Next: — →

---

> **The finale.** No new material — this note is the proof. Everything in Notes 00–17, tested the way the real world tests it: *explaining things out loud, under mild pressure, to someone who asks "but why?"* If you can work through this page without peeking, you don't just know networking — you can talk it, which is what interviews, tickets, and incident calls actually demand.

> 🌍 **The big picture:** knowledge you can't say out loud is trivia. Fluency is the deliverable: the right word, the right layer, the right mechanism, in one breath. Work this page in order, out loud, and mark honestly what you can't yet explain — those are your review targets, and every answer names the note to revisit.

---

## Part 1: Explain it in one breath

*Cover the italic answers. Each should take you under 30 seconds.*

1. **What's the difference between the Internet and the Web?**<br />*The Internet is the network — *[[07 - TCP & UDP - Making Delivery Reliable|*TCP*]]*/IP moving packets between networks (1983). The Web is one service riding on it — *[[10 - DNS, DHCP & the Web - Making It Human|*HTTP*]]*, URLs, pages (1989–91). Email and games use the same Internet without touching the Web.*
2. **Hub, **[[04 - LANs - Many Machines, One Room|switch]]**, **[[05 - IP & Routing - Connecting Networks Together|router]]** — go.**<br />*A hub repeats everything to everyone. A switch learns *[[03 - The First Conversation - Two Computers, One Wire|*MAC addresses*]]* and delivers *[[03 - The First Conversation - Two Computers, One Wire|*frames*]]* only to the right port, inside one *[[04 - LANs - Many Machines, One Room|*LAN*]]*. A router forwards packets between networks using *[[05 - IP & Routing - Connecting Networks Together|*IP addresses*]]* and *[[05 - IP & Routing - Connecting Networks Together|*routing*]]* tables.*
3. **Why does your whole house share one public **[[05 - IP & Routing - Connecting Networks Together|IP]]** address?**<br />[[08 - NAT - How One Address Serves a Whole House|*NAT*]]*/PAT at the router: it rewrites outgoing source ports and keeps a translation table so replies find the right internal device. One public address, thousands of conversations, distinguished by port.*
4. **TCP vs. **[[07 - TCP & UDP - Making Delivery Reliable|UDP]]** — when and why?**<br />*TCP when every byte must arrive, in order — web, email, files. It's a handshake, sequence numbers, ACKs, retransmission. UDP when timeliness beats completeness — calls, games, *[[10 - DNS, DHCP & the Web - Making It Human|*DNS*]]* — no setup, no resending stale data.*
5. **What happens when you type a website name? (The 20-second version.)**<br />[[10 - DNS, DHCP & the Web - Making It Human|*DHCP*]]* already gave me an address, gateway, and resolver. DNS turns the name into an IP. *[[00 - Fundamentals - The Reference Map|*ARP*]]* finds the gateway's MAC. TCP handshakes to the server across a dozen routers (with NAT at mine). *[[10 - DNS, DHCP & the Web - Making It Human|*TLS*]]* proves identity and encrypts. HTTP asks; TCP delivers every byte in order.*
6. **Why is **[[06 - Subnetting - Dividing Networks on Purpose|subnetting]]** "drawing borders"?**<br />*Moving the mask splits one address block into smaller networks. Each new border stops broadcasts, gives a router or *[[13 - Firewalls, VPNs & the DMZ - Defending the Network|*firewall*]]* a place to stand, and stops address waste. The math is just 2\^n — the borders are the design.*
7. **Why can't **[[11 - Wi-Fi Deep Dive - Networking Through Air|Wi-Fi]]** just be "wireless **[[03 - The First Conversation - Two Computers, One Wire|Ethernet]]**"?**<br />*Radio is half-duplex and shared: a radio can't listen while transmitting, so collisions can't be detected — only avoided (CSMA/CA) and inferred from missing ACKs. That one physics fact drives Wi-Fi's behavior, speeds, and security.*
8. **What does a firewall actually decide, and what's the safe default?**<br />*Which traffic may cross a boundary, by rules on addresses, ports, and — in stateful firewalls — connection state. The safe default is deny: everything blocked except deliberate exceptions.*
9. **Why did **[[09 - IPv6 - The Fix That Took 30 Years|IPv6]]** take 30 years and counting?**<br />*NAT anesthetized the address shortage IPv6 was built to cure, migration costs are real and benefits diffuse, and no one can declare a flag day on a network of billions. So we dual-stack and inch forward.*
10. **An attacker is on your LAN. What's their first move and why does it work?**<br />*ARP spoofing: answer ARP questions dishonestly so victims send gateway traffic through the attacker's machine. It works because ARP has no authentication — and *[[10 - DNS, DHCP & the Web - Making It Human|*HTTPS*]]* everywhere is what limits the damage.*

---

## Part 2: Say it right (words pros use precisely)

| People say | Pros mean | The distinction |
| --- | --- | --- |
| "Speed" | **Bandwidth** vs. **latency** | Two different measurements (Note 02). Say which one you mean |
| "The Internet is down" | A specific **layer/service** is failing | Name the layer: link, DHCP, DNS, the host itself |
| "Router" (the home box) | A **router + switch + AP + NAT + DHCP server + firewall** combo | Five jobs, one plastic shell — know which job you're troubleshooting |
| "Signal" (bars) | **Signal strength** ≠ link quality | Interference and contention can make full bars useless (Note 11) |
| "Hacked" | A **named mechanism** | ARP spoofed, phished, SYN flooded… the verb matters (Note 17) |
| "IP" | An **address**, a **protocol**, or a **version** | "The IP is 10.0.0.5" (address), "IP forwards it" (protocol), "IPv6" (version) |

---

## Part 3: Scenario drills

*Talk through your reasoning before reading the walk-through. The method from Note 16 — bisect the *[[15 - The Full Journey - What Happens When You Load a Website|*Full Journey*]]* — applies to all of them.*

1. **After a power outage, your PC shows 169.254.0.7 and nothing loads. Diagnosis in one sentence?**<br />*DHCP never answered after the reboot (Act 0 failed) — the machine self-assigned an APIPA address, so check the router/DHCP service before touching anything else.*
2. **Websites load by IP address but not by name. Your first test and your conclusion?**<br />*Test DNS directly (nslookup/dig). IP connectivity is proven, so the failure is name resolution — resolver unreachable, wrong DNS settings, or poisoned cache (Acts: 1 only).*
3. **Your game is fine at 3 PM and unplayable at 8 PM — but only on Wi-Fi, and speed tests look great. What's happening?**<br />*Evening = every neighbor home on shared air. Bandwidth tests pass in bursts, but contention means airtime delays and retries — a *[[02 - Bits & Signals - How Computers Physically Talk|*latency*]]*/jitter problem from CSMA/CA sharing, not a *[[02 - Bits & Signals - How Computers Physically Talk|*bandwidth*]]* problem (Note 11). Fix: wired, or 5/6 GHz with a cleaner channel.*
4. **A coworker's "the server is down" report: the server pings fine but the app times out. Two hypotheses?**<br />*(a) The service process is dead or its port is blocked — ping proves ICMP, not the application (Note 16's warning). (b) A firewall rule change is dropping the app port while allowing ICMP. Test the actual port (nmap/curl), not the host.*
5. **You capture traffic and see the same MAC address claiming to be both the gateway and another laptop. What are you looking at?**<br />*ARP spoofing in progress (Note 17) — one machine poisoning caches to MITM the other. Confirm via arp tables on the victim, then check for DAI/DHCP snooping on the switch.*
6. **Your company opens a branch office across town and needs it "on the same network." Options, in one breath?**<br />*Site-to-site *[[13 - Firewalls, VPNs & the DMZ - Defending the Network|*VPN*]]* over the Internet (cheap, encrypted, Note 13) or a leased line/MPLS from the carrier (pricey, private). Either way it's routing between two LANs — never stretching one broadcast domain across a city.*

---

## Part 4: The 60-second whiteboard

*The graduation test. Stand up (seriously) and deliver each of these aloud, start to finish, no notes:*

- [ ] The Full Journey — DHCP → DNS → ARP → TCP → TLS → HTTP, naming the *problem* each step solves (Note 15)
- [ ] A subnetting problem: split 192.168.1.0/24 for departments of 100, 50, and 25 — biggest first, out loud (Note 06)
- [ ] One attack and its defense, mechanism-level, for three different layers (Note 17)
- [ ] A troubleshooting run: "website won't load," bisecting acts until the broken layer names itself (Note 16)

When all four are smooth, you're not studying networking anymore — you're *speaking* it.

---

## Where to go from here

- **Labs beat pages.** Rebuild this knowledge with your hands: [[16 - Tools of the Trade - Seeing the Network|Wireshark]] captures of your own traffic (find the handshake, the DNS exchange, a TLS session), a home router you deliberately misconfigure and fix, Packet Tracer or GNS3 for routing topologies you can't afford in hardware.
- **Your Cybersecurity notes** are the natural sequel — every defense in Note 13 and attack in Note 17 has a full discipline behind it. Bring this map with you.
- **Revisit on a schedule.** The Pause & Think questions across all 18 notes are your spaced-repetition deck — when one stops being easy, that note gets a re-read.

### Final key takeaways

- [ ] Fluency = right word, right layer, right mechanism, one breath
- [ ] Every professional conversation about networks is the Full Journey plus one question: *which act, which assumption, which border?*
- [ ] You now hold the whole map: build it (01–05), deepen it (06–14), trace it end to end (15), observe it (16), attack it (17), and prove it (this page)

---

← Previous: [[17 - Thinking Like an Attacker - Exploiting the Stack]]  |  ↑ Parent: [[Networking]]  |  Next: — →
