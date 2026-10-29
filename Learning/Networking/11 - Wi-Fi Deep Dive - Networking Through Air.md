---
title: "11 - Wi-Fi Deep Dive: Networking Through Air"
domain: networking
type: lesson
lesson: 11
status: not-started
---

← Previous: [[10 - DNS, DHCP & the Web - Making It Human]]  |  ↑ Parent: [[Networking]]  |  Next: [[12 - Routing Deep Dive - How the Internet Finds Its Way]] →

---


> **Previously:** Wi-Fi began as [[03 - The First Conversation - Two Computers, One Wire|Ethernet]]'s wireless sibling — same [[04 - LANs - Many Machines, One Room|LAN]] ideas, shared air. This note goes all the way in: how Wi-Fi actually moves data, why it's built the way it is, and how its security evolved from "embarrassingly broken" to genuinely strong. If you use one network technology daily, it's this one.

> 🌍 **The big picture:** Wi-Fi is one shared conversation in the air where nobody can talk and listen at the same time. Every behavior you love or hate about it — the range, the slowdowns, the crowded-apartment collapse — is that single physics fact, plus millions of devices politely taking turns.

---

## The core difficulty: you can't hear while you're shouting

Wired Ethernet escaped collisions by giving every device a private full-duplex link to a [[04 - LANs - Many Machines, One Room|switch]]. Radio can't do that: a Wi-Fi radio transmitting on a channel **cannot listen on that same channel at the same time** — its own signal drowns everything out. Consequences:

- Wi-Fi is inherently **half-duplex** and the air around an access point is a **shared medium** — one device talks at a time per channel
- Collisions can't be *detected* mid-transmission, so Wi-Fi uses **CSMA/CA — Collision *****Avoidance***: listen first; if busy, back off a *random* amount; and crucially, the receiver sends an **ACK for every **[[03 - The First Conversation - Two Computers, One Wire|frame]] so the sender knows it landed (no ACK = assume collision/loss → retry)

That's the hidden tax behind Wi-Fi speeds: the number on the box is a best-case raw rate. Airtime spent on ACKs, backoffs, retries, and your neighbors' traffic comes out of your share. It's also why one far-away, weak-signal device can slow a whole network — it needs more airtime per bit.

---

> 🍸 **Analogy:** a Wi-Fi channel is a **cocktail-party conversation**. One person talks at a time; if two start together, both trail off and restart at random moments. A loud room (interference) makes everyone repeat themselves — and the guest mumbling from the far corner (a weak signal) makes the whole party wait longer for every sentence.

## Frequencies: the range/speed trade-off

| Band | Range | Speed potential | Obstacles | Notes |
| --- | --- | --- | --- | --- |
| **2.4 GHz** | Long (walls OK) | Low | Poor — penetrates well, which is the problem | Only **3 non-overlapping channels (1, 6, 11)** in North America — neighbors collide constantly; also shared with Bluetooth, microwaves, baby monitors |
| **5 GHz** | Shorter | High | Walls attenuate it | Many more channels → less neighbor contention; the modern default |
| **6 GHz** | Shortest | Highest | Poor penetration | Added with Wi-Fi 6E; huge clean spectrum, few legacy devices |

**Channels** are slices of the band. Two nearby networks on the *same* channel politely share airtime (they can hear each other). Two on *partially overlapping* channels (e.g., 1 and 3) just garble each other — worse than sharing. That's why 2.4 GHz guidance is always "1, 6, or 11, nothing else."

---

## The standards parade (802.11)

<table header-row="true">
<tr>
<td>
Standard
</td>
<td>
Name
</td>
<td>
Year
</td>
<td>
Band
</td>
<td>
Max rate
</td>
<td>
The big idea
</td>
</tr>
<tr>
<td>
802.11b
</td>
<td>
—
</td>
<td>
1999
</td>
<td>
2.4
</td>
<td>
11 Mbps
</td>
<td>
The one that started it all
</td>
</tr>
<tr>
<td>
802.11g
</td>
<td>
—
</td>
<td>
2003
</td>
<td>
2.4
</td>
<td>
54 Mbps
</td>
<td>
Faster, same band
</td>
</tr>
<tr>
<td>
802.11n
</td>
<td>
Wi-Fi 4
</td>
<td>
2009
</td>
<td>
2.4 + 5
</td>
<td>
600 Mbps
</td>
<td>
**MIMO** — multiple antennas/streams at once
</td>
</tr>
<tr>
<td>
802.11ac
</td>
<td>
Wi-Fi 5
</td>
<td>
2013
</td>
<td>
5
</td>
<td>
\~3.5 Gbps
</td>
<td>
Wider channels, more streams
</td>
</tr>
<tr>
<td>
802.11ax
</td>
<td>
Wi-Fi 6
</td>
<td>
2019
</td>
<td>
2.4 + 5 (+6 as 6E)
</td>
<td>
\~9.6 Gbps
</td>
<td>
**OFDMA** — serve many devices per transmission; built for *density*, not just speed
</td>
</tr>
<tr>
<td>
802.11be
</td>
<td>
Wi-Fi 7
</td>
<td>
2024
</td>
<td>
  • 6
