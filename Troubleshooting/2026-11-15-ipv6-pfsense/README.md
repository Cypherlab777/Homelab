# 🔧 Troubleshooting — IPv6 Prefix Delegation and SLAAC on pfSense

**Date:** November 15–16, 2026  
**Environment:** Personal Homelab  
**Category:** Networking / IPv6 / pfSense / Troubleshooting  
**Status:** Resolved — all three issues fixed  
**Root cause:** Mixed — missing Router Advertisements confirmed; DHCPv6 prefix-size mismatch suspected  
**Last update:** November 22, 2026

## TL;DR

- **Symptom:** IPv6 deployment on VLAN 10 (Family) was blocked by three successive issues: an apparently missing delegated prefix, no Global Unicast Address (GUA) on pfSense's `FAMILY` interface, and no GUA on end hosts.
- **Evidence:** WAN had a `/128` GUA; DHCPv6 debug logs subsequently showed an IA_PD `/64`. The generated `dhcp6c.conf` referenced `/56` and `sla-len 8`, while the ISP delegated `/64`. Router Advertisements (RA) were disabled on `FAMILY`.
- **Root cause:** RA being disabled explains the missing host SLAAC addresses (confirmed). The `/56` request versus `/64` delegation was a possible contributor to the interface assignment failure, but renegotiation occurred during the correction; causation is unconfirmed.
- **Action:** Enabled DHCPv6 debug logging, restored the `/64` request, renegotiated DHCPv6, and enabled RA in Unmanaged mode with RDNSS.
- **Status:** The delegated prefix was identified, `FAMILY` obtained a GUA, and VLAN 10 hosts obtained GUAs through SLAAC.
- **Next steps:** Validate IPv6 DNS resolution, firewall rules and network isolation.

## 1. Network Environment

| Component | Configuration / Role |
|---|---|
| ISP | VOO — DHCPv6 Prefix Delegation (PD) |
| Firewall | pfSense on CWWK N300 |
| WAN | IPv6 GUA with `/128` prefix length |
| Delegated prefix observed | `/64` (IA_PD) |
| Internal network | VLAN 10 (Family), pfSense interface `FAMILY` (`lagg0.10`) |
| Intended host configuration | SLAAC using Router Advertisements |

**Expected behavior:** pfSense receives an ISP-delegated IPv6 prefix, assigns it to VLAN 10 (Family), and advertises the prefix so end hosts can configure IPv6 addresses automatically through SLAAC.

**Address anonymization:** All example global IPv6 prefixes use the documentation range `2001:db8::/32`. Prefixes shown in different log excerpts were anonymized independently and are not intended to match one another.

## 2. Symptoms

1. **Delegated prefix not visible:** A WAN `/128` GUA was present, but the original DHCPv6 logs did not clearly show the delegated prefix.
2. **No GUA on `FAMILY`:** Track Interface pointed to WAN, yet the pfSense VLAN interface had only a link-local IPv6 address.
3. **No GUA on hosts:** Once `FAMILY` had a GUA, Windows hosts still only had link-local addresses, so SLAAC was not working.

## 3. Investigation

<a id="issue-1"></a>
### Issue 1 — Delegated Prefix Not Visible

**Step 1 — Inspect the DHCPv6 logs.** Under **Status → System Logs → DHCP**, searched for:

```text
dhcp6c
IA_PD
prefix
NoPrefixAvail
```

**Result:** The initial log detail was insufficient to establish the delegated prefix. A WAN GUA alone does not confirm Prefix Delegation.

**Step 2 — Enable DHCPv6 debug mode.** Temporarily enabled **Start DHCP6 client in debug mode** and restarted the WAN DHCPv6 negotiation.

**Result:** The logs exposed an IA_PD entry:

```text
IA_PD prefix: 2001:db8:106:2ee0::/64
```

*The address above is an anonymized documentation example.* In the tested negotiation, VOO delegated a `/64`, providing one `/64` for the internal SLAAC network.

<a id="issue-2"></a>
### Issue 2 — Track Interface Did Not Assign a GUA

**Step 1 — Verify the VLAN interface.** On **Interfaces → FAMILY**, the IPv6 Configuration Type was `Track Interface`, IPv6 Interface `WAN`, and Prefix ID `0`. Under **Status → Interfaces → FAMILY**, only a link-local address was displayed:

```text
fe80::1
```

**Step 2 — Check the requested prefix size.** A prior attempt to request `/56` via **Interfaces → WAN → DHCP6 Client Configuration** used:

```text
DHCPv6 Prefix Delegation Size : 56
Send IPv6 Prefix Hint : Enabled
```

Despite the hint, VOO still returned a `/64` in the logs:

```text
IA_PD prefix: 2001:db8:104:22e::/64
```

