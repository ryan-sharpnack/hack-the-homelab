# 07 — Documentation & Repository Conventions

## Repository

Repo name: `hack-the-homelab` (the umbrella brand repo — Ironbridge Health Alliance is the current flagship project documented within it)

## Folder structure

```
/docs/         → design documentation (01-08)
/journal/      → daily build/ops journal entries
/runbooks/     → reusable technical runbooks
/evidence/     → dated screenshots/artifacts supporting journal and runbook entries
/diagrams/     → network/architecture diagrams
/branding/     → logo, icon, and social preview assets
/configs/      → infrastructure config files committed for reference (e.g., docker-compose.yml, config.alloy)
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

## Evidence

Screenshots and other supporting artifacts live in `/evidence/YYYY-MM-DD/`, one dated folder per working session, matching the journal's date convention. Files use lowercase, hyphen-separated, descriptive names reflecting content (e.g., `fw01-libvirt-chain-verification.png`), not tool-generated timestamps. Journal entries and runbooks link to evidence using relative paths (e.g., `../evidence/2026-07-06/vlan10-corp-config.png`) rather than embedding images directly in `/journal/` or `/runbooks/`. See `evidence/README.md` for full detail.

## Diagrams

Network topology and VLAN diagrams live in `/diagrams/` as self-contained, portable SVGs.

## Branding

Logo, icon, and social preview assets live in `/branding/`. Palette and PNG regeneration commands are documented in `branding/README.md`.

## Content voice

All published material (LinkedIn, Beehiiv, GitHub write-ups) stays in the established Security Architect voice: calm, analytical, peer-to-peer, outcome-led. Brand hashtag: `#HackTheHomelab`.
