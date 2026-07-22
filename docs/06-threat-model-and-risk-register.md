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
| T-6 | Host-level cross-VLAN forwarding | Hypervisor/virtualization host misconfiguration enabling a network segmentation bypass independent of firewall policy enforcement | KVM/QEMU host; virbr1/2/3 (VLANs 10, 20, 30) |
| T-7 | Detection/alerting capability gap | Delayed breach detection due to insufficient logging/alerting — a documented contributing factor in extended attacker dwell time across numerous healthcare breach post-mortems | SIEM01 (Loki/Grafana/Alloy stack) |
| T-8 | Unpatched exposed service | Outdated/vulnerable software versions as an initial-access vector — a widely documented pattern across ransomware and opportunistic healthcare breaches | APP01 (10.50.20.20), OpenSSH 9.9 |

## Risk register

| ID | Risk | Site/system | Status |
|---|---|---|---|
| IHA-001 | One-way AD trust exposes corporate forest to compromise via a less-hardened legacy domain | Hawthorne → Ironbridge trust | Open |
| IHA-002 | Flat clinical VLAN increases lateral movement risk between hospital systems | VLAN 20 (Hospital) | Open |
| IHA-003 | Remote administrative access lacks MFA | FW01 | Open |
| IHA-004 | Kerberos audit policy not yet configured; service accounts vulnerable to credential-theft techniques | IRONBRIDGE.LOCAL, ServiceAccounts OU | Open |
| IHA-005 | Backup and recovery strategy not yet defined or tested | All sites | Open |
| IHA-006 | Host (KVM/QEMU) is multi-homed across all VLANs with IP forwarding enabled, creating a theoretical risk of bridging traffic between segments and bypassing FW01 | KVM/QEMU host; virbr1/2/3 (all VLANs) | Verified - Not Exploitable |
| IHA-007 | libvirt dnsmasq actively serving DHCP on VLAN10-Corp, VLAN20-Hosp, and VLAN30-Hawth will conflict with the future AD DHCP server role once DC01-IRONBRIDGE and DC01-HAWTHORNE are configured | KVM/QEMU host; VLAN10-Corp, VLAN20-Hosp, VLAN30-Hawth | Verified - Not Exploitable |
| IHA-008 | Stale/orphaned IP address (192.168.123.10/24) present on host bridge virbr1, outside documented network design | KVM/QEMU host; virbr1 (VLAN10-Corp bridge) | Mitigated |
| IHA-009 | SIEM01's default gateway was configured to 10.50.10.1 at OS install time, rather than the documented FW01 gateway address (10.50.10.254) | SIEM01 (10.50.10.20) | Mitigated |
| IHA-010 | SIEM01 had no outbound firewall rule permitting internet access for OS package updates | FW01 (LAN tab); SIEM01 (10.50.10.20) | Mitigated |
| IHA-011 | SIEM01's Wazuh deployment failed (installer defect) compounded by a host RAM ceiling — Wazuh's 8GB requirement exceeded the 4GB originally budgeted in Doc 04 and was never reconciled. Required architecture pivot to self-hosted Loki + Grafana + Alloy. | SIEM01 (10.50.10.20) | Mitigated |
| IHA-012 | Loki/Grafana/Alloy provides centralized log aggregation but no native alerting or file integrity monitoring (FIM), unlike the originally-planned Wazuh deployment — reduces current detection capability until Phase 4 alert-rule work closes the gap. | SIEM01 (10.50.10.20) | Open |
| IHA-013 | LDAPS (636) non-functional on DC01-IRONBRIDGE; no working server certificate bound | DC01-IRONBRIDGE (IRONBRIDGE.LOCAL) | Open |
| IHA-014 | OpenSSH 9.9 on APP01 below patched version (10.3) — 5 CVEs, CVSS v3.0 8.1 (High) | APP01 (10.50.20.20) | Open |
