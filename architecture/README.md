# NDC Security Architecture

Architecture documentation for Nexus Dynamics Corporation — 
a fully operational enterprise homelab environment built and 
maintained by Ryan Sharpnack as a Security Architect 
development project.

## What's in this folder

**NDC_Architecture.html** — Living security architecture 
document. Open in any browser. Updated as the environment 
evolves. Includes asset inventory, network topology, security 
controls, TERM risk register summary, active attack paths, 
and documented architecture gaps.

## Environment overview

- Domain: HOMELAB.LOCAL
- Networks: pentest-lab (192.168.123.0/24) · nexus-dmz (192.168.200.0/24)
- Assets: 8 VMs across Windows Server 2025, Rocky Linux 10, Ubuntu Server, pfSense CE, Kali Linux
- SIEM: Wazuh 4.7.5 with custom detection rules mapped to MITRE ATT&CK
- Assessment tools: BloodHound CE, Nessus Essentials

## Architecture evaluation framework

Every asset is evaluated daily through four lenses:

1. **Functionality** — does it do what the business needs?
2. **Exposure** — what attack surface does it create?
3. **Detection** — if attacked, will we know?
4. **Recovery** — what is the path back to normal operations?

## Current posture (2026-06-25)

- 7 risk register entries · 1 mitigated · 1 in progress · 5 open
- Highest CVSS: 9.1
- ATT&CK techniques mapped: 7
- Architecture gaps documented: 8

## Related

- [TERM Risk Register](../findings/) — full risk register with business impact statements
- [Daily ops journal](../notes/) — session-by-session work log
- [Runbooks](../runbooks/) — configuration and deployment documentation
