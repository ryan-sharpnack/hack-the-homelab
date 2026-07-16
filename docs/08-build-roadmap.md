# 08 — Build Roadmap

| Phase | Name | Scope | Exit criteria |
|---|---|---|---|
| 0 | Concept & scoping | Fresh-build decision, project scope, hardware confirmation | Complete |
| 1 | Architecture & design | Naming, network/VLAN design, identity/trust design, VM specs, compliance mapping, threat model, risk register seed, documentation conventions | Complete |
| 2 | Foundation build | Stand up FW01, DC01-IRONBRIDGE, DC01-HAWTHORNE; establish the one-way trust; baseline SIEM01 deployment and agent enrollment | Complete |
| 3 | Clinical environment build | Deploy WS01, APP01; enforce VLAN segmentation; run initial Nessus baseline scan | All Phase 1 VMs live; baseline vulnerability scan complete |
| 4 | Hardening & detection validation | GPO hardening, Kerberos audit policy, SIEM rule tuning; confirm each risk register item has a mapped detection or control | Every IHA-00x risk has a documented detection or mitigation in progress |
| 5 | Threat simulation & redesign | Run planned attack scenarios (trust abuse, Kerberoasting, lateral movement) via Kali/BloodHound/Impacket; document findings and redesign fixes; update risk register status | At least one full vulnerable-state → attack → redesign narrative documented end to end |
| 6 | Publication & outreach | Full GitHub documentation push; Beehiiv deep-dive articles; LinkedIn milestone posts; direct outreach to healthcare IT, general IT, and cybersecurity contacts | Content published and outreach conversations underway |

Publish content as each phase completes rather than waiting for the full build — momentum and visible progress are part of the strategy, not just the finished product.
