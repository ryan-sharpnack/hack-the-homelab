# Runbooks

Reusable technical procedures for Ironbridge Health Alliance — the clean, repeatable version of work first recorded in `/journal`.

## Naming convention

`IHA-RB-[CATEGORY]-[SUBJECT].md`

| Category | Covers |
|---|---|
| `WIN` | Windows systems |
| `LNX` | Linux systems |
| `NET` | Networking / firewall |
| `SEC` | Security hardening / policy |
| `SIM` | Attack simulation |
| `SIEM` | Detection / SIEM |

Example: `IHA-RB-SEC-KERBEROS-AUDIT-POLICY.md`

## What goes in a runbook

- Purpose — what this procedure achieves and why it matters
- Prerequisites
- Step-by-step procedure (specific enough to repeat exactly)
- Verification — how to confirm it worked
- Related risk register item(s), if applicable (document 06)

A runbook should be written so that following it exactly reproduces the result — no memory of "what I meant" required.
