# Cisco Catalyst 2960-X #1

## Acquisition

- Model: WS-C2960X-24TS-L
- Received: 2026-09-10
- Condition: Used
---

## Why this switch?

The Cisco Catalyst 2960-X was selected to practice and reinforce the switching concepts covered in the CCNA certification on real enterprise hardware.

It provides a physical environment for working with VLANs, trunking, Spanning Tree Protocol, EtherChannel, port security, DHCP Snooping, Dynamic ARP Inspection, SSH management and general Layer 2 troubleshooting.

Using real Cisco hardware also allows me to become familiar with physical deployment, console access, cabling, interface status, hardware inspection and operational troubleshooting beyond network simulation tools.

---

## Role in the homelab

This switch will operate as a Layer 2 access switch within the homelab.

Its main responsibilities will include:

- Connecting end devices and lab equipment
- Providing VLAN segmentation
- Carrying multiple VLANs over trunk links
- Applying Layer 2 security features
- Providing management access through a dedicated management VLAN
- Connecting to the future Layer 3 / firewall infrastructure
- Serving as a platform for CCNA review and future CCNP switching practice

---

## Initial inspection

- Visual inspection
- Power-on test
- Port LEDs verification
- Console access test

---

## Software

IOS : 15.2(7)E2

---

## Initial verification

Commands used:

```text

show version
show inventory
show interfaces status
show vlan brief
show environment

```

The switch arrived in its factory default configuration.

---

## Conclusion 

After two days of testing, the switch appears to be running perfectly.

