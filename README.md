# MPLS L3VPN with Traffic Engineering (GNS3 lab)

![Cisco IOS](https://img.shields.io/badge/Cisco-IOS%2015-1BA0D7?logo=cisco&logoColor=white)
![GNS3](https://img.shields.io/badge/GNS3-lab-2C3E50)
![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)

A service-provider lab in GNS3: two customers with **overlapping address plans** share one MPLS backbone, kept separate with **VRFs** and **MP-BGP VPNv4**, labels distributed with **LDP**, and one path steered with an **RSVP-TE** tunnel on an explicit route.

> Personal lab project. The configurations in this repository were **rebuilt in October 2026** as a clean reference lab (the original GNS3 project files were not kept). The verification commands in [docs/verification.md](docs/verification.md) describe what to check; no captured outputs are included.

## Topology

```text
   CE-A1 (AS 65001)                                         CE-A2 (AS 65001)
   192.168.1.0/24 ─┐                                     ┌─ 192.168.2.0/24
                   │            ┌──── P1 ────┐           │
                  PE1 ──────────┤  2.2.2.2   ├────────── PE2
                1.1.1.1         │     │      │         4.4.4.4
                   │            └──── P2 ────┘           │
   192.168.1.0/24 ─┘              3.3.3.3                └─ 192.168.2.0/24
   CE-B1 (AS 65002)        provider AS 65000              CE-B2 (AS 65002)

   RSVP-TE Tunnel1 (PE1 → PE2): explicit path PE1 → P2 → PE2
```

| Link | Subnet | | Loopback0 | Address |
|---|---|---|---|---|
| PE1 – P1 | 10.0.12.0/30 | | PE1 | 1.1.1.1 |
| PE1 – P2 | 10.0.13.0/30 | | P1 | 2.2.2.2 |
| P1 – P2 | 10.0.23.0/30 | | P2 | 3.3.3.3 |
| P1 – PE2 | 10.0.24.0/30 | | PE2 | 4.4.4.4 |
| P2 – PE2 | 10.0.34.0/30 | | | |

| Customer | VRF | RD / RT | CE ↔ PE | Sites |
|---|---|---|---|---|
| A | `CUST_A` | 65000:1 | eBGP, AS 65001, `as-override` | 192.168.1.0/24 ↔ 192.168.2.0/24 |
| B | `CUST_B` | 65000:2 | eBGP, AS 65002, `as-override` | 192.168.1.0/24 ↔ 192.168.2.0/24 (same as A) |

## How the pieces fit

1. **OSPF area 0** gives every core router a route to every loopback. It also carries the TE extensions (`mpls traffic-eng area 0`).
2. **LDP** builds label-switched paths between the loopbacks, so P1 and P2 forward by label and never need customer routes.
3. **VRFs** on the PEs keep each customer's routes in a separate table, which is why both customers can use 192.168.1.0/24.
4. **MP-BGP VPNv4** between PE1 and PE2 (loopback to loopback) carries the customer routes with their route distinguisher and route target.
5. **RSVP-TE** `Tunnel1` reserves 10 Mb/s along `PE1 → P2 → PE2`, with a dynamic path as fallback, and `autoroute announce` puts traffic to PE2 into the tunnel.

## Files

- `configs/*.cfg`: full configurations for PE1, PE2, P1, P2 and the four CE routers (Cisco IOSv 15.x or c7200 images)
- `docs/verification.md`: step-by-step checks (OSPF, LDP, VPNv4, VRF tables, VRF ping/traceroute, TE tunnel)

## Skills

MPLS · LDP · VRF · MP-BGP / VPNv4 · route distinguishers and route targets · eBGP CE-PE with `as-override` · OSPF-TE · RSVP-TE explicit paths · GNS3.

## License

MIT. See [LICENSE](LICENSE).
