---
title: "06 - Subnetting: Dividing Networks on Purpose"
domain: networking
type: lesson
lesson: 6
status: not-started
---

> [!navbar]
> 🏠 [[Dashboard]] · 📚 [[Learning]] · 🌐 [[Networking]] · 💻 [[Systems]] · 🐍 [[Programming]] · 🔒 [[Cybersecurity]] · 🎓 [[School]]


← Previous: [[05 - IP & Routing - Connecting Networks Together]]  |  ↑ Parent: [[Networking]]  |  Next: [[07 - TCP & UDP - Making Delivery Reliable]] →

---


> **Previously:** every device has a hierarchical [[05 - IP & Routing - Connecting Networks Together|IP address]] — network portion + host portion, split by the subnet mask. But who decides where the split goes? **You do.** Subnetting is the craft of carving one network block into smaller networks, and it's the single most testable skill in networking (Network+ and CCNA love it). This note is the math, worked slowly.

> 🌍 **The big picture:** Subnetting is border-drawing inside an address block. Every border you draw is a place where broadcasts stop, where a [[05 - IP & Routing - Connecting Networks Together|router]] or [[13 - Firewalls, VPNs & the DMZ - Defending the Network|firewall]] can stand guard, and where addresses stop being wasted. It's arithmetic — but the arithmetic *is* network design.

---

## Why subnet at all?

If hierarchy is good, more hierarchy is better — for four concrete reasons:

1. **Broadcast control.** Broadcasts stop at subnet boundaries. A college with 5,000 devices in one giant subnet would drown in broadcast chatter; split into per-building subnets, each stays quiet.
2. **Organization.** One subnet per department/floor/building makes addresses *mean* something: <code>10.2.x.x</code> = dorms, <code>10.3.x.x</code> = labs.
3. **Security.** Subnet boundaries are where routers and firewalls sit. Students and the finance office *should* be in different subnets so traffic between them can be filtered.
4. **Address efficiency.** Hand a 30-person office a /16 and you've wasted 65,000 addresses. Subnetting lets you hand out right-sized blocks.

---

> 🗺️ **Analogy:** think of an address block as a **street**. The network portion is the street name; the host portion is the house number. Subnetting is rezoning one long street into several named neighborhoods — each gets its own sign (the network address), its own PA system (the broadcast address), and a border where a gate — a router — can stand.

## The core mechanic: borrowing bits

A subnet mask is a row of 1s (network bits) followed by 0s (host bits). **Subnetting = moving the boundary right**, borrowing host bits to become network bits:

- Every bit you borrow **doubles the number of subnets** (2\^borrowed)
- Every bit you borrow **halves the hosts per subnet** (hosts = 2\^remaining host bits − 2)

Why minus 2? The all-zeros host address is the **network address** (names the subnet itself) and the all-ones is the **broadcast address** — neither can belong to a device.

| CIDR | Mask | Hosts bits | Usable hosts | Typical use |
| --- | --- | --- | --- | --- |
| /24 | 255.255.255.0 | 8 | 254 | Small office / home LAN |
| /25 | 255.255.255.128 | 7 | 126 | Half of a /24 |
| /26 | 255.255.255.192 | 6 | 62 | Small department |
| /27 | 255.255.255.224 | 5 | 30 | Tiny segment |
| /28 | 255.255.255.240 | 4 | 14 | [[13 - Firewalls, VPNs & the DMZ - Defending the Network|Small lab]] |
| /30 | 255.255.255.252 | 2 | 2 | Point-to-point router links |
| /16 | 255.255.0.0 | 16 | [[14 - The Modern Network - Old Ideas at Planetary Scale|65,534]] | A whole campus |

---

## Worked example 1: split a network

**Task:** You have <code>192.168.1.0/24</code>. You need **4 subnets**.

- 4 subnets → need 2 borrowed bits (2² = 4)
- New mask: /24 + 2 = **/26** (255.255.255.192)
- Hosts per subnet: 2⁶ − 2 = **62 usable**
- Block size: 256 − 192 = **64** addresses each, so subnets start at multiples of 64:

| Subnet | Network addr | Usable range | Broadcast |
| --- | --- | --- | --- |
| 1 | 192.168.1.0 | .1 –.62 | .63 |
| 2 | 192.168.1.64 | .65 –.126 | .127 |
| 3 | 192.168.1.128 | .129 –.190 | .191 |
| 4 | 192.168.1.192 | .193 –.254 | .255 |

> ⚡ **The magic-number shortcut:** block size = 256 − (the interesting mask octet). For /26 the last octet is 192 → block size 64 → subnets at 0, 64, 128, 192. This one trick does 90% of real-world subnetting in your head.

---

## Worked example 2: size from requirements (backwards)

