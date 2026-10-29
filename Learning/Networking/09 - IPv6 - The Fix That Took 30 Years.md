---
title: "09 - IPv6: The Fix That Took 30 Years"
domain: networking
type: lesson
lesson: 9
status: not-started
---

> [!navbar]
> 🏠 [[Dashboard]] · 📚 [[Learning]] · 🌐 [[Networking]] · 💻 [[Systems]] · 🐍 [[Programming]] · 🔒 [[Cybersecurity]] · 🎓 [[School]]


← Previous: [[08 - NAT - How One Address Serves a Whole House]]  |  ↑ Parent: [[Networking]]  |  Next: [[10 - DNS, DHCP & the Web - Making It Human]] →

---


> **Previously:** [[05 - IP & Routing - Connecting Networks Together|IPv4]] ran out of addresses, and [[08 - NAT - How One Address Serves a Whole House|NAT]] was the stopgap. In 1998 the real fix was standardized: **IPv6**. Three decades later the migration is still underway. This note explains what IPv6 changes, why it's designed the way it is, and why "better" didn't mean "fast."

> 🌍 **The big picture:** IPv6 is what the Internet looks like when addresses are *free*. Scarcity designed IPv4's hacks (NAT, broadcast shouting, hand-configuration); abundance designs IPv6's simplicity (self-configuration, multicast, an address for everything). Same internetworking idea — opposite economics.

---

## The address space, made absurd

IPv6 addresses are **128 bits**: 340,282,366,920,938,463,463,374,607,431,768,211,456 of them. That's roughly 5×10²⁸ addresses **per human alive** — enough to give every atom on Earth's surface its own address several times over. The design goal wasn't just "more"; it was "so many that we never engineer around scarcity again." Almost every IPv6 design decision flows from that abundance:

- No NAT needed — every device can have a globally unique address (end-to-end restored)
- No broadcast — with addresses to spare, **multicast** (send to a defined group) replaces shouting at everyone
- Huge [[06 - Subnetting - Dividing Networks on Purpose|subnets]] by default: a standard IPv6 [[04 - LANs - Many Machines, One Room|LAN]] is a **/64** — 18 quintillion hosts *per subnet*. [[06 - Subnetting - Dividing Networks on Purpose|Subnetting]] in IPv6 is about structure and security, never about saving addresses

---

> 🔢 **Scale check:** if IPv4's address space were the grains of sand in a bucket, IPv6's would be more grains than there are atoms in all the sand on Earth. Scarcity thinking — "save every address!" — simply stops applying, and several IPv6 design choices only make sense once you feel that.

## Reading an IPv6 address

Written as 8 groups of 4 hex digits, separated by colons:

```javascript
2001:0db8:85a3:0000:0000:8a2e:0370:7334
```

Two compression rules make this survivable:

1. **Leading zeros in a group can be dropped:** <code>0db8</code> → <code>db8</code>, <code>0000</code> → <code>0</code>
2. **One run of all-zero groups can be replaced by ****<code>::</code>** (only once per address)

So the address above becomes <code>2001:db8:85a3::8a2e:370:7334</code>. And the loopback is simply <code>::1</code>.

### The address types you must know

| Type | Prefix | IPv4 analogy | Notes |
| --- | --- | --- | --- |
| **Link-local** | <code>fe80::/10</code> | APIPA (169.254.x.x) | Auto-created on every interface, works only on the local link — but here it's *by design*, not a failure |
| **Unique local** | <code>fc00::/7</code> | Private ranges (192.168…) | Not routed on the Internet |
| **Global unicast** | <code>2000::/3</code> | Public IPv4 | The real, routable Internet |
| **Multicast** | <code>ff00::/8</code> | — (no IPv4 equivalent in daily use) | Replaces broadcast |

Every IPv6 interface normally holds **several addresses at once** — link-local + one or more global — which surprises IPv4 brains. It's normal.

---

## Configuration without (much) DHCP: SLAAC

IPv4 devices beg a [[10 - DNS, DHCP & the Web - Making It Human|DHCP]] server for an address. IPv6 devices can largely do it themselves via **SLAAC** (Stateless Address Autoconfiguration):

1. The device builds its **link-local** address automatically and verifies it's unique on the link (Duplicate Address Detection)
2. It asks the link's [[05 - IP & Routing - Connecting Networks Together|routers]] to identify themselves (**Router Solicitation**); routers also periodically *advertise* themselves (**Router Advertisements**) carrying the network's prefix
3. The device combines **prefix (from router) + interface ID (its own)** = a global address. Done — no server state involved

