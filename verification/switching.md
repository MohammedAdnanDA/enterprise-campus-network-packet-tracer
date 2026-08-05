# Switching and EtherChannel Verification

This file contains operational evidence captured from the core and distribution switches after implementing the Layer 2 and Layer 3 EtherChannels.

## Verification Summary

| Location | Devices | Port Channel | Type | Protocol | Members | Status |
|---|---|---|---|---|---|---|
| Core | CSW1–CSW2 | Port-channel1 | Layer 3 | PAgP | G1/0/2 and G1/0/3 | Operational |
| Office A Distribution | DSW-A1–DSW-A2 | Port-channel1 | Layer 2 trunk | PAgP | G1/0/4 and G1/0/5 | Operational |
| Office B Distribution | DSW-B1–DSW-B2 | Port-channel1 | Layer 2 trunk | LACP | G1/0/4 and G1/0/5 | Operational |

The verification confirms:

- All three Port-channels are operational.
- Every physical member interface is successfully bundled.
- The core EtherChannel operates as a routed Layer 3 link.
- The distribution EtherChannels operate as Layer 2 trunks.
- Office A uses PAgP.
- Office B uses LACP.
- No interface errors were recorded in the captured output.

## EtherChannel Flag Interpretation

The following flags appear in the verification output:

| Flag | Meaning |
|---|---|
| `R` | Layer 3 Port-channel |
| `S` | Layer 2 Port-channel |
| `U` | Port-channel is in use |
| `P` | Physical interface is bundled in the Port-channel |

Therefore:

```text
Po1(RU)
```

means the Port-channel is a routed Layer 3 interface and is in use.

```text
Po1(SU)
```

means the Port-channel is a Layer 2 interface and is in use.

## Core Layer EtherChannel

CSW1 and CSW2 are connected through a Layer 3 PAgP EtherChannel.

### CSW1

Command:

```cisco
show etherchannel summary
```

Relevant output:

```text
Group  Port-channel  Protocol    Ports
------+-------------+-----------+--------------------------------
1      Po1(RU)       PAgP        Gig1/0/2(P) Gig1/0/3(P)
```

This confirms:

- Port-channel1 is a Layer 3 EtherChannel.
- The channel is operational and in use.
- GigabitEthernet1/0/2 is bundled.
- GigabitEthernet1/0/3 is bundled.
- PAgP formed the channel.

Command:

```cisco
show interfaces port-channel 1
```

Relevant output:

```text
Port-channel1 is up, line protocol is up (connected)
Internet address is 10.0.0.41/30
BW 2000000 Kbit
Members in this channel: Gig1/0/2 Gig1/0/3
0 input errors, 0 CRC, 0 frame, 0 overrun, 0 ignored
0 output errors, 0 collisions
```

The logical bandwidth of `2000000 Kbit` represents the combined bandwidth of the two bundled Gigabit Ethernet links.

### CSW2

Command:

```cisco
show etherchannel summary
```

Relevant output:

```text
Group  Port-channel  Protocol    Ports
------+-------------+-----------+--------------------------------
1      Po1(RU)       PAgP        Gig1/0/2(P) Gig1/0/3(P)
```

Command:

```cisco
show interfaces port-channel 1
```

Relevant output:

```text
Port-channel1 is up, line protocol is up (connected)
Internet address is 10.0.0.42/30
BW 2000000 Kbit
Members in this channel: Gig1/0/2 Gig1/0/3
0 input errors, 0 CRC, 0 frame, 0 overrun, 0 ignored
0 output errors, 0 collisions
```

Both ends of the core EtherChannel report matching operational states and member interfaces.

## Core OSPF Connectivity

The core Port-channel also carries OSPF traffic between CSW1 and CSW2.

### CSW1 OSPF Neighbors

Command:

```cisco
show ip ospf neighbor
```

Relevant output:

