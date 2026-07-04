# VLAN Plan
 
Overview of the VLANs configured in the network, their assignment, and associated IP subnets.
 
## Overview
 
| VLAN ID | Name    | Subnet           | Gateway        | Usable Hosts | Description                  |
|---------|---------|------------------|----------------|--------------|-------------------------------|
| 10      | UserA   | 192.168.10.0/24  | 192.168.10.1   | 254          | Network for user group A      |
| 20      | UserB   | 192.168.20.0/24  | 192.168.20.1   | 254          | Network for user group B      |
| 30      | Server  | 192.168.30.0/24  | 192.168.30.1   | 254          | Network for server infrastructure |
 
## Details
 
### VLAN 10 – UserA
- **Subnet:** 192.168.10.0/24
- **Gateway:** 192.168.10.1
- **Address range:** 192.168.10.1 – 192.168.10.254
- **Broadcast:** 192.168.10.255
- **Purpose:** Endpoints for user group A
  
### VLAN 20 – UserB
- **Subnet:** 192.168.20.0/24
- **Gateway:** 192.168.20.1
- **Address range:** 192.168.20.1 – 192.168.20.254
- **Broadcast:** 192.168.20.255
- **Purpose:** Endpoints for user group B
  
### VLAN 30 – Server
- **Subnet:** 192.168.30.0/24
- **Gateway:** 192.168.30.1
- **Address range:** 192.168.30.1 – 192.168.30.254
- **Broadcast:** 192.168.30.255
- **Purpose:** Server infrastructure
  
## Notes
- Inter-VLAN routing is handled by the Layer 3 switch/router.
- Each gateway is defined as the first address (`.1`) in its respective subnet.
- Update this table accordingly when adding further VLANs.
---

 
