# Network Connection Map

This document records the physical and logical interface connections used in the enterprise campus network implementation.

The original topology and connection plan were provided as part of the Jeremy's IT Lab CCNA Mega Lab. This document records the links used during my configuration, verification, and troubleshooting work.

## Internet and Edge Connections

| Device | Interface | Connected Device | Connected Interface |
|---|---|---|---|
| R1 | G0/0/0 | ISP-A | ISP-facing interface |
| R1 | G0/1/0 | ISP-B | ISP-facing interface |
| R1 | G0/0 | CSW1 | G1/0/1 |
| R1 | G0/1 | CSW2 | G1/0/1 |

## Core Layer EtherChannel

The connection between CSW1 and CSW2 is a Layer 3 EtherChannel using PortChannel1.

| CSW1 Interface | CSW2 Interface | Port Channel |
|---|---|---|
| G1/0/2 | G1/0/2 | PortChannel1 |
| G1/0/3 | G1/0/3 | PortChannel1 |

## Core-to-Distribution Routed Links

| Core Device | Core Interface | Distribution Device | Distribution Interface |
|---|---|---|---|
| CSW1 | G1/1/1 | DSW-A1 | G1/1/1 |
| CSW2 | G1/1/1 | DSW-A1 | G1/1/2 |
| CSW1 | G1/1/2 | DSW-A2 | G1/1/1 |
| CSW2 | G1/1/2 | DSW-A2 | G1/1/2 |
| CSW1 | G1/1/3 | DSW-B1 | G1/1/1 |
| CSW2 | G1/1/3 | DSW-B1 | G1/1/2 |
| CSW1 | G1/1/4 | DSW-B2 | G1/1/1 |
| CSW2 | G1/1/4 | DSW-B2 | G1/1/2 |

## Office A Distribution EtherChannel

DSW-A1 and DSW-A2 use a Layer 2 EtherChannel named PortChannel1.

| DSW-A1 Interface | DSW-A2 Interface | Port Channel |
|---|---|---|
| G1/0/4 | G1/0/4 | PortChannel1 |
| G1/0/5 | G1/0/5 | PortChannel1 |

## Office B Distribution EtherChannel

DSW-B1 and DSW-B2 use a Layer 2 EtherChannel named PortChannel1.

| DSW-B1 Interface | DSW-B2 Interface | Port Channel |
|---|---|---|
| G1/0/4 | G1/0/4 | PortChannel1 |
| G1/0/5 | G1/0/5 | PortChannel1 |

## Office A Access-Switch Uplinks

Each access switch has one trunk connection to each Office A distribution switch.

| Access Switch | Access Interface | Distribution Switch | Distribution Interface |
|---|---|---|---|
| ASW-A1 | G0/1 | DSW-A1 | G1/0/1 |
| ASW-A1 | G0/2 | DSW-A2 | G1/0/1 |
| ASW-A2 | G0/1 | DSW-A1 | G1/0/2 |
| ASW-A2 | G0/2 | DSW-A2 | G1/0/2 |
| ASW-A3 | G0/1 | DSW-A1 | G1/0/3 |
| ASW-A3 | G0/2 | DSW-A2 | G1/0/3 |

## Office B Access-Switch Uplinks

Each access switch has one trunk connection to each Office B distribution switch.

| Access Switch | Access Interface | Distribution Switch | Distribution Interface |
|---|---|---|---|
| ASW-B1 | G0/1 | DSW-B1 | G1/0/1 |
| ASW-B1 | G0/2 | DSW-B2 | G1/0/1 |
| ASW-B2 | G0/1 | DSW-B1 | G1/0/2 |
| ASW-B2 | G0/2 | DSW-B2 | G1/0/2 |
| ASW-B3 | G0/1 | DSW-B1 | G1/0/3 |
| ASW-B3 | G0/2 | DSW-B2 | G1/0/3 |

## Office A Endpoint Connections

| Access Switch | Interface | Connected Device |
|---|---|---|
| ASW-A1 | F0/1 | LWAP1 |
| ASW-A1 | F0/2 | WLC1 |
| ASW-A2 | F0/1 | Phone1 |
| ASW-A3 | F0/1 | Phone2 |

PC1 connects through the PC port on Phone1.  
PC2 connects through the PC port on Phone2.  
Laptop1 connects wirelessly through the configured WLAN.

## Office B Endpoint Connections

| Access Switch | Interface | Connected Device |
|---|---|---|
| ASW-B1 | F0/1 | LWAP2 |
| ASW-B2 | F0/1 | Phone3 |
| ASW-B3 | F0/1 | SRV1 |

PC3 connects through the PC port on Phone3.  
Laptop2 connects wirelessly through the configured WLAN.

## Link Types

| Connection | Link Type |
|---|---|
| R1 to core switches | Layer 3 routed links |
| CSW1 to CSW2 | Layer 3 EtherChannel |
| Core to distribution switches | Layer 3 routed links |
| Distribution-switch pairs | Layer 2 EtherChannel |
| Distribution to access switches | 802.1Q trunk links |
| Access switches to endpoints | Access links |
| ASW-A1 to WLC1 | Trunk carrying Management and Wi-Fi VLANs |

## Redundancy Design

The topology provides multiple redundant paths:

- R1 connects independently to both core switches.
- The two core switches are connected using a Layer 3 EtherChannel.
- Every distribution switch connects to both core switches.
- Distribution-switch pairs are connected through Layer 2 EtherChannels.
- Every access switch connects to both distribution switches in its office.
- HSRP and Rapid PVST+ determine the active Layer 3 gateway and preferred Layer 2 forwarding path.
