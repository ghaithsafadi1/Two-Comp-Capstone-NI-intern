# Advanced Multi-VLAN Inter-VLAN Routing Lab

**Internship Project** | Network International | Packet Tracer

## Project Overview

A sophisticated enterprise network topology implementing multi-VLAN design, inter-VLAN routing (Router-on-a-Stick), OSPF, DHCP across multiple subnets, ACL-based access control, and centralized network services (NTP, DNS, HTTP). Demonstrates practical departmental network segmentation with security policies.

## Network Design

**Topology:** 2 core networks (Blue & Red clusters) interconnected via routers, 6 distinct VLANs, multiple switches, departmental subnets

**Left Network Cluster (Blue):**
- VLAN 10 (IT) — IT Department
- VLAN 15 (IT2) — IT Secondary
- VLAN 20 (HR) — Human Resources
- VLAN 25 (ITS) — IT Support/Services

**Right Network Cluster (Red):**
- VLAN 30 (ES) — Executive/Sales
- VLAN 35 (MARKETING) — Marketing Department
- VLAN 40 (SALES) — Sales Department

**Core Infrastructure:**
- Multiple 2960-24TT switches with access/trunk port configuration
- Routers with Gigabit Ethernet interfaces (Gig0/0, Gig0/1, Gig0/2, Gig0/3, etc.)
- Centralized servers providing NTP, DNS, and HTTP services
- Departmental PCs distributed across VLANs

## Technologies Implemented

### 1. **VLAN Segmentation (IEEE 802.1Q)**
- 6 departmental VLANs providing network isolation
- Trunk ports configured with 802.1Q tagging between switches
- Access ports untagged for end devices
- VLAN separation for security and traffic management

### 2. **Inter-VLAN Routing (Router-on-a-Stick)**
- Routers with subinterfaces (e.g., Fa0/0.10, Fa0/0.20) for each VLAN
- Encapsulation 802.1Q enabling cross-VLAN communication
- Default routing from each VLAN to router subinterface gateway
- Centralized routing for inter-departmental traffic

### 3. **OSPF Dynamic Routing**
- OSPF enabled between core routers connecting blue and red clusters
- Automatic route discovery for redundancy
- Multi-router convergence enabling all VLANs to communicate

### 4. **DHCP (Dynamic Host Configuration)**
- DHCP pools configured on routers for each VLAN
- Automatic IP assignment to PCs in each department
- Reduced manual configuration overhead
- Simplified network management at scale

### 5. **Access Control Lists (ACLs)**
- Standard and extended ACLs denying specific PCs from accessing servers
- Granular access control by department and user role
- Security policies enforced at router interfaces
- Prevents unauthorized access to centralized services

### 6. **DNS & HTTP Server**
- Centralized servers (Server-PT) serving both VLANs
- DNS resolution for internal hostnames and services
- HTTP web server accessible from all departments (subject to ACLs)
- Services reach across VLAN boundaries via routing

### 7. **NTP (Network Time Protocol)**
- Centralized NTP server ensuring time synchronization across network
- All devices can query for accurate time
- Critical for log correlation and security events

### 8. **HSRP (Hot Standby Router Protocol)**
- Router redundancy across both clusters
- Active/standby router configuration for gateway failover
- Virtual IP (VIP) shared between physical routers
- Automatic failover when active router goes down
- Each VLAN has a designated HSRP group for high availability
- PCs use virtual IP as default gateway, not physical router IP

## Lab Objectives

✓ Implement 6 departmental VLANs with proper IP subnetting  
✓ Configure 802.1Q trunk ports between switches  
✓ Set up Router-on-a-Stick inter-VLAN routing  
✓ Deploy OSPF for multi-router connectivity  
✓ Create DHCP pools for each VLAN  
✓ Implement ACLs to restrict server access by department  
✓ Configure DNS, HTTP, and NTP services  
✓ Deploy HSRP for router redundancy and gateway failover  
✓ Verify cross-VLAN and cross-cluster communication  
✓ Test ACL enforcement and access denial  
✓ Test HSRP failover scenarios  

## Configuration Highlights

### VLAN Configuration
```
VLAN 10 (IT)        — 10.10.10.0/24
VLAN 15 (IT2)       — 10.15.15.0/24
VLAN 20 (HR)        — 10.20.20.0/24
VLAN 25 (ITS)       — 10.25.25.0/24
VLAN 30 (ES)        — 10.30.30.0/24
VLAN 35 (MARKETING) — 10.35.35.0/24
VLAN 40 (SALES)     — 10.40.40.0/24
```

