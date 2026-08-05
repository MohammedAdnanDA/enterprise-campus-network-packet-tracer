# Troubleshooting Log

This document records the significant issues encountered while implementing and verifying the enterprise campus network in Cisco Packet Tracer.

The original topology and activity requirements were provided by Jeremy’s IT Lab. The configuration, testing, troubleshooting, verification, and documentation were completed as part of this project.

## Summary

| Issue | Area | Resolution or Final Status |
|---|---|---|
| PC1 disconnected from IP phone | Physical connectivity | Cable connection restored |
| WLAN administratively disabled | Wireless | WLAN enabled |
| Wireless client received APIPA address | DHCP/Wireless | Client association and DHCP operation restored |
| DAI enabled on incorrect VLAN | Layer 2 security | VLAN configuration corrected |
| Incorrect DHCP domain-name entry | DHCP | Corrected to `jeremysitlab.com` |
| Duplicate IPv6 address on CSW1 | IPv6 | Duplicate address removed |
| DSW-B1 VLAN 20 address conflicted with HSRP VIP | HSRP | Physical SVI address corrected |
| OSPF failover behavior inconsistent in Packet Tracer | Routing | Documented as a simulator limitation |
| Extended ACL would not remain attached to SVI | Access control | Documented as a device-image limitation |
| WLC HTTPS page timed out temporarily | Wireless management | Service recovered after advancing simulation time |
| Wireless clients retained management-subnet addresses | Wireless/DHCP | Documented as an unresolved verification observation |

---

## 1. PC1-to-IP-Phone Connection Failure

### Symptom

PC1 did not have normal network connectivity even though the access-switch configuration appeared correct.

### Investigation

The access port, VLAN configuration, DHCP service, and upstream switching were reviewed.

The problem was ultimately found at the physical layer: the PC was not correctly connected through the IP phone.

### Resolution

The Ethernet connection between PC1 and the IP phone was restored.

### Result

PC1 successfully obtained:

```text
IPv4 address: 10.1.0.11
Default gateway: 10.1.0.1
```

The incident reinforced the need to verify physical connectivity before troubleshooting higher layers.

---

## 2. WLAN Administratively Disabled

### Symptom

Wireless clients could not associate with the configured SSID.

### Investigation

The WLC configuration was inspected. The WLAN existed, but its administrative status prevented client access.

### Resolution

The WLAN was enabled through the Wireless LAN Controller.

### Result

The WLC later showed:

```text
WLAN ID: 1
Profile name: Wi-Fi
SSID: Wi-Fi
Status: Enabled
Security: WPA2-PSK
```

Two wireless clients were subsequently reported by the controller.

---

## 3. Wireless Client APIPA Address

### Symptom

A wireless laptop did not receive a valid enterprise address and instead displayed an automatically assigned address.

An APIPA address normally belongs to:

```text
169.254.0.0/16
```

This indicates that the client did not receive a DHCP response.

### Investigation

The following components were reviewed:

- Wireless association
- WLAN administrative status
- Lightweight AP registration
- DHCP server configuration
- DHCP relay configuration
- VLAN availability
- Access-point switchport configuration

### Resolution

The WLAN and client association were restored, allowing the client to communicate with the network.

### Result

The wireless clients later received DHCP-assigned addresses and successfully reached internal network resources.

The final captured addresses were from the management subnet rather than the intended wireless subnet. That separate observation is documented later in this file.

---

## 4. Incorrect DAI VLAN Configuration

### Symptom

Dynamic ARP Inspection was not operating on the intended set of VLANs.

### Investigation

The DAI configuration was compared with the VLANs present on the access switch.

### Cause

A VLAN number had been entered incorrectly in the ARP inspection configuration.

### Resolution

DAI was configured on the correct Office A VLANs:

```text
10,20,40,99
```

### Verification

The following command confirmed active inspection:

```cisco
show ip arp inspection
```

Relevant result:

```text
VLAN 10: Enabled and Active
VLAN 20: Enabled and Active
VLAN 40: Enabled and Active
VLAN 99: Enabled and Active
```

The following validation checks were also enabled:

- Source MAC validation
- Destination MAC validation
- IP address validation

---

## 5. Incorrect DHCP Domain Name

### Symptom

The Office A management DHCP pool contained an incorrect domain-name value.

### Cause

An unintended value was entered while configuring the DHCP pool.

### Resolution

The DHCP domain name was corrected to:

```text
jeremysitlab.com
```

### Result

DHCP clients later displayed:

```text
Connection-specific DNS suffix: jeremysitlab.com
```

---

## 6. Duplicate IPv6 Address on CSW1

### Symptom

CSW1 contained an unintended duplicate IPv6 address configuration.

### Risk

Duplicate addressing can cause:

- Neighbor Discovery inconsistencies
- Incorrect reachability
- Duplicate Address Detection failures
- Unstable IPv6 forwarding

### Resolution

The duplicate IPv6 address was removed, and the intended EUI-64-based configuration was retained.

### Result

CSW1 retained a unique IPv6 interface address consistent with the lab addressing design.

---

## 7. DSW-B1 VLAN 20 Address Conflict

### Symptom

DSW-B1 used the HSRP virtual IP address as its own physical SVI address on VLAN 20.

### Incorrect Condition

The device address and HSRP virtual address must be different.

Using the same address for both can prevent correct HSRP operation and create an address conflict.

### Resolution

DSW-B1 VLAN 20 was assigned its proper physical address:

```text
DSW-B1 physical SVI: 10.4.0.2
HSRP virtual IP:     10.4.0.1
```

### Result

Office B HSRP verification showed:

- DSW-B1 active for VLANs 10 and 99
- DSW-B2 active for VLANs 20 and 30
- Corresponding standby roles on the peer switch

