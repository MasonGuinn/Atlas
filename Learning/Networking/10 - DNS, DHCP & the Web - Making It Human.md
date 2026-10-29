---
title: "10 - DNS, DHCP & the Web: Making It Human"
domain: networking
type: lesson
lesson: 10
status: not-started
---


← Previous: [[09 - IPv6 - The Fix That Took 30 Years]]  |  ↑ Parent: [[Networking]]  |  Next: [[11 - Wi-Fi Deep Dive - Networking Through Air]] →

---


> **Previously:** the full transport machinery works — packets route globally ([[05 - IP & Routing - Connecting Networks Together|IP]]), conversations are reliable ([[07 - TCP & UDP - Making Delivery Reliable|TCP]]), and addressing is solved too — subnetted, NATted, and plentiful again in IPv6. But the system is still hostile to humans: you need numeric [[05 - IP & Routing - Connecting Networks Together|IP addresses]], hand-configured settings, and there's no standard way for applications to talk. This note covers the three fixes that made the Internet *usable* — and each one is also a cybersecurity battleground.

> 🌍 **The big picture:** These three protocols are the Internet's hospitality layer. **DHCP** checks you in and hands you an address. **DNS** is the phone book that turns names into numbers. **HTTP/TLS** is the visit itself — with ID checked at the door. Almost everything a human being touches on a network lives in this note.

---

## DNS: because nobody memorizes 142.250.72.14

### The problem (1980s)

Early [[01 - Why Networks Exist|ARPANET]] kept a single file, <code>HOSTS.TXT</code>, listing every computer's name and IP. One administrator at Stanford maintained it; everyone downloaded copies. With a few hundred hosts: fine. With thousands arriving: impossible — updates lagged, names collided, and the file became a bottleneck for the whole network. Sound familiar? It's the flat-name problem, back again.

### The fix (1983): a distributed, hierarchical phone book

Paul Mockapetris designed the **Domain Name System** with two ideas that mirror the lessons of this whole series:

