---
title: "04 - LANs: Many Machines, One Room"
domain: networking
type: lesson
lesson: 4
status: not-started
---

> [!navbar]
> 🏠 [[Dashboard]] · 📚 [[Learning]] · 🌐 [[Networking]] · 💻 [[Systems]] · 🐍 [[Programming]] · 🔒 [[Cybersecurity]] · 🎓 [[School]]


← Previous: [[03 - The First Conversation - Two Computers, One Wire]]  |  ↑ Parent: [[Networking]]  |  Next: [[05 - IP & Routing - Connecting Networks Together]] →

---


> **Previously:** [[03 - The First Conversation - Two Computers, One Wire|Ethernet]] let a few computers share one cable using politeness rules (CSMA/CD). This note is about what happens when the room fills up — dozens, then hundreds of machines — and why the humble **switch** quietly became one of the most important devices in networking.

> 🌍 **The big picture:** A LAN is a neighborhood, and inside it the **switch** is in charge, delivering by [[03 - The First Conversation - Two Computers, One Wire|MAC address]]. The instant a destination lives *outside* the neighborhood, a completely different system — [[05 - IP & Routing - Connecting Networks Together|IP]] and [[05 - IP & Routing - Connecting Networks Together|routing]] — takes over. Knowing which system is handling your traffic at each moment is half of understanding networking.

---

## What is a LAN, really?

A **Local Area Network** is the network in one place: your home, an office floor, a college building. Its defining traits:

- Geographically small (one site)
- Owned and run by one organization or person
- High speed, low [[02 - Bits & Signals - How Computers Physically Talk|latency]] — everything is close

Everything in the previous notes — [[03 - The First Conversation - Two Computers, One Wire|frames]], MAC addresses, Ethernet — operates *inside* a LAN. The LAN is the neighborhood; later notes are about traveling between cities.

---

## Attempt #1: the hub (and why it failed)

To wire up many computers, offices used a **hub**: a box where all the cables meet. Electrically, a hub is just a shared cable in a box — it takes every incoming signal and **repeats it out of every other port**. Every device hears everything; each NIC ignores frames not addressed to it.

Two fatal flaws followed:

1. **Collisions everywhere.** Since it's one shared medium, *all* devices take turns under CSMA/CD. 20 computers on a hub = a 20-way conversation where only one can speak at a time, with constant interruptions. A "100 Mbps hub" gave each user a tiny slice in practice.
2. **Zero privacy.** Every frame physically reaches every machine. A NIC in "promiscuous mode" can read everyone's traffic — the original LAN eavesdropping problem. (Remember this in security work: this weakness never fully went away — it evolved, see [[00 - Fundamentals - The Reference Map|ARP]] attacks in the Fundamentals note.)

> 💡 **Terminology:** a set of devices that can collide with each other is a **collision domain**. A hub = one big collision domain. The goal of LAN design became: *shrink collision domains.*

---

## The fix: the switch

A **switch** looks like a hub but thinks. It's a computer dedicated to one clever trick:

1. When a frame arrives, the switch notes the **source MAC** and which **port** it came in on, building a table: *"MAC 3C:7A… lives on port 4"* (the **CAM table**)
2. When a frame arrives for a known destination, the switch forwards it **only out that one port**
3. Unknown destination? Temporarily flood it everywhere (like a hub) until a reply teaches the switch where that MAC lives

> 📬 **Analogy:** a hub is an office worker who *shouts every message through a megaphone* and lets everyone sort out what's theirs. A switch is a **mailroom clerk** who quietly learns everyone's desk and walks each envelope straight to it — faster, private, and nobody else interrupted.

### Why this changed everything

- **Each port is its own collision domain.** Two computers on different ports can talk at the same time as two others — conversations happen in *parallel*
- **Full-duplex became possible.** With a dedicated point-to-point link to the switch, a device can send and receive simultaneously — so collisions *can't happen at all*, and CSMA/CD politely retired from wired networks
- **Privacy improved.** Traffic normally goes only where it's addressed (not perfectly — attacks exist — but the default changed from "everyone hears all" to "only the recipient hears it")

