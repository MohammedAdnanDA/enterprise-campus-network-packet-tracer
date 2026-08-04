# OSPF Verification

This file contains operational evidence captured from R1 after implementing the OSPF Area 0 routing design.

## Verification Result

| Check | Result |
|---|---|
| OSPF process | Operational |
| R1 OSPF neighbors | 2 |
| CSW1 adjacency | FULL |
| CSW2 adjacency | FULL |
| OSPF network type | Point-to-point |
| Office A routes learned | Yes |
| Office B routes learned | Yes |
| Equal-cost routing paths | Present |

R1 formed full OSPF adjacencies with both core switches:

- CSW1 router ID: `10.0.0.77`
- CSW2 router ID: `10.0.0.78`

Most internal networks have two equal-cost routes, one through each core switch. This confirms that the redundant Layer 3 paths are being used by OSPF.

## OSPF Neighbor Output

Command:

```cisco
show ip ospf neighbor
```

Output:

```text
Neighbor ID     Pri   State           Dead Time   Address         Interface
10.0.0.77         0   FULL/  -        00:00:35    10.0.0.34       GigabitEthernet0/0
10.0.0.78         0   FULL/  -        00:00:35    10.0.0.38       GigabitEthernet0/1
```

The `FULL/-` state confirms that R1 has completed the OSPF database exchange with both core switches.

The priority value of `0` and the absence of a DR or BDR role are consistent with the point-to-point OSPF network type.

## OSPF Routing Table

Command:

```cisco
show ip route ospf
```

Output:

```text
     10.0.0.0/8 is variably subnetted, 28 subnets, 4 masks
O       10.0.0.0 [110/3] via 10.0.0.34, 00:50:01, GigabitEthernet0/0
                 [110/3] via 10.0.0.38, 00:50:01, GigabitEthernet0/1
O       10.0.0.16 [110/3] via 10.0.0.34, 00:45:52, GigabitEthernet0/0
                  [110/3] via 10.0.0.38, 00:45:52, GigabitEthernet0/1
O       10.0.0.40 [110/2] via 10.0.0.34, 02:03:50, GigabitEthernet0/0
                  [110/2] via 10.0.0.38, 02:03:50, GigabitEthernet0/1
O       10.0.0.44 [110/2] via 10.0.0.34, 02:04:05, GigabitEthernet0/0
O       10.0.0.48 [110/2] via 10.0.0.34, 02:04:05, GigabitEthernet0/0
O       10.0.0.52 [110/2] via 10.0.0.34, 02:04:05, GigabitEthernet0/0
O       10.0.0.56 [110/2] via 10.0.0.34, 02:04:05, GigabitEthernet0/0
O       10.0.0.60 [110/2] via 10.0.0.38, 02:04:05, GigabitEthernet0/1
O       10.0.0.64 [110/2] via 10.0.0.38, 02:04:05, GigabitEthernet0/1
O       10.0.0.68 [110/2] via 10.0.0.38, 02:04:05, GigabitEthernet0/1
O       10.0.0.72 [110/2] via 10.0.0.38, 02:04:05, GigabitEthernet0/1
O       10.0.0.77 [110/2] via 10.0.0.34, 02:04:05, GigabitEthernet0/0
O       10.0.0.78 [110/2] via 10.0.0.38, 02:04:05, GigabitEthernet0/1
O       10.0.0.79 [110/3] via 10.0.0.34, 02:04:05, GigabitEthernet0/0
                  [110/3] via 10.0.0.38, 02:04:05, GigabitEthernet0/1
O       10.0.0.80 [110/3] via 10.0.0.34, 02:04:05, GigabitEthernet0/0
                  [110/3] via 10.0.0.38, 02:04:05, GigabitEthernet0/1
O       10.0.0.81 [110/3] via 10.0.0.34, 02:04:05, GigabitEthernet0/0
                  [110/3] via 10.0.0.38, 02:04:05, GigabitEthernet0/1
O       10.0.0.82 [110/3] via 10.0.0.34, 02:04:05, GigabitEthernet0/0
                  [110/3] via 10.0.0.38, 02:04:05, GigabitEthernet0/1
O       10.1.0.0 [110/3] via 10.0.0.34, 02:04:05, GigabitEthernet0/0
                 [110/3] via 10.0.0.38, 02:04:05, GigabitEthernet0/1
O       10.2.0.0 [110/3] via 10.0.0.34, 02:04:05, GigabitEthernet0/0
                 [110/3] via 10.0.0.38, 02:04:05, GigabitEthernet0/1
O       10.3.0.0 [110/3] via 10.0.0.34, 02:04:05, GigabitEthernet0/0
                 [110/3] via 10.0.0.38, 02:04:05, GigabitEthernet0/1
O       10.4.0.0 [110/3] via 10.0.0.34, 00:46:30, GigabitEthernet0/0
                 [110/3] via 10.0.0.38, 00:46:30, GigabitEthernet0/1
O       10.5.0.0 [110/3] via 10.0.0.34, 02:04:05, GigabitEthernet0/0
                 [110/3] via 10.0.0.38, 02:04:05, GigabitEthernet0/1
O       10.6.0.0 [110/3] via 10.0.0.34, 02:04:05, GigabitEthernet0/0
                 [110/3] via 10.0.0.38, 02:04:05, GigabitEthernet0/1
```

## Route Interpretation

The routing table confirms that R1 learned the following user and service networks through OSPF:

| Network | Purpose |
|---|---|
| `10.0.0.0/28` | Office A management |
| `10.0.0.16/28` | Office B management |
| `10.1.0.0/24` | Office A PCs |
| `10.2.0.0/24` | Office A phones |
| `10.3.0.0/24` | Office B PCs |
| `10.4.0.0/24` | Office B phones |
| `10.5.0.0/24` | Office B servers |
| `10.6.0.0/24` | Wireless clients |

For example:

```text
O 10.1.0.0 [110/3] via 10.0.0.34, GigabitEthernet0/0
             [110/3] via 10.0.0.38, GigabitEthernet0/1
```

Both paths have the same administrative distance and metric:

```text
[110/3]
```

R1 can therefore use equal-cost multipath routing through both core switches.

## Conclusion

The verification confirms:

- Stable OSPF adjacencies between R1 and both core switches
- Successful advertisement of routed links, loopbacks, and VLAN networks
- Equal-cost redundant paths through CSW1 and CSW2
- Full Layer 3 reachability information for both office locations
