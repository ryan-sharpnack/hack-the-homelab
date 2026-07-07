# Evidence

Supporting screenshots and artifacts referenced by journal entries and runbooks — the visual proof behind what's documented in prose elsewhere in the repo.

## Structure

Organized by date, matching the journal's day-based convention:

```
evidence/
  YYYY-MM-DD/
    descriptive-filename.png
```

Each dated folder holds everything captured during that working session, regardless of which runbook or journal entry ends up referencing it. A given day's evidence stays together in one place rather than being scattered across multiple runbook folders.

## Naming convention

Files use lowercase, hyphen-separated names describing their content — not tool-generated timestamps.

- Good: `fw01-libvirt-chain-verification.png`, `vlan10-corp-config.png`
- Avoid: `Screenshot_2026-07-06_23-28-45.png`

## Referencing evidence

Journal entries and runbooks link to evidence images using relative markdown paths:

```
![FW01 interface assignment](../evidence/2026-07-06/fw01-interface-assignment.png)
```

Images are not embedded directly in `/journal/` or `/runbooks/` — this folder is the single source, linked from wherever it's relevant.

## Scope

Build, configuration, and verification evidence only. Diagrams belong in `/diagrams/`; branding assets belong in `/branding/`.
