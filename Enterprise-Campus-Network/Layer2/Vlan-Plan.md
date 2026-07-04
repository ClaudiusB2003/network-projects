# VLAN Plan

Übersicht der im Netzwerk konfigurierten VLANs, ihrer Zuordnung und der zugehörigen IP-Subnetze.

## Übersicht

| VLAN ID | Name    | Subnetz          | Gateway        | Nutzbare Hosts | Beschreibung                    |
|---------|---------|------------------|----------------|----------------|----------------------------------|
| 10      | UserA   | 192.168.10.0/24  | 192.168.10.1   | 254            | Netzwerk für Benutzergruppe A    |
| 20      | UserB   | 192.168.20.0/24  | 192.168.20.1   | 254            | Netzwerk für Benutzergruppe B    |
| 30      | Server  | 192.168.30.0/24  | 192.168.30.1   | 254            | Netzwerk für Serverinfrastruktur |

## Details

### VLAN 10 – UserA
- **Subnetz:** 192.168.10.0/24
- **Gateway:** 192.168.10.1
- **Adressbereich:** 192.168.10.1 – 192.168.10.254
- **Broadcast:** 192.168.10.255
- **Zweck:** Endgeräte der Benutzergruppe A

### VLAN 20 – UserB
- **Subnetz:** 192.168.20.0/24
- **Gateway:** 192.168.20.1
- **Adressbereich:** 192.168.20.1 – 192.168.20.254
- **Broadcast:** 192.168.20.255
- **Zweck:** Endgeräte der Benutzergruppe B

### VLAN 30 – Server
- **Subnetz:** 192.168.30.0/24
- **Gateway:** 192.168.30.1
- **Adressbereich:** 192.168.30.1 – 192.168.30.254
- **Broadcast:** 192.168.30.255
- **Zweck:** Serverinfrastruktur

## Hinweise
- Inter-VLAN-Routing erfolgt über den Layer-3-Switch/Router.
- Die jeweiligen Gateways sind als erste Adresse (`.1`) im jeweiligen Subnetz definiert.
- Bei Erweiterung um weitere VLANs bitte diese Tabelle entsprechend pflegen.

---
*Letzte Aktualisierung: 2026-07-04*
