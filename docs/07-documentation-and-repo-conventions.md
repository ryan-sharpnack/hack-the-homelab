# 07 — Documentation & Repository Conventions

## Repository

Repo name: `hack-the-homelab` (the umbrella brand repo — Ironbridge Health Alliance is the current flagship project documented within it)

## Folder structure

```
/docs/         → design documentation (01-08)
/journal/      → daily build/ops journal entries
/runbooks/     → reusable technical runbooks
/diagrams/     → network/architecture diagrams
/branding/     → logo, icon, and social preview assets
```

## Runbook naming convention

`IHA-RB-[CATEGORY]-[SUBJECT].md`

Categories:
- `WIN` — Windows systems
- `LNX` — Linux systems
- `NET` — Networking/firewall
- `SEC` — Security hardening/policy
- `SIM` — Attack simulation
- `SIEM` — Detection/SIEM

Example: `IHA-RB-SEC-KERBEROS-AUDIT-POLICY.md`

## Journal convention

`journal/YYYY-MM-DD-day-N.md` — day count starts at Day 1 for this project.

## Commit cadence

End-of-day batched commits, consistent with prior practice — one clean commit per working session rather than many small ones.

## Diagrams

Network topology and VLAN diagrams live in `/diagrams/` as self-contained, portable SVGs.

## Branding

Logo, icon, and social preview assets live in `/branding/`. Palette and PNG regeneration commands are documented in `branding/README.md`.

## Content voice

All published material (LinkedIn, Beehiiv, GitHub write-ups) stays in the established Security Architect voice: calm, analytical, peer-to-peer, outcome-led. Brand hashtag: `#HackTheHomelab`.
