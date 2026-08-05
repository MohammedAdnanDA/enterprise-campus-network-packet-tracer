# NAT and Internet Failover Verification

This file contains operational evidence captured from PC1 and R1 after implementing static NAT, pool-based PAT, dual default routes, and Internet-path failover.

## Verification Summary

| Test | Result |
|---|---|
| Primary IPv4 default route | Operational |
| Floating backup default route | Operational |
| Primary Internet connectivity | Successful |
| Backup Internet connectivity | Successful |
| Automatic primary-route recovery | Successful |
| Dynamic PAT | Operational |
| Static NAT for SRV1 | Operational |
| NAT ACL matching | Confirmed |

## Normal Internet Path

During normal operation, R1 used the primary default route through ISP-A.

Command:

```cisco
show ip route 0.0.0.0
```

Output:

```text
Routing entry for 0.0.0.0/0, supernet
Known via "static", distance 1, metric 0, candidate default path
  Routing Descriptor Blocks:
  * 203.0.113.1
      Route metric is 0, traffic share count is 1
```

The administrative distance of `1` confirms that the primary static route was active.

## Primary-Path Connectivity Test

PC1 tested connectivity to the primary ISP next hop.

Command:

```text
ping 203.0.113.1
```

Output:

```text
Pinging 203.0.113.1 with 32 bytes of data:

Reply from 203.0.113.1: bytes=32 time=10ms TTL=252
Reply from 203.0.113.1: bytes=32 time=10ms TTL=252
Reply from 203.0.113.1: bytes=32 time=13ms TTL=252
Reply from 203.0.113.1: bytes=32 time=11ms TTL=252

Ping statistics for 203.0.113.1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
```

This confirms successful end-to-end connectivity from Office A through R1 and the primary Internet path.

## Dynamic PAT Verification

After PC1 generated Internet traffic, R1 displayed the following translations.

Command:

```cisco
show ip nat translations
```

Output:

```text
Pro  Inside global     Inside local       Outside local      Outside global
icmp 203.0.113.201:1   10.1.0.11:1        203.0.113.1:1      203.0.113.1:1
icmp 203.0.113.201:2   10.1.0.11:2        203.0.113.1:2      203.0.113.1:2
icmp 203.0.113.201:3   10.1.0.11:3        203.0.113.1:3      203.0.113.1:3
icmp 203.0.113.201:4   10.1.0.11:4        203.0.113.1:4      203.0.113.1:4
---  203.0.113.113     10.5.0.4           ---                ---
```

The output confirms:

- PC1 inside-local address: `10.1.0.11`
- Assigned inside-global address: `203.0.113.201`
- ICMP identifiers distinguish the individual PAT translations
- SRV1 has a permanent static NAT mapping

## Static NAT Verification

The permanent translation is:

```text
Inside local:  10.5.0.4
Inside global: 203.0.113.113
```

This confirms that SRV1 is statically mapped to the public address `203.0.113.113`.

## NAT Statistics

Command:

```cisco
show ip nat statistics
```

Output:

```text
Total translations: 5 (1 static, 4 dynamic, 4 extended)
Outside Interfaces: GigabitEthernet0/0/0 , GigabitEthernet0/1/0
Inside Interfaces: GigabitEthernet0/0 , GigabitEthernet0/1
Hits: 4  Misses: 83
Expired translations: 0

Dynamic mappings:
-- Inside Source
access-list 2 pool POOL1 refCount 4
 pool POOL1: netmask 255.255.255.248
       start 203.0.113.200 end 203.0.113.207
       type generic, total addresses 8, allocated 1 (12%), misses 0
```

This verifies:

- Both ISP-facing interfaces are configured as NAT outside interfaces.
- Both internal R1 interfaces are configured as NAT inside interfaces.
- The PAT pool is active.
- One public pool address was allocated during testing.
- Static and dynamic translations were present simultaneously.

## PAT Access List

Command:

```cisco
show access-lists 2
```

Output:

```text
Standard IP access list 2
    permit 10.1.0.0 0.0.0.255 (8 match(es))
    permit 10.2.0.0 0.0.0.255
    permit 10.3.0.0 0.0.0.255
    permit 10.4.0.0 0.0.0.255
    permit 10.6.0.0 0.0.0.255
```

The match counter on the Office A PC network confirms that PC1 traffic was selected for translation.

## Primary-Path Failure

The primary ISP-facing interface was administratively disabled:

```cisco
interface GigabitEthernet0/0/0
 shutdown
```

R1 then installed the floating backup route.

Command:

```cisco
show ip route 0.0.0.0
```

Output:

```text
Routing entry for 0.0.0.0/0, supernet
Known via "static", distance 2, metric 0, candidate default path
  Routing Descriptor Blocks:
  * 203.0.113.5
      Route metric is 0, traffic share count is 1
```

The administrative distance of `2` identifies this as the floating backup route.

## Backup-Path Connectivity Test

While the primary interface was disabled, PC1 tested the backup ISP path.

Command:

```text
ping 203.0.113.5
```

Output:

```text
Pinging 203.0.113.5 with 32 bytes of data:

Reply from 203.0.113.5: bytes=32 time<1ms TTL=252
Reply from 203.0.113.5: bytes=32 time=13ms TTL=252
Reply from 203.0.113.5: bytes=32 time<1ms TTL=252
Reply from 203.0.113.5: bytes=32 time=11ms TTL=252

Ping statistics for 203.0.113.5:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
```

This confirms that internal traffic continued through the backup ISP connection after the primary path failed.

## Primary-Path Recovery

The primary interface was restored:

```cisco
interface GigabitEthernet0/0/0
 no shutdown
```

R1 automatically returned to the lower-administrative-distance primary route.

Command:

```cisco
show ip route 0.0.0.0
```

Output:

```text
Routing entry for 0.0.0.0/0, supernet
Known via "static", distance 1, metric 0, candidate default path
  Routing Descriptor Blocks:
  * 203.0.113.1
      Route metric is 0, traffic share count is 1
```

No manual route replacement was required.

## Failover Logic

The route preference is determined by administrative distance:

```text
Primary route:
0.0.0.0/0 via 203.0.113.1
Administrative distance 1

Backup route:
0.0.0.0/0 via 203.0.113.5
Administrative distance 2
```

During normal operation, the primary route is preferred.

When the primary interface becomes unavailable, the route through `203.0.113.1` is removed and the floating route through `203.0.113.5` becomes active.

When the primary interface recovers, the route with administrative distance 1 is preferred again.

## Conclusion

The verification confirms:

- Successful Internet connectivity through the primary ISP.
- Successful dynamic PAT for an internal PC.
- A permanent static NAT mapping for SRV1.
- Correct PAT ACL matching.
- Automatic activation of the floating backup route.
- Successful client connectivity through the backup ISP.
- Automatic restoration of the preferred primary route.
