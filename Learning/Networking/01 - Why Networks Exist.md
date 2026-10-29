---
title: "01 - Why Networks Exist"
domain: networking
type: lesson
lesson: 1
status: in-progress
---

← Previous: [[00 - Fundamentals - The Reference Map]]  |  ↑ Parent: [[Networking]]  |  Next: [[02 - Bits & Signals - How Computers Physically Talk]] →

---


> **Start here.** Before protocols, ports, and models, there's a simpler question: *why connect computers at all?* Every technology in this series exists because it solved a real problem someone had. If you keep asking "what problem does this solve?", networking stops being memorization and starts being a story.

> 🌍 **The big picture:** Every technology in this series exists because of decisions made between 1969 and 1983: share resources by chopping data into *packets*, send them over whatever wires exist, and keep the network core simple while the edges do the clever work. When later notes confuse you, come back to that sentence.

---

## The world before networks

In the 1960s, a computer was a room-sized machine that cost millions of dollars. Universities and government agencies might own *one*. If you wanted to use it, you physically went to it, fed it punch cards or typed at a terminal in the same building, and waited.

This created three painful problems:

1. **Computers were stranded.** A researcher in California couldn't use an idle computer in Utah — even if their own machine was overloaded. Expensive machines sat unused while others had waiting lists.
2. **Information couldn't move.** To share data between two computers, someone literally carried magnetic tapes or punch cards between buildings. The fastest "network" of the era was a person in a car.
3. **Every computer was an island.** Different manufacturers built machines that spoke completely different languages. Even if you connected them with a wire, they couldn't understand each other.

**The core insight:** a computer by itself is a calculator. A computer *connected to other computers* becomes something far more powerful — the value grows with every machine that joins. (This is why a phone is useless alone but priceless when everyone has one.)

---

## The birth of ARPANET (1969)

The U.S. Department of Defense's research agency, **ARPA** (now DARPA), funded universities to do computer research — and got tired of buying every team its own machine. Their idea: connect the computers at different universities so researchers could **share computing power and data** across the country.

On **October 29, 1969**, the first host-to-host connection was made between UCLA and Stanford Research Institute. The first message was supposed to be <code>LOGIN</code>. The system crashed after two letters — so the first message ever sent on the network that became the Internet was literally **<code>LO</code>**.

By December 1969, ARPANET had four nodes: UCLA, Stanford, UC Santa Barbara, and the University of Utah.

> 🗓️ **A common myth:** ARPANET was *not* primarily built to survive a nuclear war. That idea existed in related research (Paul Baran's work on distributed networks), and it influenced the design philosophy — but ARPANET's stated purpose was **resource sharing**. The decentralized design it used turned out to be resilient anyway, and that resilience became one of the Internet's defining traits.

### The design decision that changed everything: packet switching

Before ARPANET, communication networks (like the telephone system) used **circuit switching**: when you made a phone call, a dedicated physical path was reserved for you end-to-end for the entire call — even during silences when nobody was talking. Hugely wasteful.

ARPANET pioneered **packet switching** instead:

- Break every message into small chunks called **packets**
- Send each packet independently — packets from many conversations share the same wires, interleaved
- Packets can take *different routes* and be reassembled at the destination
- If a link fails, packets just route around it

Think of it like the postal system versus a private courier. A courier (circuit switching) drives one dedicated route just for you. The postal service (packet switching) moves millions of letters through shared trucks and sorting centers — your letter might take a different path than the one you mailed yesterday, and the system still works if one road closes.

**Why this matters for security:** there is no single dedicated line to tap or protect. Your data is chopped up, mixed with everyone else's, and sprayed across shared infrastructure owned by strangers. Almost every networking security concept — encryption, spoofing, interception — exists because of this one design choice.

---

## The email surprise

In 1971, Ray Tomlinson sent the first network **email** — and chose the <code>@</code> symbol to separate user from machine (a symbol nobody was using in names, sitting right there on the keyboard).

Nobody planned for email to be the main application. Within a couple of years it made up the **majority of ARPANET traffic**. Lesson that repeats throughout networking history: **the killer app is rarely the one the designers expected.** The network was built for sharing computers; humans used it to talk to each other.

---

## From many networks to *the* Internet

Through the 1970s, lots of networks appeared — universities, companies, and other countries built their own. But each had its own rules, so they couldn't talk to each other. The problem had come back, one level up: *networks* were now the islands.

The fix, from Vint Cerf and Bob Kahn (1974), was [[07 - TCP & UDP - Making Delivery Reliable|TCP]]**/IP** — a shared set of protocols any network could adopt so that networks could interconnect. An "inter-network" — an **internet**. On **January 1, 1983** ("Flag Day"), ARPANET switched entirely to TCP/IP. That date is often called the Internet's birthday.

Two revolutionary ideas inside TCP/IP explain everything that follows in this series:

1. **Keep the middle dumb, the ends smart.** The network's only job is to try to deliver packets ("best effort"). Reliability, ordering, and meaning are handled by the computers at the *edges*. This is called the **end-to-end principle**, and it's why the Internet could scale to billions of devices — the core stays simple.
2. **No owner, no center.** Anyone could join by following the protocols. No permission, no central switchboard. Growth nobody had to approve.

---

## Then the Web made it explode

In 1989–1991, Tim Berners-Lee at CERN (a physics lab in Switzerland) built the **World Wide Web** — web pages, URLs, and the first browser — because physicists couldn't keep track of documents scattered across incompatible computers. In 1993, CERN put the Web in the public domain, and the graphical **Mosaic** browser made it usable by non-experts. The Internet went from a researchers' tool to a mass medium in a few years.

> ⚠️ **Web ≠ Internet.** The Internet is the underlying network (1970s–80s). The Web is one *service* that runs on top of it (1990s) — like how roads are the infrastructure and trucking companies are services that use them. Email, gaming, streaming, and the Web are all separate services riding the same Internet.

---

## The "why" in one paragraph

Networks exist to **share** — computing power, information, and communication. ARPANET proved packet switching worked; TCP/IP let independent networks merge into one Internet; email and the Web showed what people actually do with it. Every note after this one zooms into one piece of that story and asks the same question: *what problem was this solving?*

**Next →** Before any of this can work, two machines need a way to physically send information to each other. That starts with a humble idea: the bit. See [[02 - Bits & Signals - How Computers Physically Talk|02 — Bits & Signals]].

---

## 🤔 Pause & think

*Try answering out loud before peeking at the hints — explaining a thing is how it becomes yours.*

1. **What problem was ARPANET actually built to solve?**<br />*Sharing scarce, expensive computers and their resources — not surviving nuclear war. That's the myth this note busted.*
2. **If packet switching is the postal service, what everyday system is circuit switching?**<br />*A phone call: one dedicated line reserved for the whole conversation, awkward silences and all.*
3. **The Web vs. the Internet — which came first, and which runs on which?**<br />*The Internet (TCP/IP, 1983). The Web (1989–91) is one service that runs on top of it — like email before it.*

### Key takeaways

- [ ] Networks exist to share resources, information, and communication — a connected computer is worth far more than an isolated one
- [ ] **Packet switching** (shared, chunked, reroutable) beat **circuit switching** (dedicated paths) — and created both the Internet's resilience and its security challenges
- [ ] **TCP/IP** (1983) unified separate networks into one Internet using the end-to-end principle: dumb middle, smart edges
- [ ] The **Web** (1991) is a service *on* the Internet, not the Internet itself

---

← Previous: [[00 - Fundamentals - The Reference Map]]  |  ↑ Parent: [[Networking]]  |  Next: [[02 - Bits & Signals - How Computers Physically Talk]] →