**Step 3 — Inspect the generated DHCPv6 configuration.** From **Diagnostics → Command Prompt**:

```sh
cat /var/etc/dhcp6c.conf
```

Relevant configuration:

```text
id-assoc pd 0 {
    prefix ::/56 infinity;
    prefix-interface lagg0.10 {
        sla-id 0;
        sla-len 8;
    };
};
```

**Result:** The client configuration requested `/56` and specified `sla-len 8` (56 + 8 = 64), while the received prefix was `/64`. This mismatch was a plausible explanation for the assignment problem, **not a proven root cause**.

<a id="issue-3"></a>
### Issue 3 — SLAAC Not Working on VLAN 10 Hosts

**Step 1 — Check a Windows host.** Ran:

```powershell
ipconfig /all
```

**Result:** The host had only a link-local IPv6 address (for example, `fe80::2%16`) despite pfSense `FAMILY` already having a GUA. IPv4 connectivity still worked.

**Step 2 — Check Router Advertisements.** Under **Services → Router Advertisement → FAMILY**, RA was not enabled.

**Result:** Without appropriate RA messages, the hosts did not receive the prefix and default-router information needed for SLAAC.

## 4. Corrective Actions

| Issue | Action taken |
|---|---|
| 1 — Prefix visibility | Enabled DHCPv6 debug mode temporarily and restarted DHCPv6 negotiation to capture IA_PD information |
| 2 — No GUA on `FAMILY` | Restored **DHCPv6 Prefix Delegation Size: 64**, disabled **Send IPv6 Prefix Hint**, and renegotiated DHCPv6 |
| 3 — No host GUA | Enabled RA on `FAMILY` in **Unmanaged** mode, with **Router Priority: Normal** and **PREF64: Disabled**; enabled RDNSS |

After changing the WAN DHCPv6 settings, the generated file contained:

```text
id-assoc pd 0 {
    prefix-interface lagg0.10 {
        sla-id 0;
        sla-len 0;
    };
};
```

For RA DNS advertisement, the configured public IPv6 resolvers were:

```text
Cloudflare : 2606:4700:4700::1111
Google     : 2001:4860:4860::8888
```

**Note:** `dhcp6c.conf` is generated by pfSense; it was inspected, not edited directly.

## 5. Root Cause Analysis

- **Confirmed:** The ISP supplied an IA_PD `/64` during the observed negotiation. The initial inability to see it was a logging/visibility issue, not evidence that no delegation existed.
- **Confirmed:** RA was disabled on `FAMILY`. Enabling it restored SLAAC address assignment on VLAN 10 hosts.
- **Observed:** The generated client configuration referenced a `/56` request (`sla-len 8`) while the received delegated prefix was `/64`.
- **Unconfirmed:** Whether that mismatch alone prevented the VLAN interface from receiving a GUA. Returning to `/64` also triggered DHCPv6 renegotiation, so these effects were not isolated.

## 6. Validation and Status

| Check | Observed result | Status |
|---|---|---|
| Delegated prefix visible in DHCPv6 debug logs | IA_PD `/64` recorded | Confirmed |
| GUA assigned to `FAMILY` after the WAN correction | Example: `2001:db8:106:3e2a::1/64` on `lagg0.10` (anonymized) | Resolved |
| VLAN 10 hosts obtain GUAs via SLAAC | Host addresses appeared after RA was enabled | Resolved |
| IPv6 DNS, firewall and network isolation end-to-end | Separate follow-up tests planned | Outside the three resolved issues |

## 7. Summary Table

| Symptom | Key evidence | Action | Outcome |
|---|---|---|---|
| PD not visible | WAN `/128`; IA_PD found only with debug logging | Enabled DHCPv6 debug | Delegated `/64` identified |
| No GUA on `FAMILY` | `/56` request and `sla-len 8` vs delegated `/64` | Restored `/64`; renegotiated | GUA assigned; exact cause unconfirmed |
| No GUA on end hosts | RA disabled on `FAMILY` | Enabled Unmanaged RA and RDNSS | SLAAC restored |

## 8. Next Steps

- Verify IPv6 DNS resolution from VLAN 10 clients.
- Validate IPv6 firewall rules and network isolation.
- Continue separate Cisco IPv6 routing labs without assuming the ISP `/64` can be reused on multiple routed VLANs.

## 9. Lessons Learned

- A WAN GUA (`IA_NA`) and an ISP-delegated prefix (`IA_PD`) are distinct DHCPv6 concepts.
- A Prefix Hint is a request, not a guarantee that the ISP will delegate that size.
- Track Interface uses the delegated prefix, but host SLAAC also requires Router Advertisements.
- `sla-id` and `sla-len` help explain how pfSense allocates IPv6 subnets from a delegated prefix.
- Record confirmed findings separately from hypotheses whenever several changes happen during one correction.
