# IHA-RB-DET-KERBEROASTING
## Detection Runbook: Kerberoasting-Indicative Kerberos Ticket Request

**Alert Rule:** IHA-DET-01 — Kerberoasting Indicative (RC4 TGS Request)
**Risk ID:** IHA-012
**MITRE ATT&CK:** T1558.003 (Steal or Forge Kerberos Tickets: Kerberoasting)
**Severity:** High
**Scenario:** Help-desk social engineering → privileged account takeover

---

### Purpose

Detects Kerberos service ticket (TGS) requests encrypted with RC4-HMAC (`0x17`) rather than AES. Modern Windows clients request AES by default (`0x11`/`0x12`), so RC4 requests — particularly against service accounts — are a strong indicator of Kerberoasting reconnaissance or attack activity, since RC4-encrypted ticket hashes are far cheaper to crack offline than AES-encrypted ones.

### Prerequisites

- Kerberos Service Ticket Operations auditing enabled (Event ID 4769 present in Security log)
- Grafana Alloy shipping DC01-IRONBRIDGE Security log to Loki
- `instance` label populated per host in Loki

### Detection Logic

```logql
count_over_time(
  {instance="DC01-IRONBRIDGE"}
    | json
    | event_id = `4769`
    | event_data =~ ".*Name='TicketEncryptionType'>0x17.*"
    | event_data !~ ".*Name='ServiceName'>[^<]*\\$.*"
  [5m]
) > 0
```

- `event_id = 4769` — Kerberos service ticket requested
- `TicketEncryptionType=0x17` — RC4-HMAC, the Kerberoasting-indicative encryption type
- `ServiceName` exclusion filter — excludes machine accounts (`$` suffix), which legitimately use RC4 in some environments and are not useful Kerberoasting signal

### Alert Configuration

- **Folder:** IHA - Detections
- **Evaluation group:** IHA-Detections, every 1m
- **Pending period:** None (rare/binary event — fire on first occurrence)
- **Keep firing for:** None
- **Contact point:** homelab-alerts (placeholder pending SMTP config — Phase 5)
- **Labels:** `severity=high`, `mitre=t1558.003`, `risk_id=iha-012`, `scenario=helpdesk-privilege-escalation`

### Validation / Test Procedure

1. On DC01-IRONBRIDGE, create a throwaway service account with an SPN:
   ```powershell
   New-ADUser -Name "svc-test-kerb" -SamAccountName "svc-test-kerb" `
     -AccountPassword (ConvertTo-SecureString "P@ssw0rd123!" -AsPlainText -Force) `
     -Enabled $true -PasswordNeverExpires $true
   setspn -A HTTP/appsrv01.ironbridge.local IRONBRIDGE\svc-test-kerb
   ```
2. Force RC4-only support (recreates the real-world root cause — a legacy account that never had AES enabled):
   ```powershell
   Set-ADUser -Identity "svc-test-kerb" -Replace @{"msDS-SupportedEncryptionTypes"=4}
   ```
3. From WS01, as a regular domain user, request a ticket natively:
   ```powershell
   Add-Type -AssemblyName System.IdentityModel
   New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList "HTTP/appsrv01.ironbridge.local"
   ```
4. Confirm Event ID 4769 with `TicketEncryptionType: 0x17` in Event Viewer on DC01-IRONBRIDGE.
5. Confirm the same event is queryable in Grafana Explore.
6. Confirm the alert rule transitions to Firing within one evaluation cycle (~1 min).

### Evidence Captured

- Event Viewer: 4769, `0x17`, `svc-test-kerb`
- Grafana Explore: isolated single-line query result
- Grafana alert rule: Firing state screenshot

### Known Gaps / Follow-ups

- Real notification delivery not yet active (SMTP unconfigured) — alert state visible in Grafana UI only, not yet routed to email. Scheduled for Phase 5.
