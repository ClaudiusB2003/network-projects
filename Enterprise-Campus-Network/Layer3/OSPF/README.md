# OSPF Documentation

This document covers **OSPF process 1**, a single-area design (Area 0) running on 4 devices in the lab. Full configs are in the attached `.txt` files.

---

## 1. Topology Overview

Only 4 devices run OSPF. Everything else (access switches, PCs, servers, DNS/DHCP) is a plain Layer-2 or end device.

| Device | Role | Router ID |
|---|---|---|
| Edge_Router | Internet edge router | 99.99.99.99 |
| Firewall (ASAv) | Perimeter firewall | 1.1.1.1 |
| DistributionSWA | Distribution switch | 2.2.2.2 |
| DistributionSWB | Distribution switch | 3.3.3.3 |

```
**<img width="847" height="987" alt="OSPF_Topology" src="https://github.com/user-attachments/assets/8fed4331-a2b7-4af1-a75e-64d362f4da75" />**

```

Everything runs in **Area 0**. The two distribution switches also carry VLANs 10 (Users A), 20 (Users B), 30 (Servers), and 99 (Management). The Edge_Router injects a default route into OSPF.

---

## 2. Router IDs

All Router IDs are set manually via `router-id`, backed by a matching loopback interface:

| Device | Router ID | Loopback |
|---|---|---|
| Edge_Router | 99.99.99.99 | Lo0 |
| Firewall | 1.1.1.1 | Lo0 (no `nameif`, exists only for the RID) |
| DistributionSWA | 2.2.2.2 | Lo0 |
| DistributionSWB | 3.3.3.3 | Lo0 |

---

## 3. Networks (all in Area 0)

| Prefix | Purpose |
|---|---|
| 99.99.99.99/32 | Edge_Router loopback |
| 1.1.1.1/32 | Firewall loopback |
| 2.2.2.2/32 | DistributionSWA loopback |
| 3.3.3.3/32 | DistributionSWB loopback |
| 10.0.0.0/30 | Edge_Router ↔ Firewall |
| 10.100.1.0/30 | Firewall ↔ DistributionSWA |
| 10.200.2.0/30 | Firewall ↔ DistributionSWB |
| 172.16.10.0/24 | DMZ (mail/web servers) |
| 10.100.99.0/24 | VLAN 99 – Management |
| 192.168.10.0/24 | VLAN 10 – Users A |
| 192.168.20.0/24 | VLAN 20 – Users B |
| 192.168.30.0/24 | VLAN 30 – Servers |
| 0.0.0.0/0 | Default route, originated by Edge_Router |

---

## 4. Interfaces & Network Types

| Link | Network Type |
|---|---|
| Edge_Router ↔ Firewall | `point-to-point` (IOS) / `point-to-point non-broadcast` (ASA) |
| Firewall ↔ DistributionSWA | `point-to-point non-broadcast` (ASA) / `point-to-point` (IOS) |
| Firewall ↔ DistributionSWB | `point-to-point non-broadcast` (ASA) / `point-to-point` (IOS) |
| Firewall ↔ DMZ | Broadcast (no OSPF neighbor here) |

The ASA uses `non-broadcast` on all three P2P links, so it needs manual `neighbor` statements to form adjacencies:
```
neighbor 10.0.0.2
neighbor 10.100.1.2
neighbor 10.200.2.2
```
The IOS side just uses plain `point-to-point`, without the `non-broadcast` keyword. The adjacency still comes up (thanks to the ASA's `neighbor` statements), but the two ends technically disagree on network type — worth knowing if troubleshooting is ever needed.

---

## 5. Authentication

Every P2P link uses **simple plaintext OSPF authentication** with one shared key: `edv12345`.

- Applied consistently on all four links (IOS side: `ip ospf authentication` / ASA side: `ospf authentication`).
- This is Type-1 (plaintext), not MD5 — no encryption, no key-chain.
- The DMZ interface has no authentication, but that's fine since there's no OSPF neighbor there.

⚠️ Plaintext auth is fine for a lab, but MD5 or a key-chain would be recommended for production.

---

## 6. Reference Bandwidth & Cost

| Device | Reference Bandwidth |
|---|---|
| Edge_Router | 10000 |
| DistributionSWA | 10000 |
| DistributionSWB | 10000 |
| Firewall (ASA) | not set → default 100 |

⚠️ The ASA doesn't match the IOS devices, so cost calculations differ. Low impact here since the firewall has manual interface costs set anyway:

| Firewall Interface | Cost |
|---|---|
| Gi0/0 → Edge_Router | 1 |
| Gi0/1 → DistributionSWA | 10 |
| Gi0/2 → DistributionSWB | 10 |

The IOS devices don't set manual cost — it's calculated automatically (works out to 10 on Gigabit links, given reference-bandwidth 10000).

---

## 7. Other OSPF Settings

- **Passive interface:** only `Edge_Router Gi0/15` (Internet-facing) — stops OSPF from forming adjacencies toward the internet.
- **Default route:** only the Edge_Router injects `0.0.0.0/0` via `default-information originate`.
- **Redistribution:** none, apart from the default route.
- **Areas:** single Area 0 only.
- **Timers:** all default (Hello 10s / Dead 40s for P2P).

---

---

## Author
Claudius B.
