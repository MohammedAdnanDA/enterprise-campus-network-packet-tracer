# Network Services and Management

This document explains the DHCP, DNS, NTP, Syslog, SNMP, FTP, SSH, and device-discovery services implemented in the enterprise campus network.

The original topology and requirements were provided through the Jeremy's IT Lab CCNA Mega Lab. I implemented, verified, and troubleshot the services described below.

## Service Overview

| Service | Device | Purpose |
|---|---|---|
| DHCP | R1 | Dynamically assigns IPv4 settings to clients and infrastructure devices |
| DHCP Relay | Distribution switches | Forwards DHCP requests from VLANs to R1 |
| DNS | SRV1 | Resolves configured domain names to IPv4 addresses |
| NTP | R1 | Provides a common time source for network devices |
| Syslog | SRV1 | Collects logging messages from network devices |
| SNMP | Network devices | Allows centralized monitoring and management |
| FTP | SRV1 | Stores device configuration backups |
| SSH | Routers and switches | Provides encrypted remote administration |
| LLDP | Network devices | Discovers directly connected neighboring devices |

## DHCP Server

R1 operates as the centralized DHCP server for the enterprise network.

Separate DHCP pools were configured for:

### Office A

- Management VLAN 99
- PC VLAN 10
- Phone VLAN 20
- Wi-Fi VLAN 40

### Office B

- Management VLAN 99
- PC VLAN 10
- Phone VLAN 20

The server VLAN in Office B uses static addressing and therefore does not require a DHCP pool.

## DHCP Pool Configuration

Each DHCP pool provides clients with:

- An IPv4 address
- A subnet mask
- The appropriate HSRP virtual default gateway
- The DNS server address

Example configuration structure:

```cisco
ip dhcp pool A-PC
 network 10.1.0.0 255.255.255.0
 default-router 10.1.0.1
 dns-server 10.5.0.4
```

The first ten usable addresses in each client subnet were excluded from dynamic assignment.

Example:

```cisco
ip dhcp excluded-address 10.1.0.1 10.1.0.10
```

These excluded addresses were reserved for:

- HSRP virtual gateways
- Distribution-switch interfaces
- Access switches
- Wireless infrastructure
- Servers
- Other statically addressed devices

## DHCP Relay

Because R1 is not directly connected to every client VLAN, DHCP relay was configured on the distribution-switch SVIs.

The relay destination is R1 Loopback0:

```text
10.0.0.76
```

Example:

```cisco
interface vlan 10
 ip helper-address 10.0.0.76
```

The relay process works as follows:

```text
Client DHCP broadcast
        |
Distribution-switch SVI
        |
ip helper-address
        |
R1 DHCP server
```

The distribution switch converts the local DHCP broadcast into a unicast packet and forwards it to R1.

The relay information also allows R1 to determine which DHCP pool should be used for the requesting client.

## DHCP Verification

DHCP operation was verified using:

```cisco
show ip dhcp pool
show ip dhcp binding
show ip dhcp conflict
show running-config | section dhcp
```

Client addressing was checked using the IP configuration utility on the Packet Tracer endpoints.

A successful DHCP client received:

- An address from the correct subnet
- The correct subnet mask
- The HSRP virtual default gateway
- The DNS server address

## APIPA Troubleshooting

During testing, some clients temporarily received addresses from:

```text
169.254.0.0/16
```

This is an Automatic Private IP Addressing address and indicates that the client did not receive a DHCP response.

Troubleshooting included checking:

- Physical connectivity
- Switchport VLAN configuration
- Trunk allowed VLANs
- SVI status
- HSRP gateway availability
- `ip helper-address`
- DHCP pool configuration
- DHCP exclusions
- Wireless association

The client’s DHCP lease was then renewed after correcting the underlying issue.

## DNS Service

SRV1 provides DNS service using:

```text
10.5.0.4
```

R1 distributes this address to DHCP clients as their DNS server.

DNS allows clients to reach configured resources by name instead of requiring the user to enter an IPv4 address.

The basic process is:

```text
Client enters a hostname
        |
Client queries SRV1
        |
SRV1 returns the configured IPv4 address
        |
Client connects to the destination
```

DNS functionality was verified by resolving the configured DNS records from client devices.

## NTP Service

R1 provides the central time source for the internal network.

Its Loopback0 address is used as the NTP server address:

```text
10.0.0.76
```

Using a loopback address provides a stable destination that is not tied to one physical R1 interface.

NTP authentication was enabled so devices could verify that time updates came from the expected server.

The client configuration follows this structure:

```cisco
ntp authenticate
ntp trusted-key 1
ntp authentication-key 1 md5 <key>
ntp server 10.0.0.76 key 1
```

The actual authentication key is not included in this public repository.

