# Troubleshooting Notes

## Issue: SW-HQ-CORE and R-HQ never formed an OSPF adjacency

### Symptoms

After configuring OSPF on all four Layer-3 devices, `show ip ospf neighbor` on R-HQ only ever showed two neighbors (both branch routers) instead of the expected three. The HQ core switch never appeared in the neighbor table, even though its OSPF configuration matched the branch routers' pattern exactly and no errors were thrown when applying it.

### Investigation

1. Re-checked the OSPF config on SW-HQ-CORE against the design — all four `network` statements were present and correctly matched their subnets and wildcard masks. Config-side, everything looked right.
2. Ran `show ip interface brief` on both ends of the link. R-HQ's side (Gi0/0) showed `up/up`. SW-HQ-CORE's side (Gi0/1) showed `down/down` — a mismatch between the two ends of what should have been the same physical link.
3. Since OSPF can't form an adjacency over a link where the line protocol isn't up, this pointed away from a routing config issue and toward the physical connection itself.
4. Checked the cable on the topology canvas directly rather than trusting the CLI outputs in isolation — this confirmed the link had been accidentally disconnected earlier in the build (a stray click while inspecting the topology) and reconnected using **Automatic** cabling, which had silently landed on a different, unintended port.
5. After manually deleting the mis-landed cable and redrawing it with **Copper Straight-Through**, explicitly selecting GigabitEthernet0/1 on the switch and GigabitEthernet0/0 on the router, `show ip interface brief` still showed the switch side as **administratively down** — the interface itself was shut down in the running config, left over from an earlier `shutdown`/`no shutdown` bounce that didn't fully apply.

### Fix

- Re-cabled the link explicitly (rather than relying on Automatic connection mode) to guarantee both ends landed on the intended interfaces.
- Ran `no shutdown` on the switch-side interface to bring it out of administrative shutdown.
- Confirmed both ends showed `up/up`, then reconfirmed with `show ip ospf neighbor` on both devices — all three adjacencies (SW-HQ-CORE, R-BR1, R-BR2) showed **FULL** state.

### Takeaway

A routing adjacency failure isn't always a routing problem. Before re-reading OSPF configuration line by line, checking the physical/data-link layer first (`show ip interface brief`, and the literal cabling on the canvas) would have identified the actual cause faster. This also reinforced the value of setting cable connections explicitly rather than trusting "Automatic" mode on a topology with several devices in close proximity on the canvas.
