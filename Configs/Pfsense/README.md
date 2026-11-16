# IPv6 Configuration on pfSense

## 1. Introduction

After configuring my homelab with IPv4, I decided to start deploying IPv6 on my infrastructure.

My goal is to better understand how IPv6 works in an environment with multiple VLANs, especially:

- DHCPv6 and Prefix Delegation (PD)
- SLAAC and Router Advertisements (RA)
- Global Unicast Addresses (GUA)
- Unique Local Addresses (ULA)
- IPv6 static routing and OSPFv3 for my future Cisco labs

For my first deployment, I decided to enable IPv6 on VLAN 10 (Family) while keeping my existing IPv4 configuration.

### My environment

| Device | Role |
|---|---|
| VOO GigaMax | Internet provider, modem in bridge mode |
| pfSense CE | Firewall, routing, DHCPv4 and DHCPv6 client |
| Grandstream GWN7822P | Main switch |
| Grandstream GWN7660E | Wi-Fi access point |
| Cisco Catalyst 3850 (x2) | Layer 3 Distribution/Core switches |

My pfSense WAN interface is directly connected to the VOO modem operating in bridge mode.

The internal VLANs are transported through a LAGG interface using LACP.

---

## 2. IPv6 WAN Configuration

The first step was to enable DHCPv6 on my WAN interface so that pfSense could obtain a Global Unicast Address and request an IPv6 prefix from my ISP.

Under **Interfaces → WAN**, I configured the following settings:

| Setting | Configuration |
|---|---|
| IPv6 Configuration Type | DHCP6 |
| DHCPv6 Prefix Delegation Size | 64 |
| Request only an IPv6 prefix | Disabled |
| Send IPv6 Prefix Hint | Disabled |
| Do not wait for a RA | Disabled |

After applying the configuration, pfSense successfully obtained an IPv6 GUA on its WAN interface.

Example observed during my tests (IPv6 addresses anonymized):

```text
IPv6 Address : 2001:db8:100::50/128
IPv6 Gateway : fe80::abcd%igc0
```

### Why does the WAN interface have a /128 address?

A `/128` represents a single IPv6 address.

This means that VOO assigned an IPv6 address to my WAN interface, but it does not tell me which IPv6 prefix is available for my internal networks.

For that, DHCPv6 **Prefix Delegation (PD)** is required.

### IA_NA and IA_PD

During this configuration, I learned the difference between two important DHCPv6 mechanisms.

**IA_NA (Identity Association for Non-temporary Addresses)**

Allows a DHCPv6 client to request non-temporary IPv6 addresses.

In my case, pfSense uses IA_NA to request its WAN IPv6 address.

**IA_PD (Identity Association for Prefix Delegation)**

Allows a router to request one or more IPv6 prefixes that can be used on networks behind it.

In my case, pfSense requests a prefix from VOO for my internal VLANs.

I also learned that DHCPv6 does not provide the default gateway.

In IPv6, the default router is discovered through **Router Advertisements (RA)**, usually using the upstream router's link-local address (`fe80::/10`).

Another important point is that the WAN Global Unicast Address and the delegated prefix do not have to belong to the same IPv6 prefix.

---

## 3. Checking the Prefix Delegation

Once IPv6 was enabled on the WAN interface, I needed to find out which prefix size VOO had delegated to my router.

This was important because my homelab contains several VLANs, and I initially wanted to assign a separate IPv6 subnet to each of them.

### Why is the prefix size important?

On standard Ethernet and Wi-Fi networks, SLAAC normally requires a `/64` prefix.

| Delegated prefix | Available /64 subnets |
|---|---:|
| /64 | 1 |
| /60 | 16 |
| /56 | 256 |
| /48 | 65,536 |

Technically, it is possible to divide a `/64` into smaller subnets such as `/65`, `/66`, or `/68`.

However, these longer prefixes do not support standard SLAAC operation on Ethernet and Wi-Fi networks.

Static IPv6 addressing can still work in certain scenarios, especially on point-to-point links.

The limitation is therefore not the number of available IPv6 addresses, but protocol requirements and device compatibility.

### Prefix received from VOO

After checking the DHCPv6 logs, I confirmed that VOO was delegating a `/64` prefix.

Example from my logs (prefix anonymized):

```text
IA_PD prefix: 2001:db8:106:2ee0::/64
```

This means I only have one public `/64` subnet available for my internal networks using SLAAC.

I therefore decided to use this delegated prefix on VLAN 10 (Family).

