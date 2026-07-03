# Management Network
 
## Overview
 
A dedicated Management VLAN is used to administer all network infrastructure devices (firewall, distribution switches, access switches) via SSH/HTTPS, separate from user, server, and internet traffic. This isolates device management from production traffic and allows strict access control to be enforced.
 
## Configuration
 
| Parameter | Value |
|---|---|
| VLAN ID | 99 |
| Network | `10.100.99.0/24` |
| Subnet Mask | `255.255.255.0` |
| Usable Range | `10.100.99.1 – 10.100.99.254` |
 
## Address Assignment
 
| Device | IP Address | Interface Type |
|---|---|---|
| Firewall | 10.100.99.1 | Dedicated MGMT interface |
| DistributionSWA | 10.100.99.2 | SVI (VLAN 99) |
| DistributionSWB | 10.100.99.3 | SVI (VLAN 99) |
| AccessSWA | 10.100.99.4 | SVI (VLAN 99) |
| AccessSWB | 10.100.99.5 | SVI (VLAN 99) |
| AccessSWC | 10.100.99.6 | SVI (VLAN 99) |
 
## Design Notes
 
- The Firewall's IP is assigned to its **dedicated management interface**, not a data-plane interface — this keeps management traffic off the same interfaces that carry inter-zone traffic.
- All switches use an **SVI in VLAN 99** for management; VLAN 99 is trunked between Access and Distribution switches so all devices remain reachable from a single management subnet.
- VLAN 99 is **not** used as the native VLAN on any trunk (see [VTP.md](VTP.md) / VLAN design notes) — this avoids exposing the management network to VLAN hopping via double-tagging.

