# Spanning Tree Design & Verification

## Design Goal: Per-VLAN Root Placement (STP Load Balancing)

Instead of making one distribution switch the STP root for *every* VLAN — which
would leave one of the two uplinks on every access switch permanently blocked and
idle — root bridge priority was tuned per VLAN so that **both uplinks carry live
traffic, just for different VLANs**:

| VLAN | STP Root       | Root Bridge Priority |
|------|----------------|-----------------------|
| 1    | local per switch (see note) | 24577 (SWA) / 28673 (SWB) |
| 10   | DistributionSWA | 24586 |
| 20   | DistributionSWB | 24596 |
| 30   | DistributionSWA | 24606 |
| 99   | DistributionSWB | 24675 |

This is conceptually the same idea behind pairing **HSRP/GLBP active gateways**
with **STP root bridges** per VLAN: whichever distribution switch is the
active Layer 3 gateway for a VLAN is also made the STP root for that VLAN, so
traffic doesn't take an unnecessary extra hop across `Po1` before reaching its
gateway.

> **Note on VLAN 1:** both distribution switches report `"This bridge is the root"`
> for VLAN 1. This isn't a misconfiguration — VLAN 1 (native) is pruned from the
> `Po1` trunk between the two distribution switches, so each switch runs its own
> independent VLAN 1 STP domain and is correctly root within it.

## Distribution Layer Verification

`DistributionSWA#show spanning-tree` / `DistributionSWB#show spanning-tree` confirm the design:

- **VLAN 10 & 30** → root is `DistributionSWA` (`5254.00d7.3160`); `DistributionSWB`
  shows these as non-root, reaching the root via `Po1` (`Root FWD`, cost 3).
- **VLAN 20 & 99** → root is `DistributionSWB` (`5254.008a.55c5`); `DistributionSWA`
  shows these as non-root, reaching the root via `Po1` (`Root FWD`, cost 3).
- All access-facing distribution ports (`Gi0/x`, `Gi1/x`, `Gi2/x`, `Gi3/x`) are
  `Desg FWD` — expected, since distribution switches are the root (or forward
  toward it) on every access-facing link.

## Access Layer Verification

Because each access switch only trunks its own user VLAN + VLAN 99, each one
only runs 2–3 STP instances instead of 5 — a direct result of the trunk pruning
described below.

| Switch      | VLAN | Root Port (active) | Alternate (blocked) | Root Bridge   |
|-------------|------|---------------------|----------------------|----------------|
| AccessSWA   | 10   | Gi0/1 (→ SWA)       | Gi0/2 (→ SWB)        | DistributionSWA |
| AccessSWA   | 99   | Gi0/2 (→ SWB)       | Gi0/1 (→ SWA)        | DistributionSWB |
| AccessSWB   | 20   | Gi0/2 (→ SWB)       | Gi0/1 (→ SWA)        | DistributionSWB |
| AccessSWB   | 99   | Gi0/2 (→ SWB)       | Gi0/1 (→ SWA)        | DistributionSWB |
| AccessSWC   | 30   | Gi0/1 (→ SWA)       | Gi0/2 (→ SWB)        | DistributionSWA |
| AccessSWC   | 99   | Gi0/2 (→ SWB)       | Gi0/1 (→ SWA)        | DistributionSWB |

**Reading this table:**
- `AccessSWA` and `AccessSWC` actively use **both** uplinks — one for their user / server
  VLAN, one for management — which is the intended load-balancing behavior.
- `AccessSWB` currently forwards **both** its VLANs (20 and 99) through `Gi0/2`,
  leaving `Gi0/1` fully in blocking state for this switch. This is a direct
  consequence of `DistributionSWB` being root for *both* VLAN 20 and VLAN 99 —
  worth revisiting if the goal is to balance load on every access switch
  individually rather than only in aggregate.

Full raw output: [`access-swa-stp.txt`](./access-swa-stp.txt),
[`access-swb-stp.txt`](./access-swb-stp.txt),
[`access-swc-stp.txt`](./access-swc-stp.txt)

## Security Hardening – Trunk Pruning

On every access-to-distribution uplink, only the following VLANs are permitted
on the trunk:

- The access switch's own **user VLAN** or **server VLAN**
- The **management VLAN** (99)

Concretely, this means `AccessSWA` never carries VLAN 20 or 30 traffic, `AccessSWB`
never carries VLAN 10 or 30, and so on. Practical effect of this, and why it
matters:

- **Reduces the attack/blast radius** – a compromised access port or switch can
  only ever see broadcast/flood traffic for the one VLAN it's actually meant to
  serve, not the whole campus.
- **Prevents VLAN hopping** via double-tagging or misconfigured trunks, since
  VLANs that were never allowed on the trunk can't be switched onto it.
- **Fewer STP instances per access switch** – smaller topology to converge,
  smaller failure domain, easier to reason about and troubleshoot.
- Matches the general **least-privilege principle** applied to Layer 2: a
  device (or link) should only carry the traffic it needs to do its job.
