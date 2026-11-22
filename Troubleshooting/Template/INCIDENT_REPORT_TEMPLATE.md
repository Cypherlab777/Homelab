<!-- Language: English. Keep identical headings and terminology across incident reports. -->
<!-- Put the one-line AI disclosure in the Troubleshooting/README.md index, not in each report. -->
# 🔧 Troubleshooting — [Incident Name]

**Date:** [Month DD–DD, YYYY]  
**Environment:** Personal Homelab  
**Category:** Networking / [Technology] / Troubleshooting  
**Status:** [Open | Monitoring | Partially resolved | Resolved] — [precise scope]  
**Root cause:** [Confirmed | Suspected | Mixed | Unconfirmed | Unknown] — [short explanation]  
**Last update:** [Month DD, YYYY]

## TL;DR

- **Symptom:** [Observable failure; avoid guessing the cause.]
- **Evidence:** [Best concrete log, output or comparison.]
- **Root cause:** [What is proven, suspected or unknown.]
- **Action:** [Correction or workaround actually performed.]
- **Status:** [What works now; what remains unverified.]
- **Next steps:** [Only relevant follow-up work.]

<!-- Optional: include only when the incident has meaningful dated milestones. -->
## Quick Timeline

| Date / Time | Event |
|---|---|
| [Date/time] | [Observed event or documented change] |

## 1. Network Environment

| Component | Configuration / Role |
|---|---|
| [Device / interface] | [Model, firmware, interface, VLAN, IP prefix, role] |

**Expected behavior:** [Precise expected network behavior.]

## 2. Symptoms

- [Symptom 1]
- [Symptom 2]

## 3. Investigation

<!-- If there are several distinct issues, group steps under Issue 1 / Issue 2 / ... and keep stable anchors like <a id="issue-1"></a> for links. -->
### Step 1 — [Test name]

**Action:** [What was tested, with one variable changed at a time.]

**Command / Evidence:**

```text
[Short real command and relevant output]
```

**Result:** [Observation, not speculation.]

### Step 2 — [Test name]

**Action:** [Test performed.]

**Result:** [What was observed and what it does or does not establish.]

## 4. Corrective Actions

- [Change applied, where, and when.]
- [Workaround, if no confirmed fix exists.]

## 5. Root Cause Analysis

- **Confirmed:** [Evidence-backed findings.]
- **Observed:** [Correlations or direct measurements without assigning causation.]
- **Suspected:** [Plausible explanation and what would validate it.]
- **Unconfirmed:** [What cannot yet be established; use *Unknown* if nothing is known.]

<!-- Use Mixed in the header for multi-issue reports with different levels of root-cause certainty. -->

## 6. Validation and Status

| Check | Observed result | Status |
|---|---|---|
| [Verification] | [Actual output] | [Confirmed / Resolved / Pending / Failed] |

## 7. Summary Table

| Finding | Evidence | Action / Outcome |
|---|---|---|
| [Finding] | [Observed data] | [Action and effect] |

<!-- Optional: omit this entire Summary Table when it only repeats Validation and Status; renumber subsequent sections. -->

## 8. Next Steps

- [Only tests necessary to close the remaining uncertainty.]

## 9. Lessons Learned

- [Specific operational lesson that follows from the incident.]

<!-- Optional: only if concrete external vendor docs, issue tickets or related reports exist. -->
## References

- [Verified title and URL, or related internal report link]
