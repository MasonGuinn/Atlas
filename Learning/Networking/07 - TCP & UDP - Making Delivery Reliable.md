---
title: "07 - TCP & UDP: Making Delivery Reliable"
domain: networking
type: lesson
lesson: 7
status: not-started
---

> [!navbar]
> 🏠 [[Dashboard]] · 📚 [[Learning]] · 🌐 [[Networking]] · 💻 [[Systems]] · 🐍 [[Programming]] · 🔒 [[Cybersecurity]] · 🎓 [[School]]


← Previous: [[06 - Subnetting - Dividing Networks on Purpose]]  |  ↑ Parent: [[Networking]]  |  Next: [[08 - NAT - How One Address Serves a Whole House]] →

---


> **Previously:** addresses are carved into subnets, and [[05 - IP & Routing - Connecting Networks Together|IP]] delivers packets across the planet — but promises nothing: no delivery guarantee, no order, no deduplication, and no clue which *program* on your machine a packet belongs to. The **transport layer** fixes all four problems. This is where protocols stop being plumbing trivia and start shaping how every app behaves.

> 🌍 **The big picture:** IP moves packets and promises nothing. The transport layer is where an application *chooses its service*: TCP — a guaranteed, ordered conversation; UDP — fast, disposable messages. And **ports** are how thousands of these conversations share one machine without colliding.

---

## Problem 1: Which program gets this packet?

Your machine runs dozens of network programs at once — browser, game, Spotify, Discord. A packet arrives addressed to your IP. Who gets it?

**Ports** solve this. A port is a 16-bit number (0–65535) that names a program's "mailbox" on a machine:

- Packets carry a **source port** and **destination port** alongside the IPs
- Servers listen on **well-known ports**: 443 for [[10 - DNS, DHCP & the Web - Making It Human|HTTPS]], 53 for [[10 - DNS, DHCP & the Web - Making It Human|DNS]], 22 for SSH
- Your computer picks a temporary **ephemeral port** for each outgoing connection, so replies come back to the right app

> 📞 **Analogy:** [[05 - IP & Routing - Connecting Networks Together|IP address]] = a company's phone number; port = an extension. The full destination — IP + port — is called a **socket**, and a connection is uniquely identified by *two* sockets (yours and theirs). That's how one server on port 443 can hold 10,000 conversations at once without mixing them up.

With ports, the transport layer can offer two radically different services. The genius of TCP/IP is that it offers *both* and lets each application choose.

---

## TCP: the perfectionist

**Transmission Control Protocol** turns IP's unreliable postcards into a certified-mail service with a conversation. Before data flows, and all the way through, TCP does serious bookkeeping:

### The three-way handshake (setting up)

```javascript
Client ── SYN ───────► Server "I want to talk. My sequence numbers start at X."
Client ◄─ SYN-ACK ──── Server "Okay. Mine start at Y. I got your X."
Client ── ACK ───────► Server "Got your Y. We're connected."
```

Why three messages? Both sides must prove they can *send and receive*, and they exchange starting **sequence numbers** — the basis for everything below.

### During the conversation

- **Sequencing:** every byte is numbered, so the receiver can reassemble data *in order* even when packets wander different routes and arrive shuffled
- **Acknowledgments (ACKs):** the receiver constantly reports "I've received everything up to byte N"
- **Retransmission:** no ACK by the deadline? The sender assumes loss and *sends it again*. This is the resend service the [[03 - The First Conversation - Two Computers, One Wire|frame]] layer refused to provide — it lives here, at the endpoints, exactly as the end-to-end principle intended
- **Flow control:** the receiver advertises a **window** — "only send this much before waiting" — so a fast server can't drown a slow laptop
- **Congestion control:** TCP also watches for loss as a sign the *network* is overloaded and slows down voluntarily — a built-in politeness that keeps the shared Internet from collapsing under everyone's traffic (this was added in 1988 after the Internet suffered real congestion collapses, and it's a big reason the network survived its own growth)

### A worked example: 2,500 bytes, three segments

Your computer sends a 2,500-byte file on a connection starting at sequence number 1000, with a maximum segment size of 1,000 bytes:

| Segment | Seq # | Bytes carried | Receiver replies |
| --- | --- | --- | --- |
| 1 | 1000 | 1000–1999 | ACK 2000 — "send me byte 2000 next" |
| 2 | 2000 | 2000–2999 | — |
| 3 | 3000 | 3000–3499 (500 bytes) | ACK 3500 |