### Router Subinterface Configuration
```
Router(config-if)# interface Fa0/0.10
Router(config-subif)# encapsulation dot1Q 10
Router(config-subif)# ip address 10.10.10.1 255.255.255.0

Router(config-subif)# interface Fa0/0.20
Router(config-subif)# encapsulation dot1Q 20
Router(config-subif)# ip address 10.20.20.1 255.255.255.0
(repeat for all VLANs...)
```

### DHCP Pool per VLAN
```
Router(config)# ip dhcp pool VLAN10-IT
Router(dhcp-config)# network 10.10.10.0 255.255.255.0
Router(dhcp-config)# default-router 10.10.10.1
Router(dhcp-config)# dns-server 10.0.0.2

Router(config)# ip dhcp excluded-address 10.10.10.1 10.10.10.10
```

### ACL Example (Restrict HR from Server)
```
Router(config)# access-list 110 deny ip 10.20.20.0 0.0.0.255 10.0.0.0 0.255.255.255
Router(config)# access-list 110 permit ip any any
Router(config)# interface Fa0/0.20
Router(config-if)# ip access-group 110 out
```

### Switch Trunk Configuration
```
Switch(config)# interface Fa0/1
Switch(config-if)# switchport mode trunk
Switch(config-if)# switchport trunk encapsulation dot1q
Switch(config-if)# switchport trunk allowed vlan 10,15,20,25
```

### Switch Access Configuration
```
Switch(config)# interface Fa0/5
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 10
```

### HSRP Configuration Example
```
Router0(config)# interface Fa0/0.10
Router0(config-subif)# standby 10 ip 10.10.10.254
Router0(config-subif)# standby 10 priority 110
Router0(config-subif)# standby 10 preempt

Router1(config)# interface Fa0/0.10
Router1(config-subif)# standby 10 ip 10.10.10.254
Router1(config-subif)# standby 10 priority 100
Router1(config-subif)# standby 10 preempt

! PC default gateway points to virtual IP: 10.10.10.254
! If Router0 fails, Router1 becomes active automatically
```

## Verification Steps

1. **Check VLAN membership** — `show vlan brief` on all switches
2. **Verify trunk configuration** — `show interfaces trunk` on switch
3. **Test inter-VLAN routing** — Ping from one VLAN to another
4. **Verify DHCP assignment** — `ipconfig /all` on each PC
5. **Check router subinterfaces** — `show ip interface brief` on routers
6. **Display routing table** — `show ip route` to see OSPF-learned routes
7. **Test HTTP access** — Browser on each VLAN to server HTTP
8. **Verify ACL enforcement** — Attempt denied ping (should fail gracefully)
9. **Test DNS resolution** — `nslookup` from each VLAN
10. **Check NTP sync** — `show clock` on devices with NTP enabled
11. **Verify HSRP status** — `show standby brief` on both routers
12. **Test HSRP failover** — Disable active router and verify standby takes over
13. **Confirm PC gateway** — Verify PCs use virtual IP (10.x.x.254) not physical router IP

## Files Included

- `Advanced_Multi_VLAN_Lab.pkt` — Packet Tracer file with complete configuration
- Topology screenshots (blue and red clusters)
- Configuration backups and documentation
- ACL policy documentation

## Skills Demonstrated

- **VLAN Design & Implementation:** 6-VLAN departmental segmentation
- **802.1Q Trunking:** Inter-switch and switch-to-router VLAN tagging
- **Router-on-a-Stick:** Subinterface configuration for inter-VLAN routing
- **OSPF Routing:** Multi-router dynamic routing with VLAN-aware design
- **DHCP Configuration:** Multi-pool DHCP for departmental subnets
- **Access Control Lists:** Granular security policies and access denial
- **HSRP (Hot Standby Router Protocol):** Router redundancy with automatic failover
- **Network Services:** NTP, DNS, and HTTP centralized deployment
- **High Availability:** Gateway failover and service continuity
- **Troubleshooting:** Complex multi-VLAN connectivity, ACL testing, and failover scenarios
- **Enterprise Network Design:** Practical departmental isolation with redundancy and services
- **Cisco IOS CLI:** Advanced router and switch commands

## Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                    BLUE CLUSTER (Left)                          │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │ VLAN 10 (IT) | VLAN 15 (IT2) | VLAN 20 (HR) | VLAN 25 (ITS)│  │
│  │         [Switches with Access/Trunk Ports]                │  │
│  │ [PC1]   [PC2]   [PC3]   [PC4]   [PC5]   [PC6]             │  │
│  └────────────────┬───────────────────────────────────────────┘  │
│                   │                                                │
│            [Router0/Left]                                         │
│         (Subinterfaces Fa0/0.10-25)                              │
│         (OSPF, DHCP, ACLs)                                       │
│                   │                                                │
│          [Gigabit Trunk]                                         │
│                   │                                                │
└───────────────────┼────────────────────────────────────────────────┘
                    │
          ═══════[OSPF Link]═══════
                    │
