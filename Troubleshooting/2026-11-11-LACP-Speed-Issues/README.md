# 🔧 Troubleshooting — LACP Limited to 1 Gbps Instead of 2.5 Gbps

**Date:** November 11, 2026  
**Environment:** Personal Homelab  
**Category:** Networking / LACP / Troubleshooting  
**Status:** Speed limitation resolved — physical member verification pending

## 1. Network Environment

**Hardware:**
- Firewall: pfSense — CWWK Intel N300, Intel i226-V 2.5 GbE interfaces
- Switch: Grandstream GWN7822P
- Link aggregation: LACP (IEEE 802.3ad)
- pfSense: `lagg0` — `igc1` + `igc2`
- Grandstream: `LAG1` — ports `1/0/23` + `1/0/24`
- Cabling: Cat6 Ethernet cables with RJ45 female-to-female keystone couplers

**Expected behavior:** Both physical links should negotiate at 2.5 Gbps, providing an aggregate theoretical capacity of 5 Gbps across multiple traffic flows.

## 2. Background — Previous Link-Flapping Incident

The Grandstream GWN7822P had previously experienced intermittent link-flapping events affecting the LACP uplink to pfSense.

The logs showed repeated DOWN/UP transitions on port `1/0/23`, corresponding to `igc2` on pfSense.

The events were observed on both devices, indicating instability at the physical link level.

Despite the interruptions, LACP remained operational when the interfaces were connected.

To investigate the instability, the Grandstream firmware was updated from **1.0.13.6 to 1.0.15.137**.

Approximately 24 hours after the update, no further link-flapping events had been observed.

The firmware update appeared to improve stability, although the root cause of the flapping was not conclusively identified.

### Follow-Up — November 13, 2026

As of November 13, 2026, approximately five days after the Grandstream firmware update, **no further link-flapping events have been observed** on the LACP uplink between the switch and pfSense.

The connection has remained stable during this observation period.

This is an encouraging result and suggests that the firmware update may have addressed the instability. However, the exact root cause of the original link-flapping issue has not been conclusively identified.

Monitoring will continue, and the separate link-flapping troubleshooting report will be updated at a later date.

## 3. Problem — Unexpected 1 Gbps Link Speed

During the subsequent investigation of the LACP uplink, a separate speed limitation was identified.

The physical interfaces associated with LAG1 were operating at **1 Gbps instead of the expected 2.5 Gbps**.

LACP itself was functioning: both member interfaces were reported in the `bndl` state.

This indicated that the interfaces were participating in the aggregation, but not operating at their expected speed.

The issue therefore required investigating the physical links, switch ports and LAG configuration independently.

**Important:** Although this investigation followed the firmware update, the available evidence does not establish that the update caused the speed setting to change.

## 4. Troubleshooting Process

### Step 1 — Verify Interface Negotiation

Checked interface speed on both devices.

**Grandstream CLI:**
```text
show interfaces Ethernet 1/0/23 status
show interfaces Ethernet 1/0/24 status
```

**pfSense:**
```shell
ifconfig igc1
ifconfig igc2
ifconfig lagg0
```

**Observation:** The relevant physical interfaces were operating at 1000 Mbps rather than 2500 Mbps.

### Step 2 — Eliminate the Keystone as a Possible Cause

Tested the Ethernet connection without the RJ45 keystone coupler.

**Result:** Port 23 continued operating at 1 Gbps.

This showed that removing the keystone did not resolve the problem.

### Step 3 — Test Another 2.5 GbE Switch Port

Connected the pfSense interface `igc2` to Grandstream port `1/0/17` for diagnostic purposes.

**Result:** The connection negotiated successfully at 2.5 Gbps.

The test was repeated with the keystone coupler.

**Result:** The connection continued operating at 2.5 Gbps.

These observations demonstrated that the tested physical connection was capable of negotiating at the expected speed.

Port 17 was used only for troubleshooting and was not retained as an uplink.

### Step 4 — Isolate the LAG Configuration

Removed port `1/0/23` from LAG1 and tested the physical interface independently.

**Results:**

| Configuration | Negotiated speed |
|---|---|
| Port 23 included in LAG1 | 1 Gbps |
| Port 23 removed from LAG1 | 2.5 Gbps |
| Port 23 re-added to LAG1 | 1 Gbps |

This was the decisive diagnostic step.

The same physical interface could negotiate at 2.5 Gbps independently, but returned to 1 Gbps when assigned to the aggregation group.

The observations strongly suggested that the limitation was associated with the LAG configuration rather than the physical interface itself.

### Step 5 — Inspect LAG Settings in the Web Interface

Opened the Grandstream management Web UI and inspected the settings associated with LAG1.

**Finding:** The LAG speed was configured at `1000 Mbps`.

This explained the observed behavior: the configured LAG speed was limiting the link operation when the interface participated in the aggregation group.

## 5. Why the CLI Troubleshooting Did Not Resolve the Issue

The Grandstream CLI provides commands to inspect interface status and configure physical Ethernet ports.

However, **the physical member interface and the logical LAG interface have distinct configuration contexts**.

In this case, troubleshooting focused initially on the physical Ethernet interfaces.

Changing or checking the physical interface alone did not eliminate the speed limitation associated with LAG1.

The relevant configuration parameter was subsequently identified in the Web UI under the **LAG Port Settings**.

The Web UI provided direct access to the speed setting responsible for the limitation.

This does not demonstrate that the CLI is incapable of configuring LAG speed. It shows that the CLI troubleshooting performed did not resolve this particular configuration issue.

The exact CLI command failure or limitation was not recorded, so no unsupported CLI error or firmware limitation is assumed.

## 6. Resolution

Changed the LAG1 speed in the Grandstream Web UI:

**Previous setting:**
```text
LAG1 Speed: 1000 Mbps
```

**New setting:**
```text
LAG1 Speed: 2.5 Gbps
```

After applying the configuration, verified the logical aggregation interface from the CLI.

```text
show interfaces lag 1 status

lag1    connected    full    2.5G
```

**Result:** The LAG interface reported a 2.5 Gbps operating speed.

The original speed limitation was resolved at the logical LAG level.

**Remaining validation:** Verify both member interfaces individually at 2.5 Gbps and confirm their LACP aggregation state.

## 7. Root Cause Analysis

**Confirmed finding:** LAG1 was configured with a speed setting of 1000 Mbps.

**Observed effect:** Physical interfaces capable of negotiating at 2.5 Gbps operated at 1 Gbps when participating in the LAG.

**Corrective action:** Changed the LAG speed setting to 2.5 Gbps through the Grandstream Web UI.

**Unconfirmed:** Whether the firmware update modified or restored the previous LAG speed setting.

The speed limitation and the preceding link-flapping incident should therefore be treated as two separate findings unless further evidence establishes a direct relationship.

## 8. Lessons Learned

- An operational LACP aggregation does not guarantee that its physical members operate at their maximum supported speed.
- Troubleshooting should distinguish between physical interface settings and logical aggregation settings.
- Testing the same cable and interface outside the aggregation group can help isolate the cause.
- Configuration should be validated after firmware updates without assuming that every subsequent issue was caused by the update.
- Diagnostic conclusions should distinguish confirmed causes from hypotheses.
- Troubleshooting is more effective when variables are isolated and changed individually.

**Key takeaway:**

> A working LACP bundle does not guarantee maximum link speed. In this case, isolating a physical interface from its aggregation group helped identify a speed limitation configured at the logical LAG level.