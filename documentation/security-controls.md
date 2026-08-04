# Network Security Controls

This document explains the management-plane, access-control, and Layer 2 security controls implemented in the enterprise campus network.

The original topology and requirements were provided through the Jeremy's IT Lab CCNA Mega Lab. I implemented, verified, and troubleshot the security controls described below.

## Security Overview

| Security Control | Purpose |
|---|---|
| SSH version 2 | Encrypts remote device-management sessions |
| VTY access control | Restricts SSH access to authorized source networks |
| Local authentication | Requires configured usernames and secrets |
| Extended ACLs | Controls traffic between selected networks |
| Port Security | Restricts devices permitted on access ports |
| Sticky MAC learning | Dynamically learns and retains permitted MAC addresses |
| DHCP Snooping | Blocks unauthorized DHCP server messages |
| DHCP rate limiting | Limits DHCP traffic received on untrusted ports |
| Dynamic ARP Inspection | Validates ARP messages and blocks spoofed entries |
| PortFast | Accelerates endpoint-facing port activation |
| BPDU Guard | Protects the spanning-tree topology from unauthorized switches |
| DTP disabled | Prevents automatic trunk negotiation |
| Unused ports disabled | Reduces unauthorized physical network access |
| Unused native VLAN | Keeps normal user traffic away from the native VLAN |

## Secure Remote Management

SSH version 2 was used instead of Telnet for remote administration.

SSH encrypts:

- Usernames
- Passwords
- Commands
- Device responses
- Session traffic

The general configuration structure included:

```cisco
hostname <device-name>
ip domain-name <domain-name>
username <username> privilege 15 secret <secret>
crypto key generate rsa
ip ssh version 2
```

The VTY lines were configured to use the local username database and accept only SSH:

```cisco
line vty 0 15
 login local
 transport input ssh
```

Telnet was not permitted because it transmits management traffic without encryption.

Authentication values are not included in this public repository.

## Restricted SSH Access

A standard ACL was applied to the VTY lines to restrict remote management access.

General configuration structure:

```cisco
ip access-list standard SSH-ACCESS
 permit <authorized-management-network>
 deny any
```

The ACL was then applied to inbound VTY sessions:

```cisco
line vty 0 15
 access-class SSH-ACCESS in
```

The `access-class` command controls access to the device management session rather than filtering normal transit traffic through the device.

Testing included:

- Successful SSH access from an authorized source
- Failed SSH access from an unauthorized source
- Confirmation that Telnet connections were rejected

## Extended Access Control Lists

Extended ACLs were used to control traffic between selected source and destination networks.

Extended ACLs can filter traffic according to:

- Source IPv4 address
- Destination IPv4 address
- IP protocol
- TCP or UDP port number
- ICMP traffic type

General structure:

```cisco
ip access-list extended <acl-name>
 permit <protocol> <source> <destination>
 deny <protocol> <source> <destination>
 permit ip any any
```

The ACL is applied in the appropriate direction on the selected interface:

```cisco
interface <interface>
 ip access-group <acl-name> in
```

or:

```cisco
interface <interface>
 ip access-group <acl-name> out
```

ACL placement was selected according to the traffic flow and the lab requirements.

Because ACLs are processed from top to bottom, more specific rules must be placed before broader rules.

Every ACL also has an implicit final rule:

```text
deny any
```

Traffic that does not match a permit statement is denied.

## ACL Verification

ACL operation was checked using:

```cisco
show access-lists
show ip access-lists
show running-config | section access-list
show ip interface
```

Connectivity tests were performed from both permitted and denied source devices.

The ACL counters were checked to confirm that packets matched the expected entries.

## Access-Port Hardening

Endpoint-facing switchports were manually configured as access ports.

General configuration structure:

```cisco
interface <access-interface>
 switchport mode access
 switchport nonegotiate
```

Disabling DTP prevents the endpoint from negotiating a trunk connection with the switch.

