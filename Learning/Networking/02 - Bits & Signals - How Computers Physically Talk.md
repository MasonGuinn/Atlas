---
title: "02 - Bits & Signals: How Computers Physically Talk"
domain: networking
type: lesson
lesson: 2
status: not-started
---

> [!navbar]
> 🏠 [[Dashboard]] · 📚 [[Learning]] · 🌐 [[Networking]] · 💻 [[Systems]] · 🐍 [[Programming]] · 🔒 [[Cybersecurity]] · 🎓 [[School]]


← Previous: [[01 - Why Networks Exist]]  |  ↑ Parent: [[Networking]]  |  Next: [[03 - The First Conversation - Two Computers, One Wire]] →

---


> **Previously:** networks exist to share, and [[01 - Why Networks Exist|packet switching]] is the strategy. But here's the most basic problem of all: a computer's data lives as numbers in memory, and a wire is just... a wire. *How does information physically get from one machine to another?*

> 🌍 **The big picture:** Everything above this note — every protocol, app, and attack — is ultimately patterns of 1s and 0s riding on a physical signal. Higher layers pretend bits arrive perfectly; *this* layer is where physical reality (distance, noise, the speed of light) sets limits nobody can negotiate with.

---

## Everything is bits

Inside a computer, all information — text, photos, video, this note — is stored as **binary digits (bits)**: 1s and 0s. Groups of 8 bits form a **byte**. Text is encoded by agreeing that certain bit patterns mean certain characters (that's what **ASCII** and later **Unicode/UTF-8** are: giant lookup tables from numbers to characters — <code>01000001</code> = "A").

**Why binary?** Because physical systems are good at distinguishing two states: voltage high or low, light on or off, magnetized this way or that. Two states are easy to tell apart even with electrical noise — ten states would get muddy fast. Binary is the most *reliable* language a physical device can speak.

So the entire problem of networking reduces to this: **send a stream of 1s and 0s from here to there, accurately and fast.**

---

## Turning bits into signals

A **signal** is a physical quantity that varies over time to carry information. Computers encode bits onto signals in three main ways:

| Medium | Signal | Where you see it | Traits |
| --- | --- | --- | --- |
| **Copper cable** | Voltage changes (electrical pulses) | Ethernet cables | Cheap, short-range (\~100 m), can suffer electrical interference |
| **Fiber optic** | Pulses of light | Undersea cables, ISP backbones, data centers | Extremely fast, very long range, immune to electrical interference |
| **Radio waves** | Modulated wireless signals | Wi-Fi, Bluetooth, 4G/5G | Convenient, no wires — but shared airspace, interference, easier to intercept |

A device called a **network interface card (NIC)** does this translation — every phone, laptop, and server has one (or a chip doing the same job). It turns the computer's bits into signals on the medium, and incoming signals back into bits.

### Analog waves, digital data

Radio and voltage are *analog* — smooth waves. To put bits on a wave, the sender varies some property of it: its height (**amplitude**), its speed of oscillation (**frequency**), or its timing (**phase**). Deliberately changing a wave to encode data is called **modulation** — it's literally what "modem" stands for (**mo**dulator-**dem**odulator), the box that adapted computers to phone lines and now to cable/fiber.

---

### The hidden problem: agreeing on timing

Bits arrive as a continuous signal — so how does the receiver slice it into separate 1s and 0s without them blurring together? Sender and receiver agree on a **bit rate** (how long each bit lasts), and the encoding is designed so the signal itself keeps them synchronized: frequent *transitions* (changes in the signal) act like a beat the receiver's clock can lock onto, the way dancers adjust to music they can hear. Long runs with no transitions are the enemy — which is why real line encodings are built to keep transitions coming. Lose sync, and a stream of identical bits becomes uncountable mush: one more physical-layer failure that every higher layer simply experiences as "the network is broken."

## The two measurements that rule everything: bandwidth & latency

Every link can be judged by two numbers, and confusing them causes endless misunderstanding:

- **Bandwidth** — how *much* data can flow per second (bits per second: Mbps, Gbps). Think: the width of a highway.
- **Latency** — how *long* one bit takes to get there. Think: the speed limit / length of the highway.

A satellite link has enormous bandwidth but terrible latency (the signal travels \~72,000 km to orbit and back — physics sets a floor of \~240 ms). A short cable has tiny latency. **You can buy more bandwidth; you cannot beat the speed of light** — light in fiber travels \~200,000 km/s, so a New York ↔ London trip has a hard minimum of \~28 ms one-way no matter what anyone pays.

**Throughput** is what you actually get after real-world losses — like the cars that actually complete the highway trip per hour, versus the theoretical maximum.

> 🎮 **Why gamers care:** online games send tiny amounts of data, so bandwidth barely matters — but every millisecond of latency delays your actions. That's why "ping" (latency) rules gaming while "speed" (bandwidth) rules downloads. Different jobs, different metric.

---

## Sending bits in both directions

- **Simplex** — one direction only (a radio broadcast)
- **Half-duplex** — both directions, but one at a time (a walkie-talkie: "over!")
- **Full-duplex** — both directions simultaneously (a phone call)

Old [[03 - The First Conversation - Two Computers, One Wire|Ethernet]] was half-duplex, which caused the collision problem. Modern wired Ethernet is full-duplex — a separate lane each way, like a two-way street replacing a one-lane bridge.

---

## The problem bits alone can't solve

Okay: we can push bits down a wire. But raw bits raise questions that physics can't answer:

- Where does one message *end* and the next begin? A stream of 1s and 0s has no commas.
- If two devices share a cable, who talks when?
- How does a device know the bits were meant for *it*?
- What if some bits got garbled on the way?

Those are not physical problems — they're **agreement problems**. And agreements between machines are called **protocols**. The next note introduces the first real protocol machinery: how two directly connected computers hold an organized conversation.

**Next →** [[03 - The First Conversation - Two Computers, One Wire|03 — The First Conversation: Two Computers, One Wire]]

---

## 🤔 Pause & think

*Try answering out loud before peeking at the hints — explaining a thing is how it becomes yours.*

1. **Why binary, instead of ten voltage levels for digits 0–9?**<br />*Two states are far easier to tell apart through noise. More levels per signal = more chances to misread.*
2. **Your ping is 25 ms but downloads crawl at peak hour on a fast plan. Which metric is which — and which is failing?**<br />*Ping is latency, and it's fine. The crawl is throughput under congestion: the highway is full, not slow.*
3. **Why can't a pricier satellite plan fix satellite gaming lag?**<br />*That latency is distance ÷ the speed of light — a physics floor of \~240 ms, not a bandwidth limit you can pay away.*

### Key takeaways

- [ ] All data is **bits**; binary is used because two physical states are the most reliable to distinguish
- [ ] Bits ride on **electrical pulses (copper)**, **light (fiber)**, or **radio (wireless)**; the **NIC** translates
- [ ] **Bandwidth** = how much per second; **latency** = how long to arrive; **throughput** = what you actually get
- [ ] Latency is limited by physics (speed of light); bandwidth can be bought

---

← Previous: [[01 - Why Networks Exist]]  |  ↑ Parent: [[Networking]]  |  Next: [[03 - The First Conversation - Two Computers, One Wire]] →
