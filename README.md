# 🖧 Networking Practice Lab

Documenting my journey learning networking, one design at a time.

Instead of just doing quizzes, I'm designing full networks from scratch: starting from a client style brief (like a real RFP), through documentation, addressing, topology, configs, and testing. Everything is built and simulated in **Cisco Packet Tracer**.

I'm not grading myself here, the goal is to build real design instincts. Feedback, issues, and PRs pointing out mistakes are welcome 🙌

This lab list is by no means exhaustive of every networking topic out there, it's just the ones that come to mind as I go. I'll keep adding labs as I think of new scenarios worth practicing.

---

## 📌 How this repo works

Each lab has:
- A **brief** (the "client requirement"), see below
- A **solution folder** with my implementation: topology file, addressing table, device configs, and a short write-up of what I tested and why

```
/
├── README.md                 <- you are here
├── lab-01-bright-leaf/
│   ├── README.md               (my notes: assumptions, design decisions, test plan)
│   ├── topology.pkt
│   └── configs/
├── lab-02-coastline-static/
├── lab-03-coastline-eigrp/
├── lab-04-coastline-ospf/
├── lab-05-summit-ridge/
├── lab-06-northline/
└── lab-07-vantage/
```

---

## ✅ Progress Tracker

| # | Lab | Focus | Status |
|---|-----|-------|--------|
| 1 | [Bright Leaf Accounting](#lab-01-bright-leaf-accounting-small-office) | VLANs, DHCP, inter-VLAN routing | ✅ Done |
| 2 | [Coastline Retail, Static](#lab-02-coastline-retail-group-2-site-wan-static-routing) | Static routing, ACLs, multi-site addressing | ✅ Done |
| 3 | [Coastline Retail, EIGRP rebuild](#lab-03-coastline-retail-group-rebuild-with-eigrp) | EIGRP basics, reflexive ACLs | ⬜ Not started |
| 4 | [Coastline Retail, OSPF rebuild](#lab-04-coastline-retail-group-rebuild-with-ospf) | OSPF basics, EIGRP vs. OSPF comparison | ⬜ Not started |
| 5 | [Summit Ridge College](#lab-05-summit-ridge-college-campus-network) | Redundant switching, STP | ⬜ Not started |
| 6 | [Northline Logistics](#lab-06-northline-logistics-nat-security-wireless) | NAT/PAT, wireless, port security | ⬜ Not started |
| 7 | [Vantage Financial Group](#lab-07-vantage-financial-group-enterprise-capstone) | OSPF at scale, FHRP, summarization | ⬜ Not started |

*(I'll flip these to ✅ as I finish each one, and link the folder once it's up.)*

---

## Lab 01: Bright Leaf Accounting (Small Office) ✅

**The brief:** Bright Leaf Accounting is a 3-partner firm moving into a new single-floor office. Three departments, Accounting (12 staff), Admin/Reception (4 staff), and Management (3 staff), each need to be logically separated for security, but everyone needs internet access and needs to share one network printer.

**Requirements:**
- Each department on its own network segment
- Devices get addresses automatically
- Departments can't sniff each other's traffic, but everyone can reach the shared printer and the internet
- One router provides internet out
- A random person plugging into a wall jack in Admin shouldn't be able to reach the Accounting VLAN

**What I practiced:** VLANs, trunking, inter-VLAN routing, DHCP, basic switchport security.

**Constraints:** Single floor, single switch stack is fine, one router.

➡️ Solution: [`lab-01-bright-leaf/`](./lab-01-bright-leaf)

---

## Lab 02: Coastline Retail Group (2-Site WAN, Static Routing) ✅

**The brief:** Coastline Retail has a Head Office and one Branch store about 500 miles apart. Right now they email spreadsheets back and forth, they want an actual network link instead. Both sites need internal LANs, and the branch needs to reach file/print resources hosted at HQ.

**Requirements:**
- A WAN link between HQ and Branch
- Routing so Branch can reach HQ subnets and vice versa
- HQ's Finance subnet should NOT be reachable from the Branch at all
- Both sites still need internet access

**What I practiced:** Static routing, standard/extended ACLs, multi-site addressing (no overlapping subnets), and along the way added a Guest wireless VLAN at the Branch with its own isolation ACL.

**Constraint:** No dynamic routing yet, management "doesn't trust that automatic stuff." Static routes only, for now.

➡️ Solution: [`lab-02-coastline-static/`](./lab-02-coastline-static)

---

## Lab 03: Coastline Retail Group, Rebuild with EIGRP

**The brief:** Same company, same topology as Lab 02, but now I've learned EIGRP, so I'm going back to replace the static routes with a dynamic routing setup.

**What I'm practicing:** EIGRP basics (neighbor relationships, the composite bandwidth/delay metric, convergence), comparing static vs. dynamic routing tradeoffs on the same topology. Also planning to add a reflexive ACL for the Finance side, so return traffic from a legitimate session gets allowed back in dynamically instead of hand-writing a rule per protocol.

**Note:** Going with EIGRP first since it's Cisco proprietary but simpler to get running than OSPF (no areas or network-type mismatches to fight with). I'll circle back with an OSPF rebuild right after, see Lab 04.

➡️ Solution: `lab-03-coastline-eigrp/` *(coming soon)*

---

## Lab 04: Coastline Retail Group, Rebuild with OSPF

**The brief:** Same company, same topology as Labs 02 and 03, one more pass, this time with OSPF, right after EIGRP while the comparison is still fresh.

**What I'm practicing:** OSPF basics, and a direct comparison against the EIGRP version from Lab 03, what's different in the configs, what's different in the metric, what's different in convergence behavior.

➡️ Solution: `lab-04-coastline-ospf/` *(coming soon)*

---

## Lab 05: Summit Ridge College (Campus Network)

**The brief:** A small college campus with 4 buildings, Admin, Library, Science, and a Dorm block, each with its own wiring closet. Science has a small server room. The campus had an outage last semester when one switch link failed and took down half the campus, they don't want a repeat.

**Requirements:**
- Each building has its own VLANs (at minimum: Staff, Student/Labs, and Servers in Science)
- Redundant links between core switches, no single point of failure
- Dorm students can reach the Science servers, but NOT the Admin staff VLAN
- Fast convergence if a link fails

**What I'm practicing:** Multiple switches with redundant links, STP/RSTP, inter-VLAN routing at scale, cross-building ACLs.

**Constraint:** At least 3 switches with more than one physical path somewhere in the topology, the redundancy is the whole point.

➡️ Solution: `lab-05-summit-ridge/` *(coming soon)*

---

## Lab 06: Northline Logistics (NAT, Security, Wireless)

**The brief:** Northline runs a warehouse and office combo. Office staff use wired PCs; warehouse floor staff use handheld scanners over Wi-Fi. They have one public IP from their ISP. A former employee's old switch port once let a guest laptop onto the internal network from the lobby, security is now a big concern.

**Requirements:**
- Warehouse devices connect wirelessly
- Only one public IP, all internal devices still need internet access
- Guest/lobby port locked down (MAC limiting or isolated guest VLAN, internet-only)
- Warehouse scanners and office PCs logically separated, even on shared switching hardware

**What I'm practicing:** NAT/PAT (overload), wireless configuration, port security, VLAN segmentation.

**Constraint:** Only one public IP to work with, forces PAT, not static NAT.

➡️ Solution: `lab-06-northline/` *(coming soon)*

---

## Lab 07: Vantage Financial Group (Enterprise, Capstone)

**The brief:** Vantage Financial is a larger enterprise with 3 branch offices plus HQ, handling sensitive financial data. They can't tolerate a single point of failure at the network edge, and want routing that scales cleanly as they open more branches instead of a flat static-route mess.

**Requirements:**
- Dynamic routing across all sites (single or multi-area OSPF, my call, with reasoning)
- Redundant default gateway at HQ (no manual reconfig if a router/switch fails)
- ACLs so branches can only reach the specific HQ resources they need
- A clean, summarized addressing plan across all 4 sites

**What I'm practicing:** OSPF at scale, FHRP (HSRP/VRRP), route summarization, ACLs at scale, professional addressing design. This is also where I'll go back and fix the single router uplink at HQ from Lab 02.

**Constraint:** At least 4 sites total (HQ + 3 branches). This is the capstone, it pulls together everything from the labs before it.

➡️ Solution: `lab-07-vantage/` *(coming soon)*

---

## 🛠️ Tools

- Cisco Packet Tracer
- Markdown for notes/documentation

## 📚 Why I'm doing this

Reading about VLANs, EIGRP, and OSPF is one thing, actually being handed a vague, real-world-sounding requirement and having to make the design calls yourself is what actually makes it stick. This repo is me holding myself accountable to that, one lab at a time.
