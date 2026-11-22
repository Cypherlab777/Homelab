# 🔧 Troubleshooting — LACP Link Flapping Between Grandstream and pfSense

**Date:** November 8–9, 2026  
**Environment:** Personal Homelab  
**Category:** Networking / LACP / Link Flapping / Troubleshooting  
**Status:** Monitoring — no recurrence observed as of November 22, 2026  
**Root cause:** Suspected — Grandstream firmware (leading hypothesis, strong temporal correlation); not confirmed  
**Last update:** November 22, 2026

## TL;DR

- **Symptom:** Intermittent network connectivity and repeated physical link flapping on the LACP uplink between the Grandstream GWN7822P and pfSense, including an approximately two-hour overnight recurrence without configuration changes.
- **Evidence:** Grandstream Ethernet `1/0/23` and pfSense `igc2` logged matching DOWN/UP transitions. LACP was active when inspected, with no reported CRC/input/output errors.
- **Root cause:** **Suspected, not confirmed.** Grandstream firmware is the leading hypothesis: flapping occurred before the update and was not observed again for approximately two weeks afterward. Other changes occurred during the investigation, so the strong temporal correlation does not prove causation.
- **Action:** Updated Grandstream firmware from `1.0.13.6` to `1.0.15.137`, then monitored the connection.
- **Status:** No new link flapping or network instability observed through November 22 (roughly two weeks after the update). The original Hyper-V VM Internet issue also stopped occurring, without an identified cause.
- **Next steps:** Keep monitoring; if flapping returns, test the affected cable/link and check the Intel NIC details one variable at a time.

## Quick Timeline

| Date / Time | Event |
|---|---|
| Nov 8, 17:26 | Grandstream reboot initiated manually from CLI during troubleshooting |
| Nov 8, 17:36:32 | LAG1 and both member ports recorded DOWN; members recovered at 17:36:37 and LAG1 at 17:36:40 |
| Nov 8, 17:53–17:58 | Repeated link-state transitions on Grandstream Ethernet `1/0/23` |
| Nov 8, 17:56:41–17:56:45 | Matching DOWN/UP on Grandstream `1/0/23` and pfSense `igc2` |
| Overnight Nov 8–9 | Approximately two hours of additional link flapping without manual changes |
| Around Nov 9 | Grandstream firmware updated from `1.0.13.6` to `1.0.15.137` |
| Nov 10 | No recurrence observed approximately 24 hours after the update |
| Nov 11 | Separate investigation: LAG1 speed was found configured at 1000 Mbps and changed to 2.5 Gbps |
| Nov 22 | No recurrence observed after approximately two weeks; root cause still unconfirmed |

## 1. Network Environment

| Component | Configuration / Role |
|---|---|
| Firewall | pfSense on CWWK Intel N300, Intel 2.5 GbE interfaces (`igc` driver) |
| Switch | Grandstream GWN7822P |
| LACP protocol | IEEE 802.3ad |
| pfSense aggregation | `lagg0` (`igc1` + `igc2`) |
| Grandstream aggregation | `LAG1` (Ethernet `1/0/23` + `1/0/24`) |
| Affected physical link | Grandstream `1/0/23` ↔ pfSense `igc2` |
| Other member | Grandstream `1/0/24` ↔ pfSense `igc1` |

**Expected behavior:** Both physical members remain stable and the LACP uplink maintains connectivity between the Grandstream switch and pfSense.

## 2. Symptoms

- Some devices intermittently lost Internet or management connectivity, while others still worked.
- The original Hyper-V Windows 11 VM could reach local devices but not the Internet. Its relationship to the subsequent physical link flapping **was never established**.
- The Grandstream sometimes could not reach the pfSense management gateway (`192.168.99.1`).
- The affected Ethernet link repeatedly transitioned between DOWN and UP, and the flapping recurred overnight without manual changes.

## 3. Investigation

### Step 1 — Confirm Link-State Changes in Grandstream Logs

**Command:**

```text
show logging
```

**Evidence (November 8):**

```text
17:53:33 PORT-5-LINK_DOWN: Interface Ethernet1/0/23 link down
17:53:37 PORT-5-LINK_UP:   Interface Ethernet1/0/23 link up
17:56:41 PORT-5-LINK_DOWN: Interface Ethernet1/0/23 link down
17:56:45 PORT-5-LINK_UP:   Interface Ethernet1/0/23 link up
```

**Result:** Actual Ethernet link-state changes were recorded, not just lost pings.

### Step 2 — Correlate Grandstream and pfSense Logs

**Grandstream:**

```text
17:56:41 Ethernet1/0/23 LINK_DOWN
17:56:45 Ethernet1/0/23 LINK_UP
```

**pfSense:**