**Speed math:** a 24-port switch where every pair talks at 1 Gbps simultaneously moves vastly more total data than a 24-port hub capped at 100 Mbps *shared*. Switching is why LANs scaled from offices of 10 to campuses of 10,000.

---

## Broadcast: the necessary shouting

Sometimes a device genuinely needs to reach *everyone* on the LAN — for example, "whoever has [[05 - IP & Routing - Connecting Networks Together|IP address]] 192.168.1.1, tell me your MAC" (that's ARP, from the Fundamentals note). For this there's a special address: the **broadcast MAC** <code>FF:FF:FF:FF:FF:FF</code>. Switches flood broadcast frames out all ports.

- The set of devices reached by a broadcast is the **broadcast domain**
- Broadcasts are essential but expensive — *every* device must stop and process them
- Too big a broadcast domain = constant interrupting chatter ("broadcast storms" in the worst cases)

This creates a design pressure: keep broadcast domains reasonably small. The tool for that is the **VLAN** (Virtual LAN) — one physical switch carved into several *logical* LANs, each its own broadcast domain, unable to see each other. Colleges use VLANs to separate student, staff, and IoT traffic on the same hardware — a security boundary as much as an organizational one.

---

## Wi-Fi: the LAN without wires

[[11 - Wi-Fi Deep Dive - Networking Through Air|Wi-Fi]] is Ethernet's wireless sibling: same LAN concepts, same MAC addresses, same frames conceptually — but the medium is shared air, which brings back old problems:

- Air is a **shared medium** again, so Wi-Fi uses **CSMA/CA** (Collision *Avoidance*): since a radio can't easily hear a collision while transmitting, devices try extra hard not to talk over each other, and every frame gets an acknowledgment
- **Half-duplex by nature:** on a given channel, one device talks at a time in each direction
- The **access point** is essentially a bridge: it translates between the wireless LAN and the wired switch

That's why a wired connection beats Wi-Fi for gaming and calls even at the "same" speed rating: the bottleneck isn't [[02 - Bits & Signals - How Computers Physically Talk|bandwidth]] — it's everyone politely sharing the air (latency under contention).

---

## Where we stand

Inside one building, we're in great shape: switches deliver frames directly and privately between MAC addresses at high speed. But look at the boundary of the LAN:

- Switches **cannot** get a frame to another building's LAN — MAC addresses contain no location information, and no switch could hold a table of every MAC on Earth
- Broadcasts can't cross to other networks (by design — they'd melt the world)

We need a new layer: an addressing system based on *location*, devices that forward *between* networks, and a way to find paths across a planet-sized web of LANs.

**Next →** [[05 - IP & Routing - Connecting Networks Together|05 — IP & Routing: Connecting Networks Together]] — the invention of the Internet itself.

---

## 🤔 Pause & think

*Try answering out loud before peeking at the hints — explaining a thing is how it becomes yours.*

1. **A switch receives a frame for a MAC it has never seen. What does it do — and how does it learn better?**<br />*It floods the frame out all ports, hub-style. When the reply comes back, it records which port that MAC lives on.*
2. **Why did replacing hubs with switches make networks faster *****and***** more private at the same time?**<br />*Conversations run in parallel instead of sharing one collision domain — and frames are delivered only to their destination port.*
3. **Why do designers insist on keeping broadcast domains small?**<br />*Every broadcast interrupts every device. At scale that's constant chatter — VLANs and *[[06 - Subnetting - Dividing Networks on Purpose|*subnets*]]* are the pressure valve.*

### Key takeaways

- [ ] A **hub** repeats everything everywhere: one collision domain, no privacy — obsolete
- [ ] A **switch** learns which MAC lives on which port and forwards frames only where needed — parallel, full-duplex, mostly private
- [ ] **Broadcasts** reach the whole LAN; the broadcast domain must stay small; **VLANs** carve one switch into isolated logical LANs
- [ ] Wi-Fi shares air, so it avoids collisions (CSMA/CA) and is half-duplex — why wired feels faster
- [ ] LANs cannot talk to each other with MAC addresses alone — a location-based system is needed

---

← Previous: [[03 - The First Conversation - Two Computers, One Wire]]  |  ↑ Parent: [[Networking]]  |  Next: [[05 - IP & Routing - Connecting Networks Together]] →
