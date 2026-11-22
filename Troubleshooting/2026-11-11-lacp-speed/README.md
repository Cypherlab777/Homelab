# 🔧 Troubleshooting — LACP Link Speed Limited to 1 Gbps

**Date:** November 11, 2026  
**Environment:** Personal Homelab  
**Category:** Networking / LACP / Troubleshooting  
**Status:** Partially resolved — LAG speed corrected; physical members pending verification  
**Root cause:** Confirmed — LAG1 speed configured at 1000 Mbps  
**Last update:** November 22, 2026

## TL;DR

- **Symptom:** Both LACP members between pfSense and the Grandstream GWN7822P were operating at 1 Gbps instead of the expected 2.5 Gbps, although the aggregation remained operational (`bndl`).
- **Evidence:** Grandstream Ethernet `1/0/23` negotiated 2.5 Gbps alone but reverted to 1 Gbps when added to LAG1. The Grandstream Web UI showed the LAG1 speed configured to `1000 Mbps`.
- **Root cause:** LAG1 was configured for 1000 Mbps. Whether the earlier firmware update changed that setting is **unconfirmed**.
- **Action:** Changed the LAG1 speed setting to 2.5 Gbps in the Grandstream Web UI.
- **Status:** The logical LAG reported `connected full 2.5G`. Verification of *both physical members* at 2.5 Gbps remains pending.
- **Next steps:** Check negotiation and LACP state on `1/0/23` and `1/0/24`, and the matching pfSense interfaces, independently.

## Quick Timeline

| Date | Event |
|---|---|
| Nov 8–9 | Separate LACP link-flapping incident on the same uplink |
| Around Nov 9 | Grandstream firmware updated from 1.0.13.6 to 1.0.15.137 during the other investigation |
| Nov 11 | 1 Gbps limitation diagnosed; LAG1 speed changed from 1000 Mbps to 2.5 Gbps |
| Nov 22 | No new link flapping observed since the earlier firmware update; physical member-speed validation still pending |

## 1. Network Environment

| Component | Configuration / Role |
|---|---|
| Firewall | pfSense on CWWK Intel N300, Intel i226-V 2.5 GbE interfaces |
| Switch | Grandstream GWN7822P |
| Protocol | LACP (IEEE 802.3ad) |
| pfSense aggregation | `lagg0`: `igc1` + `igc2` |
| Grandstream aggregation | `LAG1`: Ethernet `1/0/23` + `1/0/24` |
| Interface mapping | `igc1` ↔ `1/0/24`; `igc2` ↔ `1/0/23` |
| Cabling | Cat6 cables with RJ45 female-to-female keystone couplers |

**Expected behavior:** Each physical member negotiates 2.5 Gbps. Two links provide up to 5 Gbps of aggregate theoretical capacity **across multiple flows**, not a 5 Gbps single-flow link.

## 2. Symptoms

- LACP was functioning: the members were reported in the `bndl` state.
- The physical links negotiated **1 Gbps**, not the expected **2.5 Gbps**.
- The limitation was found after the firmware update performed for the separate link-flapping incident. This chronology alone does **not** establish that the update changed the speed setting.

## 3. Investigation

### Step 1 — Verify the Interface Speeds

**Grandstream CLI:**

```text
show interfaces Ethernet 1/0/23 status
show interfaces Ethernet 1/0/24 status
```

**pfSense:**

```sh
ifconfig igc1
ifconfig igc2
ifconfig lagg0
```

**Result:** Relevant physical interfaces operated at 1000 Mbps instead of 2500 Mbps.

### Step 2 — Test the Cabling

Removed the RJ45 keystone coupler from the affected connection.

**Result:** Grandstream Ethernet `1/0/23` still negotiated 1 Gbps. Removing the coupler did not fix the issue.

### Step 3 — Test an Alternative Switch Port

Connected pfSense `igc2` to Grandstream Ethernet `1/0/17` temporarily, then repeated the test with the keystone coupler.