Each access port was assigned to the appropriate VLAN according to the connected device.

Examples include:

```cisco
switchport access vlan 10
```

for PCs, and:

```cisco
switchport access vlan 30
```

for the server.

Phone-facing ports used separate data and voice VLANs:

```cisco
switchport access vlan 10
switchport voice vlan 20
```

## Port Security

Port Security was enabled on selected endpoint-facing access ports.

Its purpose is to restrict which MAC addresses can use a switchport.

General configuration structure:

```cisco
interface <access-interface>
 switchport port-security
 switchport port-security mac-address sticky
```

Sticky learning allows the switch to dynamically learn the connected device’s MAC address and add it to the port-security configuration.

Port Security helps reduce the risk of:

- Unauthorized devices connecting to the network
- MAC-address table abuse
- Multiple unexpected devices using one access port
- Users moving devices to unauthorized locations

## Port-Security Verification

Port Security was verified using:

```cisco
show port-security
show port-security interface <interface>
show running-config interface <interface>
show mac address-table interface <interface>
```

The verification output was checked for:

- Port-security status
- Learned secure MAC addresses
- Sticky MAC addresses
- Violation counters
- Current device count

## DHCP Snooping

DHCP Snooping was enabled to protect the network from unauthorized DHCP servers.

DHCP Snooping classifies switchports as:

```text
Trusted
Untrusted
```

Trusted ports are allowed to send DHCP server messages.

Untrusted ports are normally endpoint-facing ports and are not permitted to send DHCP server responses.

General configuration structure:

```cisco
ip dhcp snooping
ip dhcp snooping vlan <vlan-list>
```

The uplink or trusted DHCP path was configured with:

```cisco
interface <trusted-interface>
 ip dhcp snooping trust
```

Endpoint-facing ports remained untrusted by default.

This prevents a rogue device connected to an access port from acting as a DHCP server and distributing false network settings.

## DHCP Snooping Binding Database

DHCP Snooping creates a binding table containing information learned from legitimate DHCP transactions.

The table can include:

- Client MAC address
- Assigned IPv4 address
- VLAN
- Interface
- Lease duration
- Binding type

The binding table was verified using:

```cisco
show ip dhcp snooping binding
```

This information is also used by Dynamic ARP Inspection.

## DHCP Rate Limiting

DHCP rate limiting was configured on untrusted access ports.

General structure:

```cisco
interface <access-interface>
 ip dhcp snooping limit rate <rate>
```

Rate limiting helps protect the switch and DHCP server from excessive DHCP messages.

It can reduce the impact of:

- DHCP starvation attacks
- Malfunctioning clients
- Excessive DHCP discovery traffic

The exact rate was configured according to the lab requirements.

## DHCP Snooping Verification

The following commands were used:

```cisco
show ip dhcp snooping
show ip dhcp snooping binding
show running-config | include dhcp snooping
show running-config interface <interface>
```

Verification confirmed:

- DHCP Snooping was enabled
- The required VLANs were protected
- Uplink interfaces were trusted
- Endpoint ports remained untrusted
- Client bindings appeared after successful DHCP assignment

## Dynamic ARP Inspection

Dynamic ARP Inspection was enabled to protect the network from ARP spoofing and ARP poisoning.

General configuration structure:

```cisco
ip arp inspection vlan <vlan-list>
```

DAI examines ARP messages received on untrusted ports.

For DHCP clients, it compares ARP information against the DHCP Snooping binding table.

An ARP message can be rejected when its information does not match a valid binding.

## Trusted and Untrusted DAI Ports

Access ports remain untrusted by default.

Infrastructure uplinks that legitimately carry ARP traffic between switches were configured as trusted where required:

```cisco
interface <trusted-interface>
 ip arp inspection trust
```

Trusted-port selection must be performed carefully because DAI does not inspect ARP messages received on trusted ports.

## ARP Validation

Additional ARP validation was enabled for:

- Source MAC address
- Destination MAC address
- IPv4 address

General configuration structure:

```cisco
ip arp inspection validate src-mac dst-mac ip
```

These checks verify consistency between the ARP payload and the Ethernet frame.

This helps detect malformed or spoofed ARP messages.

## DAI Verification

DAI was verified using:

```cisco
show ip arp inspection
show ip arp inspection interfaces
show ip arp inspection statistics
show ip dhcp snooping binding
show arp
```

Successful verification required:

- Correct VLAN selection
- Correct trusted uplinks
- Valid DHCP Snooping bindings
- Normal ARP communication between legitimate devices
- No unexpected inspection drops

## DAI Troubleshooting

During implementation, an incorrect VLAN entry was identified in the Office B DAI configuration.

The troubleshooting process included:

1. Checking which VLANs had DAI enabled.
2. Comparing the configured VLAN list with the Office B VLAN design.
3. Identifying the incorrect VLAN entry.
4. Removing the incorrect configuration.
5. Enabling DAI on the correct VLAN.
6. Verifying normal ARP and client connectivity.

Useful commands included:

```cisco
show ip arp inspection
show ip arp inspection interfaces
show ip dhcp snooping binding
show vlan brief
show interfaces trunk
```

This issue demonstrated that DAI depends on correct VLAN selection, trusted-interface configuration, and DHCP Snooping bindings.

## PortFast

PortFast was enabled on endpoint-facing access ports.

General configuration:

```cisco
interface <access-interface>
 spanning-tree portfast
```

PortFast allows the interface to enter the forwarding state without waiting through the normal spanning-tree transition process.

PortFast should only be enabled on ports connected to endpoints, not on links between switches.

## BPDU Guard

BPDU Guard was enabled on PortFast access ports.

General configuration:

```cisco
interface <access-interface>
 spanning-tree bpduguard enable
```

If the port receives a BPDU, BPDU Guard places the interface into an error-disabled state.

This protects the spanning-tree topology from:

- Unauthorized switches
- Accidental switch connections
- Unexpected Layer 2 loops
- Attempts to influence the root-bridge election

## Unused Interface Security

Unused switch interfaces were administratively disabled.

General configuration:

```cisco
interface range <unused-interfaces>
 shutdown
```

Disabling unused ports reduces:

- Unauthorized network access
- Accidental cabling
- Unnecessary spanning-tree participation
- Exposure to Layer 2 attacks

## Native VLAN Security

An unused VLAN was configured as the native VLAN on standard trunk links.

The native VLAN was not used for normal endpoint traffic.

General trunk structure:

```cisco
interface <trunk-interface>
 switchport mode trunk
 switchport trunk native vlan 1000
 switchport nonegotiate
```

Using an unused native VLAN reduces the risk of untagged traffic entering an active user VLAN.

## NAT Security Clarification

Static NAT and PAT were configured for Internet connectivity, but NAT is not a replacement for a firewall.

NAT changes address information and can reduce direct exposure of internal addressing, but it does not provide complete traffic inspection or security-policy enforcement.

The implemented controls relied on:

- ACLs
- Restricted management access
- Layer 2 protections
- Secure management protocols
- VLAN segmentation

## Security Verification Commands

The following commands were used to verify the security configuration:

```cisco
show access-lists
show ip access-lists
show ip ssh
show ssh
show port-security
show port-security interface <interface>
show mac address-table
show ip dhcp snooping
show ip dhcp snooping binding
show ip arp inspection
show ip arp inspection interfaces
show ip arp inspection statistics
show spanning-tree
show interfaces status
show interfaces trunk
show running-config interface <interface>
```

## Information Excluded from the Repository

The following sensitive values are intentionally omitted or sanitized:

- Local usernames
- User passwords
- Enable secrets
- RSA key details
- SNMP community strings
- NTP authentication keys
- FTP credentials
- Wireless pre-shared keys

The repository documents the implementation without publishing reusable authentication information.
