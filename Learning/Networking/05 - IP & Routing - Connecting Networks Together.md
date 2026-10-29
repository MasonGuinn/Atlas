---
title: "05 - IP & Routing: Connecting Networks Together"
domain: networking
type: lesson
lesson: 5
status: not-started
---


← Previous: [[04 - LANs - Many Machines, One Room]]  |  ↑ Parent: [[Networking]]  |  Next: [[06 - Subnetting - Dividing Networks on Purpose]] →

---


> **Previously:** inside one [[04 - LANs - Many Machines, One Room|LAN]], [[04 - LANs - Many Machines, One Room|switches]] deliver [[03 - The First Conversation - Two Computers, One Wire|frames]] by [[03 - The First Conversation - Two Computers, One Wire|MAC address]] — fast and elegant, but trapped in one building. This note solves the distance problem, and it's the heart of the whole series: **the Internet Protocol is what turns millions of separate LANs into one Internet.**

> 🌍 **The big picture:** IP is the Internet's postal system. Hierarchical addresses mean nobody needs a map of the whole planet — every router is a sorting office that only knows the *next* office. That one idea is what lets millions of separate networks behave like a single Internet.

---

## Why MAC addresses can't cross the world

Imagine the postal service tried to deliver mail using only people's *names* — no addresses. "Deliver this to Maria Garcia." Which one? Where? A mail carrier would need a directory of every human on Earth and their current location.

MAC addresses have the same flaw. They're flat — they tell you *who* a device is, nothing about *where* it is. A switch can memorize a few thousand local MACs, but no device could store directions to billions of MACs scattered across the planet, constantly moving.

**The fix:** give every device a *second* address, one based on location — an **IP address**. Then devices only need to know directions like "for anything outside my neighborhood, head toward the highway" instead of memorizing every destination.

> 📬 **The full analogy:** MAC = your name (permanent, identifies you). IP = your home address (locational, changes when you move). Local delivery uses names (switch/MAC); long-distance delivery uses addresses (router/IP). Devices need both, the way a package needs both a name and an address.

---

## IPv4: the address that built the Internet

An IPv4 address is 32 bits, written as four numbers 0–255: <code>203.0.113.45</code>. But the magic is in its structure — every address is really **two addresses in one**:

- **Network portion** — "which LAN" (like the city + ZIP in a postal address)
- **Host portion** — "which device on that LAN" (like the house number)

The [[06 - Subnetting - Dividing Networks on Purpose|subnet]]** mask** says where the split is. <code>255.255.255.0</code> (= **/24**) means "the first 24 bits are the network, the last 8 are the host" — 254 usable devices on that network. This hierarchy is the scalability trick: a router in Tokyo doesn't track your laptop — it only needs to know "the 203.0.113.0/24 neighborhood is that-a-way," the way a post office only needs the right city, not your street.

**The catch nobody planned for:** 32 bits = \~4.3 billion addresses, which in 1981 seemed inexhaustible. Then every human got a phone, a laptop, a watch, and a refrigerator. The patches that bought us decades:

- **Private addresses + **[[08 - NAT - How One Address Serves a Whole House|NAT]] — whole LANs hide behind one public IP. The ranges (10.x, 172.16–31.x, 192.168.x) from the Fundamentals note
- [[09 - IPv6 - The Fix That Took 30 Years|IPv6]] — the real fix: 128-bit addresses, so many that exhaustion stops being a meaningful concept. Adoption has been a slow, decades-long migration because IPv4's patches worked "well enough"

---

## The router: the Internet's intersection

A **router** is a device with feet in multiple networks at once (it has an IP address in *each* network it connects). Its job:

