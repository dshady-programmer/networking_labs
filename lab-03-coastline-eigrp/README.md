# Lab 03: Coastline Retail Group, Rebuild with EIGRP

Same company and topology as Lab 02, HQ and Branch connected over a WAN link, but the static routes are replaced with EIGRP. The switching layer, VLANs, and addressing are unchanged from Lab 02, this lab is focused on the routing change and a closer look at the ACLs guarding Finance and Guest.

## What changed from Lab 02

**Static routes replaced with EIGRP.** Both `HQ-RT` and `BRANCH-RT` now run `router eigrp 1`, advertising their local subnets and the shared WAN link. Each router still keeps one static default route out to the internet, EIGRP only handles reachability between HQ and Branch, it doesn't know about the internet beyond HQ's ISP link.

**EIGRP network statements use exact wildcard masks.** On `HQ-RT`, the WAN interface is advertised with `network 172.16.255.253 0.0.0.0`, an exact host match, rather than a loose classful statement. This matches the precision Branch's config already used from the start. A looser statement (no wildcard, defaulting to the classful 172.16.0.0/16 range) would technically still work here since nothing else exists in that range yet, but being exact avoids accidentally pulling in a future subnet that shouldn't be part of EIGRP.

**Duplicate default route on HQ-RT cleaned up.** Lab 02 had two static default routes pointing to the internet, one by exit interface, one by next-hop IP, left over from earlier testing. Only one remains now (`ip route 0.0.0.0 0.0.0.0 216.0.5.49`).

## A deliberate tradeoff in the ACLs

Both `FINANCE-ACL` (on HQ-RT) and `GUEST_ACL` (on Branch-RT) contain these two lines ahead of their deny rules:

```
permit tcp any any established
permit icmp any any echo-reply
```

This is a real gap. The `established` keyword only checks whether the ACK or RST bit is set in the TCP header, it doesn't confirm a session actually started from the trusted side. A packet crafted with that bit set would get through before ever reaching the deny rule underneath. The correct fix for this is a reflexive ACL, which tracks real sessions the router itself saw leave, rather than trusting a flag in the packet.

These lines are being left in place on purpose, specifically to demonstrate the syntax, since Lab 02's static version didn't include any attempt at session-aware filtering at all. It's a known, acknowledged tradeoff rather than an oversight.

**What the actual fix would look like, if the environment supported it:**

Packet Tracer's simulated IOS doesn't implement the `reflect` or `evaluate` keywords, reflexive ACLs aren't available in this environment at all. Here's what would replace the vulnerable lines on real hardware:

For Finance (the ACL sits on HQ's WAN interface, downstream from where Finance traffic actually originates, so the reflect rule has to be scoped to Finance's subnet specifically, otherwise it would start tracking every department's sessions crossing that same interface):

```
ip access-list extended FINANCE-OUT
 permit tcp 172.16.3.0 0.0.0.255 any reflect FINANCE-REFLECT
 permit udp 172.16.3.0 0.0.0.255 any reflect FINANCE-REFLECT
 permit icmp 172.16.3.0 0.0.0.255 any reflect FINANCE-REFLECT
 permit ip any any
!  (applied outbound on Gi0/0/1)

ip access-list extended FINANCE-IN
 evaluate FINANCE-REFLECT
 deny ip any 172.16.3.0 0.0.0.255
 permit ip any any
!  (applied inbound on Gi0/0/1, replaces the current FINANCE-ACL)
```

For Guest (this ACL already sits directly on the Guest subinterface, right where guest traffic originates, so no scoping is needed, `any any` is already effectively "guest only" by virtue of where it lives):

```
ip access-list extended GUEST_ACL
 permit tcp any any reflect GUEST-REFLECT
 permit udp any any reflect GUEST-REFLECT
 permit icmp any any reflect GUEST-REFLECT
 deny ip any 172.16.4.0 0.0.1.255
 deny ip any 172.16.0.0 0.0.3.255
 permit ip any any
!  (applied inbound on Gi0/0/0.50, same direction as today)

ip access-list extended GUEST_IN-ACL
 evaluate GUEST-REFLECT
 deny ip any any
!  (applied outbound on Gi0/0/0.50, new)
```

In both cases, the `reflect` lines go in the ACL applied where trusted traffic actually enters the router, and `evaluate` goes in the ACL applied where the reply traffic actually leaves it. For Finance that's outbound-then-inbound across the WAN interface, for Guest it's inbound-then-outbound at the subinterface itself, since Guest sits right at the source and Finance doesn't.

## Configs

Full running-configs for every device are in [`configs/`](./configs).

## What I'd improve next

If this environment supported reflexive ACLs (or CBAC, the older Cisco feature that does the same job through `ip inspect` commands), swapping the `established`/`echo-reply` lines for the versions above would close the gap without losing the intended behavior, Finance and Guest getting real replies to sessions they actually started, while still blocking anything that wasn't a genuine reply.