1. **Hierarchy:** names are trees — <code>mail.google.com</code> = the <code>mail</code> host, in <code>google</code>, under <code>.com</code>. Authority is *delegated*: [the.com](http://the.com) servers don't know Google's hosts; they know who to ask (<code>google.com</code>'s own servers). No single file, no single bottleneck.
2. **Caching:** answers are remembered for a while (the record's **TTL**), so most lookups are answered by a nearby **recursive resolver** from memory. This is why DNS is fast despite being planet-sized.

The lookup chain (browser cache → resolver → root → TLD → authoritative) is in the Fundamentals note; the *why* is the point here: **DNS is the Internet's solution to scaling names** — the same hierarchical trick IP used to scale addresses.

### What an answer actually looks like

Run <code>dig example.com</code> (or <code>nslookup</code>) and the reply is rows like this:

```javascript
example.com.   300   IN   A   93.184.216.34
```

Read it left to right: **name** · **TTL** — 300 seconds, how long resolvers may cache this answer · **class** — IN, the Internet · **type** — A, an IPv4 address record · **value**. Every DNS answer on Earth is rows in this shape; the planet-sized phone book is just an ocean of these rows, each served by whoever is authoritative for that name.

### Why security people obsess over DNS

- It's the first step of almost every connection — so **whoever controls your DNS controls where you go**. Malware changes DNS settings; attackers poison caches to redirect banking sites; ISPs and governments filter by DNS.
- It's also a covert channel: data can be smuggled inside fake DNS lookups (**DNS tunneling**), and defenders monitor DNS logs precisely because everything touches it.
- **DNSSEC** adds signatures so resolvers can verify answers weren't forged; **DoH/DoT** encrypt the queries themselves.

---

## DHCP: because nobody should configure this by hand

### The problem

Every device needs, before it can do anything: an IP address, a [[06 - Subnetting - Dividing Networks on Purpose|subnet]] mask, a default gateway, and a DNS server address. In the hand-configured era, an admin typed these into every machine. One typo = a device silently off the network; two devices accidentally sharing an IP = both broken. Now imagine a college with 5,000 students arriving with laptops every September.

### The fix: ask the network

**DHCP** (evolved from BOOTP, 1985 → DHCP, 1993) lets a brand-new device with *no address at all* shout a broadcast: *"I'm new here — can anyone set me up?"* A DHCP server answers with a **lease**: an IP and all the settings, loaned for a set time. That's the **DORA** exchange (Discover, Offer, Request, Acknowledge) from the Fundamentals note.

> 🏨 **Analogy:** DHCP is the **hotel front desk**. You arrive owning nothing; the desk assigns your room (IP address), hands you the [[11 - Wi-Fi Deep Dive - Networking Through Air|Wi-Fi]] password and the directions to the lobby (DNS server + default gateway), and notes the room is yours *until checkout* (the lease).

Notice the chicken-and-egg elegance: the device broadcasts using the special source "address" <code>0.0.0.0</code> and the all-devices broadcast destination — using [[04 - LANs - Many Machines, One Room|LAN]] mechanics to *request the configuration that makes the higher layers possible*.

> ⚠️ **Security note:** DHCP has no authentication — any device can answer the shout. A **rogue DHCP server** can hand a victim a malicious gateway or DNS server and own their traffic from the first second. On a related note, when DHCP fails entirely, devices self-assign <code>169.254.x.x</code> (APIPA) — a useful forensic clue that you're looking at a machine that couldn't get configured.

---

## HTTP & the Web: the application that ate the world

### The problem (1989)

Tim Berners-Lee's issue at CERN was documents: research lived on scattered, incompatible computers. His fix combined three inventions:

- **URL** — one standard way to *name* any document anywhere (<code>https://example.com/page</code>)
- **HTTP** — one simple protocol to *fetch* it: a text request (<code>GET /page</code>), a text response with a status code (<code>200 OK</code>)
- **HTML** — one simple format to *write* it, with links to other documents

### Why HTTP won

It was deliberately simple and **stateless**: each request stands alone; the server needn't remember you between clicks. That made servers cheap and scalable — and it's why **cookies** had to be invented later to fake memory (logins, carts) on top. When you hear "the web is stateless," that design bargain is what it means.

### HTTPS: bolting on trust

Plain HTTP is postcards again — readable and modifiable by every hop (remember the dozen [[05 - IP & Routing - Connecting Networks Together|routers]]). **HTTPS** wraps HTTP in **TLS** encryption, which provides three things the original stack never had:

1. **Confidentiality** — eavesdroppers see scrambled bits
2. **Integrity** — tampering is detectable
3. **Authentication** — certificates prove the server really is who it claims to be (issued by trusted Certificate Authorities — which is why browser padlock warnings matter)

Almost all the security in everyday browsing lives *here*, in one application-layer protocol — not in IP, not in TCP. The lower layers never got fixed; we armored the top. That's the end-to-end principle one last time: the network stays dumb, the endpoints take responsibility.

---

## Where we stand

Every piece is now on the table: signals, [[03 - The First Conversation - Two Computers, One Wire|frames]], [[04 - LANs - Many Machines, One Room|switches]], IP, [[05 - IP & Routing - Connecting Networks Together|routing]], TCP, DNS, DHCP, HTTP, TLS. The final note runs one real request through *all* of them in order — because the true test of understanding networking is telling the whole story at once.

**Next →** The wired stack is complete and human-usable. Now cut the cable: [[11 - Wi-Fi Deep Dive - Networking Through Air|11 — Wi-Fi Deep Dive: Networking Through Air]].

---

## 🤔 Pause & think

*Try answering out loud before peeking at the hints — explaining a thing is how it becomes yours.*

1. **Why did the single HOSTS.TXT file have to die?**<br />*One hand-maintained file for a growing network meant update lag, name collisions, and a total bottleneck. DNS replaced it with hierarchy + caching.*
2. **A computer's address starts with 169.254. Which protocol failed, and what did the computer do?**<br />*DHCP never answered, so the machine self-assigned an APIPA address — present on the LAN, connected to nothing.*
3. **HTTP is stateless, yet your shopping cart remembers you. How?**<br />*Cookies: the server hands your browser a token to present with every request — faked memory on top of a forgetful protocol.*

### Key takeaways

- [ ] **DNS** scales names the way IP scales addresses: hierarchy + delegation + caching (1983)
- [ ] **DHCP** automates configuration via broadcast (DORA) — and its lack of authentication enables rogue-server attacks
- [ ] **HTTP** is simple and stateless by design; cookies fake the memory; **HTTPS/TLS** adds encryption, integrity, and authentication at the application layer
- [ ] Security is concentrated at the edges/top of the stack — the core protocols were never made trustworthy

---

← Previous: [[09 - IPv6 - The Fix That Took 30 Years]]  |  ↑ Parent: [[Networking]]  |  Next: [[11 - Wi-Fi Deep Dive - Networking Through Air]] →
