# VTP (VLAN Trunking Protocol) Configuration

## Overview

VLAN Trunking Protocol (VTP) is used in this lab to synchronize VLAN configuration across all switches in the domain, so VLANs only need to be created once — on the VTP Server — and are then propagated automatically to all Client switches.

## Configuration

| Parameter | Value |
|---|---|
| VTP Domain | `lan.local` |
| VTP Version | 2 |
| VTP Password | `vtpadmin` |
| VTP Server | `DistributionSWA` |

## Role Assignment

| Device | VTP Mode | Reason |
|---|---|---|
| DistributionSWA | Server | Source of truth for VLAN database; VLANs are created/modified here |
| DistributionSWB | Client | Receives VLAN updates, does not originate changes |
| AccessSWA | Client | Receives VLAN updates, does not originate changes |
| AccessSWB | Client | Receives VLAN updates, does not originate changes |
| AccessSWC | Client | Receives VLAN updates, does not originate changes |

> Update this table to reflect the actual configured mode on each switch (`show vtp status`).

## Security & Design Notes

- **VTP password** is configured to prevent unauthorized switches from joining the domain and overwriting the VLAN database. It must be identical on every switch in the domain — a mismatch will cause a switch to reject VTP updates.
- **Revision number risk:** a switch with a *higher* configuration revision number than the current server can overwrite the entire VLAN database when connected to the domain — even if it's in Client mode. Before adding a new switch to this lab, reset its revision number with `vtp mode transparent` → `vtp mode client` (or a `delete flash:vlan.dat` + reload), to avoid accidentally wiping the VLAN config.
- **VTP Version 2** was chosen for Token Ring support compatibility and consistent domain-wide behavior; this lab does not use Token Ring, so V1 vs. V2 has no functional impact here beyond consistency.
- For production environments, **VTP Transparent mode** or **VTP Off** is often preferred over Server/Client, since a misconfigured or compromised switch in Server mode can disrupt VLANs across the entire domain. This lab intentionally uses Server/Client mode to practice and understand VTP propagation behavior.

