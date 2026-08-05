# Security and DHCP Verification

This file contains operational evidence for DHCP, DHCP Snooping, Dynamic ARP Inspection, Port Security, SSH management restrictions, and access-control behavior.

## Verification Summary

| Security Control | Verification Result |
|---|---|
| Centralized DHCP server | Operational |
| DHCP address bindings | Confirmed |
| DHCP Snooping | Enabled |
| DHCP Snooping binding table | Populated |
| DHCP rate limiting | Enabled on access port |
| Trusted DHCP uplinks | Configured |
| Dynamic ARP Inspection | Active |
| ARP source MAC validation | Enabled |
| ARP destination MAC validation | Enabled |
| ARP IP validation | Enabled |
| Port Security | Secure-up |
| Sticky MAC learning | Confirmed |
| Port-security violations | 0 |
| SSH-only VTY access | Configured |
| VTY source ACL | Applied |
| Extended inter-office ACL | Defined but not attached |

## DHCP Server Verification

R1 operates as the centralized DHCP server.

Command:

```cisco
show ip dhcp binding
```

Output:

```text
IP address       Hardware address       Type
10.0.0.11        0002.4AE7.1601         Automatic
10.0.0.12        00E0.B0AD.0060         Automatic
10.0.0.14        0003.E493.89CA         Automatic
10.1.0.11        0001.4290.A4D1         Automatic
10.0.0.28        00E0.F9EA.1401         Automatic
```

The bindings confirm that R1 successfully assigned addresses to management devices and PC1.

PC1 received:

```text
IPv4 address: 10.1.0.11
MAC address:  0001.4290.A4D1
Pool:         Office A PC network
```

## DHCP Pool Verification

Command:

```cisco
show ip dhcp pool
```

The following pools were present:

| Pool | Network | Active Leases |
|---|---|---:|
| A-Mgmt | 10.0.0.0/28 | 3 |
| A-PC | 10.1.0.0/24 | 1 |
| A-Phone | 10.2.0.0/24 | 0 |
| B-Mgmt | 10.0.0.16/28 | 1 |
| B-PC | 10.3.0.0/24 | 0 |
| B-Phone | 10.4.0.0/24 | 0 |
| Wi-Fi | 10.6.0.0/24 | 0 |

A pool with zero current leases is still configured and available. It only indicates that no active client held a lease from that pool when the output was captured.

## DHCP Snooping Verification

DHCP Snooping was verified on ASW-A2.

Command:

```cisco
show ip dhcp snooping
```

Relevant output:

```text
Switch DHCP snooping is enabled

DHCP snooping is configured on following VLANs:
10,20,40,99

Insertion of option 82 is disabled
Option 82 on untrusted port is not allowed
Verification of hwaddr field is enabled

Interface                  Trusted    Rate limit (pps)
FastEthernet0/1            no         15
```

This confirms:

- DHCP Snooping is globally enabled.
- VLANs 10, 20, 40, and 99 are protected.
- The endpoint-facing port is untrusted.
- DHCP traffic on FastEthernet0/1 is limited to 15 packets per second.
- Option 82 insertion is disabled according to the lab requirements.

## DHCP Snooping Binding Table

Command:

```cisco
show ip dhcp snooping binding
```

Output:

```text
MacAddress          IpAddress        Lease(sec)  Type           VLAN  Interface
00:01:42:90:A4:D1   10.1.0.11        0           dhcp-snooping  10    FastEthernet0/1

Total number of bindings: 1
```

The binding matches the address assigned by R1:

```text
MAC address: 00:01:42:90:A4:D1
IPv4 address: 10.1.0.11
VLAN: 10
Interface: FastEthernet0/1
```

This binding can be used by Dynamic ARP Inspection to validate ARP traffic from PC1.

## Trusted Uplinks

The two distribution-switch uplinks were configured as trusted for both DHCP Snooping and Dynamic ARP Inspection.

Relevant running configuration:

```cisco
interface GigabitEthernet0/1
 switchport trunk native vlan 1000
 switchport trunk allowed vlan 10,20,40,99
 ip arp inspection trust
 ip dhcp snooping trust
 switchport mode trunk
 switchport nonegotiate

interface GigabitEthernet0/2
 switchport trunk native vlan 1000
 switchport trunk allowed vlan 10,20,40,99
 ip arp inspection trust
 ip dhcp snooping trust
 switchport mode trunk
 switchport nonegotiate
```

