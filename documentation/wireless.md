# Wireless Network Implementation

This document explains the Wireless LAN Controller, lightweight access points, WLAN, wireless security, client VLAN, DHCP behavior, association, and troubleshooting performed in the enterprise campus network.

The original topology and requirements were provided through the Jeremy's IT Lab CCNA Mega Lab. I implemented, verified, and troubleshot the wireless configuration described below.

## Wireless Architecture

The network uses a controller-based wireless design.

```text
Wireless clients
       |
Lightweight Access Points
       |
Management VLAN 99
       |
Wireless LAN Controller
       |
Wi-Fi VLAN 40
       |
Distribution switches and routed network
```

The wireless infrastructure includes:

- One Cisco Wireless LAN Controller
- Two lightweight access points
- One centrally managed WLAN
- WPA2-PSK security
- AES encryption
- A separate VLAN for wireless clients
- Centralized client traffic forwarding

## Wireless Devices

| Device | Location | Purpose |
|---|---|---|
| WLC1 | Office A | Central management of the WLAN and lightweight APs |
| LWAP1 | Office A | Provides wireless coverage for Office A |
| LWAP2 | Office B | Provides wireless coverage for Office B |
| Laptop1 | Office A | Wireless client |
| Laptop2 | Office B | Wireless client |

Both access points are controlled by WLC1.

## Management Addressing

| Device | Management Network | IPv4 Address |
|---|---|---|
| WLC1 | VLAN 99 | 10.0.0.7 |
| LWAP1 | Office A VLAN 99 | 10.0.0.11 |
| LWAP2 | Office B VLAN 99 | 10.0.0.28 |

VLAN 99 is used for wireless infrastructure management.

The lightweight access points receive network connectivity through access ports assigned to VLAN 99.

## Wireless Client Network

Wireless clients use VLAN 40 in Office A’s addressing plan.

| Setting | Value |
|---|---|
| VLAN | 40 |
| Subnet | 10.6.0.0/24 |
| Default gateway | 10.6.0.1 |
| WLC dynamic interface | 10.6.0.4/24 |
| DHCP server | R1 |
| DNS server | SRV1 at 10.5.0.4 |

The HSRP virtual IP address `10.6.0.1` serves as the default gateway for wireless clients.

## WLC Management Interface

The WLC management interface uses:

```text
IPv4 address: 10.0.0.7
Management VLAN: 99
```

The connection between WLC1 and ASW-A1 carries both management and wireless-client traffic.

The switchport is configured as a trunk carrying:

```text
VLAN 99 — WLC management traffic
VLAN 40 — Wireless client traffic
```

VLAN 99 is the native VLAN on this specific WLC trunk because the WLC management interface sends untagged traffic.

General switch configuration structure:

```cisco
interface <wlc-facing-interface>
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 40,99
 switchport nonegotiate
```

DTP was disabled because the WLC does not dynamically negotiate an 802.1Q trunk.

## Dynamic Wireless Interface

A dynamic interface was created on WLC1 for wireless client traffic.

| Setting | Value |
|---|---|
| Interface purpose | Wireless clients |
| VLAN identifier | 40 |
| IPv4 address | 10.6.0.4 |
| Subnet mask | 255.255.255.0 |
| Default gateway | 10.6.0.1 |
| DHCP server | 10.0.0.76 |

The DHCP server address points to R1 Loopback0.

The dynamic interface connects the WLAN to VLAN 40.

The traffic path is:

```text
Wireless client
       |
Lightweight AP
       |
CAPWAP tunnel
       |
WLC1
       |
Dynamic interface for VLAN 40
       |
ASW-A1 trunk
       |
Distribution switches
```

## Lightweight Access Points

The APs operate as lightweight access points rather than autonomous APs.

A lightweight AP depends on the Wireless LAN Controller for:

- WLAN configuration
- SSID information
- Wireless security settings
- Radio management
- Client association management
- Centralized forwarding behavior

The AP-facing switchports use management VLAN 99.

General configuration structure:

```cisco
interface <ap-facing-interface>
 switchport mode access
 switchport access vlan 99
 switchport nonegotiate
 spanning-tree portfast
 spanning-tree bpduguard enable
```

## CAPWAP Operation

The lightweight APs communicate with WLC1 using CAPWAP.

CAPWAP separates wireless operations into:

- Control traffic
- Client data traffic

Control traffic is used for AP registration and management.

Client data traffic is tunneled from the AP to the WLC because FlexConnect local switching was not used.

## Centralized Switching

The wireless clients are centrally switched through WLC1.

This means a wireless client connected through LWAP2 in Office B can still have its traffic tunneled to WLC1 in Office A before entering VLAN 40.

The path can therefore be:

```text
Laptop2
   |
LWAP2 in Office B
   |
Enterprise network
   |
WLC1 in Office A
   |
VLAN 40
```

