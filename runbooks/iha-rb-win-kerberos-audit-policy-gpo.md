# IHA-RB-WIN-KERBEROS-AUDIT-POLICY-GPO

**Category:** Windows
**Related:** IHA-004, Day 17 (Phase 4, Task 2)
**Applies to:** Domain controllers requiring Kerberos ticket-request visibility (Authentication Service + Service Ticket Operations)

## Purpose

Enables Advanced Audit Policy logging for Kerberos Authentication Service (event 4768, TGT requests) and Kerberos Service Ticket Operations (event 4769, service ticket requests) via a dedicated, linked GPO — feeding the Kerberoasting-indicative and cross-forest-authentication LogQL detection rules. Establishes a verification method that distinguishes true GPO-driven auditing from a local Windows default that can look identical at first glance.

## Prerequisites

- Domain Admin (or delegated GPO-edit) rights on the target domain
- Access to Group Policy Management Console (`gpmc.msc`) on a domain controller
- Existing Alloy → Loki pipeline already ingesting the Security log from the target DC (see `iha-rb-win-alloy-agent-deployment.md` or the original DC deployment runbook) — this policy generates no value without that pipeline already in place

## Procedure

1. **Baseline check before making changes** — run this first, and don't assume a blank result means nothing is configured:
   ```
   auditpol /get /category:"Account Logon"
   ```
   If Kerberos Authentication Service / Service Ticket Operations show **Success** only (not "Success and Failure"), do not assume a GPO is responsible — this can be a local Windows default with no GPO backing it at all. Confirm by checking whether **Default Domain Controllers Policy** actually has these subcategories configured (`Computer Configuration → Policies → Windows Settings → Security Settings → Advanced Audit Policy Configuration → Audit Policies → Account Logon`). If it shows "Not Configured" across the board, the Success-only baseline is local, not GPO-driven, and needs its own dedicated GPO to be authoritative and documentable.

2. **Create a dedicated GPO** (don't edit Default Domain Controllers Policy directly — keeps this auditable and traceable to a named risk register item):
   - GPMC → right-click **Group Policy Objects** → **New** → name: `IHA-Kerberos-Audit-Policy`

3. **Edit the GPO** → navigate to:
   ```
   Computer Configuration → Policies → Windows Settings → Security Settings →
   Advanced Audit Policy Configuration → Audit Policies → Account Logon
   ```

4. **Enable both subcategories for Success AND Failure:**
   - **Audit Kerberos Authentication Service** — double-click → check "Configure the following audit events" → check both Success and Failure → OK
   - **Audit Kerberos Service Ticket Operations** — same

5. **Enable the legacy-override setting** — critical, and easy to skip:
   ```
   Computer Configuration → Policies → Windows Settings → Security Settings → Local Policies → Security Options
   ```
   Find **"Audit: Force audit policy subcategory settings (Windows Vista or later) to override audit policy category settings"** → Enable. Without this, legacy basic audit categories can silently override the advanced subcategory settings you just configured, resulting in zero events despite a seemingly correct config.

6. **Link the GPO** to the Domain Controllers OU (right-click the OU → **Link an Existing GPO** → select the new GPO).

7. **Force policy refresh and re-verify:**
   ```
   gpupdate /force
   auditpol /get /category:"Account Logon"
   ```
   Confirm both subcategories now read **Success and Failure**.

## Verification

Trigger a real Kerberos event (any normal domain logon generates 4768/4769 traffic) and confirm it lands in Loki:
```
{instance="<DC_HOSTNAME>"} |= "4769"
```

## Troubleshooting / Gotchas

- **A subcategory showing "Success" alone is not proof of a working GPO** — verify the GPO itself is linked and applying (`gpresult /r` or `gpresult /h report.html`) rather than trusting `auditpol` output in isolation. `auditpol` shows the *effective* policy regardless of source (local default, legacy category, or advanced GPO subcategory), so it can't tell you which one is actually responsible.
- **Other subcategories in the same category may reset to "No Auditing"** once a new GPO takes authority over that category — e.g. Credential Validation dropped to unconfigured here, since the new GPO only explicitly set two of the four Account Logon subcategories. Check the full category output after applying, not just the two subcategories you intended to change, so you know what else shifted.
- **GPOs don't cross a one-way forest trust.** If a second forest (e.g. a HAWTHORNE-side DC) also needs this policy, it requires its own separately created and linked GPO in that forest — this GPO only applies within the forest it was created in.

## Related Documentation

- Risk register: IHA-004
- `iha-rb-win-sacl-fim-configuration.md` (same Advanced Audit Policy Configuration path, different category)
