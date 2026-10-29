---
title: "12 - Routing Deep Dive: How the Internet Finds Its Way"
domain: networking
type: lesson
lesson: 12
status: not-started
---


← Previous: [[11 - Wi-Fi Deep Dive - Networking Through Air]]  |  ↑ Parent: [[Networking]]  |  Next: [[13 - Firewalls, VPNs & the DMZ - Defending the Network]] →

---


> **Previously:** we said [[05 - IP & Routing - Connecting Networks Together|routers]] forward packets hop-by-hop using [[05 - IP & Routing - Connecting Networks Together|routing]] tables, each knowing only the *next step*. But where do those tables come from? A planet-scale, constantly-changing map maintained by nobody in particular. This note is how that miracle — and occasional catastrophe — works.

> 🌍 **The big picture:** Routing is map-making by gossip with rules. *Inside* an organization, everyone shares one map and computes their own best path (OSPF). *Between* organizations, it's business contracts and trust (BGP). The Internet's resilience — and its biggest outages — both come from this arrangement.

---

## The one rule that picks every route: longest prefix match

When several table entries could match a destination, the router picks the **most specific** one — the longest matching prefix.

Destination <code>192.168.1.50</code>:

- <code>192.168.0.0/16</code> matches (16 bits) → "toward building A"
- <code>192.168.1.0/24</code> matches (24 bits) → "toward the 3rd floor"
- **/24 wins.** More specific = more trustworthy information.

And the least specific route of all, <code>0.0.0.0/0</code> — the **default route** — matches *everything*, so it's used only when nothing better exists. Your home router's entire "table" is basically: [[04 - LANs - Many Machines, One Room|LAN]] → direct; everything else → the ISP (default). Internet backbone routers are the opposite extreme: **no default route**, full tables of \~1 million prefixes. They're called the **Default-Free Zone**.

---

## Static routing: hand-drawn maps

An admin can type routes manually. Static routing is:

- ✅ Predictable, zero overhead, no protocol to attack
- ❌ Doesn't adapt — if a link dies, the route points into the void until a human edits it

Right tool for tiny, stable networks and deliberate choke points (like forcing all traffic past a [[13 - Firewalls, VPNs & the DMZ - Defending the Network|firewall]]). Absurd for anything that changes.

---

## Dynamic routing: routers gossip with rules

Dynamic protocols let routers *learn* and *re-learn* the map automatically. Two philosophies:

### Distance vector — "I heard it from a neighbor"

Each router tells its neighbors: "I can reach network X, and it's N hops from me." Routers believe the smallest hop count. (Protocol: **RIP**.)

- Simple — but slow to adapt (**convergence** can take minutes), prone to counting-to-infinity loops when links fail, and blindly trusting: a lying or buggy neighbor poisons everyone
- Mostly legacy now, but its *idea* — trust announcements, pick best metric — underlies everything

### Link-state — "Give everyone the same map"

Each router floods *facts* ("my link to router B is up, cost 10") to all routers in the area. Everyone assembles the identical map, then independently computes best paths with **Dijkstra's shortest-path algorithm**. (Protocol: **OSPF**, used inside companies/campuses.)

- Fast convergence (seconds), scales to large organizations, no blind trust of *paths* (only of raw facts)
- Costs memory/CPU — every router holds the whole map

|  | Distance vector (RIP) | Link-state (OSPF) |
| --- | --- | --- |
| Knowledge | Rumors from neighbors | Full map, self-computed |
| Metric | Hop count | Configurable link cost |
| Convergence | Slow (minutes) | Fast (seconds) |
| Scale | Small networks | Large organizations |

---

> 🗺️ **Analogy check:** distance vector is **asking the next town** how far the city is, and trusting the answer. Link-state is giving **every driver the same road atlas** and letting each compute their own route. BGP is neither — it's **trucking companies following contracts**: the route chosen is the profitable one, not the shortest.

## BGP: the Internet's actual nervous system

