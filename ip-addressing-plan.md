# IP Addressing Plan

## LAN subnets

| Site | VLAN | Purpose | Subnet | Gateway |
|---|---|---|---|---|
| HQ | 10 | Sales | 192.168.10.0/24 | 192.168.10.1 |
| HQ | 20 | IT / Server | 192.168.20.0/24 | 192.168.20.1 |
| HQ | 30 | Guest | 192.168.30.0/24 | 192.168.30.1 |
| Branch 1 | 40 | Staff | 192.168.40.0/24 | 192.168.40.1 |
| Branch 2 | 50 | Staff | 192.168.50.0/24 | 192.168.50.1 |

## Transit / WAN links (point-to-point, /30)

| Link | Subnet | Side A | Side B |
|---|---|---|---|
| SW-HQ-CORE &harr; R-HQ | 10.0.0.0/30 | SW-HQ-CORE: 10.0.0.1 | R-HQ: 10.0.0.2 |
| R-HQ &harr; R-BR1 | 10.0.12.0/30 | R-HQ: 10.0.12.1 | R-BR1: 10.0.12.2 |
| R-HQ &harr; R-BR2 | 10.0.13.0/30 | R-HQ: 10.0.13.1 | R-BR2: 10.0.13.2 |

## DHCP

Each VLAN/subnet excludes the first 10 addresses (reserved for static/infrastructure use) and hands out the rest automatically. DHCP for the HQ VLANs is served by SW-HQ-CORE; each branch router serves DHCP for its own local subnet.

## Routing

OSPF area 0, running on SW-HQ-CORE, R-HQ, R-BR1, and R-BR2 — every subnet above (LAN and transit) is advertised into the process.

## Access control

ACL 100 on SW-HQ-CORE, applied inbound on VLAN 30 (Guest):
- Deny: 192.168.30.0/24 &rarr; 192.168.20.0/24
- Permit: everything else
