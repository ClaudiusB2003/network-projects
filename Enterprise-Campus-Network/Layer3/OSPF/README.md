# OSPF Documentation – Enterprise Campus & Security Lab

This README documents exclusively the **OSPF process 1** (single-area, Area 0).

Document structure:
1. [High-Level Overview](#1-high-level-overview) – topology, devices, router IDs
2. [Details](#2-details) – router IDs & loopbacks, networks per area, interface/network types, authentication, reference bandwidth & costs, other OSPF options, notable issues

Raw configurations are attached as separate `.cfg` files (see [Attached Files](#attached-files)).

---

## 1. High-Level Overview

Only 4 of the roughly 16 devices in the lab speak OSPF. The rest (AccessSW A/B/C, iosvl2-0, DHCP/DNS server, PCs, mail server, web server) are plain Layer-2 switches or end devices without a routing process.

| Device | Role | Router ID |
|---|---|---|
| **Edge_Router** | Internet edge router | `99.99.99.99` |
| **Firewall** (ASAv) | Perimeter firewall / L3 hub | `1.1.1.1` |
| **DistributionSWA** | Distribution layer | `2.2.2.2` |
| **DistributionSWB** | Distribution layer | `3.3.3.3` |

```
                         Internet (DHCP, Gi0/15)
                               │  (passive-interface)
                        ┌──────┴──────┐
                        │ Edge_Router │  RID 99.99.99.99
                        └──────┬──────┘
                          10.0.0.0/30
                        ┌──────┴──────────────┐
                        │      Firewall       │  RID 1.1.1.1
                        │  + DMZ 172.16.10.0/24
                        └───┬──────────────┬───┘
              10.100.1.0/30 │              │ 10.200.2.0/30
                    ┌───────┴───┐    ┌──────┴────┐
                    │DistribSWA │────│DistribSWB │
                    │RID 2.2.2.2│ PC │RID 3.3.3.3│
                    └───────────┘    └───────────┘
```

All networks live in **Area 0** (pure single-area design). The distribution switches additionally distribute VLANs 10 (UserA), 20 (UserB), 30 (Server), and 99 (MGMT), plus a default route injected by the Edge_Router (`default-information originate`).

---

## 2. Details

### 2.1 Router IDs & Loopbacks

| Device | Router ID | Source | Loopback Interface |
|---|---|---|---|
| Edge_Router | 99.99.99.99 | manual: `router-id 99.99.99.99` | Lo0 = 99.99.99.99/32 |
| Firewall (ASA) | 1.1.1.1 | manual: `router-id 1.1.1.1` | Lo0 = 1.1.1.1/32 (no `nameif`, exists only for the RID) |
| DistributionSWA | 2.2.2.2 | manual: `router-id 2.2.2.2` | Lo0 = 2.2.2.2/32 |
| DistributionSWB | 3.3.3.3 | **automatic** – no `router-id` configured, OSPF picks the highest loopback IP | Lo0 = 3.3.3.3/32 |

> Since DistributionSWB's RID is not statically set, adding a higher loopback address later would change the router ID on the next OSPF process restart. For consistency with the other three devices, an explicit `router-id 3.3.3.3` command is recommended.

### 2.2 Networks per Device (all Area 0)

**Edge_Router**
```
network 10.0.0.0 0.0.0.3 area 0            ! Transit to the firewall (Outside)
network 99.99.99.99 0.0.0.0 area 0         ! own loopback
```

**Firewall**
```
network 1.1.1.1 255.255.255.255 area 0
network 10.0.0.0 255.255.255.252 area 0     ! to Edge_Router
network 10.100.1.0 255.255.255.252 area 0   ! to DistributionSWA
network 10.200.2.0 255.255.255.252 area 0   ! to DistributionSWB
network 172.16.10.0 255.255.255.0 area 0    ! DMZ segment
```

**DistributionSWA**
```
network 2.2.2.2 0.0.0.0 area 0
network 10.100.1.0 0.0.0.3 area 0     ! to the firewall
network 10.100.99.0 0.0.0.255 area 0  ! VLAN 99 (MGMT)
network 192.168.10.0 0.0.0.255 area 0 ! VLAN 10 (UserA)
network 192.168.20.0 0.0.0.255 area 0 ! VLAN 20 (UserB)
network 192.168.30.0 0.0.0.255 area 0 ! VLAN 30 (Server)
```

**DistributionSWB**
```
network 3.3.3.3 0.0.0.0 area 0
network 10.100.99.0 0.0.0.255 area 0  ! VLAN 99 (MGMT)
network 10.200.2.0 0.0.0.3 area 0     ! to the firewall
network 192.168.10.0 0.0.0.255 area 0
network 192.168.20.0 0.0.0.255 area 0
network 192.168.30.0 0.0.0.255 area 0
```

**All prefixes at a glance:**

| Prefix | Description | Announced by |
|---|---|---|
| 99.99.99.99/32 | Loopback Edge_Router | Edge_Router |
| 1.1.1.1/32 | Loopback Firewall | Firewall |
| 2.2.2.2/32 | Loopback DistributionSWA | DistributionSWA |
| 3.3.3.3/32 | Loopback DistributionSWB | DistributionSWB |
| 10.0.0.0/30 | Transit Edge_Router ↔ Firewall | Edge_Router, Firewall |
| 10.100.1.0/30 | Transit Firewall ↔ DistributionSWA | Firewall, DistributionSWA |
| 10.200.2.0/30 | Transit Firewall ↔ DistributionSWB | Firewall, DistributionSWB |
| 172.16.10.0/24 | DMZ segment (mail/web server via iosvl2-0) | Firewall |
| 10.100.99.0/24 | VLAN 99 – Management | DistributionSWA, DistributionSWB |
| 192.168.10.0/24 | VLAN 10 – UserA | DistributionSWA, DistributionSWB |
| 192.168.20.0/24 | VLAN 20 – UserB | DistributionSWA, DistributionSWB |
| 192.168.30.0/24 | VLAN 30 – Server | DistributionSWA, DistributionSWB |
| 0.0.0.0/0 | Default route toward the Internet | Edge_Router (`default-information originate`) |

### 2.3 Interfaces & Network Types

| Link | Interface A | Interface B | IP A | IP B | Network Type A | Network Type B |
|---|---|---|---|---|---|---|
| Edge_Router ↔ Firewall | Gi0/0 | Gi0/0 (Outside) | 10.0.0.2/30 | 10.0.0.1/30 | `point-to-point` | `point-to-point non-broadcast` |
| Firewall ↔ DistributionSWA | Gi0/1 (DistributionSWA) | Gi1/1 | 10.100.1.1/30 | 10.100.1.2/30 | `point-to-point non-broadcast` | `point-to-point` |
| Firewall ↔ DistributionSWB | Gi0/2 (DistributionSWB) | Gi1/2 | 10.200.2.1/30 | 10.200.2.2/30 | `point-to-point non-broadcast` | `point-to-point` |
| Firewall ↔ DMZ | Gi0/8 | — (no OSPF neighbor) | 172.16.10.254/24 | — | Broadcast (default) | — |

**On the `non-broadcast` network type on the firewall:** since this type does not send multicast hellos, explicit `neighbor` statements are mandatory on the ASA:
```
neighbor 10.200.2.2
neighbor 10.100.1.2
neighbor 10.0.0.2
```
The IOS peers are missing the `non-broadcast` keyword; a plain `point-to-point` type (multicast hellos) is enough on their side. Both ends still form an adjacency because the ASA sends unicast hellos to its known neighbors thanks to the `neighbor` entries — but this is a type mismatch between the two ends and worth keeping in mind if adjacency issues ever show up.

### 2.4 Authentication

All four point-to-point links between the OSPF routers use **simple (plaintext) OSPF authentication** with the same pre-shared key:

| Link | Commands (IOS side) | Commands (ASA side) |
|---|---|---|
| Edge_Router ↔ Firewall | `ip ospf authentication` / `ip ospf authentication-key edv12345` | `ospf authentication` / `ospf authentication-key edv12345` |
| Firewall ↔ DistributionSWA | – | `ospf authentication` / `ospf authentication-key edv12345` |
| Firewall ↔ DistributionSWB | – | `ospf authentication` / `ospf authentication-key edv12345` |
| DistributionSWA ↔ Firewall (Gi1/1) | `ip ospf authentication` / `ip ospf authentication-key edv12345` | – |
| DistributionSWB ↔ Firewall (Gi1/2) | `ip ospf authentication` / `ip ospf authentication-key edv12345` | – |

- Key: `edv12345` (identical on every interface, no key rotation configured)
- This is **Type-1 (plaintext) authentication**, not MD5 (`message-digest`) — no interface uses `authentication message-digest` or a `key-chain`. For a production environment, MD5- or key-chain-based authentication would be advisable.
- The firewall's DMZ interface (172.16.10.0/24) has **no** OSPF authentication — not critical, since there's no OSPF neighbor there anyway.

### 2.5 Reference Bandwidth & Costs

| Device | `auto-cost reference-bandwidth` | Note |
|---|---|---|
| Edge_Router | 10000 (Mbps, i.e. 10 Gbps) | explicitly set |
| DistributionSWA | 10000 | explicitly set |
| DistributionSWB | 10000 | explicitly set |
| Firewall (ASA) | *not configured* → ASA default (100 Mbps) | **inconsistency!** |

> **Notable issue:** the three IOS devices use `reference-bandwidth 10000`, while the ASA has no explicit value (ASA default = 100). Reference bandwidth should be consistent across the whole OSPF process, otherwise both sides calculate link cost differently. Since most links have a manually set cost anyway (see below), the practical impact is small, but a unified value would be cleaner.

**Manually set interface costs (firewall only):**

| Interface | Cost |
|---|---|
| Gi0/0 (Outside → Edge_Router) | 1 |
| Gi0/1 (→ DistributionSWA) | 10 |
| Gi0/2 (→ DistributionSWB) | 10 |

The IOS devices (Edge_Router, DistributionSWA/B) have **no** manual interface cost set — the automatic calculation from bandwidth/reference-bandwidth applies there (for Gigabit interfaces with reference-bandwidth 10000, that works out to a cost of 10).

### 2.6 Other OSPF Options

- **Passive interfaces:** only `Edge_Router Gi0/15` (Internet uplink) is passive — prevents accidental OSPF adjacencies toward the Internet.
- **Default route injection:** `default-information originate` only on the Edge_Router, distributing the static default route (`ip route 0.0.0.0 0.0.0.0 Gi0/15`) as an external route into the backbone.
- **Redistribution:** no further redistribution (no BGP/static besides the default route) configured.
- **Stub/NSSA areas:** none — pure single-Area-0 design, no area types in use.
- **Timers (Hello/Dead):** not manually changed anywhere → the defaults for the respective network type apply (P2P: Hello 10s / Dead 40s).

### 2.7 Summary of Notable Issues (for troubleshooting/hardening)

1. **Reference-bandwidth inconsistency:** ASA uses the default (100), IOS devices use 10000 → should be aligned.
2. **Network-type mismatch:** ASA sides are `point-to-point non-broadcast`, IOS peers are just `point-to-point` → currently only works because of the `neighbor` statements on the ASA, but is not clean.
3. **Authentication is plaintext only (Type 1)** — no MD5/key-chain, could be improved from a security standpoint.
4. **DistributionSWB has no explicit `router-id`** — consistency risk if the loopback ever changes.

---

## Attached Files

| File | Content |
|---|---|
| `Edge_Router.cfg` | Full IOS configuration of the edge router |
| `DistributionSWA.cfg` | Full IOS configuration of DistributionSWA |
| `DistributionSWB.cfg` | Full IOS configuration of DistributionSWB |
| `Firewall_ASA.cfg` | Full ASA configuration of the firewall (incl. certificate chains) |

These files are extracted 1:1 from the YAML lab export (`configuration.content`), unabridged.