</td>
<td>
\~46 Gbps (theoretical)
</td>
<td>
Multi-link operation — use several bands *simultaneously*
</td>
</tr>
</table>

Notice the shift: early standards chased *peak speed for one device*; Wi-Fi 6+ chases *efficiency for many devices* — because the modern problem isn't one fast laptop, it's 60 chatty smart-home gadgets.

---

## Wi-Fi security: a 25-year repair job

| Generation | Year | What happened |
| --- | --- | --- |
| **WEP** | 1997 | "Wired Equivalent Privacy" — encryption with fatal math flaws. Crackable in *minutes* with free tools. A cautionary tale: it was weakened partly by export rules and reviewed by too few cryptographers |
| **WPA** | 2003 | Emergency patch (TKIP) to run on WEP hardware — better, still flawed |
| **WPA2** | 2004 | The real fix: **AES** encryption. Security now rests on the **4-way handshake** when a device joins — and on your passphrase. Weak passphrase = crackable handshake capture (this is the classic Wi-Fi attack: capture handshake → brute-force offline) |
| **WPA3** | 2018 | Replaces the handshake with **SAE**, which resists offline brute force and gives *forward secrecy* — capturing traffic today doesn't help decrypt it tomorrow even if the password leaks |

Other traps that persist across generations:

- **WPS** (the push-button/PIN join feature) has a design flaw making its PIN brute-forceable — disable it
- **Evil twins:** an attacker names an AP identically to a legit one; devices roam to the stronger signal. Public Wi-Fi + a fake portal is the classic coffee-shop attack
- **Open networks** have no encryption at all — your protection there is [[10 - DNS, DHCP & the Web - Making It Human|HTTPS]]/[[13 - Firewalls, VPNs & the DMZ - Defending the Network|VPN]], not the Wi-Fi

---

## Mesh & roaming

One AP rarely covers a house well (remember: higher bands hate walls). **Mesh systems** deploy multiple cooperating APs with a shared SSID; your device **roams** between them as you move. Standards (802.11k/r/v) help devices pick better APs and hand off faster — before them, phones would cling to a distant AP at crawl speed ("sticky client" problem).

---

## Connecting the dots

- Wi-Fi *is* the physical + data link layer over radio: MACs, frames, [[00 - Fundamentals - The Reference Map|ARP]], [[10 - DNS, DHCP & the Web - Making It Human|DHCP]] — all of it still applies unchanged above the air
- Its shared-air etiquette (CSMA/CA, ACKs) is Ethernet's CSMA/CD reinvented for a medium where collision *detection* is impossible
- WPA2/3 protect only the air link — the moment traffic hits the wired LAN it's ordinary networking again, which is why end-to-end encryption ([[10 - DNS, DHCP & the Web - Making It Human|TLS]]) still matters

**Next →** Back to the wired core, one level up: how routers *learn* the Internet's map. [[12 - Routing Deep Dive - How the Internet Finds Its Way|12 — Routing Deep Dive: How the Internet Finds Its Way]].

---

## 🤔 Pause & think

*Try answering out loud before peeking at the hints — explaining a thing is how it becomes yours.*

1. **Why does Wi-Fi use collision *****avoidance***** while old Ethernet used collision *****detection*****?**<br />*A radio can't hear anything while it's transmitting — its own signal drowns the channel. Collisions can't be detected mid-send, so they must be prevented, and losses inferred from missing ACKs.*
2. **Why are '1, 6, or 11' the only right channel answers on 2.4 GHz?**<br />*They're the only three channels that don't overlap. Partially-overlapping channels garble each other — worse than politely sharing one.*
3. **An attacker captures your WPA2 handshake at a café. What can they do with it — and why does WPA3 stop them?**<br />*Take it home and brute-force your passphrase offline, at full GPU speed. WPA3's SAE handshake makes offline guessing useless and adds forward secrecy.*

### Key takeaways

- [ ] Radio is half-duplex and shared → **CSMA/CA + per-frame ACKs**; real Wi-Fi throughput is always well under the box number
- [ ] 2.4 GHz = range, 3 clean channels (1/6/11); 5/6 GHz = speed and capacity, less wall penetration
- [ ] Standards evolved from peak-speed (b/g/n/ac) to density-efficiency (Wi-Fi 6/7, OFDMA, multi-link)
- [ ] Security arc: WEP (broken) → WPA2 (AES, handshake — protect your passphrase) → WPA3 (SAE, forward secrecy); disable WPS; beware evil twins

---

← Previous: [[10 - DNS, DHCP & the Web - Making It Human]]  |  ↑ Parent: [[Networking]]  |  Next: [[12 - Routing Deep Dive - How the Internet Finds Its Way]] →
