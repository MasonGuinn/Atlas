---
title: "08 - NAT: How One Address Serves a Whole House"
domain: networking
type: lesson
lesson: 8
status: not-started
---

> [!navbar]
> 🏠 [[Dashboard]] · 📚 [[Learning]] · 🌐 [[Networking]] · 💻 [[Systems]] · 🐍 [[Programming]] · 🔒 [[Cybersecurity]] · 🎓 [[School]]


← Previous: [[07 - TCP & UDP - Making Delivery Reliable]]  |  ↑ Parent: [[Networking]]  |  Next: [[09 - IPv6 - The Fix That Took 30 Years]] →

---


> **Previously:** programs got ports, and IPv4 got scarce. Combine those two facts and you get the biggest patch in Internet history: **NAT**. Your home has exactly **one** public [[05 - IP & Routing - Connecting Networks Together|IP address]] — yet your phone, laptop, console, and TV all use the Internet *simultaneously*. This note explains the sleight of hand, why purists hate it, and why it refuses to die.

> 🌍 **The big picture:** NAT is a receptionist for an entire building: one public face, a perfect memory of who's talking to whom, and no way in for uninvited guests. A 1994 stopgap that saved IPv4, quietly reshaped the Internet into clients-plus-cloud — and has been mistaken for a security guard ever since.

---

## The problem (early 1990s)

IPv4's \~4.3 billion addresses started looking finite alarmingly early. Giving every device on Earth its own public address was impossible — but most devices mostly *consume* services rather than host them. In 1994, RFC 1631 proposed a stopgap: **let a whole network share a small pool of public addresses — or even one.**

NAT was designed as a temporary bridge to [[09 - IPv6 - The Fix That Took 30 Years|IPv6]]. Like many "temporary" technologies, it worked well enough to become permanent: it's now load-bearing for the entire consumer Internet.

---

## How NAT works (the version in your house: PAT)

Technically, home [[05 - IP & Routing - Connecting Networks Together|routers]] run **PAT** (Port Address Translation), also called NAT overload — and it works by abusing the port numbers:

1. Your laptop (<code>192.168.1.50</code>, private) opens a connection to a web server. Source port: <code>51173</code>.
2. The packet reaches your router. The router **rewrites the source **[[05 - IP & Routing - Connecting Networks Together|IP]] to the home's public address (say <code>73.140.22.9</code>) and the **source port** to a fresh, unique one it picks (say <code>62001</code>).
3. The router records this in its **NAT translation table**: <code>192.168.1.50:51173 ↔ 73.140.22.9:62001</code>
4. The server replies to <code>73.140.22.9:62001</code> — as far as the server knows, *the router* is the customer.
5. The router looks up port <code>62001</code>, rewrites the destination back to <code>192.168.1.50:51173</code>, and forwards the reply across the [[04 - LANs - Many Machines, One Room|LAN]] (machinery).

Your phone does the same thing at the same moment with a different rewritten port — that's the whole trick: **ports give one IP address 65,535 separate return addresses per protocol**, so thousands of internal devices can share it.

| NAT flavor | Mapping | Used where |
| --- | --- | --- |
| **Static NAT** | One internal IP ↔ one public IP, permanently | Servers that must be reachable |
| **Dynamic NAT** | Internal IPs draw from a pool of public IPs | Legacy corporate setups |
| **PAT (overload)** | Many internal IPs ↔ one public IP, distinguished by ports | Every home router on Earth |

---

> 📞 **Analogy:** the router is a **hotel switchboard operator**. Guests (your devices) have room numbers nobody outside knows (private IPs); callers from outside know only the hotel's one number (your public IP). The operator places outgoing calls, jots down "room 412 is talking to the pharmacy on line 62001," and routes return calls by that note. A stranger who phones the hotel demanding "room 412" gets nowhere — there's no note, and no instruction to connect them.

## Consequence 1: incoming connections break (and port forwarding)

NAT state is created by *outgoing* traffic. If a packet arrives from the Internet with no matching table entry, the router has **nowhere to send it** and drops it. Great for accidental security; terrible if you *want* to host something — a game server, a camera, a website.

**Port forwarding** is the manual fix: a permanent rule saying "anything arriving on public port 25565 → send to <code>192.168.1.50:25565</code>." It's deliberately punching a hole — which is exactly why it's a security decision, not just a convenience (each forwarded port is a door the whole Internet can knock on).

This asymmetry — clients fine, servers painful — quietly reshaped the Internet's architecture: it's a big reason everything moved to **the cloud**, where servers live on real public addresses and your devices only ever make outbound connections.

## Consequence 2: NAT is *not* a firewall (but acts like half of one)

Because unsolicited inbound packets get dropped, NAT *feels* like a [[13 - Firewalls, VPNs & the DMZ - Defending the Network|firewall]]. But:

- It does nothing about **outbound** connections — malware on your laptop phones home happily (outbound creates a table entry!)
- It inspects nothing; it filters by *state*, not by *rules* (that's a real firewall's job)
- **UPnP**, a protocol letting apps ask the router to open port forwards automatically, is convenient and a notorious security hole — malware uses it too

Treating NAT as your security boundary is a classic beginner mistake. It's a side-effect, not a design.

## Consequence 3: peer-to-peer gets hard

Games, voice calls, and file sharing often want devices to talk *directly*. Behind two NATs, neither side can open a connection to the other. An entire ecosystem of workarounds exists — **STUN** (discover your public mapping), **TURN** (relay through a public server when direct fails), **ICE** (try everything, keep what works). Every Discord call and game lobby quietly runs this machinery. This friction is the strongest technical argument for IPv6's restore-the-end-to-end vision.

---

## Connecting the dots

- NAT is why **private address ranges** matter: they're the *inside* addresses NAT translates between
- The translation table is a **state table** — the same concept stateful firewalls use
- NAT appears in the [[15 - The Full Journey - What Happens When You Load a Website|Full Journey]] — now you know exactly what the router was doing
- It was supposed to be retired by **IPv6**... which is next

**Next →** [[09 - IPv6 - The Fix That Took 30 Years|09 — IPv6: The Fix That Took 30 Years]]

---

## 🤔 Pause & think

*Try answering out loud before peeking at the hints — explaining a thing is how it becomes yours.*

1. **A stranger on the Internet tries to open a connection straight to your laptop at home. What stops them — exactly?**<br />*No NAT table entry exists for that traffic, so the router has nowhere to forward it and drops it. No rule blocked it; the state did.*
2. **If NAT drops unsolicited inbound traffic, why isn't it a firewall?**<br />*It has no rules and no inspection, and it happily passes outbound connections — including malware phoning home. Firewalls decide by policy; NAT decides by memory.*
3. **Two laptops behind one router visit the same website at the same moment. Why don't the replies get crossed?**<br />*The router rewrote each connection's source port to a unique value and remembers which laptop each value belongs to.*

### Key takeaways

- [ ] **PAT** lets a whole network share one public IP by rewriting source ports and keeping a translation table
- [ ] NAT state is created by outbound traffic → unsolicited inbound packets die → hosting needs **port forwarding**
- [ ] NAT ≠ firewall: it's a state side-effect with no rules and no outbound protection
- [ ] NAT broke direct peer-to-peer, spawning STUN/TURN/ICE — and entrenched the client/cloud model of the Internet

---

← Previous: [[07 - TCP & UDP - Making Delivery Reliable]]  |  ↑ Parent: [[Networking]]  |  Next: [[09 - IPv6 - The Fix That Took 30 Years]] →