The access port remains untrusted, while infrastructure uplinks are trusted.

## Dynamic ARP Inspection Verification

Command:

```cisco
show ip arp inspection
```

Relevant output:

```text
Source Mac Validation      : Enabled
Destination Mac Validation : Enabled
IP Address Validation      : Enabled

Vlan     Configuration    Operation
10       Enabled          Active
20       Enabled          Active
40       Enabled          Active
99       Enabled          Active
```

This confirms that DAI is active on the required Office A VLANs.

The following validation checks are enabled:

- Source MAC validation
- Destination MAC validation
- IPv4 address validation

DAI uses DHCP Snooping bindings to identify invalid or spoofed ARP packets received on untrusted ports.

## DAI Interface Trust

Command:

```cisco
show ip arp inspection interfaces
```

Relevant output:

```text
Interface        Trust State     Rate(pps)
Fa0/1            Untrusted              15
```

FastEthernet0/1 is an untrusted endpoint port and is subject to ARP inspection.

The running configuration confirms that GigabitEthernet0/1 and GigabitEthernet0/2 are trusted uplinks.

## Port Security Verification

Port Security was verified on ASW-A2 FastEthernet0/1.

Command:

```cisco
show port-security interface fastEthernet0/1
```

Output:

```text
Port Security              : Enabled
Port Status                : Secure-up
Violation Mode             : Restrict
Aging Time                 : 0 mins
Aging Type                 : Absolute
SecureStatic Address Aging : Disabled
Maximum MAC Addresses      : 2
Total MAC Addresses        : 1
Configured MAC Addresses   : 0
Sticky MAC Addresses       : 1
Last Source Address:Vlan   : 0001.4290.A4D1:10
Security Violation Count   : 0
```

This confirms:

- Port Security is enabled.
- The interface is operational in `Secure-up` state.
- Restrict mode is configured.
- The port permits up to two MAC addresses for the connected phone and PC.
- One sticky MAC address was learned.
- No security violations were recorded.

## SSH Management Access Control

A standard ACL restricts remote management access to the Office A PC network.

Command:

```cisco
show access-lists 1
```

Output:

```text
Standard IP access list 1
    permit 10.1.0.0 0.0.0.255
```

The ACL is applied to both VTY line ranges.

Command:

```cisco
show running-config | begin line vty 0 4
```

Output:

```text
line vty 0 4
 access-class 1 in
 logging synchronous
 login local
 transport input ssh

line vty 5 15
 access-class 1 in
 logging synchronous
 login local
 transport input ssh
```

This confirms:

- Only sources in `10.1.0.0/24` are permitted to initiate VTY sessions.
- The local user database is used for authentication.
- SSH is accepted.
- Telnet is not accepted.

The `access-class` protects the device management plane rather than normal transit traffic.

## Extended ACL Observation

The extended ACL exists on DSW-A1:

```cisco
ip access-list extended OfficeA_to_OfficeB
 permit icmp 10.1.0.0 0.0.0.255 10.3.0.0 0.0.0.255
 deny ip 10.1.0.0 0.0.0.255 10.3.0.0 0.0.0.255
 permit ip any any
```

Its intended policy is:

- Permit ICMP from Office A PCs to Office B PCs.
- Deny other IP traffic between those two PC networks.
- Permit unrelated traffic.

However, verification showed:

```text
Inbound access list is not set
```

The command to apply the ACL was accepted at the configuration prompt but was not retained in the running configuration on this Packet Tracer switch image.

Therefore, the ACL is documented as defined but not actively enforced. The repository does not claim that this specific transit ACL test succeeded.

## Conclusion

The verified controls demonstrate:

- Centralized DHCP address assignment
- DHCP Snooping on user and management VLANs
- A valid DHCP Snooping binding for PC1
- Trusted infrastructure uplinks and untrusted endpoint ports
- DHCP rate limiting
- Active Dynamic ARP Inspection
- Source MAC, destination MAC, and IP validation
- Port Security with sticky MAC learning
- Zero port-security violations
- SSH-only remote management
- Source-based restrictions on VTY access

The extended inter-office ACL was successfully defined but could not be verified as attached to the SVI in the Packet Tracer device image used for this lab.
