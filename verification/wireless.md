# Wireless Network Verification

This document records the operational verification of the Wireless LAN Controller, WLAN, lightweight access points, wireless clients, DHCP addressing, and wireless connectivity.

## Verification Summary

| Component | Result |
|---|---|
| WLC management interface | Operational |
| WLC reachable from wired client | Confirmed |
| WLAN ID 1 | Enabled |
| SSID | Wi-Fi |
| WLAN security | WPA2-PSK |
| Dynamic Wi-Fi interface | Configured |
| Wi-Fi VLAN | VLAN 40 |
| LWAP1 registration | Registered |
| LWAP2 registration | Registered |
| Wireless client association | Confirmed |
| Internal server connectivity | Confirmed |
| Intended VLAN 40 client addressing | Not confirmed during capture |

## WLC Management Interface

WLC1 uses the following management configuration:

```text
Management IP address: 10.0.0.7
Subnet mask:           255.255.255.240
Default gateway:       10.0.0.1
DNS server:            10.5.0.4
```

The management interface belongs to the Office A management network:

```text
10.0.0.0/28
```

## WLC Reachability

PC1 successfully reached WLC1.

Command:

```text
ping 10.0.0.7
```

Result:

```text
Packets: Sent = 4, Received = 4, Lost = 0
Loss: 0%
```

This confirmed that the WLC management interface was operational and reachable from the wired Office A network.

The WLC HTTPS management page was also successfully opened from PC1.

## WLAN Configuration

The configured WLAN was verified through the WLC web interface.

```text
WLAN ID:                  1
Profile name:             Wi-Fi
SSID:                     Wi-Fi
Administrative status:    Enabled
Security policy:          WPA2 with PSK authentication
Interface/Interface Group: Wi-Fi
```

This confirms that the `Wi-Fi` SSID was enabled and mapped to the WLC dynamic interface named `Wi-Fi`.

## Dynamic Wi-Fi Interface

The WLC dynamic interface uses the following configuration:

```text
Interface name:       Wi-Fi
VLAN identifier:      40
IP address:           10.6.0.4
Subnet mask:          255.255.255.0
Default gateway:      10.6.0.1
Primary DHCP server:  10.0.0.76
Physical port:        1
Active port:          1
DHCP proxy mode:      Global
ACL:                  none
```

The intended wireless client network is:

```text
10.6.0.0/24
```

The first-hop redundant gateway for this network is:

```text
10.6.0.1
```

## Lightweight Access Point Registration

Both lightweight access points were registered with WLC1.

| Access Point | Management IP | Model | Status | Mode | Ethernet Speed |
|---|---|---|---|---|---|
| LWAP1 | 10.0.0.11 | AIR-CAP3702I-A-K9 | REG | FlexConnect | 100 Mbps |
| LWAP2 | 10.0.0.27 | AIR-CAP3702I-A-K9 | REG | FlexConnect | 100 Mbps |

The WLC monitor page reported:

```text
Total access points: 2
Operational access points: 2
Down access points: 0
```

Both 802.11a/n/ac and 802.11b/g/n radio networks were enabled.

## FlexConnect Operation

Both lightweight access points were operating in FlexConnect mode.

The Office A access-switch configuration showed:

```cisco
interface FastEthernet0/1
 switchport access vlan 99
 switchport mode access
```

This interface connects WLC1 to the management VLAN.

The access-point-facing interface was configured as a trunk:

```cisco
interface FastEthernet0/2
 switchport trunk native vlan 99
 switchport trunk allowed vlan 40,99
 switchport mode trunk
 switchport nonegotiate
 spanning-tree portfast trunk
```

This design permits:

- Native VLAN 99 for access-point management traffic
- Tagged VLAN 40 for wireless client traffic

The distribution uplinks also allow VLANs 40 and 99.

## Wireless Client Association

The WLC reported:

```text
Current wireless clients: 2
Excluded clients:         0
Disabled clients:         0
```

At the time of capture:

```text
LWAP1 clients: 2
LWAP2 clients: 0
```

This confirmed that wireless clients were associated through the controller-managed wireless infrastructure.

Client distribution between APs can change based on association state and the Packet Tracer simulation.

## Laptop1 Addressing

Laptop1 received:

```text
IPv4 address:    10.0.0.12
Subnet mask:     255.255.255.240
Default gateway: 10.0.0.1
DNS suffix:      jeremysitlab.com
```

Laptop1 successfully reached the wireless VLAN gateway after initial convergence:

```text
Destination: 10.6.0.1
Sent:        4
Received:    2
Lost:        2
```

Laptop1 successfully reached SRV1:

```text
Destination: 10.5.0.4
Sent:        4
Received:    4
Lost:        0
```

## Laptop2 Addressing

Laptop2 received:

```text
IPv4 address:    10.0.0.14
Subnet mask:     255.255.255.240
Default gateway: 10.0.0.1
DNS suffix:      jeremysitlab.com
```

Laptop2 reached the wireless VLAN gateway:

```text
Destination: 10.6.0.1
Sent:        4
Received:    3
Lost:        1
```

Laptop2 successfully reached SRV1:

```text
Destination: 10.5.0.4
Sent:        4
Received:    4
Lost:        0
```

The initial missed gateway replies were followed by successful replies and are consistent with initial ARP resolution or Packet Tracer wireless convergence.

## Addressing Observation

Although the WLAN was configured for VLAN 40 and the `10.6.0.0/24` network, both wireless clients held addresses from the management subnet during verification:

```text
Laptop1: 10.0.0.12/28
Laptop2: 10.0.0.14/28
```

Therefore, this repository does not claim that wireless client placement into VLAN 40 was successfully verified during the final evidence capture.

The verified results are limited to:

- WLAN availability
- WPA2-PSK configuration
- WLC operation
- AP registration
- Wireless client association
- Routed connectivity to the wireless gateway
- Routed connectivity to the internal server

This observation is documented rather than concealed or represented as a successful VLAN 40 DHCP assignment.

## Packet Tracer Behavior

During testing, the WLC HTTPS page temporarily returned a request timeout even though the WLC remained reachable through ICMP.

The interface became available again after:

- Confirming Realtime simulation mode
- Advancing Packet Tracer time
- Waiting for the simulated WLC service to respond
- Reopening the browser

No WLC or switch configuration change was required to restore web access.

## Conclusion

The wireless infrastructure successfully demonstrated:

- Operational WLC management
- An enabled WPA2-PSK WLAN
- A VLAN 40 dynamic interface
- Two registered FlexConnect access points
- Two associated wireless clients
- Reachability to the wireless gateway
- Reachability to the internal server
- WLC management through HTTPS

The final client captures showed management-subnet addresses rather than VLAN 40 addresses. This is explicitly documented as an unresolved verification observation and is not presented as a successfully validated WLAN-to-VLAN assignment.