Organizations' networks — ISPs, Google, your college's provider — are **Autonomous Systems (AS)**, each with an **AS number**. *Between* ASes, one protocol runs everything: **BGP** (Border Gateway Protocol). BGP is distance-vector's descendant, grown up and political:

- ASes announce **"I can reach these **[[05 - IP & Routing - Connecting Networks Together|IP]]** prefixes"** to their peers; announcements chain: AS1 → AS2 → AS3 means AS3 learns a *path* (the sequence of ASes), not just a hop count
- Path choice is **policy, not physics**: an ISP may prefer the route that makes/saves money (peering agreements, customer > peer > transit economics) over the shortest path. Your packets take the *business-optimal* route, not the geographic one
- The system runs on **trust**: BGP has no built-in verification that you own the prefixes you announce

### When trust breaks: prefix hijacks

- **2008:** Pakistan, trying to block YouTube domestically, announced a bogus, more-specific route for YouTube's prefix. Longest-prefix-match (above) made the lie *more attractive* — and YouTube vanished globally for \~2 hours, its traffic routed to Pakistan
- **2019:** a small ISP's misconfiguration leaked routes that dragged huge swaths of Cloudflare/Amazon traffic through a congested network in Pennsylvania

Fixes exist (**RPKI** — cryptographic certificates of prefix ownership, increasingly enforced), illustrating a recurring theme of this series: the Internet's original protocols optimized for *working*, and security has been retrofitted ever since ([[10 - DNS, DHCP & the Web - Making It Human|DNS]], [[10 - DNS, DHCP & the Web - Making It Human|DHCP]], BGP).

---

## Loops, TTL, and why traceroute works

Routing during changes can briefly form **loops**: A sends to B, B to C, C back to A. Packets would circle forever — except every IP packet carries a **TTL (Time To Live)**, decremented at each hop; at zero, the router discards it and sends back an ICMP "Time Exceeded" message naming itself.

That safety mechanism doubles as a diagnostic goldmine: [[16 - Tools of the Trade - Seeing the Network|traceroute]] sends packets with TTL 1, 2, 3… forcing each successive router to identify itself. The hop list *is* TTL, weaponized for good.

---

## Connecting the dots

- Routing tables are why the hierarchy matters: prefixes aggregate — an ISP announces *one* summary route for thousands of customers (supernetting), keeping the global table near a million entries instead of billions
- Convergence time explains real-world behavior: after an outage, routes can take minutes to settle — the Internet "heals," but not instantly
- Everything here is control-plane machinery: it *builds the map*; the data plane (forwarding) is just longest-prefix-match at wire speed

**Next →** An open, trusting network needs defending at its borders. [[13 - Firewalls, VPNs & the DMZ - Defending the Network|13 — Firewalls, VPNs & the DMZ: Defending the Network]].

---

## 🤔 Pause & think

*Try answering out loud before peeking at the hints — explaining a thing is how it becomes yours.*

1. **A destination matches both a /16 route and a /24 route. Which wins, and what's the principle called?**<br />*The /24 — longest prefix match. The most specific information wins.*
2. **Why do your packets sometimes take a geographically silly route?**<br />*BGP chooses by business policy — money and peering contracts between ASes — not by distance or speed.*
3. **What stops a routing loop from circulating a packet forever?**<br />*The TTL field: decremented at every hop and discarded at zero — the same mechanism traceroute exploits.*

### Key takeaways

- [ ] **Longest prefix match** decides every forwarding decision; <code>0.0.0.0/0</code> (default) is the least specific route of all
- [ ] Static = hand-drawn, fine for tiny/stable; dynamic = self-updating
- [ ] **OSPF** (link-state, full map, Dijkstra) runs *inside* organizations; **BGP** (path-vector, policy + trust) runs *between* them — the Internet is BGP
- [ ] BGP's trust model enables prefix hijacks; RPKI is the retrofit. **TTL** prevents infinite loops and powers traceroute

---

← Previous: [[11 - Wi-Fi Deep Dive - Networking Through Air]]  |  ↑ Parent: [[Networking]]  |  Next: [[13 - Firewalls, VPNs & the DMZ - Defending the Network]] →