```text
Neighbor ID     State       Address         Interface
10.0.0.76       FULL/-      10.0.0.33       GigabitEthernet1/0/1
10.0.0.78       FULL/DR     10.0.0.42       Port-channel1
10.0.0.80       FULL/-      10.0.0.50       GigabitEthernet1/1/2
10.0.0.79       FULL/-      10.0.0.46       GigabitEthernet1/1/1
10.0.0.81       FULL/-      10.0.0.54       GigabitEthernet1/1/3
10.0.0.82       FULL/-      10.0.0.58       GigabitEthernet1/1/4
```

### CSW2 OSPF Neighbors

Relevant output:

```text
Neighbor ID     State       Address         Interface
10.0.0.76       FULL/-      10.0.0.37       GigabitEthernet1/0/1
10.0.0.79       FULL/-      10.0.0.62       GigabitEthernet1/1/1
10.0.0.81       FULL/-      10.0.0.70       GigabitEthernet1/1/3
10.0.0.80       FULL/-      10.0.0.66       GigabitEthernet1/1/2
10.0.0.82       FULL/-      10.0.0.74       GigabitEthernet1/1/4
10.0.0.77       FULL/BDR    10.0.0.41       Port-channel1
```

Both core switches have six full OSPF adjacencies:

- R1
- The other core switch
- DSW-A1
- DSW-A2
- DSW-B1
- DSW-B2

The DR and BDR states on Port-channel1 result from the default broadcast OSPF network type used on that interface.

## Office A Distribution EtherChannel

DSW-A1 and DSW-A2 use a Layer 2 PAgP EtherChannel.

### DSW-A1

Command:

```cisco
show etherchannel summary
```

Output:

```text
Group  Port-channel  Protocol    Ports
------+-------------+-----------+--------------------------------
1      Po1(SU)       PAgP        Gig1/0/4(P) Gig1/0/5(P)
```

This confirms:

- Port-channel1 operates at Layer 2.
- The channel is in use.
- Both physical links are bundled.
- PAgP formed the channel.

Command:

```cisco
show interfaces port-channel 1
```

Relevant output:

```text
Port-channel1 is up, line protocol is up (connected)
BW 2000000 Kbit
Members in this channel: Gig1/0/4 Gig1/0/5
0 input errors, 0 CRC, 0 frame, 0 overrun, 0 ignored
0 output errors, 0 collisions
```

This Port-channel carries the Office A VLAN trunks between DSW-A1 and DSW-A2.

## Office B Distribution EtherChannel

DSW-B1 and DSW-B2 use a Layer 2 LACP EtherChannel.

### DSW-B1

Command:

```cisco
show etherchannel summary
```

Output:

```text
Group  Port-channel  Protocol    Ports
------+-------------+-----------+--------------------------------
1      Po1(SU)       LACP        Gig1/0/4(P) Gig1/0/5(P)
```

This confirms:

- Port-channel1 operates at Layer 2.
- The channel is in use.
- Both physical links are bundled.
- LACP formed the channel.

Command:

```cisco
show interfaces port-channel 1
```

Relevant output:

```text
Port-channel1 is up, line protocol is up (connected)
BW 2000000 Kbit
Members in this channel: Gig1/0/5 Gig1/0/4
0 input errors, 0 CRC, 0 frame, 0 overrun, 0 ignored
0 output errors, 0 collisions
```

This Port-channel carries the Office B VLAN trunks between DSW-B1 and DSW-B2.

## Protocol Comparison

| Feature | PAgP | LACP |
|---|---|---|
| Standard | Cisco proprietary | IEEE standard |
| Core usage | CSW1–CSW2 | Not used |
| Office A usage | DSW-A1–DSW-A2 | Not used |
| Office B usage | Not used | DSW-B1–DSW-B2 |
| Operational result | Successful | Successful |

The project demonstrates configuration and verification of both major EtherChannel negotiation protocols.

## Conclusion

The switching verification confirms:

- The routed core EtherChannel is operational.
- The Office A Layer 2 EtherChannel is operational.
- The Office B Layer 2 EtherChannel is operational.
- All physical member links are bundled successfully.
- The logical Port-channel interfaces are up/up.
- The expected PAgP and LACP protocols are being used.
- OSPF successfully operates across the core Port-channel.
- No packet, CRC, collision, or interface errors were observed in the captured evidence.
