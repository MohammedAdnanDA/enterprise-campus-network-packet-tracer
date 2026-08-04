# VLAN and Layer 2 Switching Design

This document explains the VLAN segmentation, trunking, VTP, EtherChannel, access-port, and spanning-tree configuration used in the enterprise campus network.

The original topology and requirements were provided through the Jeremy's IT Lab CCNA Mega Lab. I implemented, verified, and troubleshot the configurations described below.

## VLAN Design

### Office A

| VLAN | Name | Purpose |
|---:|---|---|
| 10 | PCs | Wired user devices |
| 20 | Phones | Voice traffic |
| 40 | Wi-Fi | Wireless clients |
| 99 | Management | Switches, WLC, and lightweight AP management |
| 1000 | Unused native VLAN | Native VLAN on standard trunk links |

### Office B

| VLAN | Name | Purpose |
|---:|---|---|
| 10 | PCs | Wired user devices |
| 20 | Phones | Voice traffic |
| 30 | Servers | Server network containing SRV1 |
| 99 | Management | Switch and lightweight AP management |
| 1000 | Unused native VLAN | Native VLAN on standard trunk links |

VLAN segmentation separates devices according to their function and limits the size of each Layer 2 broadcast domain.

## VTP Configuration

VTP version 2 was used to distribute VLAN information within each office.

| Setting | Value |
|---|---|
| VTP version | 2 |
| Domain name | JeremysITLab |
| Distribution switch role | One VTP server per office |
| Access switch role | VTP client |

VLANs were created on the selected distribution switch acting as the VTP server. The access switches learned the VLAN database through VTP.

Office A and Office B use different VLAN sets, so VLAN propagation was managed within each office’s switching domain.

## Distribution-Switch EtherChannels

### Office A

DSW-A1 and DSW-A2 use a Layer 2 EtherChannel configured with PAgP.

| Setting | Value |
|---|---|
| Port channel | PortChannel1 |
| Protocol | PAgP |
| Member mode | Desirable on both switches |
| Link type | Layer 2 trunk |

PAgP is Cisco proprietary. The `desirable` mode actively attempts to form the EtherChannel.

### Office B

DSW-B1 and DSW-B2 use a Layer 2 EtherChannel configured with LACP.

| Setting | Value |
|---|---|
| Port channel | PortChannel1 |
| Protocol | LACP |
| Member mode | Active on both switches |
| Link type | Layer 2 trunk |

LACP is an open-standard EtherChannel protocol. The `active` mode actively attempts to form the channel.

## Trunk Configuration

Links between access and distribution switches were configured manually as 802.1Q trunks.

DTP was disabled using `switchport nonegotiate` because the trunk role was already known and did not need to be dynamically negotiated.

### Office A Trunks

Office A trunks carry:

```text
VLANs 10,20,40,99
```

The unused VLAN 1000 is configured as the native VLAN.

### Office B Trunks

Office B trunks carry:

```text
VLANs 10,20,30,99
```

The unused VLAN 1000 is configured as the native VLAN.

Using an unused native VLAN reduces the risk of untagged user traffic being placed into an active production VLAN.

## WLC1 Connection

WLC1 connects to ASW-A1 through a trunk link that carries:

```text
VLAN 99 — WLC management
VLAN 40 — Wi-Fi clients
```

VLAN 99 is configured as the untagged native VLAN on this specific connection because the WLC management interface uses untagged traffic.

DTP was disabled on the WLC connection.

## Access-Port Roles

### Lightweight Access Points

The LWAP switchports use VLAN 99 for management.

The APs do not use FlexConnect. Wireless client traffic is centrally switched through the Wireless LAN Controller rather than being locally switched by the AP.

### IP Phones and PCs

Phone-facing ports support two VLANs:

```text
Access VLAN 10 — PC data traffic
Voice VLAN 20 — IP phone traffic
```

The PC connects through the IP phone’s PC port, allowing voice and data traffic to use one physical switchport while remaining logically separated.

### Server

SRV1 connects through a manually configured access port in Office B VLAN 30.

### Access-Port Security Settings

Endpoint-facing ports were manually configured as access ports and had DTP disabled.

PortFast and BPDU Guard were enabled on ports connected to endpoints.

- PortFast allows an endpoint-facing port to enter the forwarding state without waiting through normal spanning-tree transition timers.
- BPDU Guard disables the port if it receives a spanning-tree BPDU, helping prevent an unauthorized switch from being connected.

## Rapid PVST+

Rapid PVST+ was enabled on all access and distribution switches.

Rapid PVST+ maintains a separate spanning-tree instance for each VLAN and provides faster convergence than traditional 802.1D spanning tree.

The spanning-tree root bridge for each VLAN was aligned with the active HSRP gateway.

### Office A

| VLAN | HSRP Active Router | STP Root Bridge |
|---:|---|---|
| 99 | DSW-A1 | DSW-A1 |
| 10 | DSW-A1 | DSW-A1 |
| 20 | DSW-A2 | DSW-A2 |
| 40 | DSW-A2 | DSW-A2 |

### Office B

| VLAN | HSRP Active Router | STP Root Bridge |
|---:|---|---|
| 99 | DSW-B1 | DSW-B1 |
| 10 | DSW-B1 | DSW-B1 |
| 20 | DSW-B2 | DSW-B2 |
| 30 | DSW-B2 | DSW-B2 |

The HSRP standby router was configured with the next-lowest spanning-tree priority.

This alignment prevents traffic from taking an unnecessarily indirect Layer 2 path to reach the active default gateway.

## Unused Interfaces

All unused access and distribution switch interfaces were administratively disabled.

This reduces:

- Unauthorized physical access
- Accidental connections
- Unnecessary spanning-tree participation
- Exposure to Layer 2 attacks

## Verification Commands

The following commands were used to verify the Layer 2 configuration:

```cisco
show vlan brief
show interfaces trunk
show interfaces switchport
show etherchannel summary
show pagp neighbor
show lacp neighbor
show vtp status
show spanning-tree
show spanning-tree root
show interfaces status
```
