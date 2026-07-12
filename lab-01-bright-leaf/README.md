# Lab 01: Bright Leaf Accounting (Small Office)

A small office network for a 3-partner accounting firm with three departments: Accounting, Admin/Reception, and Management. Each department needed its own segment, DHCP, shared access to one printer, and internet access, while unauthorized devices shouldn't be able to reach anything sensitive.

## Topology

![Topology](topology.PNG)

- **1 router** (`BrightLeaf-RT`), handles inter-VLAN routing, DHCP, and NAT out to the internet
- **1 router** (`ISP`), simulates the internet, with loopback addresses used to test reachability
- **3 access switches** (`SW01`, `SW02`, `SW03`), connected in a triangle for redundancy, with EtherChannel bundling the links between them
- **1 wireless access point** and **1 shared printer**, both connected to SW01

## Addressing

| VLAN | Department | Subnet | Gateway |
|---|---|---|---|
| 10 | Accounting | 192.168.0.0/27 | 192.168.0.1 |
| 20 | Admin/Reception | 192.168.0.32/28 | 192.168.0.33 |
| 30 | Management (includes switch management SVIs) | 192.168.0.48/28 | 192.168.0.49 |
| 99 | Unused ports (dead VLAN) | n/a | n/a |
| 999 | Native VLAN for trunk links | n/a | n/a |

Subnet sizes were picked based on expected staff count per department, with room to grow. DHCP hands out addresses in each VLAN, with the router, gateway, and printer addresses excluded from the DHCP pool so they can't be handed out to a random device.

## Design decisions

**Inter-VLAN routing is allowed between all departments.** The requirement was that departments shouldn't be able to sniff each other's traffic, which VLAN segmentation already takes care of, since each department is its own broadcast domain. On top of that, routing between VLANs is left open on purpose, so departments can send documents to each other and everyone can reach the shared printer, which lives in the Management VLAN. This was a deliberate choice to support normal day to day office communication, not an oversight.

**Switch management lives directly on VLAN 30, not a separate VLAN.** An earlier version of this build put the switch management SVIs on a separate VLAN 50, addressed inside the same subnet as VLAN 30. That didn't work: VLANs are separate broadcast domains no matter what IP subnet is assigned to them, so a Management PC's ARP request for the SVI address never left VLAN 30 to reach VLAN 50, and neither the router (no VLAN 50 subinterface) nor the switches (Layer 2 only) could route it either. The fix was to drop VLAN 50 and configure the switch SVIs directly on VLAN 30 instead, so the Management PCs and the switch management interfaces sit in the same broadcast domain and can reach each other with no routing hop required.

**Only Management can SSH or Telnet into the switches.** A standard ACL (`MGMT_RESTRICT`) is applied to the VTY lines on every switch, permitting only addresses from the Management subnet and denying everyone else. Since the switch management SVI now lives directly on VLAN 30, VLAN membership is already the main thing keeping other departments out, a device has to be on VLAN 30 to even reach the SVI at all. The ACL is kept anyway as a second layer: it still blocks a VLAN 30 device that's been given a static IP outside the expected subnet, and it keeps the boundary explicit in the config in case the network changes later (for example, if VLAN 30 ever spans more than one subnet).

**Unused switch ports are isolated in a dead VLAN (99) and administratively shut down.** VLAN 99 is deliberately left out of every trunk's allowed VLAN list, so even without the shutdown, a device plugged into one of these ports has no path off its local switch. The `shutdown` command adds a second layer on top of that.

**Trunk links use a dedicated native VLAN (999), not VLAN 1.** Using the default VLAN as a native VLAN is a common attack surface for VLAN hopping. Moving the native VLAN to an unused, deliberately empty VLAN closes that off.

**SW01 is the STP root, SW02 is the secondary root.** Spanning-tree priorities are set explicitly (SW01 lowest, SW02 next) rather than left to a default election, so the root position is predictable and won't shift unexpectedly if a new switch is added later. Rapid PVST+ is used for faster convergence than legacy STP.

**Redundant links between switches use EtherChannel (LACP).** Each pair of switches has two physical links bundled into a single logical link, so losing one cable doesn't mean losing the connection, traffic just shifts to the remaining link in the bundle.

**NAT overload (PAT) gets the whole office online through one public IP.** All three department subnets are translated behind the router's single public facing address on the link to the ISP router.

## Configs

Full running-configs for every device are in [`configs/`](./configs).

## What I'd improve next

If departments ever needed to be walled off from each other (not just separated from sniffing each other), the next step would be adding ACLs on the router's subinterfaces to control specifically what can pass between VLANs, rather than leaving routing fully open.