Accurate and synchronized timestamps are important for:

- Syslog analysis
- Security investigations
- Troubleshooting
- Configuration auditing
- Correlating events across multiple devices

## NTP Verification

NTP was verified using:

```cisco
show clock
show ntp associations
show ntp status
```

A synchronized device should identify R1 as its time source.

## Syslog Service

SRV1 acts as the centralized Syslog server:

```text
10.5.0.4
```

Network devices were configured to send logging messages to SRV1.

Example:

```cisco
logging host 10.5.0.4
service timestamps log datetime msec
```

Centralized logging allows events from multiple devices to be viewed in one location.

Examples of logged events include:

- Interface status changes
- Configuration changes
- Authentication activity
- Port-security violations
- Routing-neighbor changes
- Device warnings and errors

Timestamps were enabled so events could be correlated accurately.

## Syslog Verification

Syslog was verified by:

1. Opening the Syslog service on SRV1.
2. Generating a device event, such as shutting down and restoring an interface.
3. Confirming that the event appeared in the server log.

Example event generation:

```cisco
interface g0/1
 shutdown
 no shutdown
```

## SNMP

SNMP was configured to support centralized network monitoring.

SNMP allows a management system to retrieve information such as:

- Interface status
- Interface counters
- Device uptime
- CPU and memory information
- Routing information
- System identification

The configured community value is not included in this public documentation.

Example configuration structure:

```cisco
snmp-server community <community> ro
```

The `ro` option provides read-only access and prevents the monitoring system from modifying the device configuration.

## FTP Service

SRV1 provides FTP service for network-device configuration backups.

The server address is:

```text
10.5.0.4
```

Device configurations can be copied to the FTP server using a command such as:

```cisco
copy running-config ftp:
```

The device then requests:

- The remote FTP server address
- An FTP username
- An FTP password
- The destination filename

Example backup filenames can identify the device and configuration type:

```text
R1-running-config
CSW1-running-config
DSW-A1-running-config
```

FTP credentials are not included in the public repository.

## SSH Remote Management

SSH version 2 was configured for encrypted remote access to routers and switches.

SSH is preferred over Telnet because SSH encrypts:

- Usernames
- Passwords
- Commands
- Session data

The general configuration includes:

```cisco
hostname R1
ip domain-name example.local
username <user> privilege 15 secret <secret>
crypto key generate rsa
ip ssh version 2
```

VTY lines were configured to:

- Authenticate using the local username database
- Accept SSH connections
- Reject Telnet access

Example:

```cisco
line vty 0 15
 login local
 transport input ssh
```

Passwords, usernames, domain values, and cryptographic details are omitted or sanitized in the public repository.

## Restricted Management Access

A standard ACL was applied to the VTY lines to restrict which source network could establish SSH sessions.

Example structure:

```cisco
ip access-list standard SSH-ACCESS
 permit <authorized-management-network>
 deny any

line vty 0 15
 access-class SSH-ACCESS in
```

This adds another security layer beyond username and password authentication.

An unauthorized device may still be able to reach the management IP with ICMP, but its SSH connection is denied by the VTY access-class.

## SSH Verification

SSH was verified using:

```cisco
show ip ssh
show ssh
show access-lists
```

Testing included:

- A successful connection from an authorized management device
- A denied connection from an unauthorized source
- Confirmation that Telnet was not accepted

## Device Discovery

CDP was disabled, and LLDP was enabled for neighbor discovery.

CDP is Cisco proprietary, while LLDP is an IEEE vendor-neutral protocol.

Example:

```cisco
no cdp run
lldp run
```

LLDP was used to identify directly connected devices and their interfaces.

Verification commands included:

```cisco
show lldp
show lldp neighbors
show lldp neighbors detail
```

## LLDP Interface Control

LLDP transmission was disabled on the specified LWAP-facing access-switch interface while retaining the required discovery behavior elsewhere.

Example:

```cisco
interface f0/1
 no lldp transmit
```

LLDP can be controlled independently in each direction:

```cisco
lldp transmit
lldp receive
```

This allows the administrator to control whether an interface sends or processes LLDP advertisements.

## Verification Commands

The following commands were used to verify network services and management access:

```cisco
show ip dhcp pool
show ip dhcp binding
show ip dhcp conflict
show ip interface brief
show running-config | include helper-address
show clock
show ntp associations
show ntp status
show logging
show snmp
show ip ssh
show ssh
show access-lists
show lldp neighbors
show lldp neighbors detail
ping
```

## Security Note

Authentication information is intentionally excluded from this repository, including:

- Enable secrets
- Local usernames and passwords
- NTP authentication keys
- SNMP community values
- FTP credentials
- Wireless passwords

This preserves the technical documentation without publishing reusable credentials.
