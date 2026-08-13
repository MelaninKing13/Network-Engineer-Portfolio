# Project 01: Small Business Office Network

**Client:** Ellis Marketing Solutions  
**Designed by:** Raymond Ellis  
**Platform:** Cisco Packet Tracer  
**Project Type:** Small Business Network Design & Security  
**Focus:** VLANs, DHCP, Inter-VLAN Routing, ACLs, SSH, Port Security, Troubleshooting

---

## Project Overview

This project simulates the design and implementation of a secure small-business network for **Ellis Marketing Solutions**.

The company consists of three departments:

- **Sales**
- **Accounting**
- **IT**

The goal was to create a segmented and manageable network that automatically assigns IP addresses, allows controlled communication between departments, provides secure remote device management, and follows basic network-security best practices.

The network was built and tested entirely in **Cisco Packet Tracer** as part of my hands-on CCNA and Network Engineering studies.

---

## Network Objectives

The network was designed to meet the following requirements:

- Separate Sales, Accounting, and IT into individual VLANs
- Automatically assign IP addresses using DHCP
- Configure inter-VLAN routing using Router-on-a-Stick
- Allow IT to communicate with all departments
- Prevent direct communication between Sales and Accounting
- Provide secure SSH management for networking devices
- Restrict remote administration to the IT network
- Implement switch port security
- Disable unused switch ports
- Document and verify the completed network

---

## Network Topology

The network contains:

- **1 Cisco 2911 Router**
- **2 Cisco 2960 Switches**
- **9 Client PCs**
- **3 VLANs**

### Logical Design

```text
                         R1
                    Cisco 2911
                         |
                  802.1Q Trunk
                         |
                        SW1
                   Cisco 2960
                    /        \
                   /          \
           Sales + IT       802.1Q Trunk
                                |
                               SW2
                          Cisco 2960
                                |
                           Accounting
```

---

## VLAN & IP Addressing Plan

| Department | VLAN | Network | Default Gateway |
|---|---:|---|---|
| Sales | 10 | 192.168.10.0/24 | 192.168.10.1 |
| Accounting | 20 | 192.168.20.0/24 | 192.168.20.1 |
| IT | 30 | 192.168.30.0/24 | 192.168.30.1 |

### Infrastructure Management Addresses

| Device | Management IP |
|---|---|
| R1 | 192.168.30.1 |
| SW1 | 192.168.30.2 |
| SW2 | 192.168.30.3 |

Addresses `.1` through `.10` were excluded from the DHCP pools so they could be reserved for gateways, switches, servers, or other infrastructure devices.

---

## VLAN Configuration

### VLAN 10 — Sales

```text
192.168.10.0/24
Gateway: 192.168.10.1
```

### VLAN 20 — Accounting

```text
192.168.20.0/24
Gateway: 192.168.20.1
```

### VLAN 30 — IT

```text
192.168.30.0/24
Gateway: 192.168.30.1
```

---

## Router-on-a-Stick

Inter-VLAN routing was implemented using subinterfaces on R1.

```text
G0/0.10 → VLAN 10 → 192.168.10.1
G0/0.20 → VLAN 20 → 192.168.20.1
G0/0.30 → VLAN 30 → 192.168.30.1
```

Each subinterface uses **802.1Q encapsulation** to identify traffic belonging to its respective VLAN.

This allows one physical router interface to provide routing services for multiple VLANs.

---

## DHCP Configuration

R1 serves as the DHCP server for all three departments.

Three DHCP pools were configured:

```text
SALES
ACCOUNTING
IT
```

All nine client PCs successfully receive:

- IPv4 address
- Subnet mask
- Default gateway

from their appropriate VLAN DHCP pool.

Example:

```text
Sales PC
IP Address: 192.168.10.x
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.10.1
```

---

## Trunking

802.1Q trunk links carry traffic for multiple VLANs between network devices.

### SW1

```text
Gi0/1 → Trunk to R1
Gi0/2 → Trunk to SW2
```

### SW2

```text
Gi0/1 → Trunk to SW1
```

The trunk links carry VLANs:

```text
10
20
30
```

---

## Security Policy

After confirming that unrestricted inter-VLAN routing was working, Extended Access Control Lists were configured to enforce the company's security requirements.

### Required Communication

```text
Sales      X      Accounting

Sales      ↔      IT

Accounting ↔      IT
```

### SALES-FILTER

The Sales network is prevented from initiating traffic toward Accounting.

```text
deny ip 192.168.10.0 0.0.0.255 192.168.20.0 0.0.0.255
permit ip any any
```

