# Ironbridge Health Alliance — Design Documentation

**Status:** Design phase complete — ready for build

## What this is

Ironbridge Health Alliance (IHA) is a simulated multi-site healthcare enterprise built as a hands-on security architecture portfolio project. It exists to demonstrate real-world blue team, GRC, and enterprise security architecture skills — network segmentation, identity architecture, detection engineering, compliance mapping, and risk management — in an environment realistic enough to hold up in front of healthcare IT, general IT, and cybersecurity hiring conversations.

## How to read this

Read in numbered order — each document builds on the last.

| # | Document | Covers |
|---|---|---|
| 01 | Project charter | Purpose, goals, success criteria |
| 02 | Network architecture | VLANs, IP scheme, routing, segmentation |
| 03 | Identity architecture | AD forest design, trust relationship, OU plan |
| 04 | System build specifications | VM list, specs, asset inventory (real vs. simulated) |
| 05 | Compliance & GRC mapping | HIPAA / NIST / CIS control mapping |
| 06 | Threat model & risk register | Threat scenarios and the initial risk register |
| 07 | Documentation & repo conventions | GitHub structure, naming conventions |
| 08 | Build roadmap | Phased build plan, from foundation through publication |

## Org quick reference

- **Organization:** Ironbridge Health Alliance (IHA)
- **Root domain:** IRONBRIDGE.LOCAL
- **Sites:** Corporate HQ, Ironbridge Regional Medical Center (hospital), Hawthorne Family Clinic (acquired, legacy domain)
- **Hardware:** Single-host lab (16GB RAM / 500GB HDD), KVM/QEMU
- **Content brand:** Hack the Homelab
