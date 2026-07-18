# OSPF-Dokumentation – Enterprise Campus & Security Lab

Diese README dokumentiert ausschließlich den **OSPF-Prozess 1** (Single-Area, Area 0) aus dem Lab `Enterprise_Campus___Security.yaml`.

Aufbau des Dokuments:
1. [Grober Überblick](#1-grober-überblick) – Topologie, Geräte, Router-IDs
2. [Details](#2-details) – Router-IDs & Loopbacks, Networks je Area, Interface-/Network-Types, Authentication, Reference Bandwidth & Costs, Sonstige OSPF-Optionen, Auffälligkeiten

Rohkonfigurationen liegen als separate `.cfg`-Dateien bei (siehe [Beiliegende Dateien](#beiliegende-dateien)).

---

## 1. Grober Überblick

Nur 4 von ca. 16 Geräten im Lab sprechen OSPF. Der Rest (AccessSW A/B/C, iosvl2-0, DHCP/DNS-Server, PCs, Mailserver, Webserver) sind reine Layer-2-Switches bzw. Endgeräte ohne Routing-Prozess.

| Gerät | Rolle | Router-ID |
|---|---|---|
| **Edge_Router** | Internet-Edge-Router | `99.99.99.99` |
| **Firewall** (ASAv) | Perimeter-Firewall / L3-Hub | `1.1.1.1` |
| **DistributionSWA** | Distribution-Layer | `2.2.2.2` |
| **DistributionSWB** | Distribution-Layer | `3.3.3.3` |

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

Alle Netze liegen in **Area 0** (reines Single-Area-Design). Die Distribution-Switches verteilen zusätzlich die VLANs 10 (UserA), 20 (UserB), 30 (Server) und 99 (MGMT) sowie eine vom Edge_Router injizierte Default-Route (`default-information originate`).

---

## 2. Details

### 2.1 Router-IDs & Loopbacks

| Gerät | Router-ID | Herkunft | Loopback-Interface |
|---|---|---|---|
| Edge_Router | 99.99.99.99 | manuell: `router-id 99.99.99.99` | Lo0 = 99.99.99.99/32 |
| Firewall (ASA) | 1.1.1.1 | manuell: `router-id 1.1.1.1` | Lo0 = 1.1.1.1/32 (kein `nameif`, dient nur der RID) |
| DistributionSWA | 2.2.2.2 | manuell: `router-id 2.2.2.2` | Lo0 = 2.2.2.2/32 |
| DistributionSWB | 3.3.3.3 | **automatisch** – kein `router-id` gesetzt, OSPF zieht die höchste Loopback-IP | Lo0 = 3.3.3.3/32 |

> Da bei DistributionSWB die RID nicht statisch gesetzt ist, würde ein späteres Hinzufügen einer höheren Loopback-Adresse die Router-ID beim nächsten OSPF-Neustart ändern. Zur Konsistenz mit den anderen drei Geräten wäre ein expliziter `router-id 3.3.3.3`-Befehl empfehlenswert.

### 2.2 Networks je Gerät (alle Area 0)

**Edge_Router**
```
network 10.0.0.0 0.0.0.3 area 0            ! Transit zur Firewall (Outside)
network 99.99.99.99 0.0.0.0 area 0         ! eigenes Loopback
```

**Firewall**
```
network 1.1.1.1 255.255.255.255 area 0
network 10.0.0.0 255.255.255.252 area 0     ! zu Edge_Router
network 10.100.1.0 255.255.255.252 area 0   ! zu DistributionSWA
network 10.200.2.0 255.255.255.252 area 0   ! zu DistributionSWB
network 172.16.10.0 255.255.255.0 area 0    ! DMZ-Segment
```

**DistributionSWA**
```
network 2.2.2.2 0.0.0.0 area 0
network 10.100.1.0 0.0.0.3 area 0     ! zur Firewall
network 10.100.99.0 0.0.0.255 area 0  ! VLAN 99 (MGMT)
network 192.168.10.0 0.0.0.255 area 0 ! VLAN 10 (UserA)
network 192.168.20.0 0.0.0.255 area 0 ! VLAN 20 (UserB)
network 192.168.30.0 0.0.0.255 area 0 ! VLAN 30 (Server)
```

**DistributionSWB**
```
network 3.3.3.3 0.0.0.0 area 0
network 10.100.99.0 0.0.0.255 area 0  ! VLAN 99 (MGMT)
network 10.200.2.0 0.0.0.3 area 0     ! zur Firewall
network 192.168.10.0 0.0.0.255 area 0
network 192.168.20.0 0.0.0.255 area 0
network 192.168.30.0 0.0.0.255 area 0
```

**Alle Präfixe im Überblick:**

| Präfix | Beschreibung | Announced von |
|---|---|---|
| 99.99.99.99/32 | Loopback Edge_Router | Edge_Router |
| 1.1.1.1/32 | Loopback Firewall | Firewall |
| 2.2.2.2/32 | Loopback DistributionSWA | DistributionSWA |
| 3.3.3.3/32 | Loopback DistributionSWB | DistributionSWB |
| 10.0.0.0/30 | Transit Edge_Router ↔ Firewall | Edge_Router, Firewall |
| 10.100.1.0/30 | Transit Firewall ↔ DistributionSWA | Firewall, DistributionSWA |
| 10.200.2.0/30 | Transit Firewall ↔ DistributionSWB | Firewall, DistributionSWB |
| 172.16.10.0/24 | DMZ-Segment (Mail-/Webserver via iosvl2-0) | Firewall |
| 10.100.99.0/24 | VLAN 99 – Management | DistributionSWA, DistributionSWB |
| 192.168.10.0/24 | VLAN 10 – UserA | DistributionSWA, DistributionSWB |
| 192.168.20.0/24 | VLAN 20 – UserB | DistributionSWA, DistributionSWB |
| 192.168.30.0/24 | VLAN 30 – Server | DistributionSWA, DistributionSWB |
| 0.0.0.0/0 | Default-Route Richtung Internet | Edge_Router (`default-information originate`) |

### 2.3 Interfaces & Network-Types

| Link | Interface A | Interface B | IP A | IP B | Network-Type A | Network-Type B |
|---|---|---|---|---|---|---|
| Edge_Router ↔ Firewall | Gi0/0 | Gi0/0 (Outside) | 10.0.0.2/30 | 10.0.0.1/30 | `point-to-point` | `point-to-point non-broadcast` |
| Firewall ↔ DistributionSWA | Gi0/1 (DistributionSWA) | Gi1/1 | 10.100.1.1/30 | 10.100.1.2/30 | `point-to-point non-broadcast` | `point-to-point` |
| Firewall ↔ DistributionSWB | Gi0/2 (DistributionSWB) | Gi1/2 | 10.200.2.1/30 | 10.200.2.2/30 | `point-to-point non-broadcast` | `point-to-point` |
| Firewall ↔ DMZ | Gi0/8 | — (kein OSPF-Nachbar) | 172.16.10.254/24 | — | Broadcast (Default) | — |

**Zum Network-Type `non-broadcast` auf der Firewall:** Da bei diesem Typ keine Multicast-Hellos verschickt werden, sind auf der ASA explizite `neighbor`-Statements zwingend nötig:
```
neighbor 10.200.2.2
neighbor 10.100.1.2
neighbor 10.0.0.2
```
Auf den IOS-Gegenstellen fehlt das Schlüsselwort `non-broadcast`, dort reicht der einfache `point-to-point`-Typ (Multicast-Hellos). Beide Seiten bilden trotzdem eine Adjazenz, weil die ASA dank der `neighbor`-Einträge Unicast-Hellos an die bekannten Nachbarn schickt – das ist aber eine Typ-Inkonsistenz zwischen den Enden und sollte im Hinterkopf behalten werden, falls künftig Adjazenzprobleme auftreten.

### 2.4 Authentication

Alle vier Punkt-zu-Punkt-Links zwischen den OSPF-Routern nutzen **einfache (Plaintext) OSPF-Authentifizierung** mit demselben Pre-Shared-Key:

| Link | Befehle (IOS-Seite) | Befehle (ASA-Seite) |
|---|---|---|
| Edge_Router ↔ Firewall | `ip ospf authentication` / `ip ospf authentication-key edv12345` | `ospf authentication` / `ospf authentication-key edv12345` |
| Firewall ↔ DistributionSWA | – | `ospf authentication` / `ospf authentication-key edv12345` |
| Firewall ↔ DistributionSWB | – | `ospf authentication` / `ospf authentication-key edv12345` |
| DistributionSWA ↔ Firewall (Gi1/1) | `ip ospf authentication` / `ip ospf authentication-key edv12345` | – |
| DistributionSWB ↔ Firewall (Gi1/2) | `ip ospf authentication` / `ip ospf authentication-key edv12345` | – |

- Key: `edv12345` (identisch auf allen Interfaces, keine Key-Rotation konfiguriert)
- Es handelt sich um **Type-1 (Plaintext) Authentication**, nicht MD5 (`message-digest`) – auf keinem Interface findet sich `authentication message-digest` oder ein `key-chain`. Für produktive Umgebungen wäre MD5- oder Key-Chain-basierte Authentifizierung empfehlenswert.
- Die DMZ-Schnittstelle der Firewall (172.16.10.0/24) hat **keine** OSPF-Authentifizierung – unkritisch, da dort kein OSPF-Nachbar existiert.

### 2.5 Reference Bandwidth & Costs

| Gerät | `auto-cost reference-bandwidth` | Bemerkung |
|---|---|---|
| Edge_Router | 10000 (Mbps, entspricht 10 Gbps) | explizit gesetzt |
| DistributionSWA | 10000 | explizit gesetzt |
| DistributionSWB | 10000 | explizit gesetzt |
| Firewall (ASA) | *nicht konfiguriert* → ASA-Standardwert (100 Mbps) | **Inkonsistenz!** |

> **Wichtige Auffälligkeit:** Die drei IOS-Geräte nutzen `reference-bandwidth 10000`, die ASA aber keinen expliziten Wert (ASA-Default = 100). Reference-Bandwidth sollte im gesamten OSPF-Prozess konsistent sein, sonst berechnen beide Seiten Kosten pro Link unterschiedlich. Bei reinen P2P-Links mit größtenteils manuell gesetzter Cost (s.u.) ist der praktische Effekt gering, sauberer wäre aber ein einheitlicher Wert.

**Manuell gesetzte Interface-Costs (nur auf der Firewall):**

| Interface | Cost |
|---|---|
| Gi0/0 (Outside → Edge_Router) | 1 |
| Gi0/1 (→ DistributionSWA) | 10 |
| Gi0/2 (→ DistributionSWB) | 10 |

Auf den IOS-Geräten (Edge_Router, DistributionSWA/B) ist **keine** manuelle Interface-Cost gesetzt – dort gilt die automatische Berechnung aus Bandbreite/Reference-Bandwidth (bei Gigabit-Interfaces und Reference-BW 10000 ergibt sich Cost = 10).

### 2.6 Sonstige OSPF-Optionen

- **Passive Interfaces:** Nur `Edge_Router Gi0/15` (Internet-Uplink) ist passiv – verhindert versehentliche OSPF-Adjazenzen Richtung Internet.
- **Default-Route-Injection:** `default-information originate` nur auf dem Edge_Router, verteilt die statische Default-Route (`ip route 0.0.0.0 0.0.0.0 Gi0/15`) als externe Route ins Backbone.
- **Redistribution:** Keine weitere Redistribution (kein BGP/Static außer der Default-Route) konfiguriert.
- **Stub/NSSA-Areas:** Keine – reines Single-Area-0-Design, keine Area-Typen im Einsatz.
- **Timers (Hello/Dead):** Nirgends manuell verändert → es gelten die Defaults des jeweiligen Network-Types (P2P: Hello 10s / Dead 40s).

### 2.7 Zusammenfassung der Auffälligkeiten (für Troubleshooting/Hardening)

1. **Reference-Bandwidth-Inkonsistenz:** ASA nutzt Default (100), IOS-Geräte nutzen 10000 → sollte angeglichen werden.
2. **Network-Type-Mismatch:** ASA-Seiten sind `point-to-point non-broadcast`, IOS-Gegenstellen nur `point-to-point` → funktioniert aktuell nur wegen der `neighbor`-Statements auf der ASA, ist aber unsauber.
3. **Authentication ist nur Plaintext (Type 1),** kein MD5/Key-Chain – aus Sicherheitssicht verbesserungswürdig.
4. **DistributionSWB hat keine explizite `router-id`** – Konsistenzrisiko bei künftigen Loopback-Änderungen.

---

## Beiliegende Dateien

| Datei | Inhalt |
|---|---|
| `Edge_Router.cfg` | Vollständige IOS-Konfiguration des Edge-Routers |
| `DistributionSWA.cfg` | Vollständige IOS-Konfiguration von DistributionSWA |
| `DistributionSWB.cfg` | Vollständige IOS-Konfiguration von DistributionSWB |
| `Firewall_ASA.cfg` | Vollständige ASA-Konfiguration der Firewall (inkl. Zertifikatsketten) |

Diese Dateien sind 1:1 aus dem YAML-Lab-Export (`configuration.content`) extrahiert, ohne Kürzung.