### ACCOUNTING-FILTER

The Accounting network is prevented from initiating traffic toward Sales.

```text
deny ip 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255
permit ip any any
```

Testing confirmed that the ACLs successfully blocked unauthorized communication while allowing IT connectivity.

---

## Secure Remote Management

SSH was configured for secure command-line management of:

```text
R1
SW1
SW2
```

Remote administration was restricted to the **IT VLAN**.

IT management network:

```text
192.168.30.0/24
```

Telnet was not used.

Additional security features included:

- Local administrator account
- Enable secret
- SSH version 2
- RSA encryption keys
- VTY authentication
- IT-only VTY access
- Login warning banner
- Password encryption

---

## Switch Port Security

Port Security was enabled on employee access ports.

Configuration included:

- Maximum of **1 MAC address per port**
- Sticky MAC address learning
- Restrict violation mode
- Monitoring of security violations

This helps prevent unauthorized devices from being connected to employee switch ports.

---

## Unused Port Hardening

Unused switch interfaces were administratively disabled.

This reduces the chance of unauthorized devices being connected to unused network ports.

Unused interfaces were also labeled for easier administration and documentation.

---

## Connectivity Testing

Testing was performed before and after implementing security policies.

### Before ACLs

All VLANs successfully communicated through R1.

```text
Sales ↔ Accounting
Sales ↔ IT
Accounting ↔ IT
```

This verified:

- VLAN configuration
- Trunking
- DHCP
- Default gateways
- Router-on-a-Stick
- Inter-VLAN routing

### After ACLs

Testing confirmed:

```text
Sales → Accounting       BLOCKED
Accounting → Sales       BLOCKED

Sales → IT               SUCCESS
Accounting → IT          SUCCESS
IT → Sales               SUCCESS
IT → Accounting          SUCCESS
```

---

## Troubleshooting Experience

One of the most valuable parts of this project involved diagnosing a connectivity and DHCP failure.

Initially, client PCs were unable to obtain DHCP addresses or communicate with their default gateway even though:

- VLANs were correctly configured
- Router subinterfaces were up
- DHCP pools were configured
- Access ports were correctly assigned

Troubleshooting included:

- Static IPv4 configuration
- ICMP ping testing
- VLAN verification
- Trunk verification
- MAC address-table inspection
- Interface status verification
- Router subinterface verification
- Packet Tracer Simulation Mode

The MAC address table helped identify that R1 was physically connected to **SW1 Gi0/1**, while the router trunk had originally been configured on a different interface.

After correcting the trunk configuration, connectivity was restored and DHCP began functioning successfully.

This troubleshooting process reinforced the importance of verifying the **actual physical and logical network path instead of relying only on documentation or assumptions**.

---

## Skills Demonstrated

This project provided hands-on practice with:

- Cisco IOS CLI
- IPv4 Addressing
- Subnetting
- VLAN Configuration
- Access Ports
- 802.1Q Trunking
- Router-on-a-Stick
- Inter-VLAN Routing
- DHCP
- Extended ACLs
- SSH
- Management SVIs
- Port Security
- MAC Address Tables
- Interface Hardening
- Network Documentation
- Layer 2 Troubleshooting
- Layer 3 Troubleshooting
- ICMP Testing
- Network Security Fundamentals

---

## Project Screenshots

Screenshots included with this project:

```text
01-Final-Topology.png
02-VLAN-Configuration.png
03-Trunk-Verification.png
04-DHCP-Bindings.png
05-ACL-Security.png
06-SSH-Management.png
07-Port-Security.png
08-ACL-Connectivity-Test.png
```

---

## Project Files

```text
Project-01-Small-Business-Network/
│
├── README.md
├── Project-01-Small-Business-Network.pkt
│
└── Screenshots/
    ├── 01-Final-Topology.png
    ├── 02-VLAN-Configuration.png
    ├── 03-Trunk-Verification.png
    ├── 04-DHCP-Bindings.png
    ├── 05-ACL-Security.png
    ├── 06-SSH-Management.png
    ├── 07-Port-Security.png
    └── 08-ACL-Connectivity-Test.png
```

---

## What I Learned

This project strengthened my understanding of how switching, routing, addressing, and security work together within a business network.

The troubleshooting portion was particularly valuable because it required working through the network layer-by-layer rather than simply rebuilding the configuration.

I gained additional hands-on experience using Cisco IOS commands to configure, verify, troubleshoot, secure, and document a functioning multi-VLAN network.

This project is part of my ongoing preparation for the **Cisco CCNA certification** and my transition into an entry-level **Network Engineer / Network Technician** career.