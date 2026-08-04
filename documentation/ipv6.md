# IPv6 Implementation

This document explains the IPv6 addressing, EUI-64 interface configuration, link-local connectivity, static routing, default routes, and failover configuration implemented in the enterprise campus network.

The original topology and requirements were provided through the Jeremy's IT Lab CCNA Mega Lab. I implemented, verified, and troubleshot the IPv6 configuration described below.

## IPv6 Overview

IPv6 was implemented on the edge and core portions of the network.

The implementation included:

- IPv6 unicast routing
- Global unicast addresses
- EUI-64 interface identifiers
- Automatically generated link-local addresses
- Recursive static routes
- Fully specified static routes
- A primary IPv6 default route
- A floating IPv6 default route
- IPv6 Internet-path failover

The documentation prefix `2001:db8::/32` was used throughout the lab.

## IPv6 Routing Enablement

IPv6 packet forwarding was enabled on the Layer 3 devices participating in IPv6 routing:

```cisco
ipv6 unicast-routing
```

This command allows the device to forward IPv6 packets between interfaces.

Without it, the device can use IPv6 for its own communication but does not operate as an IPv6 router.

## IPv6 Addressing Plan

| Link | Device and Interface | IPv6 Addressing |
|---|---|---|
| R1 to ISP-A | R1 G0/0/0 | 2001:db8:a::2/64 |
| R1 to ISP-B | R1 G0/1/0 | 2001:db8:b::2/64 |
| R1 to CSW1 | R1 G0/0 | 2001:db8:a1::/64 using EUI-64 |
| R1 to CSW1 | CSW1 G1/0/1 | 2001:db8:a1::/64 using EUI-64 |
| R1 to CSW2 | R1 G0/1 | 2001:db8:a2::/64 using EUI-64 |
| R1 to CSW2 | CSW2 G1/0/1 | 2001:db8:a2::/64 using EUI-64 |
| CSW1 to CSW2 | CSW1 PortChannel1 | Automatically generated link-local address |
| CSW1 to CSW2 | CSW2 PortChannel1 | Automatically generated link-local address |

The exact EUI-64 interface identifiers are not listed because they are generated from each interface’s MAC address.

## ISP-Facing IPv6 Addresses

R1 connects to two ISP paths.

### Primary ISP Connection

```cisco
interface g0/0/0
 ipv6 address 2001:db8:a::2/64
```

The next-hop router on this network is:

```text
2001:db8:a::1
```

### Backup ISP Connection

```cisco
interface g0/1/0
 ipv6 address 2001:db8:b::2/64
```

The next-hop router on this network is:

```text
2001:db8:b::1
```

The first ISP path is preferred during normal operation. The second ISP path provides backup connectivity.

## EUI-64 Addressing

EUI-64 was used on the routed links between R1 and the core switches.

Example configuration:

```cisco
interface g0/0
 ipv6 address 2001:db8:a1::/64 eui-64
```

With EUI-64, the device automatically generates the 64-bit interface identifier using information derived from the interface MAC address.

An IPv6 global unicast address consists of:

```text
64-bit network prefix + 64-bit interface identifier
```

For example:

```text
2001:db8:a1::/64 + generated EUI-64 identifier
```

The resulting interface address can be viewed with:

```cisco
show ipv6 interface brief
```

## Link-Local Addressing

Every IPv6-enabled interface automatically receives a link-local address from:

```text
fe80::/10
```

Link-local addresses are used only on the local Layer 2 link and are not routed across the network.

They are commonly used for:

- Neighbor Discovery
- Router communication
- Next-hop identification
- IPv6 routing protocols
- Static routes using link-local next hops

## Core EtherChannel IPv6 Configuration

The Layer 3 EtherChannel between CSW1 and CSW2 was configured for IPv6 using:

```cisco
interface port-channel1
 ipv6 enable
```

The `ipv6 enable` command creates an IPv6 link-local address without assigning a global unicast prefix.

This allowed the two core switches to communicate over the EtherChannel using their link-local addresses.

The generated addresses were verified with:

```cisco
show ipv6 interface brief
```

## IPv6 Static Routing

Static IPv6 routes were used to control forwarding between the edge router, core switches, and ISP connections.

The general syntax is:

```cisco
ipv6 route <destination-prefix> <next-hop>
```

