# 03 — Identity Architecture

## Forests and domains

| Domain | Role | Hardening posture |
|---|---|---|
| IRONBRIDGE.LOCAL | Root forest — Corporate + Hospital | Modern, actively hardened (Phase 1 baseline, tightened through the build) |
| HAWTHORNE.LOCAL | Legacy forest — acquired clinic | Deliberately under-hardened at first, representing real acquired-entity tech debt |

## Trust relationship

**IRONBRIDGE.LOCAL trusts HAWTHORNE.LOCAL (one-way).**

This is the realistic direction for an acquisition: Hawthorne's staff need access to shared corporate resources (email, file shares, VPN), so the parent forest extends trust to the acquired one. It also defines the attack surface at the center of this project's story — Hawthorne is the weaker, older domain, and the trust is the path a compromise there could ride into the corporate forest.

**Design decision:** the trust starts in its default, broad configuration in Phase 1 (no selective authentication) — matching how these trusts usually exist in the real world immediately post-acquisition. Scoping it down with selective authentication is a planned Phase 4/5 redesign deliverable, not a Phase 1 requirement. Building it insecurely first, then fixing it, is more valuable content than building it correctly from day one.

## OU structure — IRONBRIDGE.LOCAL

- `OU=Corporate` — HQ staff, SOC/NOC accounts
- `OU=Hospital` — clinical staff, WS01
- `OU=ServiceAccounts` — scoped, monitored for credential-theft techniques (e.g. Kerberoasting)
- `OU=Admins` — administrative accounts (tiered admin model documented as a target-state improvement even if simplified at lab scale)

## OU structure — HAWTHORNE.LOCAL

- Minimal, flat structure — intentionally reflects a smaller, less-mature legacy environment
- Local clinic staff accounts only

## Account and naming conventions

- Service accounts prefixed `svc-` for easy identification and monitoring
- Admin accounts separate from daily-use accounts for every human identity