---

## 8. OSPF Failover Behavior in Packet Tracer

### Symptom

Some OSPF failover tests did not always converge or update exactly as expected after a simulated link failure.

### Investigation

Normal-state OSPF operation was verified first.

R1 formed full adjacencies with both core switches:

```text
CSW1 router ID: 10.0.0.77
CSW2 router ID: 10.0.0.78
State: FULL
```

R1 also learned internal routes through both core switches using equal-cost paths.

### Observation

Packet Tracer does not reproduce every IOS routing protocol behavior with full production-device fidelity. Some topology changes may require additional simulation time, manual traffic generation, or interface cycling before the displayed state updates.

### Final Status

Normal OSPF adjacency and route-learning operation were verified.

The repository does not claim production-grade convergence measurements from Packet Tracer.

---

## 9. Extended ACL Not Retained on VLAN 10 SVI

### Intended Policy

The following extended ACL was created:

```cisco
ip access-list extended OfficeA_to_OfficeB
 permit icmp 10.1.0.0 0.0.0.255 10.3.0.0 0.0.0.255
 deny ip 10.1.0.0 0.0.0.255 10.3.0.0 0.0.0.255
 permit ip any any
```

Its intended behavior was:

- Permit ICMP from Office A PCs to Office B PCs
- Deny other traffic between those PC networks
- Permit unrelated traffic

### Symptom

The following command was accepted without an error:

```cisco
interface vlan 10
 ip access-group OfficeA_to_OfficeB in
```

However, verification showed:

```text
Inbound access list is not set
```

The command also did not appear under `interface Vlan10` in the running configuration.

### Final Status

The ACL definition exists, but its attachment to the SVI could not be retained on the Packet Tracer multilayer-switch image.

The repository therefore documents the ACL as defined but does not claim that the inter-office filtering policy was operationally enforced.

---

## 10. Temporary WLC HTTPS Timeout

### Symptom

The WLC management page at:

```text
https://10.0.0.7
```

temporarily displayed a request timeout.

### Investigation

WLC management reachability was tested from PC1:

```text
Packets sent:     4
Packets received: 4
Packet loss:      0%
```

This proved that the Layer 3 path and WLC management address were operational.

### Resolution

The following actions restored the simulated HTTPS service:

1. Confirmed that Packet Tracer was in Realtime mode
2. Advanced simulation time
3. Waited for the WLC service to respond
4. Closed and reopened the simulated web browser

### Result

The WLC HTTPS interface became accessible again without changing WLC or switch configuration.

---

## 11. Wireless Client VLAN Addressing Observation

### Intended Design

The WLAN was mapped to the WLC dynamic interface named `Wi-Fi`:

```text
VLAN identifier:     40
Interface address:   10.6.0.4/24
Default gateway:     10.6.0.1
Primary DHCP server: 10.0.0.76
```

The access points operated in FlexConnect mode.

The AP-facing switchport allowed:

```text
Native VLAN: 99
Allowed VLANs: 40,99
```

### Captured Client Addresses

During final verification, the laptops held:

```text
Laptop1: 10.0.0.12/28
Laptop2: 10.0.0.14/28
```

These addresses belong to the management subnet rather than the intended wireless subnet:

```text
Intended wireless subnet: 10.6.0.0/24
```

### What Was Successfully Verified

Despite the addressing observation:

- The WLAN was enabled
- WPA2-PSK was configured
- Both lightweight APs were registered
- Two clients were associated
- Both laptops reached `10.6.0.1`
- Both laptops reached SRV1 at `10.5.0.4`

### Final Status

The VLAN 40 client DHCP assignment was not confirmed in the final capture.

No additional network configuration was changed solely to force a different verification result. The discrepancy is recorded transparently rather than represented as a successful VLAN 40 assignment.

---

## 12. NAT and Internet Failover Testing

### Verification Process

Internet connectivity was tested from PC1 through R1.

The primary route used:

```text
Next hop: 203.0.113.1
Administrative distance: 1
```

PAT translated PC1:

```text
Inside local:  10.1.0.11
Inside global: 203.0.113.201
```

### Failover Test

The primary Internet-facing interface was shut down.

R1 selected the floating backup route:

```text
Next hop: 203.0.113.5
Administrative distance: 2
```

PC1 successfully reached the backup ISP address.

After restoring the primary interface, R1 returned to the preferred route through `203.0.113.1`.

### Result

Static-route failover and normal PAT operation were verified.

A NAT translation-table snapshot was not captured specifically during backup-path operation, so the repository does not claim direct NAT table evidence from that exact stage.

---

## Troubleshooting Method Used

The general troubleshooting sequence followed throughout the project was:

1. Verify physical links and device power
2. Confirm interface status
3. Check VLAN membership and trunk permissions
4. Verify addressing and default gateways
5. Inspect HSRP and spanning-tree roles
6. Check OSPF adjacencies and routing tables
7. Verify DHCP pools, bindings, and relay addresses
8. Inspect DHCP Snooping and DAI trust boundaries
9. Verify ACL placement, direction, and persistence
10. Test connectivity from source to destination
11. Distinguish configuration errors from Packet Tracer limitations
12. Record unresolved observations without overstating results

## Conclusion

The troubleshooting process identified and resolved multiple physical, switching, addressing, redundancy, security, and wireless issues.

It also identified several Packet Tracer-specific limitations or inconsistent behaviors. Those cases are documented explicitly so the repository distinguishes between:

- Successfully verified operation
- Correctly configured but incompletely verified features
- Simulator-specific limitations
- Unresolved observations

This provides a more accurate engineering record than presenting every configured feature as fully successful.
