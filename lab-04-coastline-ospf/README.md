# Lab 04: Coastline Retail Group, Rebuild with OSPF

Same company and topology as Labs 02 and 03, HQ and Branch connected over a WAN link, this time with the routing swapped from EIGRP to OSPF. Switching, VLANs, and addressing are unchanged. Both routers run `router ospf 1` inside a single area (Area 0), with each router still keeping one static default route out to the internet, OSPF only handles reachability between HQ and Branch, same division of responsibility as the EIGRP version.

## Two ways to enable OSPF, used deliberately

The two routers configure OSPF differently on purpose, to show both methods work:

**HQ-RT uses the `network` statement style**, the classic approach where OSPF is enabled under the routing process itself:

```
router ospf 1
 router-id 10.10.10.10
 log-adjacency-changes
 network 172.16.0.0 0.0.3.255 area 0
 network 172.16.255.253 0.0.0.0 area 0
```

**BRANCH-RT uses the per-interface style instead**, where OSPF is enabled directly on each interface rather than through a network statement:

```
interface GigabitEthernet0/0/0.10
 ip address 172.16.4.1 255.255.255.128
 ip ospf 1 area 0
```

Both achieve the same result, the interface just needs to end up in the OSPF process and the right area, but they read very differently. The `network` statement style centralizes everything in one place, easy to see the whole picture in one block, but it's easy to get the wildcard mask wrong on a busy router with lots of subnets. The per-interface style is more explicit, each interface states its own OSPF membership right where its IP address is configured, which reads cleanly interface by interface but means the OSPF picture is scattered across the whole config instead of centralized. Both routers also set an explicit router-id rather than letting OSPF pick one automatically from the highest loopback or interface address, which keeps the ID predictable rather than depending on what interfaces happen to exist.

## The ACLs are unchanged from Lab 03

`FINANCE-ACL` and `GUEST_ACL` still carry the same `permit tcp any any established` / `permit icmp any any echo-reply` lines discussed in the [Lab 03 README](../lab-03-coastline-eigrp/README.md). Same tradeoff, same reasoning, left in deliberately to demonstrate the syntax, with the actual reflexive ACL replacement documented there since Packet Tracer doesn't support `reflect`/`evaluate`. Nothing changed on the ACL side for this lab, the focus here was purely the routing protocol swap.

## Configs

Full running-configs for every device are in [`configs/`](./configs).

## What I'd improve next

Running an actual side-by-side comparison against the EIGRP version, timing convergence after a simulated link failure, comparing `show ip route` output to see how OSPF cost and EIGRP's composite metric differ for the same path, would be a good follow-up exercise, but isn't something this lab covers.