An IPv6 default route uses:

```text
::/0
```

This represents all IPv6 destinations not already matched by a more specific route.

## Recursive Static Routes

A recursive static route specifies only the next-hop IPv6 address.

General structure:

```cisco
ipv6 route <destination-prefix> <next-hop-address>
```

The router must perform a second routing-table lookup to determine which exit interface reaches the specified next hop.

This second lookup is why the route is called recursive.

## Fully Specified Static Routes

A fully specified IPv6 static route includes both:

- The exit interface
- The next-hop IPv6 address

General structure:

```cisco
ipv6 route <destination-prefix> <exit-interface> <next-hop-address>
```

This format is particularly important when a link-local address is used as the next hop because a link-local address is valid only on its local link.

The exit interface identifies which link contains that next-hop address.

## Primary IPv6 Default Route

R1 uses ISP-A as the preferred IPv6 Internet path.

The primary next hop is:

```text
2001:db8:a::1
```

The route follows this structure:

```cisco
ipv6 route ::/0 2001:db8:a::1
```

Because it uses the normal static-route administrative distance, this route is preferred while the primary path remains available.

## Floating IPv6 Default Route

A second IPv6 default route uses ISP-B as the backup path.

The backup next hop is:

```text
2001:db8:b::1
```

This route was configured with an administrative distance higher than the primary route.

A floating static route remains inactive while a route with a lower administrative distance is available.

The forwarding behavior is:

```text
Normal operation:
::/0 through ISP-A

Primary path unavailable:
::/0 through ISP-B
```

## IPv6 Failover

Failover was tested by disabling the primary ISP-facing interface on R1.

```cisco
interface g0/0/0
 shutdown
```

The IPv6 routing table was then checked to confirm that the backup default route became active.

```cisco
show ipv6 route
show ipv6 route ::/0
```

Connectivity was tested through the backup ISP path.

The primary interface was restored using:

```cisco
interface g0/0/0
 no shutdown
```

After the primary path recovered, the lower-administrative-distance default route became preferred again.

## IPv6 Neighbor Discovery

IPv6 does not use ARP.

Instead, IPv6 Neighbor Discovery uses ICMPv6 to perform functions such as:

- Discovering neighboring devices
- Resolving IPv6 addresses to Layer 2 addresses
- Detecting duplicate addresses
- Discovering routers
- Maintaining neighbor reachability information

The IPv6 neighbor table was viewed using:

```cisco
show ipv6 neighbors
```

The output identifies:

- Neighbor IPv6 addresses
- Link-layer addresses
- Neighbor states
- Local interfaces

## IPv6 Connected Routes

When an IPv6 address is configured on an active interface, the device installs:

- A connected route for the interface prefix
- A local `/128` route for its own interface address

Example routing-table entries:

```text
C   2001:db8:a1::/64
L   2001:db8:a1::<interface-id>/128
```

The `C` entry represents the directly connected network.

The `L` entry represents the device’s own interface address.

## Verification Process

IPv6 verification included:

1. Confirming that IPv6 unicast routing was enabled.
2. Checking IPv6 interface addresses.
3. Confirming that interfaces were operational.
4. Checking automatically generated link-local addresses.
5. Reviewing connected and static IPv6 routes.
6. Checking the active IPv6 default route.
7. Reviewing the IPv6 neighbor table.
8. Pinging directly connected IPv6 neighbors.
9. Testing the primary Internet path.
10. Disabling the primary path and testing backup connectivity.
11. Restoring the primary path and confirming route recovery.

## Verification Commands

The following commands were used during IPv6 implementation and testing:

```cisco
show ipv6 interface brief
show ipv6 interface
show ipv6 route
show ipv6 route static
show ipv6 route ::/0
show ipv6 neighbors
show running-config | include ipv6
show running-config interface <interface>
ping <ipv6-address>
traceroute <ipv6-address>
```

## Implementation Summary

The IPv6 implementation demonstrated:

- Enabling IPv6 packet forwarding
- Assigning global unicast addresses
- Generating interface identifiers with EUI-64
- Using link-local addressing
- Configuring recursive and fully specified static routes
- Configuring primary and floating IPv6 default routes
- Verifying IPv6 neighbors and routes
- Testing IPv6 Internet-path failover
