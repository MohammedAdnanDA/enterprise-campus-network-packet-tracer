# IPv4 Addressing Plan

This document records the IPv4 addressing used in the enterprise campus network lab.

The original addressing plan was provided as part of Jeremy's IT Lab CCNA Mega Lab. I used it to configure, verify, and troubleshoot the completed network.

## User and Service Networks

| Location | VLAN | Purpose | Subnet | HSRP Virtual Gateway |
|---|---:|---|---|---|
| Office A | 99 | Management | 10.0.0.0/28 | 10.0.0.1 |
| Office A | 10 | PCs | 10.1.0.0/24 | 10.1.0.1 |
| Office A | 20 | Phones | 10.2.0.0/24 | 10.2.0.1 |
| Office A | 40 | Wi-Fi | 10.6.0.0/24 | 10.6.0.1 |
| Office B | 99 | Management | 10.0.0.16/28 | 10.0.0.17 |
| Office B | 10 | PCs | 10.3.0.0/24 | 10.3.0.1 |
| Office B | 20 | Phones | 10.4.0.0/24 | 10.4.0.1 |
| Office B | 30 | Servers | 10.5.0.0/24 | 10.5.0.1 |

## Internet Connections

| Link | Subnet | Device | IPv4 Address |
|---|---|---|---|
| R1 to ISP-A | 203.0.113.0/30 | ISP-A | 203.0.113.1 |
| R1 to ISP-A | 203.0.113.0/30 | R1 G0/0/0 | 203.0.113.2 |
| R1 to ISP-B | 203.0.113.4/30 | ISP-B | 203.0.113.5 |
| R1 to ISP-B | 203.0.113.4/30 | R1 G0/1/0 | 203.0.113.6 |

R1 obtained its ISP-facing addresses through DHCP.

## R1 and Core Links

| Link | Subnet | Device and Interface | IPv4 Address |
|---|---|---|---|
| R1 to CSW1 | 10.0.0.32/30 | R1 G0/0 | 10.0.0.33 |
| R1 to CSW1 | 10.0.0.32/30 | CSW1 G1/0/1 | 10.0.0.34 |
| R1 to CSW2 | 10.0.0.36/30 | R1 G0/1 | 10.0.0.37 |
| R1 to CSW2 | 10.0.0.36/30 | CSW2 G1/0/1 | 10.0.0.38 |
| CSW1 to CSW2 | 10.0.0.40/30 | CSW1 PortChannel1 | 10.0.0.41 |
| CSW1 to CSW2 | 10.0.0.40/30 | CSW2 PortChannel1 | 10.0.0.42 |

## Core-to-Distribution Routed Links

| Link | Subnet | Device and Interface | IPv4 Address |
|---|---|---|---|
| CSW1 to DSW-A1 | 10.0.0.44/30 | CSW1 G1/1/1 | 10.0.0.45 |
| CSW1 to DSW-A1 | 10.0.0.44/30 | DSW-A1 G1/1/1 | 10.0.0.46 |
| CSW1 to DSW-A2 | 10.0.0.48/30 | CSW1 G1/1/2 | 10.0.0.49 |
| CSW1 to DSW-A2 | 10.0.0.48/30 | DSW-A2 G1/1/1 | 10.0.0.50 |
| CSW1 to DSW-B1 | 10.0.0.52/30 | CSW1 G1/1/3 | 10.0.0.53 |
| CSW1 to DSW-B1 | 10.0.0.52/30 | DSW-B1 G1/1/1 | 10.0.0.54 |
| CSW1 to DSW-B2 | 10.0.0.56/30 | CSW1 G1/1/4 | 10.0.0.57 |
| CSW1 to DSW-B2 | 10.0.0.56/30 | DSW-B2 G1/1/1 | 10.0.0.58 |
| CSW2 to DSW-A1 | 10.0.0.60/30 | CSW2 G1/1/1 | 10.0.0.61 |
| CSW2 to DSW-A1 | 10.0.0.60/30 | DSW-A1 G1/1/2 | 10.0.0.62 |
| CSW2 to DSW-A2 | 10.0.0.64/30 | CSW2 G1/1/2 | 10.0.0.65 |
| CSW2 to DSW-A2 | 10.0.0.64/30 | DSW-A2 G1/1/2 | 10.0.0.66 |
| CSW2 to DSW-B1 | 10.0.0.68/30 | CSW2 G1/1/3 | 10.0.0.69 |
| CSW2 to DSW-B1 | 10.0.0.68/30 | DSW-B1 G1/1/2 | 10.0.0.70 |
| CSW2 to DSW-B2 | 10.0.0.72/30 | CSW2 G1/1/4 | 10.0.0.73 |
| CSW2 to DSW-B2 | 10.0.0.72/30 | DSW-B2 G1/1/2 | 10.0.0.74 |

