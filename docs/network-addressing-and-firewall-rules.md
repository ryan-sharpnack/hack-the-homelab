# Network Addressing & Firewall Rule Set

**Component:** FW01 (pfSense) — IP Addressing Plan & Inter-VLAN Firewall Rules
**Date:** July 8, 2026 (Day 25 of 1,000) — updated July 13, 2026 (IHA Day 8, SIEM01 egress rules) — updated July 15, 2026 (IHA Day 10, Wazuh→Loki pivot: legacy agent ports disabled, Loki push rule added)
**Status:** Implemented and verified in pfSense GUI

---

## IP Addressing Scheme

| Host | VLAN | Subnet | Assigned IP |
|---|---|---|---|
| DC01-IRONBRIDGE | VLAN10-Corp | 10.50.10.0/24 | **10.50.10.10** |
| SIEM01 | VLAN10-Corp | 10.50.10.0/24 | **10.50.10.20** |
| WS01 | VLAN20-Hosp | 10.50.20.0/24 | **10.50.20.10** |
| APP01 | VLAN20-Hosp | 10.50.20.0/24 | **10.50.20.20** |
| DC01-HAWTHORNE | VLAN30-Hawth | 10.50.30.0/24 | **10.50.30.10** |

**Convention:** `.1–.9` reserved for future infra/gateway aliases. `.10`, `.20`, `.30`... reserved for statically-assigned servers. Gateway for each VLAN is `.254`, held by FW01 (pfSense LAN/OPT1/OPT2 interfaces).

> **⚠️ Known gotcha — `.1` vs. `.254`:** On this build, libvirt's isolated virtual networks (`vlan10-corp`, `vlan20-hosp`, `vlan30-hawth`) each assign the **host's own bridge interface** (`virbr1`, etc.) the `.1` address on their subnet — e.g., `10.50.10.1` on `virbr1` — independent of whether FW01 is powered on. This is normal libvirt behavior for isolated networks, not a misconfiguration, and it falls within the `.1–.9` reserved range above.
>
> The risk: if a guest VM's default gateway is ever mistakenly set to `.1` instead of the documented `.254`, the host bridge will silently answer ARP/ICMP for `.1`, making the VM *appear* to have working connectivity to "a gateway" even when FW01 is completely powered off. This exact scenario occurred on SIEM01 (its OS installer set the gateway to `10.50.10.1` at install time) and cost real troubleshooting time before the root cause was found — the symptom looked like a routing/NAT problem when it was actually a wrong gateway address masked by an unrelated host-level address.
>
> **Takeaway:** `.254` is the only valid gateway address for any guest VM on these subnets. If a VM can ping its "gateway" while FW01 is shut down, that is a red flag, not a good sign — treat it as a configuration error to investigate, not a connectivity success.

---

## Firewall Rule Set

All rules below sit on a default-deny base — no "allow any" rules exist on any interface. pfSense evaluates rules per-interface based on the direction traffic *enters*, so bidirectional relationships (e.g., the AD trust) require matching rule sets on both interfaces involved.

### LAN tab (traffic originating from VLAN10-Corp)

| Source | Destination | Port | Protocol | Purpose |
|---|---|---|---|---|
| 10.50.10.10 (DC-IRONBRIDGE) | 10.50.30.10 (DC-HAWTHORNE) | 53 | TCP/UDP | DNS |
| 10.50.10.10 | 10.50.30.10 | 88 | TCP/UDP | Kerberos |
| 10.50.10.10 | 10.50.30.10 | 135 | TCP | RPC Endpoint Mapper |
| 10.50.10.10 | 10.50.30.10 | 389 | TCP/UDP | LDAP |
| 10.50.10.10 | 10.50.30.10 | 445 | TCP | SMB |
| 10.50.10.10 | 10.50.30.10 | 464 | TCP/UDP | Kerberos password change |
| 10.50.10.10 | 10.50.30.10 | 636 | TCP | LDAP SSL |
| 10.50.10.10 | 10.50.30.10 | 3268 | TCP | Global Catalog LDAP |
| 10.50.10.10 | 10.50.30.10 | 3269 | TCP | Global Catalog LDAP SSL |
| 10.50.10.10 | 10.50.30.10 | 49152–65535 | TCP | RPC dynamic range |
| 10.50.10.20 (SIEM01) | Any / WAN | 80 (HTTP) | TCP | SIEM01 outbound HTTP (apt/updates) |
| 10.50.10.20 (SIEM01) | Any / WAN | 443 (HTTPS) | TCP | SIEM01 outbound HTTPS (apt/updates) |
| 10.50.10.20 (SIEM01) | Any / WAN | 53 (DNS) | UDP | SIEM01 outbound DNS resolution |

