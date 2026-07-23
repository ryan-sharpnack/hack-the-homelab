# IHA-RB-WIN-SACL-FIM-CONFIGURATION

**Category:** Windows
**Related:** IHA-012, Day 17 (Phase 4, Task 3)
**Applies to:** File Integrity Monitoring on sensitive DC paths (SYSVOL\Policies) across one or more forests, without a dedicated FIM product

## Purpose

Implements a no-new-tool FIM approach using native Windows Object Access auditing (SACLs) on SYSVOL\Policies, feeding the existing Loki/Alloy pipeline. Replaces the FIM capability lost in the Wazuh-to-Loki pivot (see IHA-011) without introducing a new product. Must be repeated per forest — GPOs and, separately, filesystem SACLs, do not cross a forest trust boundary.

## Prerequisites

- Domain Admin rights in each forest being configured
- Alloy already forwarding the Security log from each target DC to Loki (SACL-driven events land in the Security log as event ID 4663/4656 — no separate log source or Alloy config change needed)
- Completed: `iha-rb-win-kerberos-audit-policy-gpo.md` is a useful reference for the GPO-creation mechanics reused here, though this is a different audit category

## Procedure — repeat entirely for each forest

1. **Create a dedicated GPO:**
   GPMC → **Group Policy Objects** → New → `IHA-FIM-Object-Access-Audit-Policy`

2. **Edit the GPO** → navigate to:
   ```
   Computer Configuration → Policies → Windows Settings → Security Settings →
   Advanced Audit Policy Configuration → Audit Policies → Object Access
   ```

3. **Enable Audit File System** for both Success and Failure. (Other Object Access subcategories can remain Not Configured — only File System is needed for this use case.)

4. **Link the GPO** to that forest's Domain Controllers OU, then:
   ```
   gpupdate /force
   auditpol /get /category:"Object Access"
   ```
   Confirm **File System** shows Success and Failure.

5. **Apply the actual SACL** to the target path — enabling the audit category alone logs nothing; Windows only generates events for objects with an explicit SACL attached.
   - Navigate to `C:\Windows\SYSVOL\domain\Policies`
   - Right-click → **Properties** → **Security** tab → **Advanced** → **Auditing** tab → **Add**
   - **Principal:** Everyone · **Type:** All · **Applies to:** This folder, subfolders and files
   - **Click "Clear all" first**, then **"Show advanced permissions"** — the basic permissions list (Read, Write, Modify, etc.) does not include Delete or Change Permissions, which are the actual tamper indicators needed here
   - Check: **Create files/write data**, **Create folders/append data**, **Delete subfolders and files**, **Delete**, **Change permissions**
   - OK through all dialogs to save

6. **Generate a real test event** — don't rely on configuration alone as proof it's working:
   - Create a throwaway GPO (e.g. `IHA-SACL-Test-Delete-Me`)
   - **Delete that same GPO from Group Policy Objects** — not from a different location. A test object created and deleted from the wrong container/path will not trigger the SACL on the intended path, and will look like a silent failure even when the GPO and SACL are both correctly configured.

7. **Verify in Grafana Explore:**
   ```
   {instance="<DC_HOSTNAME>"} |= "4663"
   ```
   Confirm entries land shortly after the test, and that the event data references the correct object path/GUID for the test GPO you just deleted.

## Verification

Full closure requires all of the following, per host, per forest:
- GPO created, linked, and confirmed via `auditpol` showing File System = Success and Failure
- SACL confirmed present under Advanced → Auditing on the target folder
- A real test event (4663) confirmed landing in Loki for that specific `instance` label

Configuration alone (steps 1–5) is not sufficient to consider this closed — the verified test event (steps 6–7) is the actual exit condition.

## Troubleshooting / Gotchas

- **"auditpol shows Success and Failure but no events appear in Loki"** — almost always means either (a) no SACL has actually been applied to the object yet (category enabled ≠ object audited), or (b) the test action was performed against the wrong object/path, so the SACL never triggered. Re-check the Auditing tab on the exact folder before assuming a pipeline problem.
- **GPOs and SACLs are both forest-scoped.** A trust relationship (even a fully configured cross-forest trust) does not propagate either GPO application or filesystem auditing settings across forests. Each forest needs its own independent build of this entire procedure.
- **A pre-existing, narrower auditing entry may already be present** on the target folder (e.g. a default `Everyone / Success / Delete / This folder only` entry) before you add your own. This is harmless and redundant with a broader entry added per this runbook — no need to remove it, just don't mistake it for evidence that full coverage is already in place.

## Related Documentation

- Risk register: IHA-011, IHA-012
- `iha-rb-win-kerberos-audit-policy-gpo.md` (same Advanced Audit Policy Configuration path, different category)
- Doc 05 (Compliance & GRC Mapping) — AU-6 row, referenced from IHA-012