Routers advertise; hosts self-configure. DHCPv6 still exists for organizations that want central control and for handing out [[10 - DNS, DHCP & the Web - Making It Human|DNS]] servers — but it's optional, inverting IPv4's dependency.

**Privacy wrinkle:** the original way to build the interface ID used the [[03 - The First Conversation - Two Computers, One Wire|MAC address]] — meaning your device would carry the *same* identifier on every network on Earth: a super-cookie burned into hardware. Modern systems instead use random, rotating **temporary addresses** (privacy extensions). Security thinking is baked into IPv6 in ways IPv4 had to retrofit.

### ARP's replacement: Neighbor Discovery

IPv6 deletes [[00 - Fundamentals - The Reference Map|ARP]] entirely. Its jobs — finding the neighbor's link-layer address, finding routers, checking duplicates — move to **Neighbor Discovery Protocol (NDP)**, running over **ICMPv6** using *multicast* instead of broadcast. Same mission, new machinery — and the same family of attacks (rogue router advertisements ≈ rogue DHCP).

---

## So why isn't migration done?

- **NAT worked too well.** The pain IPv6 solves was anesthetized in 1994. No flag day is possible on a network of billions owned by millions of parties (contrast the 1983 switch, when [[01 - Why Networks Exist|ARPANET]] was small).
- **Dual stack:** most devices and networks run IPv4 *and* IPv6 side-by-side, content providers serve both, and your traffic picks whichever works better (browsers race them — "Happy Eyeballs"). You likely already use IPv6 daily (mobile networks especially) without knowing.
- **Cost/benefit:** for many organizations IPv6 brings new tooling, training, and risk — for an benefit (more addresses) they don't personally feel. Adoption inches upward (\~45%+ of Google's users as of the mid-2020s) rather than flipping.

> 🛡️ **Security implications:** the vast address space kills casual scanning (an attacker can't sweep a /64 — it would take centuries), but *your* misconfigurations matter more: an accidentally-open IPv6 path can bypass an IPv4-only [[13 - Firewalls, VPNs & the DMZ - Defending the Network|firewall]]'s rules entirely. Defenders must audit **both** stacks — dual stack means dual attack surface.

---

## Connecting the dots

- IPv6 assumes the hierarchy, the [[05 - IP & Routing - Connecting Networks Together|routing]], the transport — [[07 - TCP & UDP - Making Delivery Reliable|TCP]]/[[07 - TCP & UDP - Making Delivery Reliable|UDP]] are unchanged; only the network layer swapped
- NDP replaces ARP; SLAAC absorbs much of DHCP's role
- Every prior IPv4 concept has a deliberate IPv6 answer — scarcity thinking → abundance thinking is the theme

**Next →** The addressing story is complete — v4, subnets, NAT, v6. Now the services that make all of it usable by actual humans: [[10 - DNS, DHCP & the Web - Making It Human|10 — DNS, DHCP & the Web: Making It Human]].

---

## 🤔 Pause & think

*Try answering out loud before peeking at the hints — explaining a thing is how it becomes yours.*

1. **Why does one IPv6 interface normally hold several addresses at once?**<br />*Different jobs: a link-local address for the local link, plus global address(es) for the Internet. IPv4 forced a single address to do everything.*
2. **ARP is gone in IPv6. What replaced it, and what carries it?**<br />*Neighbor Discovery Protocol, running over ICMPv6 — and using multicast instead of broadcast.*
3. **IPv6 is objectively better. Why is the migration 30 years old and still unfinished?**<br />*NAT anesthetized the pain IPv6 treats, migration costs are real while benefits are diffuse — and a network of billions has no one who can declare a flag day.*

### Key takeaways

- [ ] IPv6 = 128-bit addresses in hex, designed for *abundance*: no NAT, no broadcast, /64 subnets by default
- [ ] Every interface has multiple addresses — link-local (<code>fe80::</code>) is automatic and normal
- [ ] **SLAAC** lets hosts self-configure from router advertisements; **NDP/ICMPv6** replaces ARP
- [ ] Migration is slow because NAT anesthetized the pain; dual stack means **two** attack surfaces to defend

---

← Previous: [[08 - NAT - How One Address Serves a Whole House]]  |  ↑ Parent: [[Networking]]  |  Next: [[10 - DNS, DHCP & the Web - Making It Human]] →