### OPT1 tab (traffic originating from VLAN20-Hosp)

| Source | Destination | Port | Protocol | Purpose | Status |
|---|---|---|---|---|---|
| 10.50.20.0/24 | 10.50.10.20 (SIEM01) | 1514 | TCP | Legacy Wazuh agent event data | **Disabled 2026-07-15** — superseded by Loki/Alloy pivot |
| 10.50.20.0/24 | 10.50.10.20 | 1515 | TCP | Legacy Wazuh agent enrollment | **Disabled 2026-07-15** — superseded by Loki/Alloy pivot |

### OPT2 tab (traffic originating from VLAN30-Hawth)

| Source | Destination | Port | Protocol | Purpose | Status |
|---|---|---|---|---|---|
| 10.50.30.10 (DC-HAWTHORNE) | 10.50.10.20 (SIEM01) | 1514 | TCP | Legacy Wazuh agent event data | **Disabled 2026-07-15** — superseded by Loki/Alloy pivot |
| 10.50.30.10 | 10.50.10.20 | 1515 | TCP | Legacy Wazuh agent enrollment | **Disabled 2026-07-15** — superseded by Loki/Alloy pivot |
| 10.50.30.10 | 10.50.10.20 | 3100 | TCP | Grafana Alloy → Loki push (Windows Event Log forwarding) | **Active, added 2026-07-15** |
| 10.50.30.10 | 10.50.10.10 (DC-IRONBRIDGE) | 53, 88, 135, 389, 445, 464, 636, 3268, 3269 | TCP/UDP as above | AD trust (return path) | Active |
| 10.50.30.10 | 10.50.10.10 | 49152–65535 | TCP | RPC dynamic range | Active |

---

## Design Notes

**Bidirectional AD trust rules across LAN and OPT2:** pfSense firewall rules are evaluated per-interface based on the direction traffic enters that interface, not by logical relationship between two hosts. Because the IRONBRIDGE.LOCAL ↔ HAWTHORNE.LOCAL AD trust requires bidirectional traffic (DNS, Kerberos, LDAP, RPC, SMB) between DC01-IRONBRIDGE and DC01-HAWTHORNE, matching rule sets are required on both the LAN tab (governing traffic originating from VLAN10-Corp) and the OPT2 tab (governing traffic originating from VLAN30-Hawth). This duplication is intentional pfSense behavior, not a documentation error.

**RPC dynamic range (49152–65535):** this is the widest rule in the set by port count. It reflects the standard Windows RPC dynamic port allocation and is required for AD replication and management traffic. Flagged as a candidate for future hardening — Windows supports restricting RPC to a smaller static range via registry configuration, which would allow this rule to be narrowed in a later phase.

**SIEM01 outbound egress (added Day 8, 2026-07-13):** SIEM01 required outbound internet access for OS package updates. Rather than a broad "allow any outbound" rule, three narrowly scoped rules were added — one host (`10.50.10.20`), one purpose each (HTTP, HTTPS, DNS) — consistent with the least-privilege posture applied throughout this rule set. ICMP (ping) was deliberately left unpermitted, as it was never an actual requirement, only a diagnostic convenience during troubleshooting; ping to `1.1.1.1` from SIEM01 is expected to fail even with these rules in place.

**Wazuh → Loki/Grafana/Alloy pivot (2026-07-15, IHA Day 10):** SIEM01's original Wazuh deployment was replaced due to an installer defect combined with a hardware resource ceiling on the host (see risk register / journal for full rationale). This changed the firewall requirements: the legacy Wazuh agent ports (1514 enrollment, 1515 event data) on OPT1 and OPT2 were **disabled, not deleted**, preserving the audit trail of what was originally provisioned and why it changed. A new rule was added on OPT2 — `10.50.30.10 → 10.50.10.20:3100 TCP` — scoped to the single HAWTHORNE host rather than the full VLAN30 subnet, consistent with the least-privilege pattern already applied to the AD-trust rules on this same interface. No equivalent rule was needed on the LAN tab for DC01-IRONBRIDGE, since IRONBRIDGE and SIEM01 share the same L2 segment and that traffic never traverses FW01 at all. The OPT1 legacy rules remain disabled with no replacement, since VLAN20 (WS01/APP01) has no active log-forwarding agent at this time.

**Reference screenshots:** GUI confirmation of the applied rule sets for each interface are archived at `day25-14-pfsense-lan-rules-final.png`, `day25-15-pfsense-opt1-rules-final.png`, `day25-16-pfsense-opt2-rules-final.png`, and `siem01-pfsense-egress-rules.png` (evidence folder: `evidence/2026-07-13/`). *(Add today's OPT1/OPT2 rule-state screenshots to `evidence/2026-07-15/` and reference them here once captured.)*