This behavior is expected in a centrally switched controller-based WLAN.

## WLAN Configuration

A WLAN was created and mapped to the dynamic interface associated with VLAN 40.

The WLAN configuration included:

- A WLAN profile
- An SSID
- Mapping to the Wi-Fi dynamic interface
- WPA2 personal security
- AES encryption
- A pre-shared key
- Administrative enablement

The exact SSID and pre-shared key are omitted from the public repository.

## Wireless Security

The WLAN uses:

```text
WPA2-PSK
AES encryption
```

WPA2-PSK authenticates clients using a shared wireless password.

AES encrypts wireless traffic between the client and the access point.

The pre-shared key is intentionally excluded from this repository.

## WLAN Administrative State

Creating a WLAN does not automatically guarantee that clients can use it.

The WLAN must also be administratively enabled.

During troubleshooting, the WLAN configuration existed but remained disabled.

The issue was corrected by enabling the WLAN from the WLC interface.

This demonstrated the difference between:

```text
WLAN configured
```

and:

```text
WLAN configured and enabled
```

## Client Association

Each laptop was configured with:

- The correct SSID
- WPA2-PSK authentication
- AES encryption
- The correct pre-shared key
- DHCP addressing

After successful authentication and association, the client requested an IPv4 address from R1.

A successfully connected wireless client should receive:

- An address from `10.6.0.0/24`
- A subnet mask of `255.255.255.0`
- A default gateway of `10.6.0.1`
- A DNS server address of `10.5.0.4`

## AP Selection

Both access points advertise the same WLAN.

A client may associate with either access point depending on factors such as:

- Signal strength
- Client position
- Radio availability
- Existing association state
- Packet Tracer simulation behavior

During testing, a wireless client associated with an access point in the opposite office.

Because both APs advertise the same centrally managed WLAN, this did not necessarily indicate an incorrect WLAN configuration.

The associated AP was confirmed through WLC client and AP information.

## DHCP and APIPA Troubleshooting

A wireless client temporarily received an APIPA address from:

```text
169.254.0.0/16
```

An APIPA address indicates that the client did not receive a valid DHCP response.

The troubleshooting process included checking:

1. Whether the WLAN was enabled.
2. Whether the client was associated with an AP.
3. Whether the SSID and pre-shared key were correct.
4. Whether the WLAN was mapped to the correct dynamic interface.
5. Whether the WLC dynamic interface used VLAN 40.
6. Whether VLAN 40 was allowed on the WLC trunk.
7. Whether the VLAN 40 SVI was operational.
8. Whether the HSRP virtual gateway was reachable.
9. Whether DHCP relay was configured.
10. Whether R1 had the correct Wi-Fi DHCP pool.

After correcting the wireless state and renewing DHCP, the client received an address from the correct subnet.

## Wireless Connectivity Testing

Wireless connectivity was tested in stages.

### Stage 1: Local Addressing

The client was checked for:

- A valid IPv4 address
- Correct subnet mask
- Correct default gateway
- Correct DNS server

### Stage 2: Default Gateway

The client tested connectivity to:

```text
10.6.0.1
```

This verified communication through the WLAN, WLC, VLAN 40, and HSRP gateway.

### Stage 3: Internal Services

The client tested connectivity to internal destinations such as:

```text
10.5.0.4
```

This verified routing between the Wi-Fi VLAN and server VLAN.

### Stage 4: Internet Connectivity

The client tested external connectivity through R1, NAT, and the active Internet path.

### Stage 5: DNS Resolution

The client tested access using a configured hostname to verify DNS operation.

## Wireless Verification

The WLC interface was used to verify:

- AP registration
- AP management addresses
- WLAN status
- WLAN-to-interface mapping
- Security configuration
- Associated clients
- Client IPv4 addresses
- Client-to-AP association

The switches and routed network were verified using commands such as:

```cisco
show vlan brief
show interfaces trunk
show interfaces switchport
show ip interface brief
show standby brief
show ip route
show ip dhcp binding
show mac address-table
show arp
ping
traceroute
```

## Troubleshooting Summary

Wireless issues identified and investigated included:

- A configured WLAN that remained disabled
- Clients receiving APIPA addresses
- DHCP lease renewal problems
- Confirming VLAN 40 across the WLC trunk
- Confirming the dynamic-interface mapping
- Checking AP registration with WLC1
- Checking which AP a client had associated with
- A client associating with an AP in the opposite office
- Packet Tracer wireless simulation behavior

## Information Excluded from the Repository

The following wireless authentication information is intentionally omitted:

- Wireless pre-shared key
- Administrative passwords
- WLC login credentials
- Other reusable authentication values

The repository documents the wireless design and implementation without exposing credentials.
