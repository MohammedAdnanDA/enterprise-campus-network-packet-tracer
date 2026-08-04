# Routing and Redundancy Design

This document explains the Layer 3 routing, OSPF, HSRP, default-route, EtherChannel, and Internet failover configuration used in the enterprise campus network.

The original topology and requirements were provided through the Jeremy's IT Lab CCNA Mega Lab. I implemented, verified, and troubleshot the routing and redundancy configuration described below.

## Layer 3 Architecture

The network uses a hierarchical design:

```text
Internet
   |
  R1
   |
Core switches
   |
Distribution switches
   |
Access switches and endpoints
```

Layer 3 routing is performed by:

- R1
- CSW1 and CSW2
- DSW-A1 and DSW-A2
- DSW-B1 and DSW-B2

The access switches operate primarily at Layer 2 and use management IP addresses in VLAN 99.

## IPv4 Routing Enablement

IPv4 routing was enabled on all core and distribution switches:

```cisco
ip routing
```

This allows the multilayer switches to:

- Route traffic between VLANs
- Participate in OSPF
- Forward traffic over routed interfaces
- Maintain Layer 3 routing tables

## Layer 3 EtherChannel

CSW1 and CSW2 are connected through a Layer 3 EtherChannel using PortChannel1.

| Device | PortChannel1 Address |
|---|---|
| CSW1 | 10.0.0.41/30 |
| CSW2 | 10.0.0.42/30 |

The EtherChannel uses PAgP, a Cisco-proprietary negotiation protocol.

Both switches actively attempt to form the channel using PAgP desirable mode.

The physical member interfaces were converted to routed ports before being added to the channel.

Example configuration structure:

```cisco
interface range g1/0/2-3
 no switchport
 channel-group 1 mode desirable

interface port-channel1
 no switchport
 ip address 10.0.0.41 255.255.255.252
```

The Layer 3 EtherChannel provides:

- Increased bandwidth
- Logical bundling of multiple physical links
- Redundancy if one member link fails
- A single routed interface for OSPF

## OSPF Design

OSPF process ID 1 and Area 0 were used throughout the internal routed network.

```cisco
router ospf 1
```

All routers and multilayer switches participate in the same backbone area:

```text
Area 0
```

## OSPF Router IDs

Each device uses its Loopback0 IP address as its manually configured OSPF router ID.

| Device | OSPF Router ID |
|---|---|
| R1 | 10.0.0.76 |
| CSW1 | 10.0.0.77 |
| CSW2 | 10.0.0.78 |
| DSW-A1 | 10.0.0.79 |
| DSW-A2 | 10.0.0.80 |
| DSW-B1 | 10.0.0.81 |
| DSW-B2 | 10.0.0.82 |

Example:

```cisco
router ospf 1
 router-id 10.0.0.77
```

Using loopback addresses provides a stable router ID that does not depend on the state of a physical interface.

## OSPF Interface Configuration

R1 had OSPF enabled directly under its LAN-facing interfaces.

Example:

```cisco
interface g0/0
 ip ospf 1 area 0
```

The switches used precise network statements matching each Layer 3 interface address.

Example:

```cisco
router ospf 1
 network 10.0.0.45 0.0.0.0 area 0
```

Using a wildcard mask of `0.0.0.0` matches only the exact configured interface address.

## Passive Interfaces

Loopback interfaces were configured as passive because they must be advertised into OSPF but do not form neighbor relationships.

Example:

```cisco
router ospf 1
 passive-interface loopback0
```

Distribution-switch SVIs were also configured as passive where OSPF neighbors were not expected.

Passive interfaces:

- Advertise their connected subnets
- Do not send OSPF Hello packets
- Do not attempt to form OSPF adjacencies

This reduces unnecessary OSPF traffic and prevents endpoints from participating in routing.

## OSPF Point-to-Point Network Type

Physical routed links between OSPF neighbors were configured as point-to-point.

Example:

```cisco
interface g1/1/1
 ip ospf network point-to-point
```

Point-to-point mode was appropriate because each routed link connects exactly two OSPF devices.

Benefits include:

- No DR or BDR election
- Simpler neighbor formation
- Faster and more predictable convergence

The Layer 3 PortChannel between CSW1 and CSW2 remained on its default OSPF network type because Packet Tracer did not support the required point-to-point configuration on that interface.

## OSPF Default-Route Advertisement

R1 acts as the OSPF Autonomous System Boundary Router for Internet access.

R1 advertises its default route into OSPF:

```cisco
router ospf 1
 default-information originate
```

This allows the internal routers and Layer 3 switches to learn:

```text
0.0.0.0/0 through R1
```

Internal devices therefore send traffic for unknown external destinations toward R1.

## Primary and Floating Default Routes

R1 has two Internet connections.

### Primary Default Route

```cisco
ip route 0.0.0.0 0.0.0.0 203.0.113.1
```

This route uses the default static-route administrative distance of 1.

