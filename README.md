# Enterprise Campus Network – Cisco Packet Tracer

This project demonstrates the implementation, configuration, verification, security, redundancy, and troubleshooting of a multi-site enterprise campus network using Cisco Packet Tracer.

The network includes core, distribution, and access layers across two office locations, along with routing, switching, network services, security controls, IPv6, Internet connectivity, and centralized wireless management.

## Network Topology

![Enterprise Campus Network Topology](topology/enterprise-campus-topology.png)

## Project Origin and Scope

This project is based on the CCNA Mega Lab created by Jeremy's IT Lab.

The original topology and implementation requirements were provided as part of the lab. My work included:

- Configuring routers, Layer 3 switches, access switches, wireless devices, and network services
- Implementing routing, switching, redundancy, security, IPv6, and wireless connectivity
- Verifying network operation using Cisco show commands and connectivity tests
- Troubleshooting configuration, physical connectivity, DHCP, routing, NAT, failover, and wireless issues
- Documenting the completed implementation and verification process

I do not claim ownership of the original lab topology or requirements.

## Technologies Implemented

### Switching

- VLANs and access ports
- 802.1Q trunking
- Native VLAN configuration
- DTP disabled on manually configured links
- VTP version 2
- Layer 2 EtherChannel
- PAgP and LACP
- Rapid PVST+
- PortFast
- BPDU Guard

### Routing and Redundancy

- Inter-VLAN routing
- Layer 3 EtherChannel
- OSPF Area 0
- Passive OSPF interfaces
- Point-to-point OSPF network types
- HSRPv2 default-gateway redundancy
- Static default routes
- Floating static routes
- Internet link failover

### Network Services

- DHCP server configuration
- DHCP relay
- DNS
- NTP
- SNMP
- Syslog
- FTP
- SSH remote management
- CDP and LLDP

### NAT and Access Control

- Static NAT
- Pool-based dynamic PAT
- Standard ACLs
- Extended ACLs
- Restricted SSH management access

### Layer 2 Security

- Port Security
- Sticky MAC address learning
- DHCP Snooping
- DHCP rate limiting
- Dynamic ARP Inspection
- Source MAC, destination MAC, and IP validation

### IPv6

- IPv6 unicast routing
- Global unicast addressing
- EUI-64 interface identifiers
- Link-local addressing
- IPv6 static default routes
- IPv6 floating static route

### Wireless

- Cisco Wireless LAN Controller
- Lightweight Access Points
- Dynamic wireless interface
- WLAN and SSID configuration
- WPA2-PSK
- AES encryption
- Wireless client association

## Repository Contents

This repository will contain:

- The completed network topology
- Device configuration files
- IP addressing and VLAN documentation
- Routing and redundancy documentation
- Security configuration documentation
- Verification command outputs
- Connectivity and failover test results
- Troubleshooting notes
- Packet Tracer limitations observed during the lab
