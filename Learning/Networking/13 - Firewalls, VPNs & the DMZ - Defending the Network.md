---
title: "13 - Firewalls, VPNs & the DMZ: Defending the Network"
domain: networking
type: lesson
lesson: 13
status: not-started
---


← Previous: [[12 - Routing Deep Dive - How the Internet Finds Its Way]]  |  ↑ Parent: [[Networking]]  |  Next: [[14 - The Modern Network - Old Ideas at Planetary Scale]] →

---


> **Previously:** we've built a global network whose founding protocols trust everyone — the recurring theme of this series. [[08 - NAT - How One Address Serves a Whole House|NAT]] accidentally blocks some inbound traffic — but accidental isn't a strategy. This note is the deliberate version: the security architecture that sits at a network's borders. *This is where networking becomes cybersecurity — your field.*

> 🌍 **The big picture:** The core network trusts everyone, so security gets added at the borders: decide what may cross (firewall), quarantine what must be public (DMZ), encrypt what must travel (VPN). Nothing here is new magic — it's the fundamentals from the earlier notes, given authority.

---

## Firewalls: policy, enforced at a choke point

A **firewall** is a device (or software) sitting where traffic must pass, enforcing rules: *what may cross, in which direction, between whom.* The concept evolved in generations:

| Generation | How it decides | Blind spot |
| --- | --- | --- |
| **Packet filter** (stateless) | Each packet alone vs. rules on IP/port/protocol | No memory — can't tell a real reply from a forged one |
| **Stateful inspection** | Tracks *connections* (the NAT table idea): replies to your outbound requests are expected; random inbound packets are not | Still can't see *inside* allowed traffic |
| **Next-gen / application-aware** | Identifies the actual app and content (even on standard ports), integrates threat intelligence, user identity | Cost, complexity, and encryption limits what it can inspect |

> 🚪 **Analogy:** a firewall is a **bouncer with a list**. *Stateless* = he checks your face against the list every single time and remembers nothing. *Stateful* = he remembers letting you in, so your return from the smoking area is waved through. *Next-gen* = he also checks what you're carrying, and whether it matches what you claimed to be bringing.

### ACLs and the default-deny philosophy

The rule list is an **ACL** (Access Control List), evaluated top-to-bottom, first match wins. Security-minded design follows **default deny**: block everything, then explicitly allow what's needed. Every allow rule is a considered exception. (The opposite default — allow all, block known-bad — fails open and is how breaches happen.)

A minimal home/SMB example:

```javascript
ALLOW inside → outside any port (users browsing out)
ALLOW replies (stateful) (their answers return)
DENY outside → inside any port (nobody initiates inward)
ALLOW outside → DMZ server port 443 (the one public exception)
```

Notice: this is just the [[05 - IP & Routing - Connecting Networks Together|routing]] choke point + the ports + connection state, combined into policy. Firewalls aren't new magic — they're fundamentals with authority.

---

## The DMZ: a lobby for strangers

Some servers *must* be reachable by the public — a website, a game server. Putting them on the internal [[04 - LANs - Many Machines, One Room|LAN]] means one compromise exposes everything. The classic answer is the **DMZ** (demilitarized zone, borrowed from the Korean border): a separate [[06 - Subnetting - Dividing Networks on Purpose|subnet]], between two firewall rule sets, holding only the public-facing systems.

- Internet → DMZ: allowed, but only on the service's port
- DMZ → internal LAN: **denied** (or near enough) — so a hacked web server is a foothold in the lobby, not the vault
- Internal LAN → DMZ: allowed for administration

Recognize the pattern? It's the LAN and [[06 - Subnetting - Dividing Networks on Purpose|subnetting]] ideas applied to security: **segmentation** — [[04 - LANs - Many Machines, One Room|VLANs]] and subnets as blast-radius limiters. A flat network lets an attacker roam; a segmented one makes them re-break-in at every border. Modern "zero trust" thinking is this same instinct pushed further: don't fully trust *any* segment, verify every flow.

---

## VPNs: a private tunnel through public wires

A **Virtual Private Network** solves a scary problem: how do two sites (or one remote worker and the office) communicate across the untrusted Internet as if locally connected?

The recipe: **tunneling + encryption**