Now lose segment 2 on the way. Segments 1 and 3 arrive — but the receiver can only ACK *contiguous* bytes, so it repeats **ACK 2000** and holds segment 3 aside. The duplicate ACK tells the sender exactly what's missing; it resends segment 2, and the receiver's next ACK jumps to **3500** — the complete file, in order, despite the loss. Those four numbers *are* TCP: sequencing, acknowledgment, retransmission, reassembly.

### Closing the conversation

TCP also ends politely: one side sends **FIN** ("I'm done sending"), the other ACKs it, and eventually sends its own FIN, which gets ACKed in return — a four-step goodbye, often compressed to three. That's why connections linger in states like TIME\_WAIT in the <code>netstat</code> output: the endpoint keeps the socket on the books briefly, in case a stray late packet shows up needing a polite refusal instead of confusion.

### The cost

All that care isn't free: handshake round-trips before any data, ACK traffic, retransmission delays, and in-order delivery means one lost packet stalls everything behind it (**head-of-line blocking**). For a file download, perfect trade. For a live voice call... not so much.

---

## UDP: the minimalist

**User Datagram Protocol** adds *almost nothing* to IP: just ports and a checksum. No handshake, no ACKs, no retransmission, no ordering. Fire and forget.

Why would anyone choose that?

- **Speed/**[[02 - Bits & Signals - How Computers Physically Talk|latency]]**:** zero setup, zero waiting. In a live game or call, a *late* packet is worthless — you'd rather lose a frame of audio than freeze the stream waiting for a resend
- **Simplicity:** tiny overhead (8-byte header vs TCP's 20+)
- **Broadcast/multicast:** TCP is strictly one-to-one conversation; UDP can shout to many

**DNS** runs on UDP for exactly these reasons (one quick question, one quick answer — if it's lost, just ask again). And when apps *do* need reliability over UDP, they build their own flavor of it — modern **QUIC** (which powers [[10 - DNS, DHCP & the Web - Making It Human|HTTP]]/3) is reliability re-implemented over UDP to escape TCP's head-of-line blocking.

|  | TCP | UDP |
| --- | --- | --- |
| Mental model | Phone call / certified mail | Postcard / shouting across a room |
| Setup | 3-way handshake | None |
| Delivery | Guaranteed, in-order | Best-effort, any order |
| Used by | Web, email, SSH, file transfer | DNS, games, streaming, voice, QUIC |

---

## A security lens (you'll need this)

- **Port scanning** (Nmap) works by probing ports and reading TCP's reactions: SYN → SYN-ACK means *open*; RST means *closed*; silence suggests *filtered*. The handshake's politeness leaks information.
- A **SYN flood** sends SYNs and never answers, filling the server's half-open connection table — a DoS attack on the handshake itself.
- UDP's no-handshake design makes it easy to **spoof** source IPs, enabling reflection/amplification attacks (e.g., via open DNS servers).

---

## Where we stand

Computers can now have reliable, ordered, per-program conversations across a planet of unreliable packet delivery. The stack is complete enough to build applications on — but one thing is still painfully manual: in 1983, to reach a computer you had to *know its numeric IP address*, and every machine had to be *configured by hand* with its address, mask, and gateway.

**Next →** TCP/IP's design assumed every device owns a public address — then IPv4 ran out. The workaround that saved it abuses the very ports you just learned: [[08 - NAT - How One Address Serves a Whole House|08 — NAT: How One Address Serves a Whole House]].

---

## 🤔 Pause & think

*Try answering out loud before peeking at the hints — explaining a thing is how it becomes yours.*

1. **Why does the handshake need *****three***** messages instead of two?**<br />*Both sides must prove they can send and receive, and exchange starting sequence numbers. Two messages would leave one side's receive path unconfirmed.*
2. **You're on a live voice call and a packet vanishes. Should the app resend it?**<br />*No — by the time it arrived, it would be stale audio played out of context. Exactly why calls and games choose UDP.*
3. **One server on port 443 holds 10,000 conversations at once. What keeps them from getting mixed up?**<br />*Each connection is a unique socket pair: (server IP: 443) + (client IP: ephemeral port).*

### Key takeaways

- [ ] **Ports** deliver packets to the right program; IP + port = a **socket**; servers use well-known ports, clients use ephemeral ones
- [ ] **TCP** = handshake + sequence numbers + ACKs + retransmission + flow/congestion control → guaranteed, ordered delivery at the cost of latency and overhead
- [ ] **UDP** = ports + checksum and nothing else → fast and simple; apps that need reliability over it (QUIC) build their own
- [ ] Congestion control is TCP voluntarily slowing down — a key reason the shared Internet scales

---

← Previous: [[06 - Subnetting - Dividing Networks on Purpose]]  |  ↑ Parent: [[Networking]]  |  Next: [[08 - NAT - How One Address Serves a Whole House]] →
