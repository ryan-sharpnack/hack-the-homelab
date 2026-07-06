# 06 — Threat Model & Risk Register

## Threat model

Built around documented, publicly known healthcare ransomware and breach patterns — not hypothetical scenarios. The goal is a credible "vulnerable state → discovery → redesign" story for each one.

| # | Scenario | Real-world pattern it reflects | Where it lives in this build |
|---|---|---|---|
| T-1 | Legacy trust abuse | Acquired-entity trust relationships used as a lateral movement path in M&A-driven breaches | HAWTHORNE.LOCAL → IRONBRIDGE.LOCAL trust |
| T-2 | Unpatched SMB / lateral spread | The 2017 WannaCry pattern — worm-style spread across an unpatched, flat network | Hospital clinical VLAN (VLAN 20) |
| T-3 | Missing MFA on remote access | Credential-based compromise of remote access without MFA, seen in multiple major recent healthcare breaches | FW01 VPN / remote admin access |
| T-4 | Kerberoasting on service accounts | Credential theft via weak Kerberos service account hygiene | `OU=ServiceAccounts`, IRONBRIDGE.LOCAL |
| T-5 | Weak backup/recovery posture | Ransomware impact amplified by inadequate backup strategy | Contingency planning control (see document 05) |

## Risk register (seed)

| ID | Risk | Site/system | Status |
|---|---|---|---|
| IHA-001 | One-way AD trust exposes corporate forest to compromise via a less-hardened legacy domain | Hawthorne → Ironbridge trust | Open |
| IHA-002 | Flat clinical VLAN increases lateral movement risk between hospital systems | VLAN 20 (Hospital) | Open |
| IHA-003 | Remote administrative access lacks MFA | FW01 | Open |
| IHA-004 | Kerberos audit policy not yet configured; service accounts vulnerable to credential-theft techniques | IRONBRIDGE.LOCAL, ServiceAccounts OU | Open |
| IHA-005 | Backup and recovery strategy not yet defined or tested | All sites | Open |

Update `Status` (Open → Mitigated, with date and remediation summary) as each item is addressed during the build. Tracked over time, this register is itself strong portfolio content — it's the difference between saying you found risks and showing you closed them.