1. Receive a packet
2. Look at the **destination IP** (not MAC — we've left the neighborhood)
3. Consult its **routing table**: a list of "to reach network X, send the packet to next-hop router Y (or deliver directly if X is one of my networks)"
4. Forward it. Repeat at every router along the path — a trip across the Internet typically crosses **10–20 routers ("hops")**

Routers don't know the whole path — each one knows only the *next step*, like highway signs that say "Seattle: this exit direction" without describing Seattle's streets. (Try <code>tracert</code> / <code>traceroute</code> on your own machine to watch the hops your packets take — it uses **ICMP**, IP's companion protocol for diagnostics, the same one <code>ping</code> uses.)

### The default gateway: the only direction most devices know

Your laptop's routing table is beautifully ignorant:

- Destination is on *my* LAN → deliver directly (well, via [[00 - Fundamentals - The Reference Map|ARP]] + switch)
- Destination is *anywhere else* → hand it to the **default gateway** (your router) and let it figure things out

That single rule is enough for billions of devices to use a network none of them can map.

### How do routers learn their tables?

Big routers talk to each other using routing protocols (like [[12 - Routing Deep Dive - How the Internet Finds Its Way|BGP]], the Border Gateway Protocol that runs *between* the giant networks — ISPs and tech companies — that make up the Internet; and [[12 - Routing Deep Dive - How the Internet Finds Its Way|OSPF]] inside large organizations). They constantly exchange "I can reach these networks" announcements and compute best paths. When you read about a "BGP incident" taking down a chunk of the Internet, it's this system being misconfigured or lied to — the Internet literally runs on routers trusting each other's directions.

---

### Anatomy of an IP packet (the fields that actually matter)

| Field | Job |
| --- | --- |
| **Source / Destination IP** | Who sent it, where it's going — the addresses this whole note is about |
| **TTL** | Hop countdown; at zero the packet is discarded (this stops loops — and powers traceroute) |
| **Protocol** | What's inside: TCP (6), UDP (17), ICMP (1) — tells the destination which transport protocol gets the payload |
| **Header checksum** | Integrity check of the header only — recalculated at every hop, because TTL keeps changing |

Notice what's *absent*: no port numbers (transport's job), no sequence numbers, no delivery promises. The header is minimal on purpose — backbone routers process millions of these per second, and every extra field would be a tax on all of them.

## IP's fine print: it promises nothing

Read IP's actual guarantees, because they're shocking:

- ❌ No guarantee the packet arrives
- ❌ No guarantee it arrives *once*, or *in order*
- ❌ No guarantee it arrives uncorrupted (IP has only a weak header checksum)
- ✅ It will *try* ("best effort delivery")

This was deliberate (remember the end-to-end principle): a simple, fast, unreliable core that any technology can run over — IP has run over phone lines, [[03 - The First Conversation - Two Computers, One Wire|Ethernet]], [[11 - Wi-Fi Deep Dive - Networking Through Air|Wi-Fi]], satellites, and literal carrier pigeons (as a joke RFC, but it *worked*). Fixing the unreliability is left to the endpoints — and that fix is the star of the next note.

---

## Where we stand

Your packet can now find its way from a dorm in Pasco to a server in Virginia through a dozen routers, using hierarchical addresses and hop-by-hop forwarding. But IP alone gives you *postcards dropped in a mailbox*: no confirmation, no order, no re-sends, and no way to say which *program* on the destination machine should get the data.

**Next →** Before any of this runs, someone has to decide where each address's network/host split goes — and how big each network gets. That craft is next: [[06 - Subnetting - Dividing Networks on Purpose|06 — Subnetting: Dividing Networks on Purpose]].

---

## 🤔 Pause & think

*Try answering out loud before peeking at the hints — explaining a thing is how it becomes yours.*

1. **Your laptop knows exactly one direction for 'anywhere else.' What is it called, and why is that enough?**<br />*The default gateway — your router. Hierarchy means each device only needs the next hop, never the whole map.*
2. **A packet crosses 14 routers on its way to a server. How many of them know the full path?**<br />*None. Each knows only its own next hop — like highway signs pointing toward a city they can't describe.*
3. **Name three things IP does *****not***** guarantee.**<br />*Delivery, arrival order, and no duplicates. (There's also no real error correction — only a weak header checksum.) Best effort only.*

### Key takeaways

- [ ] IP addresses are **hierarchical** (network + host), which is what makes planet-scale routing possible; MAC addresses are flat, which is why they can't be
- [ ] **Routers** forward packets hop-by-hop using **routing tables**; each router knows only the next step
- [ ] The **default gateway** rule — local? deliver directly; anything else? hand it to the router — is all most devices know
- [ ] IP is **best-effort**: no delivery, order, or duplication guarantees — by design; reliability is the endpoints' problem

---

← Previous: [[04 - LANs - Many Machines, One Room]]  |  ↑ Parent: [[Networking]]  |  Next: [[06 - Subnetting - Dividing Networks on Purpose]] →
