# HSRP and Spanning Tree Verification

This file contains operational evidence captured from the four distribution switches after implementing HSRPv2 and Rapid PVST+.

## Verification Result

| Office | VLAN | Purpose | HSRP Active | STP Root | Status |
|---|---:|---|---|---|---|
| A | 10 | PCs | DSW-A1 | DSW-A1 | Aligned |
| A | 20 | Phones | DSW-A2 | DSW-A2 | Aligned |
| A | 40 | Wi-Fi | DSW-A2 | DSW-A2 | Aligned |
| A | 99 | Management | DSW-A1 | DSW-A1 | Aligned |
| B | 10 | PCs | DSW-B1 | DSW-B1 | Aligned |
| B | 20 | Phones | DSW-B2 | DSW-B2 | Aligned |
| B | 30 | Servers | DSW-B2 | DSW-B2 | Aligned |
| B | 99 | Management | DSW-B1 | DSW-B1 | Aligned |

The active HSRP gateway and Rapid PVST+ root bridge are aligned for every VLAN.

This ensures that the preferred Layer 2 forwarding path leads directly to the active Layer 3 default gateway.

## Office A HSRP Verification

### DSW-A1

Command:

```cisco
show standby brief
```

Output:

```text
Interface   Grp  Pri P State    Active          Standby         Virtual IP
Vl10        2    105 P Active   local           10.1.0.3        10.1.0.1
Vl20        3    100   Standby  10.2.0.3        local           10.2.0.1
Vl40        4    100   Standby  10.6.0.3        local           10.6.0.1
Vl99        1    105 P Active   local           10.0.0.3        10.0.0.1
```

DSW-A1 is active for VLANs 10 and 99.

### DSW-A2

Command:

```cisco
show standby brief
```

Output:

```text
Interface   Grp  Pri P State    Active          Standby         Virtual IP
Vl10        2    100   Standby  10.1.0.2        local           10.1.0.1
Vl20        3    105 P Active   local           10.2.0.2        10.2.0.1
Vl40        4    105 P Active   local           10.6.0.2        10.6.0.1
Vl99        1    100   Standby  10.0.0.2        local           10.0.0.1
```

DSW-A2 is active for VLANs 20 and 40.

## Office A Spanning Tree Verification

### DSW-A1

VLANs 10 and 99 identify DSW-A1 as the root bridge:

```text
VLAN0010
Spanning tree enabled protocol rstp
Root ID    Priority    10
           Address     000C.CFE7.2765
           This bridge is the root
```

```text
VLAN0099
Spanning tree enabled protocol rstp
Root ID    Priority    99
           Address     000C.CFE7.2765
           This bridge is the root
```

For VLANs 20 and 40, DSW-A1 uses Port-channel1 to reach DSW-A2:

```text
VLAN0020
Root ID    Priority    20
           Address     00D0.BC16.1213
           Cost        3
           Port        29(Port-channel1)
```

```text
VLAN0040
Root ID    Priority    40
           Address     00D0.BC16.1213
           Cost        3
           Port        29(Port-channel1)
```

### DSW-A2

VLANs 20 and 40 identify DSW-A2 as the root bridge:

```text
VLAN0020
Spanning tree enabled protocol rstp
Root ID    Priority    20
           Address     00D0.BC16.1213
           This bridge is the root
```

```text
VLAN0040
Spanning tree enabled protocol rstp
Root ID    Priority    40
           Address     00D0.BC16.1213
           This bridge is the root
```

For VLANs 10 and 99, DSW-A2 uses Port-channel1 to reach DSW-A1:

```text
VLAN0010
Root ID    Priority    10
           Address     000C.CFE7.2765
           Cost        3
           Port        29(Port-channel1)
```

```text
VLAN0099
Root ID    Priority    99
           Address     000C.CFE7.2765
           Cost        3
           Port        29(Port-channel1)
```

## Office B HSRP Verification

### DSW-B1

Command:

```cisco
show standby brief
```

Output:

```text
Interface   Grp  Pri P State    Active          Standby         Virtual IP
Vl10        2    105 P Active   local           10.3.0.3        10.3.0.1
Vl20        3    100   Standby  10.4.0.3        local           10.4.0.1
Vl30        4    100   Standby  10.5.0.3        local           10.5.0.1
Vl99        1    105 P Active   local           10.0.0.19       10.0.0.17
```

DSW-B1 is active for VLANs 10 and 99.

### DSW-B2

Command:

```cisco
show standby brief
```

Output:

```text
Interface   Grp  Pri P State    Active          Standby         Virtual IP
Vl10        2    100   Standby  10.3.0.2        local           10.3.0.1
Vl20        3    105 P Active   local           10.4.0.2        10.4.0.1
Vl30        4    105 P Active   local           10.5.0.2        10.5.0.1
Vl99        1    100   Standby  10.0.0.18       local           10.0.0.17
```

DSW-B2 is active for VLANs 20 and 30.

## Office B Spanning Tree Verification

### DSW-B1

VLANs 10 and 99 identify DSW-B1 as the root bridge:

```text
VLAN0010
Spanning tree enabled protocol rstp
Root ID    Priority    10
           Address     0001.635B.4C6A
           This bridge is the root
```

```text
VLAN0099
Spanning tree enabled protocol rstp
Root ID    Priority    99
           Address     0001.635B.4C6A
           This bridge is the root
```

For VLANs 20 and 30, DSW-B1 uses Port-channel1 to reach DSW-B2:

```text
VLAN0020
Root ID    Priority    20
           Address     0050.0F7B.3C01
           Cost        3
           Port        29(Port-channel1)
```

```text
VLAN0030
Root ID    Priority    30
           Address     0050.0F7B.3C01
           Cost        3
           Port        29(Port-channel1)
```

### DSW-B2

VLANs 20 and 30 identify DSW-B2 as the root bridge:

```text
VLAN0020
Spanning tree enabled protocol rstp
Root ID    Priority    20
           Address     0050.0F7B.3C01
           This bridge is the root
```

```text
VLAN0030
Spanning tree enabled protocol rstp
Root ID    Priority    30
           Address     0050.0F7B.3C01
           This bridge is the root
```

For VLANs 10 and 99, DSW-B2 uses Port-channel1 to reach DSW-B1:

```text
VLAN0010
Root ID    Priority    10
           Address     0001.635B.4C6A
           Cost        3
           Port        29(Port-channel1)
```

```text
VLAN0099
Root ID    Priority    99
           Address     0001.635B.4C6A
           Cost        3
           Port        29(Port-channel1)
```

## Conclusion

The verification confirms:

- HSRPv2 is operational across all eight VLAN gateway groups.
- Each HSRP group has an active and standby distribution switch.
- Rapid PVST+ is operating on the distribution layer.
- STP root-bridge placement matches the active HSRP gateway.
- The two distribution switches share the active gateway workload.
- Port-channel1 provides the preferred inter-distribution path for VLANs rooted on the opposite switch.