```text
17:56:41 igc2: link state changed to DOWN
17:56:45 igc2: link state changed to UP
```

**Result:** Matching timestamps on both ends confirmed that the **same physical connection** went down and came back up. These logs do not identify which endpoint or component caused the event.

### Step 3 — Inspect LACP and Interface Counters

**pfSense:**

```sh
ifconfig -a
```

**Relevant observations:**

```text
lagg0: status: active
laggproto lacp lagghash l2,l3,l4
laggport: igc1 flags=1c<ACTIVE,COLLECTING,DISTRIBUTING>
laggport: igc2 flags=1c<ACTIVE,COLLECTING,DISTRIBUTING>
```

Grandstream LACP counters showed the members exchanging LACP packets when inspected. No CRC, input or output errors were reported in the reviewed counters.

**Result:** The bundle was operational at the time of inspection, but its historical physical link stability was not guaranteed by these counters.

### Step 4 — Isolate the Timing of the Incident

- At 17:36, LAG1 and both member ports went down and recovered, roughly ten minutes after the manually initiated switch reboot recorded at 17:26. This timing alone does not establish a causal relationship with the later unexplained flapping.
- The affected member `1/0/23` subsequently flapped repeatedly, with corroboration from pfSense `igc2`.
- The problem occurred again for approximately two hours overnight while no manual changes were being made.
- A pfSense packet-capture test coincided with the return of some connectivity, but **no causal effect of starting the capture was established**.

**Result:** The overnight recurrence made a simple explanation based solely on the Hyper-V/VLAN reconfiguration less credible.

## 4. Corrective Actions

- Updated the Grandstream GWN7822P firmware from `1.0.13.6` to `1.0.15.137`.
- Monitored switch and pfSense behavior for a recurrence after the change.
- No cable replacement, Intel NIC driver/NVM update or confirmed hardware repair was performed during this investigation.

## 5. Root Cause Analysis

- **Confirmed:** The physical link `1/0/23` ↔ `igc2` experienced repeated DOWN/UP transitions, corroborated by both endpoints' logs.
- **Observed:** Flapping recurred without manual changes, then no further occurrences were observed after the firmware upgrade, for approximately two weeks through November 22.
- **Suspected (leading hypothesis):** Grandstream firmware `1.0.13.6`. Flapping occurred before the update to `1.0.15.137` and did not recur during approximately two weeks of observation through November 22. The temporal correlation is strong, but other changes were made during the broader investigation (manual switch reboot, ISP box restart, removal of unused legacy pfSense VLAN definitions, and LAG1 speed correction on November 11). No controlled reproduction, rollback, or vendor confirmation isolated the firmware as the cause.
- **Unconfirmed alternatives:** Cable/connector, physical port, Intel 2.5 GbE NIC or driver/negotiation behavior, or a LACP interaction. **The physical link speed during the flapping window was not recorded**, so a speed-related contribution cannot be ruled in or out; see the [LACP link-speed report](/README.md).
- **Not established:** A Layer 2 loop, broadcast storm, or causal relationship between the original Hyper-V Internet issue and the physical link flapping.

## 6. Validation and Status

| Check | Observation | Status |
|---|---|---|
| Link-state evidence | Matching Grandstream and pfSense logs | Confirmed |
| LACP at inspection time | Both members active, collecting and distributing | Confirmed at inspection |
| New flapping after firmware update | None observed through Nov 22 | No recurrence observed |
| Physical cause of the original flapping | Not isolated | Unknown |
| Original Hyper-V VM Internet issue | No longer occurring, without an explanation | Unexplained |
| Related LAG1 speed correction (Nov 11) | Logical LAG configured to 2.5 Gbps; individual member validation still pending | Separate investigation |

**Assessment:** Grandstream firmware remains the **leading suspect** because of the strong before/after correlation. This is not a confirmed firmware defect or a proven permanent fix.

## 7. Next Steps

If the fault returns:

1. Save complete logs and timestamps from both endpoints.
2. Replace only the affected cable for `1/0/23` ↔ `igc2` and observe the result.
3. Test a compatible alternate switch port in a controlled maintenance window if needed.
4. Record the Intel NIC model, driver and NVM version before considering supported changes.
5. Isolate variables (single-link or different negotiated speed) while maintaining management access.

## 8. Lessons Learned

- Correlating timestamps across two devices can confirm which physical link experienced a fault.
- Active LACP and zero CRC errors at one point in time do **not** rule out earlier link flapping.
- Separate confirmed link-state evidence from explanations that remain hypotheses.
- A strong before/after correlation is valuable evidence, but not proof of a particular firmware defect.
- Change one variable at a time and keep dated follow-up observations.

## References

- [LACP link speed limited to 1 Gbps — separate investigation](../lacp-speed/README.md)
