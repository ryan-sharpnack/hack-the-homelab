# 05 — Compliance & GRC Mapping

Every technical control built into Ironbridge Health Alliance is mapped to the regulatory or framework language a real healthcare compliance function would use. This is the GRC-translation skill — the ability to state a technical finding in the business/regulatory language executives and auditors actually use.

| Control area | Technical control | HIPAA | NIST 800-53 | NIST 800-66 | CIS Controls |
|---|---|---|---|---|---|
| Network segmentation | VLAN isolation between corporate, clinical, and legacy segments | §164.312(a)(1) Access Control | SC-7 Boundary Protection | Access control safeguards | CIS Control 12 — Network Infrastructure Management |
| Audit logging & monitoring | Centralized log aggregation via Grafana Loki + Alloy (Windows Event Log collection, both forests); alerting rules and file integrity monitoring (FIM) not yet implemented — tracked as Phase 3 backlog | §164.312(b) Audit Controls | AU-2 (implemented — log generation/retention), AU-6 (pending — no active alerting configured) | Audit controls implementation — partial | CIS Control 8 — Audit Log Management (logging implemented; alerting pending) |
| Identity & access management | AD hardening, GPO baseline, trust scoping | §164.312(a)(2)(i) Unique User ID | AC-2, AC-6 | Person/entity authentication | CIS Control 5/6 — Account & Access Management |
| Vulnerability management | Nessus scanning, patch cadence | §164.308(a)(1) Risk Analysis | RA-5 Vulnerability Scanning | Risk analysis & management | CIS Control 7 — Continuous Vulnerability Management |
| Contingency planning | Backup and recovery design | §164.308(a)(7) Contingency Plan | CP-9, CP-10 | Data backup plan | CIS Control 11 — Data Recovery |
| Transmission security | Encrypted remote access, MFA on VPN/admin access | §164.312(e)(1) Transmission Security | SC-8, IA-2 | Transmission security | CIS Control 6.5 — MFA for Remote Access |

This table is a living document — update it as each control is actually implemented during the build, not just planned. The delta between "planned" and "implemented" is itself worth tracking; it's evidence of follow-through.

**Revision note (2026-07-15, IHA Day 9/10):** the Audit logging & monitoring row was updated following the Wazuh → Loki/Grafana/Alloy pivot. This is a deliberate re-scoping, not a like-for-like renaming — Wazuh natively provided alerting and FIM as built-in capabilities that the current stack does not yet replicate. AU-6 and the CIS 8 alerting component are marked pending rather than implemented until the Phase 3 LogQL alert-rule work closes that gap. FIM specifically has no current equivalent in this stack and would require a separate tool if that capability is to be represented as real rather than planned.