**Result:** The link negotiated 2.5 Gbps in both tests. Port `1/0/17` was **only used for diagnosis**; it is not a member of LAG1 in the final topology.

### Step 4 — Isolate the LAG Configuration

Tested the same Ethernet `1/0/23` interface inside and outside LAG1:

| Configuration | Negotiated speed |
|---|---|
| `1/0/23` in LAG1 | 1 Gbps |
| `1/0/23` removed from LAG1 | 2.5 Gbps |
| `1/0/23` added back to LAG1 | 1 Gbps |

**Result:** The speed limitation followed LAG membership rather than the isolated physical port.

### Step 5 — Inspect LAG1 in the Web UI

Opened the Grandstream Web UI and inspected **LAG Port Settings**.

**Result:** LAG1's configured speed was `1000 Mbps`. This was the decisive finding.

The CLI checks of physical ports did not resolve the issue because the limiting setting was associated with the logical LAG configuration. This does **not** prove that the CLI cannot configure LAG speed; no corresponding CLI failure was recorded.

## 4. Corrective Actions

Changed the Grandstream **LAG1 speed** through the Web UI:

```text
Before: LAG1 Speed = 1000 Mbps
After:  LAG1 Speed = 2.5 Gbps
```

No cable replacement or Intel NIC driver/firmware update was required for this specific correction.

## 5. Root Cause Analysis

- **Confirmed:** LAG1 was configured at `1000 Mbps` in the Grandstream Web UI.
- **Observed:** The same physical interface could negotiate 2.5 Gbps outside LAG1 but only 1 Gbps when participating in LAG1.
- **Confirmed corrective effect:** Changing the LAG setting resulted in the logical LAG reporting 2.5 Gbps.
- **Unconfirmed:** Whether the earlier Grandstream firmware update modified or restored the LAG speed setting. The prior link flapping and this speed limitation are separate findings unless a common cause is demonstrated.

## 6. Validation and Status

**Post-change Grandstream CLI check:**

```text
show interfaces lag 1 status

lag1    connected    full    2.5G
```

| Check | Observed result | Status |
|---|---|---|
| LAG1 speed setting | Changed from 1000 Mbps to 2.5 Gbps | Resolved |
| Logical LAG interface | `connected full 2.5G` | Confirmed |
| Ethernet `1/0/23` and `1/0/24` individually | No final two-member speed verification recorded | Pending |
| LACP aggregation after member verification | Final individual member check not recorded | Pending |

**Scope of resolution:** The logical LAG speed limitation is corrected. The CLI output alone is not proof that both physical members have individually negotiated 2.5 Gbps.

## 7. Summary Table

| Diagnostic test | Result | Interpretation |
|---|---|---|
| Remove keystone | Still 1 Gbps | Keystone removal did not fix the issue |
| Use temporary port `1/0/17` | 2.5 Gbps, with/without keystone | Tested connection supports 2.5 Gbps |
| Remove `1/0/23` from LAG1 | 2.5 Gbps outside the LAG | Speed limitation associated with LAG membership |
| Inspect LAG1 settings | Speed set to 1000 Mbps | Confirmed configuration limitation |
| Change LAG1 speed | Logical interface reports 2.5 Gbps | Correction effective at logical LAG level |

## 8. Next Steps

- Verify Grandstream Ethernet `1/0/23` and `1/0/24` individually at 2.5 Gbps.
- Verify corresponding pfSense `igc2` and `igc1` negotiated speeds.
- Confirm both members' LACP state after the speed change.
- Keep the older link-flapping investigation separate; do not infer an unproven link to the speed setting.

## 9. Lessons Learned

- An operational LACP bundle does not guarantee that its physical members run at their expected speed.
- Check **physical interfaces** and **logical aggregation interfaces** separately.
- Isolating the same port inside and outside the LAG was more informative than changing cables alone.
- Verify configuration after a firmware update without automatically assigning causation to the update.
- Distinguish a confirmed configuration fault from an unconfirmed explanation for how that configuration arose.

## References

- [LACP link flapping between Grandstream and pfSense — separate investigation](../link-flapping/README.md)
