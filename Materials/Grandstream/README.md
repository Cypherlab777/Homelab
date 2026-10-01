# Grandstream GWN7822P

![Grandstream GWN7822P](./Pictures/GWN7822P.png)

## Acquisition

- Model: GWN7822P
- Received: 2026-10-01
- Condition: New

### Package Contents

![Package Contents](./Pictures/Package-contents.png)

- Grandstream GWN7822P
- Power Cord Anti-Trip
- Power Cable
- Ground Cable
- Rack Bracket Kit

---

## Why this switch?

I mainly chose this switch because of its price-to-feature ratio. It offers:

- 16 × 1 GbE ports
- 8 × 2.5 GbE ports
- 4 × 10 GbE SFP+ ports
- PoE support
- Many Layer 2 security features
- Static and dynamic routing capabilities
- Console, SSH, Web GUI and cloud management

It also supports routing protocols such as OSPF and RIPng, which gives me more possibilities to experiment with Layer 3 features later.

Management can be done through the console port with a CLI that I found quite complete and fairly close to Cisco IOS, but also through SSH, the Web GUI or cloud management.

Grandstream is not as mature or widely used as vendors like Cisco or Aruba, but the feedback I found about the brand and this model was generally positive.

For the price, the GWN7822P gives me a lot of features to experiment with, and I think it will be a good platform to keep learning and growing my homelab over the next few years.

---

## Role in the homelab

The GWN7822P will act as the main switch of my home network and homelab.

Its main responsibilities will include:

- Acting as the central switch for the house
- Separating the family network from the different lab environments using VLANs
- Connecting the pfSense firewall to the rest of the network
- Providing connectivity for the Cisco lab and future Proxmox infrastructure
- Powering future wireless access points and other PoE devices
- Providing high-speed uplinks through its 2.5 GbE and 10 GbE SFP+ interfaces
- Serving as the central point for VLAN distribution across the homelab

---

## Initial inspection

- Visual inspection
- Power-on test
- Port LED verification
- Console access test

---

## Software

- Firmware version: 1.0.13.6

---

## Initial verification

Commands used:

```text
show version
show running-config
show interfaces all
show ip interfaces
show spanning-tree
show vlan

```
---

## Observations

During the initial setup, I noticed a few particularities with the GWN7822P.

The console connection uses 115200 baud, instead of the 9600 baud I was used to with Cisco equipment. The switch is protected by default with an administrator account, and the initial password is printed on the device label.

I also noticed some inconsistencies in the CLI interface descriptions. Ports 14-24 are reported as Ten Gigabit Ethernet even though they are 2.5 GbE copper ports, while ports 25-28 are reported as EtherChannel even though they are actually 10 GbE SFP+ interfaces.

The CLI is quite complete and feels close to Cisco IOS, but the syntax can sometimes be a little particular and requires some adaptation.

---

## Conclusion

The switch is currently working perfectly and no major issues have been found so far.

Testing will continue over the next few days so I can get a better overall impression of the switch and form a more complete opinion after using it in different scenarios.