┌───────────────────┼────────────────────────────────────────────────┐
│                   │                                                 │
│            [Router1/Right]                                         │
│         (Subinterfaces Fa0/0.30-40)                               │
│         (OSPF, DHCP, ACLs)                                        │
│                   │                                                 │
│  ┌────────────────┴───────────────────────────────────────────┐  │
│  │ VLAN 30 (ES) | VLAN 35 (MARKETING) | VLAN 40 (SALES)       │  │
│  │         [Switches with Access/Trunk Ports]                 │  │
│  │ [PC7]   [PC8]   [PC9]   [PC10]   [PC11]   [PC12]           │  │
│  └───────────────────────────────────────────────────────────┘  │
│                    RED CLUSTER (Right)                            │
└─────────────────────────────────────────────────────────────────┘

         ┌────────────────────────────────────┐
         │  Central Services (Both Clusters)   │
         │  • NTP Server                       │
         │  • DNS Server                       │
         │  • HTTP Web Server                  │
         │  (Accessible via routing, ACL-gated)
         └────────────────────────────────────┘
```

## How to Use This Lab

1. Open `Advanced_Multi_VLAN_Lab.pkt` in Cisco Packet Tracer
2. Review VLAN assignments and routing design
3. Examine router configuration for subinterfaces and OSPF
4. Check ACL rules on router interfaces
5. Test PC-to-PC communication within and across VLANs
6. Verify server access (DNS, HTTP) with and without ACL restrictions
7. Modify ACLs to experiment with access policies
8. Test DHCP by releasing/renewing PC leases

## Key Learnings

- **VLAN Isolation:** Departments can operate independently yet interconnect
- **802.1Q Tagging:** Enables multiple VLANs over single physical links
- **Router-on-a-Stick Scalability:** One router can serve many VLANs via subinterfaces
- **OSPF in Multi-VLAN Networks:** Dynamic routing works seamlessly across departments
- **HSRP for High Availability:** Single virtual gateway IP masks dual routers; automatic failover without client reconfiguration
- **ACLs for Security:** Granular access control without physical separation
- **Centralized Services:** NTP, DNS, HTTP reach all departments across VLAN boundaries
- **DHCP at Scale:** Multiple pools reduce manual IP assignment overhead
- **Redundancy Design:** Eliminates single points of failure at gateway layer
- **Troubleshooting Multi-VLAN:** Test methodology (check VLAN membership → trunking → routing → ACLs → HSRP status)

## Advanced Concepts Tested

- **VLAN Trunking Protocol (VTP)** — Optional: configure VTP domain for automatic VLAN propagation
- **Spanning Tree Protocol (STP)** — Multiple switches can run STP for loop prevention
- **Private VLANs (PVLANs)** — Restrict inter-VLAN communication at Layer 2
- **Route Summarization** — Aggregate VLAN routes for scalability
- **DHCP Snooping & DAI** — Optional security enhancements against rogue DHCP servers

## Future Enhancements

- Implement EIGRP as alternative to OSPF
- Add ACL logging to monitor access attempts
- Integrate Wireless Access Points (WAP) with VLAN assignment
- Configure QoS policies per VLAN
- Add site-to-site VPN between clusters
- Implement VLAN access control lists (VACLs) at switch layer
- Deploy RADIUS or TACACS+ for centralized authentication
- Add switch redundancy with Spanning Tree Protocol (STP) optimization
- Implement HSRP tracking for priority-based failover based on link status

## Author Notes

This lab demonstrates advanced network design solving real organizational requirements:
- **IT Department:** Isolated on dedicated VLANs (10, 15, 25)
- **HR Department:** Restricted access to sensitive servers (VLAN 20 with ACL)
- **Executive Suite:** Separated on VLAN 30 with elevated access
- **Marketing & Sales:** Dedicated VLANs (35, 40) for departmental operations
- **High Availability:** HSRP ensures no single router failure impacts network accessibility

The implementation shows how enterprise networks balance connectivity, security, and reliability through VLAN segmentation, policy-based access control, and redundant gateway architecture. The HSRP implementation demonstrates production-ready network design where gateway failover is transparent to end users.

---

**Internship Period:** [Your dates]  
**Skills:** VLAN Design, 802.1Q Trunking, Inter-VLAN Routing, OSPF, DHCP, ACLs, HSRP, NTP/DNS/HTTP, Router-on-a-Stick, Gateway Redundancy, Network Security  
**Certification Track:** Cisco CCNA (Network Design, Operations & High Availability)
