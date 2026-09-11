# 🏠 Homelab Infrastructure Project

This repository documents the design, deployment, operation, and evolution of my personal IT homelab.

The project was created to move beyond simulated environments such as Cisco Packet Tracer and GNS3 and apply the networking knowledge acquired through my CompTIA Network+ and Cisco CCNA studies to real enterprise hardware.

The environment is being built progressively around Cisco switching, MikroTik, pfSense, Proxmox, Windows Server, Active Directory, and Linux.

Its purpose is not only to build a working infrastructure, but to document the complete engineering process:

**design → deployment → verification → troubleshooting → improvement**

The homelab will also serve as a practical platform for future CCNP and Security+ studies.

---

## 🚧 Current status

The homelab is currently in the early deployment stage.

### Completed

- ✅ First Cisco Catalyst 2960-X acquired and validated
- ✅ Initial hardware inspection and console access completed
- ✅ IOS and factory-default configuration verified
- ✅ First device documentation published

### In progress

- 🔄 Cisco lab deployment
- 🔄 Physical network and VLAN design
- 🔄 Repository structure and documentation
- 🔄 Homelab architecture planning

### Upcoming

- ⬜ Second Cisco Catalyst 2960-X
- ⬜ Cisco Catalyst 3650 switches
- ⬜ MikroTik central switch
- ⬜ pfSense firewall/router
- ⬜ 15U rack deployment
- ⬜ VLAN segmentation and inter-VLAN routing
- ⬜ Proxmox virtualization platform
- ⬜ Integration of Windows Server, Active Directory, and Linux systems
---

## 🎯 Objectives

The main objectives of this homelab are to:

- Apply CCNA networking knowledge on physical enterprise hardware
- Design and operate a segmented multi-VLAN network
- Practice Layer 2 and Layer 3 networking
- Implement inter-VLAN routing
- Deploy routing, firewalling, and NAT with pfSense
- Practice structured network troubleshooting
- Deploy Windows Server, Active Directory, and Linux services
- Learn virtualization with Proxmox
- Build a secure remote administration environment
- Apply network security features on real equipment
- Document configurations, failures, troubleshooting, and solutions
- Support future CCNP and Security+ studies

---

## 🏗️ Target architecture

The homelab is being built progressively around the following target architecture:

```text
                    Internet
                       │
                 VOO ISP modem
                  (bridge mode)
                       │
                    pfSense
          routing / firewall / NAT
                       │
                    MikroTik
            central Layer 2 switching
                       │
        ┌──────────────┼──────────────┐
        │              │              │
   Family network   Cisco lab   Virtualization lab
                        │              │
                 Cisco switches     Proxmox
                        │              │
                  CCNA / CCNP     Windows Server
                  networking       Active Directory
                  laboratories     Linux / Security+
```

### Component roles

- **VOO ISP modem** — Internet access, planned bridge mode
- **pfSense** — main router, firewall, NAT, inter-VLAN routing, and Internet gateway
- **MikroTik** — central Layer 2 switching and VLAN transport
- **Cisco lab** — physical Cisco switching and routing practice
- **Proxmox lab** — virtualization platform for Windows Server, Active Directory, Linux, and security labs

The exact architecture will evolve as the project grows and design decisions are validated through deployment.

---

## 🌐 Planned network segmentation

The initial segmentation plan is:

| VLAN | Purpose |
|---|---|
| VLAN 10 | Family network |
| VLAN 50 | Homelab network |
| VLAN 100 | Virtualization / servers |
| Management VLAN | Network device administration |

Inter-VLAN routing and firewall policies will be handled by pfSense.

The main goal is to isolate the lab environment from the family network while still allowing controlled Internet access and management traffic.

---

## 🖥️ Hardware

Each major device has its own documentation folder under `Materials/`.

| Status | Device | Role |
|---|---|---|
| ✅ | Cisco Catalyst WS-C2960X-24TS-L #1 | Layer 2 access switch |
| ⬜ | Cisco Catalyst WS-C2960X-24TS-L #2 | Layer 2 access switch |
| ⬜ | Cisco Catalyst 3650 #1 | Layer 3 distribution / collapsed-core |
| ⬜ | Cisco Catalyst 3650 #2 | Layer 3 distribution / collapsed-core |
| ⬜ | MikroTik switch | Central VLAN switching |
| ⬜ | Mini PC — pfSense | Firewall / router |
| ⬜ | Mini PC — Proxmox | Virtualization host |
| ⬜ | 15U rack | Physical infrastructure |

