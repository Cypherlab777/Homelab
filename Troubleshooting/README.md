# 🔧 Network Troubleshooting — Homelab

Real troubleshooting investigations conducted in my personal homelab. Each report focuses on symptoms, evidence, diagnostic steps, corrective actions, validation, and remaining uncertainty.

## Incident Reports

| Incident | Date | Status | Root cause |
|---|---|---|---|
| [LACP link flapping](2026-11-08-link-flapping//README.md) | Nov 8–9, 2026 | Monitoring | Suspected — firmware; unconfirmed |
| [LACP link speed limited to 1 Gbps](2026-11-11-lacp-speed/README.md) | Nov 11, 2026 | Partially resolved — physical checks pending | Confirmed — LAG1 speed setting |
| [IPv6 Prefix Delegation and SLAAC on pfSense](/2026-11-15-ipv6-pfsense//README.md) | Nov 15–16, 2026 | Resolved | Mixed — cause varies by issue |

## Documentation Format

All reports follow a shared [incident report template](Template/INCIDENT_REPORT_TEMPLATE.md). **Confirmed** findings are distinguished from **suspected** explanations and **unconfirmed** causes. A *Mixed* designation means a report covers multiple issues with different levels of certainty.

## AI Assistance

AI tools were used to support technical research, log analysis, and documentation.
