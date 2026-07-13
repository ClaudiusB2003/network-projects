# DHCP Server Configuration
 
## Configuration
 
```
ip dhcp excluded-address 192.168.10.1 192.168.10.10
ip dhcp excluded-address 192.168.20.1 192.168.20.10
!
ip dhcp pool UserA
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.254 
 domain-name lan.local
!
ip dhcp pool UserB
 network 192.168.20.0 255.255.255.0
 default-router 192.168.20.254 
 domain-name lan.local
```
 
## Overview
 
| Pool Name | Network | Default Router | Domain Name | Excluded Addresses |
|---|---|---|---|---|
| UserA | 192.168.10.0/24 | 192.168.10.254 | lan.local | 192.168.10.1 – 192.168.10.10 |
| UserB | 192.168.20.0/24 | 192.168.20.254 | lan.local | 192.168.20.1 – 192.168.20.10 |
 
## Notes
 
- Both pools now consistently reserve addresses `.1 – .10` for static devices/infrastructure (e.g. access switch management IPs, printers, etc.).
- The `domain-name` (`lan.local`) is handed out identically in both pools as the DNS suffix for clients.
- **Regarding `.254` vs. `.1`:** This is **not an error** — `.254` is the **HSRP virtual IP** (standby group between DistributionSWA and DistributionSWB), not the physical interface address of an individual switch (which is `.1` / `.2`). Clients should always receive the VIP as their gateway so that, in the event of a switch failure, the other switch automatically takes over. The VLAN plan listing `.1` as the gateway should be corrected to reflect this, rather than changing the DHCP configuration.
- No pool is defined for the Server VLAN (`192.168.30.0/24`) — this is expected if servers continue to use static addressing.
