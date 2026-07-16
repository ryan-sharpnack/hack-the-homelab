# Ironbridge Health Alliance

Ironbridge Health Alliance (IHA) is a simulated multi-site healthcare enterprise — corporate HQ, a hospital, and a regional clinic acquired mid-build, legacy domain and all — built from the ground up as a security architecture portfolio project.

Healthcare is one of the most complex environments in IT: clinical systems, legacy infrastructure, strict regulatory requirements (HIPAA, HITECH), and a track record of some of the highest-profile security incidents of the past decade. 
This project treats that complexity as a feature, not an obstacle — an environment realistic enough to architect, defend, break, and redesign end to end.

## Repo structure

| Folder | Contents |
|---|---|
| [`/docs`](./docs) | Design documentation — architecture, identity, compliance mapping, threat model, risk register, build roadmap |
| [`/diagrams`](./diagrams) | Network and topology diagrams (SVG) |
| [`/branding`](./branding) | Logo, icon, and social preview assets |
| [`/journal`](./journal) | Day-by-day build log |
| [`/runbooks`](./runbooks) | Reusable technical runbooks (`IHA-RB-[CATEGORY]-[SUBJECT].md`) |
| [`/evidence`](./evidence) | Dated screenshots/artifacts supporting journal and runbook entries |
| [`/configs`](./configs) | Infrastructure config files (e.g., `docker-compose.yml`, `config.alloy`) |

## Quick facts

- **Org:** Ironbridge Health Alliance — Corporate HQ, Ironbridge Regional Medical Center, Hawthorne Family Clinic
- **Root domain:** `IRONBRIDGE.LOCAL` (one-way trust from `HAWTHORNE.LOCAL`)
- **Hardware:** Single-host lab — Dell OptiPlex 3040, 16GB RAM, 500GB HDD, KVM/QEMU
- **Status:** Design phase complete — build in progress

Part of the **Hack the Homelab** project series. `#HackTheHomelab`