**Task:** A department needs **50 hosts**. What mask?

- Need 2\^h − 2 ≥ 50 → h = 6 host bits (2⁶−2 = 62 ✓; h=5 gives only 30 ✗)
- Mask = 32 − 6 = **/26**. Done — always solve for host bits *first*, then convert.

---

## VLSM: different needs, different sizes

Real networks aren't uniform. **Variable Length Subnet Masking** means carving one block into *unequal* subnets. **Golden rule: allocate biggest first**, or you'll fragment the space and strand unusable gaps.

**Task:** Carve <code>192.168.10.0/24</code> for: a lab needing 100 hosts, an office needing 50, a classroom needing 25, and a router-to-router link needing 2.

| Need | Host bits | Mask | Block | Assignment | Usable |
| --- | --- | --- | --- | --- | --- |
| 100 | 7 (126) | /25 | 128 | 192.168.10.0/25 | .1–.126 |
| 50 | 6 (62) | /26 | 64 | 192.168.10.128/26 | .129–.190 |
| 25 | 5 (30) | /27 | 32 | 192.168.10.192/27 | .193–.222 |
| 2 | 2 (2) | /30 | 4 | 192.168.10.224/30 | .225–.226 |

Total used: 128 + 64 + 32 + 4 = 228 of 256 — and <code>.228–.255</code> remains free for growth. Every requirement met, nothing wasted on a giant flat network.

---

## Connecting the dots

- [[04 - LANs - Many Machines, One Room|VLANs]] are how you *implement* subnets on [[04 - LANs - Many Machines, One Room|switches]] — normally one VLAN = one subnet
- Devices in different subnets **must** talk through a router — that's the point; it's also where filtering happens
- [[10 - DNS, DHCP & the Web - Making It Human|DHCP]] runs one *scope* (address pool) per subnet — a relay agent forwards requests across subnets
- Going the *other* direction — merging small blocks into one summary route — is **supernetting/route aggregation**, how ISPs keep [[05 - IP & Routing - Connecting Networks Together|routing]] tables small

**Next →** Addresses are carved, assigned, and routed. Now the question changes: how do two *programs* hold a reliable, ordered conversation across this network? [[07 - TCP & UDP - Making Delivery Reliable|07 — TCP & UDP: Making Delivery Reliable]].

---

## ✏️ Practice — subnetting drills

*Answer each out loud before reading the italic solution. Speed comes from reps; accuracy comes first.*

1. **How many usable hosts in a /27?**<br />*5 host bits → 2⁵ − 2 = ****30****.*
2. **Split ****<code>192.168.5.0/24</code>**** into subnets holding at least 25 hosts each. How many do you get, and what's the mask?**<br />*25 hosts needs 5 host bits (30 usable) → ****/27****. Borrowing 3 bits gives 2³ = ****8 subnets****.*
3. **A device is ****<code>172.16.4.37/20</code>****. What's its network address and broadcast address?**<br />*/20 → the interesting octet is the third: block size 256 − 240 = 16, so networks start at 0, 16, 32… 4 falls in the first block → network ****172.16.0.0****, broadcast ****172.16.15.255****.*
4. **VLSM: carve ****<code>192.168.20.0/24</code>**** for 60 hosts, 25 hosts, and a 2-host router link.**<br />*Biggest first: 60 → /26 at .0 (usable .1–.62); 25 → /27 at .64 (usable .65–.94); 2 → /30 at .96 (usable .97–.98). Everything from .100 up is still free.*

## 🤔 Pause & think

*Try answering out loud before peeking at the hints — explaining a thing is how it becomes yours.*

1. **Why is the hosts formula 2\^h − 2 instead of just 2\^h?**<br />*The all-zeros host address names the network itself, and all-ones is the broadcast address — neither can belong to a device.*
2. **You need at least 6 subnets out of a /24. How many bits do you borrow, and what's the new mask?**<br />*3 bits (2³ = 8 ≥ 6) → /27 — leaving 30 usable hosts per subnet.*
3. **In VLSM, why must the biggest block be allocated first?**<br />*Big blocks need large, aligned, contiguous space. Carving small ones first fragments the range and strands unusable gaps.*

### Key takeaways

- [ ] Subnetting = borrowing host bits to create more, smaller networks — tradeoff: subnets double per bit, hosts halve per bit
- [ ] Usable hosts = 2\^host bits − 2 (network + broadcast are reserved)
- [ ] Block size = 256 − mask octet → subnets start at multiples of the block size
- [ ] **VLSM**: allocate biggest subnet first to avoid fragmentation

---

← Previous: [[05 - IP & Routing - Connecting Networks Together]]  |  ↑ Parent: [[Networking]]  |  Next: [[07 - TCP & UDP - Making Delivery Reliable]] →