> 🔧 **Troubleshooting:** At this stage, I couldn't find the prefix delegated by VOO in the DHCPv6 logs. I had to enable DHCPv6 debug mode to identify it.
>
> [See troubleshooting — Prefix Delegation](../../Troubleshooting/pfsense/IPv6/README.md#issue-1)

---

## 4. Configuring Track Interface on VLAN 10

Once I identified my delegated `/64`, I configured the FAMILY interface to use the IPv6 prefix received from VOO.

Under **Interfaces → FAMILY**:

| Setting | Value |
|---|---|
| IPv4 Configuration Type | Static IPv4 |
| IPv6 Configuration Type | Track Interface |
| IPv6 Interface | WAN |
| IPv6 Prefix ID | 0 |
| Block private networks | Disabled |
| Block bogon networks | Disabled |

### How does Track Interface work?

Track Interface allows a pfSense internal interface to use part of the IPv6 prefix delegated to another interface, usually the WAN.

In my case:

```text
               VOO
                |
       DHCPv6 Prefix Delegation
                |
           /64 Prefix
                |
             pfSense
            WAN (igc0)
                |
         Track Interface
                |
       FAMILY (lagg0.10)
                |
          VLAN 10 — /64
```

Because VOO only delegates a single `/64`, I use Prefix ID `0`.

This allows pfSense to assign the delegated prefix to my FAMILY interface.

Another advantage of Track Interface is that it can automatically follow prefix changes made by the ISP.

I actually observed several different delegated prefixes during my tests.

### Result

After correcting my configuration, the FAMILY interface successfully obtained a Global Unicast Address.

Example observed (IPv6 address anonymized):

```text
Interface : lagg0.10
IPv6 Address : 2001:db8:106:3e2a::1/64
```

My VLAN 10 now has a public IPv6 prefix.

> 🔧 **Troubleshooting:** Even though Track Interface was correctly configured, VLAN 10 was not receiving any GUA. I had to inspect the `dhcp6c.conf` file to better understand how pfSense was handling Prefix Delegation.
>
> [See troubleshooting — Track Interface](../../Troubleshooting/pfsense/IPv6/README.md#issue-2)

---

## 5. Configuring Router Advertisements (SLAAC)

After assigning the IPv6 prefix to VLAN 10, the next step was to allow the end hosts on my family network to automatically configure their IPv6 addresses.

For this, I configured **Router Advertisements (RA)** on pfSense.

Under **Services → Router Advertisement → FAMILY**:

| Setting | Value |
|---|---|
| Router Mode | Unmanaged |
| Router Priority | Normal |
| PREF64 | Disabled |

### Why Unmanaged?

Unmanaged mode allows end hosts to use SLAAC to automatically configure their IPv6 addresses.

Unlike stateful DHCPv6, SLAAC does not require a DHCPv6 server to individually assign addresses to clients.

The router advertises the IPv6 prefix, and the devices generate their own addresses.

This is particularly suitable for the end hosts on my family network.

### Difference between Track Interface and SLAAC

It is important to understand that these are two different mechanisms.

**Track Interface:**

Allows pfSense to assign the delegated IPv6 prefix to VLAN 10.

**Router Advertisements + SLAAC:**

Allow the end hosts on VLAN 10 to receive the information needed to automatically configure their IPv6 addresses.

Router Advertisements also allow hosts to discover their IPv6 default router.

### DNS Configuration

I also enabled DNS server advertisement through Router Advertisements.

The **RDNSS (Recursive DNS Server)** option allows IPv6 hosts to discover DNS servers through RA messages without requiring DHCPv6 for DNS configuration.

For now, I decided to use public IPv6 DNS servers from Cloudflare and Google:

| Provider | IPv6 DNS Address |
|---|---|
| Cloudflare | `2606:4700:4700::1111` |
| Google | `2001:4860:4860::8888` |

I haven't studied the pfSense DNS Resolver configuration in detail yet, so I preferred to use public DNS servers for this first deployment.

Later, I may configure the pfSense DNS Resolver to centralize DNS resolution on my network.

### Result

After enabling Router Advertisements, the end hosts on VLAN 10 were able to obtain their IPv6 Global Unicast Addresses through SLAAC.

> 🔧 **Troubleshooting:** Although pfSense had a GUA on the FAMILY interface, the end hosts were still only receiving link-local IPv6 addresses. The problem was that Router Advertisements were not enabled on VLAN 10.
>
> [See troubleshooting — SLAAC](../../Troubleshooting/pfsense/IPv6/README.md#issue-3)

---

## 6. IPv6 Security and Firewall Rules

Enabling IPv6 also requires checking the firewall rules.

In IPv4, my family network uses private addresses and NAT to access the Internet.

With IPv6, the end hosts on VLAN 10 can have their own Global Unicast Addresses.

These addresses are globally routable, but this does not automatically mean that the devices are accessible from the Internet.

pfSense still controls the traffic through its firewall rules.

### Important points

- IPv4 firewall rules do not automatically apply to IPv6.
- IPv6 firewall rules must preserve segmentation between VLANs.
- Aliases containing only IPv4 networks are not enough to filter IPv6 networks.
- ICMPv6 is essential for several IPv6 mechanisms.
- NDP (Neighbor Discovery Protocol) replaces ARP in IPv6.
- Path MTU Discovery also depends on ICMPv6.

For this reason, ICMPv6 should not be blocked indiscriminately.

Complete validation of my IPv6 firewall rules and network isolation is still an additional step to document.

---

## 7. Future IPv6 Plans for My Homelab

For now, I don't want to enable IPv6 across my entire infrastructure.

I prefer to proceed gradually and use my Cisco Lab to learn more about IPv6 routing.

### Planned architecture

| VLAN / Network | IPv6 | Purpose |
|---|---|---|
| VLAN 10 — Family | GUA /64 via Track Interface + SLAAC | Dual-stack family network |
| VLAN 50 — Cisco Transit | Planned static ULA | IPv6 communication between pfSense and Cisco |
| VLAN 99 — Management | IPv4 only for now | Keep management stable |
| VLAN 100 — Virtualization | IPv4 only for now | Keep current configuration |
| VLAN 200–230 — Cisco Lab | Planned ULA | IPv6 routing, SVIs and HSRP for IPv6 |
| Cisco P2P links | Planned ULA /127 | Static routing and OSPFv3 |

### Why use ULA addresses?

Unique Local Addresses (ULA) belong to the `fc00::/7` address space.

For locally generated ULA prefixes, the `fd00::/8` range is used.

These addresses are not intended to be routed on the public Internet, but they can be routed between different VLANs within my homelab.

This will allow me to work on IPv6 without depending on the size of the public prefix delegated by VOO.

### Choosing a Global ID

I haven't selected the final static IPv6 addresses for my Cisco Lab yet.

My plan is to generate a ULA `/48` prefix, following the recommendations of RFC 4193.

This involves generating a **pseudo-random 40-bit Global ID**, which helps reduce the risk of address conflicts with other networks using ULA.

The ULA address structure is:

```text
| 8 bits | 40 bits   | 16 bits   | 64 bits       |
| fd     | Global ID | Subnet ID | Interface ID |
```

The first 48 bits form the ULA prefix, while the following 16 bits can be used to identify individual `/64` subnets.

For example, a fictional prefix could be:

```text
fda3:7b92:18c4::/48
```

From this `/48`, I could create up to 65,536 different `/64` subnets.

I will choose my actual Global ID and final addressing plan before starting the configuration.

### Example of a possible addressing plan

| Network | Example ULA prefix |
|---|---|
| VLAN 50 | `fda3:7b92:18c4:50::/64` |
| VLAN 200 | `fda3:7b92:18c4:200::/64` |
| VLAN 210 | `fda3:7b92:18c4:210::/64` |
| VLAN 220 | `fda3:7b92:18c4:220::/64` |
| VLAN 230 | `fda3:7b92:18c4:230::/64` |
| Reserved space for P2P links | `fda3:7b92:18c4:251::/64` |

These are only examples. I have not generated my final prefix yet, and none of these addresses are currently configured.

IPv6 subnet identifiers are hexadecimal values. I plan to use a naming convention based on my VLAN numbers to make the addressing plan easier to read.

### Point-to-point /127 links

For my future IPv6 routing labs, I also want to use dedicated Layer 3 interfaces between my Cisco Catalyst 3850 switches.

I plan to use IPv6 `/127` prefixes for these links.

With a `/127`, only one bit remains for the interface identifier:

```text
128 - 127 = 1 bit
2^1 = 2 addresses
```

Both addresses can be used on a point-to-point link between routers, as described in RFC 6164.

Fictional example:

```text
DC1 ↔ DC2 Link

Prefix : fda3:7b92:18c4:251::/127

DC1 : fda3:7b92:18c4:251::/127
DC2 : fda3:7b92:18c4:251::1/127
```

I also plan to use IPv4 and IPv6 on some of these links so that I can practice OSPFv2 and OSPFv3 in a dual-stack environment.

OSPFv3 uses IPv6 link-local addresses to establish neighbor adjacencies. Static ULA addresses will mainly help with connectivity testing and troubleshooting.

---

## 8. What I Learned

This first IPv6 deployment helped me better understand several mechanisms that I previously knew mainly from theory.

Here are the main things I learned:

- A WAN IPv6 `/128` address and a delegated IPv6 prefix are two different things.
- IA_NA is used to request IPv6 addresses, while IA_PD is used to request IPv6 prefixes.
- DHCPv6 does not provide the default gateway.
- SLAAC normally requires a `/64` prefix on Ethernet and Wi-Fi networks.
- Track Interface allows pfSense to use and follow a delegated IPv6 prefix.
- Router Advertisements are required for router discovery and SLAAC.
- RDNSS allows DNS information to be advertised through Router Advertisements.
- Requesting a `/56` does not guarantee that the ISP will delegate one.
- ULA addresses allow an internal IPv6 network to be designed independently of the ISP.
- `/127` prefixes are suitable for point-to-point links between routers.

For the next steps, I plan to continue learning IPv6 on my Cisco Catalyst switches, especially static routing, OSPFv3 and redundancy.

My goal is to progress step by step while keeping my family network stable.