Smaller accessories such as console cables, patch cables, keystone modules, and patch-panel components are documented separately under `Materials/Accessory/`.

---

## 🧱 Cisco lab

The Cisco lab is intended to provide a physical environment for practicing technologies previously studied in simulation.

Planned areas include:

### Switching and Layer 2

- VLAN design and implementation
- 802.1Q trunking
- Access and trunk port configuration
- Rapid PVST+
- Spanning Tree root bridge design
- EtherChannel
- Layer 2 redundancy
- Physical cabling and interface management

### Layer 2 security

- Port Security
- DHCP Snooping
- Dynamic ARP Inspection
- BPDU Guard
- Secure management access

### Management

- Dedicated management VLANs
- SSH management
- Device hardening
- Interface and hardware verification
- Logging and operational checks

### Layer 3 networking

- Inter-VLAN routing
- Layer 3 switching
- Static routing
- Dynamic routing protocols
- IPv4 routing
- IPv6 routing

### First-hop redundancy

- HSRP for IPv4
- HSRPv2 for IPv6
- Active/standby gateway redundancy
- Gateway failover verification

### Network architecture

- Collapsed-core design
- Access / distribution roles
- Layer 2 and Layer 3 redundancy
- Segmented network design
- Multi-VLAN enterprise-style topology

### Troubleshooting and validation

- Structured troubleshooting methodology
- Real hardware verification
- Interface and link troubleshooting
- VLAN and trunk troubleshooting
- STP troubleshooting
- Routing troubleshooting
- Redundancy and failover testing

The long-term objective is to reuse the same physical infrastructure for more advanced CCNP-oriented labs.

---

## 🖥️ Virtualization and systems lab

The virtualization and systems lab will be used to deploy, integrate, and troubleshoot Windows and Linux environments inside the homelab.

### Virtualization

- Proxmox deployment and administration
- Virtual machine provisioning
- Virtual networking
- Resource allocation and management
- Snapshots and backups
- Integration with the physical network infrastructure

### Windows Server

- Windows Server deployment
- Server role installation and administration
- Active Directory Domain Services
- Group Policy
- DNS
- DHCP
- File and storage services
- User and computer management
- Authentication and authorization

### Windows clients

- Windows client deployment
- Domain join
- User profile management
- Group Policy application
- DNS and DHCP client behavior
- Authentication troubleshooting
- Client/server connectivity testing

### Linux systems

- Linux server deployment
- Network configuration
- User and permission management
- SSH administration
- DNS and service integration
- Basic server hardening
- Integration with Active Directory and the physical network

### Identity and access management

- Active Directory users, groups, and organizational units
- Authentication
- Authorization
- Group Policy
- Administrative delegation
- Domain services troubleshooting

### Network services

- DNS
- DHCP
- File services
- Name resolution
- IP address allocation
- Service availability testing
- Integration with VLANs and routing

### Logging and monitoring

- Windows Event Viewer
- Centralized logging
- System monitoring
- Service monitoring
- Network monitoring
- Basic alerting
- Troubleshooting from logs and events

### Security

- System hardening
- Access control
- Firewall rules
- Least privilege principles
- Secure remote administration
- Authentication security
- Security logging
- Security+-oriented labs

### Integration and troubleshooting

- Integration with the physical Cisco network
- VLAN-based server segmentation
- Client/server connectivity
- DNS and DHCP troubleshooting
- Authentication troubleshooting
- Domain connectivity issues
- Network path verification
- Structured systems troubleshooting

A dedicated Proxmox host is planned.

In the meantime, Windows Server and Active Directory labs are already being performed on Hyper-V using a laptop with 32 GB of RAM.

---

## 📚 Documentation approach

Documentation is intentionally separated into two areas.

### `Materials/`

The `Materials/` directory acts as the equipment identity card.

Each device folder may contain:

- Why the equipment was selected
- Acquisition information
- Physical condition
- Initial inspection
- Software version
- Hardware verification
- Photos
- Current status

Example:

    Materials/
    └── Cisco-Switches/
        └── Cisco-2960X-01/
            ├── README.md
            └── images/

### `Labs/`

The lab folders document what is actually built with the equipment.

