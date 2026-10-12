# Verification checklist

Run the commands in this order; each step depends on the previous one.

| # | Where | Command | What you should see |
|---|---|---|---|
| 1 | every core router | `show ip ospf neighbor` | all core neighbours in `FULL` state |
| 2 | PE1 | `show ip route 4.4.4.4` | PE2's loopback learned via OSPF |
| 3 | every core router | `show mpls ldp neighbor` | an LDP session with each directly connected core router |
| 4 | PE1 | `show mpls forwarding-table 4.4.4.4 32` | an outgoing label towards PE2's loopback |
| 5 | PE1 | `show bgp vpnv4 unicast all summary` | PE2 (4.4.4.4) established, prefixes received |
| 6 | PE1 | `show ip route vrf CUST_A` / `vrf CUST_B` | each VRF has its own 192.168.2.0/24 route, learned from 4.4.4.4 |
| 7 | PE1 | `show bgp vpnv4 unicast rd 65000:1 192.168.2.0` | route target 65000:1 and a VPN label |
| 8 | CE-A1 | `ping 192.168.2.1 source 192.168.1.1` | replies from CE-A2 only |
| 9 | CE-B1 | `ping 192.168.2.1 source 192.168.1.1` | replies from CE-B2, isolated from customer A despite identical addresses |
| 10 | PE1 | `traceroute vrf CUST_A 192.168.2.1` | MPLS labels shown on the core hops |
| 11 | PE1 | `show mpls traffic-eng tunnels brief` | `Tunnel1` up/up, path option 1 (explicit) in use |
| 12 | PE1 | `show mpls traffic-eng tunnels Tunnel1` | explicit route 10.0.13.2 → 10.0.34.2, 10 000 kb/s reserved |
| 13 | P2 | `show ip rsvp reservation` | the 10 Mb/s reservation for the tunnel |

**Failover test:** shut down `GigabitEthernet0/1` on PE1 (link to P2). The tunnel should come back up on path option 2 (dynamic, via P1); step 11 shows the new path, and the VRF pings keep working.
