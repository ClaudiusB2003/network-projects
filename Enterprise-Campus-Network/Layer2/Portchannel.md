# LACP Port-Channel Configuration (Distribution Switches)

This document describes the configuration of a Layer 2 LACP Port-Channel between two distribution switches using EtherChannel (Port-Channel 1).

---

## Overview

- **Protocol:** LACP (Link Aggregation Control Protocol)
- **Port-Channel ID:** 1
- **Mode:** Active (LACP)
- **Trunk VLANs:** 10, 20, 30, 99
- **Encapsulation:** IEEE 802.1Q

---
## Port-Channel Configuration

### Distribution Switch A (DistributionSWA)

```bash
interface Port-channel1
 description #LACP-Portchannel#
 switchport trunk allowed vlan 10,20,30,99
 switchport trunk encapsulation dot1q

 interface GigabitEthernet3/2
 switchport trunk allowed vlan 10,20,30,99
 switchport trunk encapsulation dot1q
 negotiation auto
 channel-group 1 mode active

interface GigabitEthernet3/3
 switchport trunk allowed vlan 10,20,30,99
 switchport trunk encapsulation dot1q
 negotiation auto
 channel-group 1 mode active
```
 
### Distribution Switch B (DistributionSWB)
```bash
interface Port-channel1
 description #LACP-Portchannel#
 switchport trunk allowed vlan 10,20,30,99
 switchport trunk encapsulation dot1q

 interface GigabitEthernet3/2
 switchport trunk allowed vlan 10,20,30,99
 switchport trunk encapsulation dot1q
 switchport mode trunk
 negotiation auto
 channel-group 1 mode active

interface GigabitEthernet3/3
 switchport trunk allowed vlan 10,20,30,99
 switchport trunk encapsulation dot1q
 switchport mode trunk
 negotiation auto
 channel-group 1 mode active
 ```