### Floating Backup Route

```cisco
ip route 0.0.0.0 0.0.0.0 203.0.113.5 2
```

This route has an administrative distance of 2.

Because the primary route has the lower administrative distance, it is preferred during normal operation.

```text
Primary route: AD 1
Backup route:  AD 2
```

The backup route remains in the configuration but does not enter the routing table until the primary route becomes unavailable.

## Internet Failover Test

The primary Internet interface was disabled to verify failover:

```cisco
interface g0/0/0
 shutdown
```

Before the failure, R1 used:

```text
0.0.0.0/0 via 203.0.113.1
Administrative distance 1
```

After the interface was disabled, R1 installed:

```text
0.0.0.0/0 via 203.0.113.5
Administrative distance 2
```

The active route was verified using:

```cisco
show ip route 0.0.0.0
```

Internal clients successfully reached the Internet through the backup route.

The primary interface was then restored:

```cisco
interface g0/0/0
 no shutdown
```

The lower-distance primary route automatically became active again.

## Packet Tracer OSPF Limitation

During the failover test, Packet Tracer required the OSPF default-route advertisement command to be removed and re-entered:

```cisco
router ospf 1
 no default-information originate
 default-information originate
```

This forced Packet Tracer to refresh the default route advertised to internal OSPF devices.

On real Cisco equipment, the normal OSPF process should react automatically when the active default route changes.

A production design may also use IP SLA and object tracking to verify reachability beyond the directly connected ISP router.

## HSRPv2 Gateway Redundancy

HSRP version 2 was configured on the distribution switches to provide redundant default gateways.

Endpoints use a virtual IP address instead of the physical address of one distribution switch.

Example:

```text
PC default gateway: 10.1.0.1

DSW-A1: 10.1.0.2
DSW-A2: 10.1.0.3
HSRP VIP: 10.1.0.1
```

If the active distribution switch fails, the standby switch can take ownership of the virtual IP address.

## Office A HSRP Roles

| VLAN | Purpose | HSRP Group | Virtual IP | Active Router | Standby Router |
|---:|---|---:|---|---|---|
| 99 | Management | 1 | 10.0.0.1 | DSW-A1 | DSW-A2 |
| 10 | PCs | 2 | 10.1.0.1 | DSW-A1 | DSW-A2 |
| 20 | Phones | 3 | 10.2.0.1 | DSW-A2 | DSW-A1 |
| 40 | Wi-Fi | 4 | 10.6.0.1 | DSW-A2 | DSW-A1 |

## Office B HSRP Roles

| VLAN | Purpose | HSRP Group | Virtual IP | Active Router | Standby Router |
|---:|---|---:|---|---|---|
| 99 | Management | 1 | 10.0.0.17 | DSW-B1 | DSW-B2 |
| 10 | PCs | 2 | 10.3.0.1 | DSW-B1 | DSW-B2 |
| 20 | Phones | 3 | 10.4.0.1 | DSW-B2 | DSW-B1 |
| 30 | Servers | 4 | 10.5.0.1 | DSW-B2 | DSW-B1 |

The active HSRP role is divided between the two distribution switches instead of placing all VLANs on one device.

This provides basic load distribution:

```text
DSW-1 active for Management and PCs
DSW-2 active for Phones and Wi-Fi or Servers
```

## HSRP Priority and Preemption

The preferred active router was configured with a priority five higher than the default value of 100.

Example:

```cisco
standby version 2
standby 2 ip 10.1.0.1
standby 2 priority 105
standby 2 preempt
```

A higher priority causes that switch to become the active router.

Preemption allows the preferred switch to take back the active role after recovering from a failure.

Without preemption, the current active router could remain active even after the higher-priority switch returns.

## HSRP and Spanning Tree Alignment

The active HSRP router for each VLAN was also configured as the spanning-tree root bridge for that VLAN.

This prevents traffic from taking an indirect path.

Without alignment, traffic could:

```text
Access switch
   |
STP root distribution switch
   |
Cross-link to other distribution switch
   |
HSRP active gateway
```

With alignment, the preferred Layer 2 path leads directly to the active Layer 3 gateway.

## Redundancy Summary

The network includes redundancy at multiple levels:

- Two Internet paths on R1
- Primary and floating default routes
- Two core switches
- Layer 3 EtherChannel between core switches
- Dual core connections for every distribution switch
- Two distribution switches per office
- Layer 2 EtherChannel between distribution switches
- Dual uplinks from each access switch
- HSRP default-gateway redundancy
- Rapid PVST+ Layer 2 path control
- OSPF dynamic route convergence

## Verification Commands

The following commands were used to verify routing and redundancy:

```cisco
show ip interface brief
show etherchannel summary
show ip ospf neighbor
show ip ospf interface brief
show ip protocols
show ip route
show ip route ospf
show ip route 0.0.0.0
show standby brief
show standby
show spanning-tree root
ping
traceroute
```
