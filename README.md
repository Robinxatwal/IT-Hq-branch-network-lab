# HQ + Branch Office Network Design

A three-site enterprise network built and configured from scratch in Cisco Packet Tracer — designed, addressed, and tested end-to-end rather than following a pre-built lab scenario.

## Scenario

A small company ("TechRetail Co.") with one headquarters and two branch offices needed:

- Logical separation between departments at HQ (Sales, IT/Server, Guest)
- A default-gateway-per-department setup that scales past a single flat network
- Reliable routing between HQ and both branches over WAN links
- Automatic IP addressing for every device
- A basic security boundary keeping guest traffic away from internal servers

## Topology

See `topology-diagram.png` for the full layout. In short:

- **HQ**: a Layer-3 core switch handles inter-VLAN routing for three VLANs (Sales, IT/Server, Guest), with a router (`R-HQ`) providing the WAN uplinks out to both branches.
- **Branch 1 / Branch 2**: each has its own router and access switch, with a single staff VLAN.
- All four Layer-3 devices (the HQ core switch, R-HQ, and both branch routers) run OSPF area 0.

## Design decisions

- **L3 switching at HQ instead of router-on-a-stick.** Inter-VLAN routing happens directly on the core switch via SVIs, which is closer to how this would actually be built in a real small-business network, and keeps the edge router free to focus purely on WAN connectivity.
- **OSPF over static routing or EIGRP.** OSPF is vendor-neutral and the protocol most commonly asked about in interviews, so it made more sense as the primary routing protocol for a portfolio piece than a Cisco-only option.
- **A Guest VLAN ACL.** Guest traffic is explicitly blocked from reaching the IT/Server subnet, while everything else (including internet-bound traffic) stays open — a deliberate, explainable security decision rather than a flat, unsegmented network.

## Addressing

See `ip-addressing-plan.md` for the full subnet table.

## Configs

The `configs/` folder holds the full running-config export from each device:

- `R-HQ.txt`
- `SW-HQ-CORE.txt`
- `R-BR1.txt`
- `SW-BR1.txt`
- `R-BR2.txt`
- `SW-BR2.txt`

## Verification

Tested and confirmed working:
- Full OSPF adjacency (state FULL) across all four routing devices
- End-to-end ping connectivity between every VLAN and both branch sites
- DHCP correctly scoping addresses per VLAN/subnet
- The Guest VLAN ACL: blocked from the IT/Server subnet, unaffected everywhere else

Screenshots of `show ip ospf neighbor`, `show ip route`, DHCP bindings, and the ACL test are in `screenshots/`.

## What I'd add next

- NAT and a simulated ISP uplink for real internet-bound traffic
- A second WAN path with a floating static route for redundancy
- EIGRP on a secondary link as a routing-protocol comparison

## Troubleshooting notes

See `troubleshooting-writeup.md` for a real issue hit and resolved during the build — not everything worked on the first try, and that process is part of the point of this project.
