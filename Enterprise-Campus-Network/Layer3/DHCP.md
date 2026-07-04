# DHCP Server Configuration
 
## Configuration
 
```
ip dhcp excluded-address 192.168.10.1 192.168.10.10
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
 
| Pool Name | Network           | Default Router  | Domain Name | Excluded Addresses          |
|-----------|-------------------|------------------|-------------|------------------------------|
| UserA     | 192.168.10.0/24   | 192.168.10.254   | lan.local   | 192.168.10.1 – 192.168.10.10 |
| UserB     | 192.168.20.0/24   | 192.168.20.254   | lan.local   | none                          |
 
## Notes
 
- The excluded address range `192.168.10.1 – 192.168.10.10` reserves these addresses in the **UserA** network so the DHCP server does not assign them (e.g. for static devices, printers, or infrastructure).
- No excluded address range is currently configured for **UserB**. If static/reserved addresses are needed there too, add a corresponding `ip dhcp excluded-address` line.
- ⚠️ The `default-router` value for both pools is `.254`, while the [VLAN Plan](./VLAN-Plan-EN.md) defines the gateway as `.1` for each subnet. Please verify which address is actually configured on the router/Layer 3 interface and align this documentation (and/or the config) accordingly.
- No DHCP pool is defined for the **Server** VLAN (192.168.30.0/24) — this is expected if servers use static IP addressing.
- The `domain-name` command sets the DNS suffix (`lan.local`) handed out to DHCP clients in both pools.
---

 