## Loopback Addresses

| Device | Interface | IPv4 Address |
|---|---|---|
| R1 | Loopback0 | 10.0.0.76/32 |
| CSW1 | Loopback0 | 10.0.0.77/32 |
| CSW2 | Loopback0 | 10.0.0.78/32 |
| DSW-A1 | Loopback0 | 10.0.0.79/32 |
| DSW-A2 | Loopback0 | 10.0.0.80/32 |
| DSW-B1 | Loopback0 | 10.0.0.81/32 |
| DSW-B2 | Loopback0 | 10.0.0.82/32 |

The loopback addresses were used as OSPF router IDs. R1's Loopback0 was also used as the DHCP relay destination, NTP server address, and internal management address.

## Office A HSRP Addresses

| VLAN | Purpose | DSW-A1 | DSW-A2 | Virtual IP | Active Router |
|---:|---|---|---|---|---|
| 99 | Management | 10.0.0.2/28 | 10.0.0.3/28 | 10.0.0.1 | DSW-A1 |
| 10 | PCs | 10.1.0.2/24 | 10.1.0.3/24 | 10.1.0.1 | DSW-A1 |
| 20 | Phones | 10.2.0.2/24 | 10.2.0.3/24 | 10.2.0.1 | DSW-A2 |
| 40 | Wi-Fi | 10.6.0.2/24 | 10.6.0.3/24 | 10.6.0.1 | DSW-A2 |

## Office B HSRP Addresses

| VLAN | Purpose | DSW-B1 | DSW-B2 | Virtual IP | Active Router |
|---:|---|---|---|---|---|
| 99 | Management | 10.0.0.18/28 | 10.0.0.19/28 | 10.0.0.17 | DSW-B1 |
| 10 | PCs | 10.3.0.2/24 | 10.3.0.3/24 | 10.3.0.1 | DSW-B1 |
| 20 | Phones | 10.4.0.2/24 | 10.4.0.3/24 | 10.4.0.1 | DSW-B2 |
| 30 | Servers | 10.5.0.2/24 | 10.5.0.3/24 | 10.5.0.1 | DSW-B2 |

## Access Switch Management Addresses

| Device | Management Interface | IPv4 Address | Default Gateway |
|---|---|---|---|
| ASW-A1 | VLAN 99 | 10.0.0.4/28 | 10.0.0.1 |
| ASW-A2 | VLAN 99 | 10.0.0.5/28 | 10.0.0.1 |
| ASW-A3 | VLAN 99 | 10.0.0.6/28 | 10.0.0.1 |
| ASW-B1 | VLAN 99 | 10.0.0.20/28 | 10.0.0.17 |
| ASW-B2 | VLAN 99 | 10.0.0.21/28 | 10.0.0.17 |
| ASW-B3 | VLAN 99 | 10.0.0.22/28 | 10.0.0.17 |

## Infrastructure and Server Addresses

| Device | IPv4 Address | Purpose |
|---|---|---|
| WLC1 | 10.0.0.7 | Wireless LAN Controller management |
| SRV1 | 10.5.0.4/24 | DNS, FTP and Syslog server |
| WLC1 Wi-Fi Dynamic Interface | 10.6.0.4/24 | Wireless client VLAN interface |

## NAT Public Addresses

| Function | Inside Address or Networks | Public Address |
|---|---|---|
| Static NAT for SRV1 | 10.5.0.4 | 203.0.113.113 |
| Dynamic PAT pool | Office A PCs and Phones, Office B PCs and Phones, and Wi-Fi | 203.0.113.200–203.0.113.207 |