1. Take the original packet — the whole thing, [[05 - IP & Routing - Connecting Networks Together|IP]] header and all
2. **Encrypt it** (so the dozen [[05 - IP & Routing - Connecting Networks Together|routers]] see only noise)
3. Wrap it in a *new* IP packet addressed between the two VPN endpoints — the tunnel
4. At the far end: unwrap, decrypt, and deliver the original packet onto the local network as if it had never left

| Type | Connects | Classic use |
| --- | --- | --- |
| **Site-to-site** | Two whole networks | Branch office ↔ HQ over the Internet instead of a leased line |
| **Remote access** | One device ↔ a network | Working from home: your laptop "virtually" sits in the office LAN |

Protocols you've heard of: **IPsec** (the venerable built-in-encryption suite), **OpenVPN** and **WireGuard** (modern, simpler — WireGuard especially is praised for a tiny, auditable codebase).

> 📺 **About consumer "VPN" apps:** they use the identical tunnel trick, but the far end is the *provider's* server, not your office. Your ISP then sees only a tunnel; the provider sees everything your ISP would have. You've **moved** trust, not eliminated it — great against coffee-shop snoops and ISP tracking, meaningless if you don't trust the provider. (And no, a VPN does not make you "anonymous.")

---

## Watching the traffic: IDS/IPS

Firewalls enforce rules; **Intrusion Detection/Prevention Systems** look for *badness*:

- **IDS** — watches (often a copy of traffic via a [[04 - LANs - Many Machines, One Room|switch]]'s mirror/SPAN port) and *alerts* on suspicious patterns
- **IPS** — sits in-line and can *block* in real time, at the cost of possibly blocking legit traffic when wrong

Detection styles mirror antivirus: **signature-based** (known attack patterns — fast, blind to novel attacks) vs. **anomaly-based** (learn "normal," flag deviations — catches new things, cries wolf more). Today these functions are usually folded into next-gen firewalls and endpoint tools rather than separate boxes.

---

## The philosophy: defense in depth

No single layer in this note is sufficient — firewalls get misconfigured, VPN credentials get phished, segments get bridged by a careless cable. So real architectures **stack independent controls**: segmentation *and* stateful filtering *and* encryption *and* monitoring *and* patched hosts (the endpoint layer your Cybersecurity notes will cover). An attacker must defeat *all* of them, in sequence, without tripping any alarm. That's the practical form of the end-to-end principle's security corollary: **trust nothing by default; verify at every boundary.**

**Next →** Borders defended — now look at what the network *became*: cloud, CDNs, and planet-scale everything, built entirely from parts you already know. [[14 - The Modern Network - Old Ideas at Planetary Scale|14 — The Modern Network: Old Ideas at Planetary Scale]].

---

## 🤔 Pause & think

*Try answering out loud before peeking at the hints — explaining a thing is how it becomes yours.*

1. **Why is default-deny safer than default-allow?**<br />*Default-allow fails open: everything you didn't think of is permitted. Default-deny fails closed: only deliberate exceptions exist, and each one can be audited.*
2. **The public web server in the DMZ gets hacked. Why isn't the internal LAN automatically lost?**<br />*The inner firewall denies DMZ → LAN flows. The attacker is stuck in the lobby and must breach a second, separate boundary.*
3. **What does a consumer VPN app actually change about your traffic?**<br />*Who can see it. Your ISP sees only a tunnel; the VPN provider sees what your ISP would have. Trust moved — it didn't vanish.*

### Key takeaways

- [ ] Firewalls enforce **ACL** rules at choke points; **stateful** beats stateless because it understands connections; **default deny** is the professional posture
- [ ] A **DMZ** isolates public servers so a compromise doesn't reach the internal LAN — segmentation = blast-radius control
- [ ] A **VPN** = tunnel (encapsulation) + encryption: site-to-site joins networks, remote-access joins one device; consumer VPNs *move* trust to the provider
- [ ] **Defense in depth**: stack controls so no single failure is fatal

---

← Previous: [[12 - Routing Deep Dive - How the Internet Finds Its Way]]  |  ↑ Parent: [[Networking]]  |  Next: [[14 - The Modern Network - Old Ideas at Planetary Scale]] →
