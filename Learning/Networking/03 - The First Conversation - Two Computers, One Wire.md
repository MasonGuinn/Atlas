---
title: "03 - The First Conversation: Two Computers, One Wire"
domain: networking
type: lesson
lesson: 3
status: not-started
---

← Previous: [[02 - Bits & Signals - How Computers Physically Talk]]  |  ↑ Parent: [[Networking]]  |  Next: [[04 - LANs - Many Machines, One Room]] →

---


> **Previously:** we can turn bits into signals and push them down a wire. Now two computers are plugged into the same cable. They immediately face the problems of any conversation: *Who am I talking to? When does a message start and end? What if it arrives damaged?*

> 🌍 **The big picture:** This is the smallest complete network conversation: *who are you* (MAC address), *here's a packaged chunk* (frame), *did it arrive intact* (checksum)? Every grander system later in the series is this conversation plus a way to cross between networks.

---

## Problem 1: Identity — the MAC address

If a signal travels down a shared wire, every attached device can "hear" it. So each device needs a name so senders can say who a message is *for*.

The solution was to give every network card a **permanent, unique hardware address** at the factory: the **MAC address** (Media Access Control) — 48 bits, written as six pairs of hex digits:

```javascript
3C:7A:9F:12:B8:01
```

- The **first half** identifies the manufacturer (assigned to companies by the IEEE)
- The **second half** is a serial number the manufacturer assigns
- In theory, no two NICs on Earth share a MAC address

> 📬 **Analogy:** a MAC address is like a person's *name spoken aloud in a room* — it identifies someone locally, in earshot. It is **not** like a postal address: it says nothing about *where* the device is, and it never changes when the device moves. Hold onto that limitation — it's the entire reason [[05 - IP & Routing - Connecting Networks Together|IP addresses]] had to be invented.

---

## Problem 2: Framing — where does a message begin and end?

A raw bit stream has no punctuation. The fix is to send data in structured chunks called **frames**. A frame wraps the data with:

| Part | Purpose |
| --- | --- |
| **Destination MAC** | Who this is for |
| **Source MAC** | Who sent it (so you can reply) |
| **Type** | What's inside (e.g., an IP packet) |
| **Data (payload)** | The actual content (46–1500 bytes in classic Ethernet) |
| **Frame Check Sequence (FCS)** | A checksum — a mathematical fingerprint of the frame |

That checksum solves **Problem 3: damage.** The receiver recomputes the fingerprint; if it doesn't match, the frame was corrupted in transit and is simply *discarded*. Notice what's missing: nobody *fixes* it or asks for a resend at this level. This layer detects damage and drops the frame — getting a replacement is a higher layer's problem (that's [[07 - TCP & UDP - Making Delivery Reliable|TCP]]'s job). **Layers dividing labor** is a theme you'll see everywhere.

---

## The invention: Ethernet (1973)

At Xerox PARC in California, **Robert Metcalfe** (and colleagues) designed **Ethernet**, named after the "ether" — the invisible medium 19th-century physicists wrongly believed carried light. Ethernet defined the frame, the MAC addressing, and — critically — the *etiquette* for a shared wire.

### Problem 4: Who talks when? (CSMA/CD)

Early Ethernet was one shared cable for a whole office. If two computers transmitted at once, their signals garbled each other — a **collision**. Metcalfe's rules, **CSMA/CD** (Carrier Sense Multiple Access with Collision Detection), borrowed from polite conversation:

1. **Listen before speaking** — if the wire is busy, wait
2. **If two speak at once, both stop** — detect the collision, go silent
3. **Wait a random amount of time** — then try again (random, so they don't collide forever in lockstep)

It worked beautifully at small scale and badly at large scale: as more devices joined, more time was wasted on collisions. Imagine a meeting room where every interruption triggers random pauses — fine for 5 people, chaos for 500. Fixing this properly required replacing the shared wire itself, which is the story.

---

## What's actually inside the cable tray at a college or office

When you plug into a wall jack today, you're using Ethernet's descendants:

- Cables are rated by category — **Cat5e, Cat6, Cat6a** — each supporting higher speeds over the standard **100-meter** limit
- Speeds negotiated between devices: 100 Mbps → 1 Gbps → 10 Gbps and beyond
- The classic Ethernet *rules* survived even as the shared cable died: same frames, same MAC addresses, same 1500-byte payload tradition

That 1500 number follows you everywhere — the standard **MTU** (Maximum Transmission Unit) of Ethernet shapes packet sizes across the whole Internet to this day.

---

## Where we stand

Two (or a few) computers on one wire can now have an organized, addressed, error-checked conversation. But notice the limits of everything so far:

- MAC addresses only work for devices **on the same local wire** — there's no way to say "deliver this to a computer in another building, city, or country"
- The shared cable doesn't scale — collisions eat it alive
- There's still no concept of *finding a path* through multiple networks

**Next →** First, fix the local problem: [[04 - LANs - Many Machines, One Room|04 — LANs: Many Machines, One Room]] (hubs, switches, and the death of collisions). Then we tackle distance itself.

---

## 🤔 Pause & think

*Try answering out loud before peeking at the hints — explaining a thing is how it becomes yours.*

1. **Why does every frame carry the *****source***** MAC, not just the destination?**<br />*So the receiver knows who to reply to — and so *[[04 - LANs - Many Machines, One Room|*switches*]]* can learn where the sender lives.*
2. **A frame arrives with a bad checksum. What happens — and just as important, what does *****not***** happen?**<br />*It's silently discarded. Nobody resends it at this layer; recovery is TCP's job.*
3. **Why can't a MAC address be used to deliver data to another city?**<br />*It contains zero location information. It's a name, not an address.*

### Key takeaways

- [ ] Every NIC has a **MAC address** — a permanent hardware identity that only has meaning on the local network
- [ ] Data travels in **frames**: addressed chunks with a checksum (FCS) that lets receivers detect and discard corruption
- [ ] **Ethernet** (1973) defined frames and shared-wire etiquette (**CSMA/CD**) — listen first, back off randomly after collisions
- [ ] This layer delivers locally and detects damage, but does **not** resend — reliability belongs to a higher layer

---

← Previous: [[02 - Bits & Signals - How Computers Physically Talk]]  |  ↑ Parent: [[Networking]]  |  Next: [[04 - LANs - Many Machines, One Room]] →