Each lab follows a consistent structure:

1. **Objective**
2. **Topology**
3. **Configuration**
4. **Verification**
5. **Troubleshooting**
6. **Solution**
7. **Lessons learned**

Failed attempts are documented alongside successful implementations.

The goal is to show the real engineering process rather than only the final working configuration.

---

## 🔐 Security principles

### Infrastructure

The homelab is designed with security in mind.

Planned and implemented practices include:

- SSH-only administration
- No Telnet
- Dedicated management VLAN
- Local authentication
- Firewall policies
- Network segmentation
- DHCP Snooping
- Dynamic ARP Inspection
- Port Security
- BPDU Guard
- Secure device credentials
- Centralized logging
- Monitoring
- Controlled inter-VLAN communication
- Restricted management access
- Separation between family and lab networks

### Repository

Sensitive information is never intentionally committed.

Repository rules include:

- No passwords
- No private keys
- No real public IP addresses
- No API tokens
- No sensitive credentials
- No unsanitized configuration backups
- Configuration files sanitized before publication
- Secrets checked before commits

---

## 📂 Repository structure

    Homelab/
    │
    ├── README.md
    ├── CHANGELOG.md
    ├── .gitignore
    │
    ├── Materials/
    │   ├── Cisco-Switches/
    │   │   └── Cisco-2960X-01/
    │   │       ├── README.md
    │   │       └── images/
    │   ├── MikroTik/
    │   ├── pfSense/
    │   ├── Proxmox/
    │   ├── Rack/
    │   └── Accessory/
    │
    ├── Labs/
    │   ├── Cisco/
    │   └── Virtualization/
    │
    ├── Configs/
    │   ├── Cisco/
    │   ├── pfSense/
    │   └── MikroTik/
    │
    ├── Diagrams/
    │   ├── Physical/
    │   └── Logical/
    │
    └── Troubleshooting/

---

## 🛠️ Phase 1 — Network infrastructure

- [x] First Cisco Catalyst 2960-X acquired and validated
- [ ] Second Cisco Catalyst 2960-X acquired
- [ ] First Cisco Catalyst 3650 switches acquired
- [ ] Second Cisco Catalyst 3650 switches acquired
- [ ] MikroTik switch acquired
- [ ] pfSense mini PC acquired
- [ ] 15U rack installed
- [ ] Core network architecture deployed
- [ ] VLAN segmentation implemented
- [ ] Inter-VLAN routing and firewall policies deployed
- [ ] Cisco lab fully integrated into the homelab

---

## 🖥️ Phase 2 — Virtualization and systems

- [x] Windows Server and Active Directory labs started on Hyper-V (Laptop)
- [ ] Proxmox host acquired
- [ ] Proxmox virtualization platform deployed
- [ ] Windows Server and Active Directory environment migrated to Proxmox
- [ ] Linux server environment deployed
- [ ] Virtualization lab integrated with the physical network
- [ ] Monitoring and centralized logging implemented
- [ ] Backup strategy implemented

---

## 🔐 Phase 3 — Advanced networking and security

Future areas include:

- Advanced firewall policies
- VPN
- AAA
- Centralized authentication
- Network monitoring
- Syslog
- SNMP
- Automation
- Advanced routing
- High availability
- Redundancy
- Network hardening
- Remote administration
- CCNP-oriented labs
- Security+-oriented labs

---

## 📸 Project evolution

The physical and logical evolution of the homelab is documented over time.

Device photos are stored in the corresponding:

`Materials/<device>/images/`

Network diagrams are stored under:

    Diagrams/
    ├── Physical/
    └── Logical/

A chronological summary of major changes is maintained in:

`CHANGELOG.md`

---

## 🧪 Troubleshooting philosophy

Troubleshooting is treated as an important part of the project.

Problems are documented whenever possible with:

- Symptoms
- Initial assumptions
- Verification commands
- Root cause
- Corrective action
- Final validation
- Lessons learned

The goal is not to hide failures, but to use them as part of the learning process.

---

## 🚀 Project philosophy

This homelab is a long-term learning environment.

The objective is not to build everything at once, but to progressively design, deploy, test, break, troubleshoot, improve, and document the infrastructure.

The repository is therefore intended to show not only the final architecture, but also the decisions, mistakes, troubleshooting process, and lessons learned during the project.

**Work in progress — continuously evolving.**