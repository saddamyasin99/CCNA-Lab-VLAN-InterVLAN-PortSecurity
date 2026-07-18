# CCNA Lab  – VLAN Configuration, Inter-VLAN Routing (Router-on-a-Stick) & Port Security

## Objective
This lab focuses on segmenting a switched network into multiple VLANs, enabling communication between those VLANs using the router-on-a-stick method, and securing access-layer switch ports with Port Security. The goal was to build a functioning multi-VLAN topology, verify Layer 2 segmentation, restore Layer 3 connectivity between VLANs through a single router interface, and harden edge ports against unauthorized device access.

## Topology Overview
The topology consists of:
- **1 Layer 2 Switch** with two VLANs configured
- **1 Router (ISR4321)** connected to the switch via a single trunk link
- **6 PCs** split across two departments/VLANs

| VLAN | Name | Subnet | Ports |
|------|------|--------|-------|
| 10 | CS | 192.168.10.0/24 | Fa0/1 – Fa0/3 |
| 20 | Science | 192.168.20.0/24 | Fa0/4 – Fa0/6 |

- **PC0, PC1, PC2** → VLAN 10 (192.168.10.0/24)
- **PC3, PC4, PC5** → VLAN 20 (192.168.20.0/24)
- **Fa0/7** on the switch → trunk link to the router's Gig0/0/0
- Router subinterfaces handle inter-VLAN routing (router-on-a-stick)

## Configuration Steps

### 1. VLAN Creation
Created and named two VLANs on the switch:
```
Switch(config)#vlan 10
Switch(config-vlan)#name CS
Switch(config-vlan)#exit
Switch(config)#vlan 20
Switch(config-vlan)#name Science
Switch(config-vlan)#exit
```

### 2. Assigning Access Ports to VLANs
```
Switch(config)#interface range fa0/1-3
Switch(config-if-range)#switchport mode access
Switch(config-if-range)#switchport access vlan 10
Switch(config-if-range)#exit

Switch(config)#interface range fa0/4-6
Switch(config-if-range)#switchport mode access
Switch(config-if-range)#switchport access vlan 20
Switch(config-if-range)#exit
```

### 3. Trunk Configuration (Switch–Router Link)
Fa0/7 was configured as a trunk port to carry traffic for both VLANs to the router:
```
Switch(config)#interface fa0/7
Switch(config-if)#switchport mode trunk
Switch(config-if)#switchport trunk allowed vlan 10,20
Switch(config-if)#exit
```

### 4. Router-on-a-Stick (Sub-interface Configuration)
The router's physical interface was activated, and two logical sub-interfaces were created — one per VLAN — each tagged with 802.1Q encapsulation and assigned the VLAN's default gateway address.
```
Router(config)#interface gig0/0/0
Router(config-if)#no shutdown

Router(config)#interface gig0/0/0.10
Router(config-subif)#encapsulation dot1q 10
Router(config-subif)#ip address 192.168.10.10 255.255.255.0
Router(config-subif)#exit

Router(config)#interface gig0/0/0.20
Router(config-subif)#encapsulation dot1q 20
Router(config-subif)#ip address 192.168.20.10 255.255.255.0
Router(config-subif)#exit
```
Each sub-interface came up (`LINEPROTO-5-UPDOWN ... changed state to up`) confirming correct trunk/encapsulation matching between the switch and router.

### 5. Port Security on Access Ports
Port Security was enabled on the access-layer ports (Fa0/2, Fa0/3, Fa0/4, Fa0/6) to restrict each port to a single learned MAC address and shut the port down on a violation:
```
Switch(config)#interface fa0/3
Switch(config-if)#switchport mode access
Switch(config-if)#switchport port-security
Switch(config-if)#exit

Switch(config)#interface fa0/4
Switch(config-if)#switchport port-security
Switch(config-if)#switchport mode access
Switch(config-if)#exit

Switch(config)#interface fa0/6
Switch(config-if)#switchport mode access
Switch(config-if)#switchport port-security
Switch(config-if)#exit
```
*(Note: `switchport port security` without the hyphen returned an "Invalid input" error — corrected to `switchport port-security`.)*

## Verification

### VLAN Table (`show vlan`)
Confirms VLAN 10 (CS) and VLAN 20 (Science) are active, with the correct ports assigned to each, and VLAN 1 retaining all unused/default ports.

### Port Security Status (`show port-security interface`)
For Fa0/2, Fa0/3, and Fa0/4:
- Port Security: **Enabled**
- Port Status: **Secure-up**
- Violation Mode: **Shutdown**
- Maximum MAC Addresses: **1**
- Aging: **Disabled**

This confirms each port will learn only one MAC address and shut down automatically if a second device is detected — protecting against unauthorized device connections or MAC spoofing.

### Inter-VLAN Connectivity Test (Ping)
With the router-on-a-stick configuration in place, PCs in different VLANs successfully reached each other across subnets, confirming that inter-VLAN routing was functioning correctly:

- Ping from a VLAN 10 host to `192.168.20.2` and `192.168.20.3` — replies received (initial packet loss on first ping is expected ARP resolution delay in Packet Tracer, not a fault).
- Ping from a VLAN 20 host to `192.168.10.1` (0% loss) and `192.168.10.3` — replies received.

Both tests confirm full connectivity between VLAN 10 and VLAN 20 through the router's sub-interfaces, validating that:
1. VLAN tagging (802.1Q) is functioning correctly across the trunk.
2. The router is correctly routing between the two subnets.
3. End devices in different VLANs can communicate despite Layer 2 isolation.

## Key Takeaways
- ✅ VLANs successfully segment broadcast domains at Layer 2, isolating VLAN 10 (CS) and VLAN 20 (Science) traffic.
- ✅ A single trunk link (Fa0/7) can carry multiple VLANs to a router using 802.1Q tagging.
- ✅ Router-on-a-stick restores Layer 3 connectivity between VLANs using logical sub-interfaces on one physical interface — no need for a dedicated interface per VLAN.
- ✅ Port Security adds an access-layer defense by limiting each port to one MAC address and enforcing a shutdown violation policy.
- ✅ Encapsulation mismatches or missing `no shutdown` commands are common causes of sub-interface failure — always verify sub-interface state after configuration.
- ✅ Minor syntax errors (e.g., missing hyphen in `port-security`) are a normal part of the CLI learning curve and are easily corrected.

## Tools Used
- Cisco Packet Tracer
- Cisco IOS CLI (Switch & Router)

---
#CCNA #Networking #Cisco #PacketTracer #VLAN #InterVLANRouting #PortSecurity #GitHub #NetworkEngineering
