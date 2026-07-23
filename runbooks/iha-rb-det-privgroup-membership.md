# IHA-RB-DET-PRIVGROUP-MEMBERSHIP
## Detection Runbook: Privileged Group Membership Change

**Alert Rule:** IHA-DET-02 — Privileged Group Membership Change
**Risk ID:** IHA-012
**MITRE ATT&CK:** T1098 (Account Manipulation)
**Severity:** High
**Scenario:** Help-desk social engineering → privileged account takeover

---

### Purpose

Detects additions to high-privilege AD security groups (Domain Admins, Enterprise Admins, Schema Admins, Administrators). Unexpected membership changes to these groups are a common indicator of successful privilege escalation following account takeover.

### Prerequisites

- **Security Group Management** audit subcategory enabled — must be defined at the GPO level (Default Domain Controllers Policy), not just via local `auditpol`, or DC's periodic GPO refresh will silently revert it (see Known Gaps below)
- Grafana Alloy shipping both DCs' Security logs to Loki
- `instance` label populated per host in Loki

### Detection Logic

```logql
count_over_time(
  {instance=~"DC01-.*"}
    | json
    | event_id =~ `4728|4732|4756`
    | event_data =~ ".*Name='TargetUserName'>(Domain Admins|Enterprise Admins|Schema Admins|Administrators)<.*"
  [5m]
) > 0
```

- `4728` — member added to a security-enabled global group
- `4732` — member added to a security-enabled local group
- `4756` — member added to a security-enabled universal group
- `TargetUserName` filter restricts to the four sensitive group names; broader group changes are out of scope for this rule

### Alert Configuration

- **Folder:** IHA - Detections
- **Evaluation group:** IHA-Detections, every 1m
- **Pending period:** None
- **Keep firing for:** None
- **Contact point:** homelab-alerts (placeholder pending SMTP config — Phase 5)
- **Labels:** `severity=high`, `mitre=t1098`, `risk_id=iha-012`, `scenario=helpdesk-privilege-escalation`

### Validation / Test Procedure

1. Confirm audit policy is active at the GPO level:
   ```powershell
   auditpol /get /subcategory:"Security Group Management"
   ```
   If not enabled, define it in `Default Domain Controllers Policy` via `gpmc.msc`:
   `Computer Configuration → Policies → Windows Settings → Security Settings → Advanced Audit Policy Configuration → Account Management → Security Group Management → Define: Success`
   Then force refresh: `gpupdate /force`
2. Add the test account to a privileged group:
   ```powershell
   Add-ADGroupMember -Identity "Domain Admins" -Members "svc-test-kerb"
   ```
3. Confirm Event ID 4728 in Event Viewer, with Subject (actor) and Member (target account) fields populated correctly.
4. Confirm the alert rule transitions to Firing within one evaluation cycle (~1 min).
5. **Clean up immediately after verification:**
   ```powershell
   Remove-ADGroupMember -Identity "Domain Admins" -Members "svc-test-kerb" -Confirm:$false
   ```

### Evidence Captured

- Event Viewer: 4728, Subject `IRONBRIDGE\Administrator`, Member `svc-test-kerb`, Task Category "Security Group Management"
- Grafana alert rule: Firing state screenshot

### Known Gaps / Follow-ups

- **GPO vs. local auditpol:** Local `auditpol /set` changes on domain controllers do not persist — Default Domain Controllers Policy GPO refresh (default every 5 minutes) silently resets any Advanced Audit Policy subcategory not explicitly defined at the GPO level. Confirmed via Event ID 4719 (audit policy change) timestamp correlation. Any future detection rule relying on a Security log subcategory should be checked against existing GPO coverage before assuming local auditpol is sufficient.
- Real notification delivery not yet active (SMTP unconfigured) — scheduled for Phase 5.
