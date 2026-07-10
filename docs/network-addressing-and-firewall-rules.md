# Network Addressing & Firewall Rule Set

**Component:** FW01 (pfSense) — IP Addressing Plan & Inter-VLAN Firewall Rules
**Date:** July 8, 2026 (Day 25)
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

### OPT1 tab (traffic originating from VLAN20-Hosp)

| Source | Destination | Port | Protocol | Purpose |
|---|---|---|---|---|
| 10.50.20.0/24 | 10.50.10.20 (SIEM01) | 1514 | TCP | Wazuh agent event data |
| 10.50.20.0/24 | 10.50.10.20 | 1515 | TCP | Wazuh agent enrollment |

### OPT2 tab (traffic originating from VLAN30-Hawth)

| Source | Destination | Port | Protocol | Purpose |
|---|---|---|---|---|
| 10.50.30.10 (DC-HAWTHORNE) | 10.50.10.20 (SIEM01) | 1514 | TCP | Wazuh agent event data |
| 10.50.30.10 | 10.50.10.20 | 1515 | TCP | Wazuh agent enrollment |
| 10.50.30.10 | 10.50.10.10 (DC-IRONBRIDGE) | 53, 88, 135, 389, 445, 464, 636, 3268, 3269 | TCP/UDP as above | AD trust (return path) |
| 10.50.30.10 | 10.50.10.10 | 49152–65535 | TCP | RPC dynamic range |

---

## Design Notes

**Bidirectional AD trust rules across LAN and OPT2:** pfSense firewall rules are evaluated per-interface based on the direction traffic enters that interface, not by logical relationship between two hosts. Because the IRONBRIDGE.LOCAL ↔ HAWTHORNE.LOCAL AD trust requires bidirectional traffic (DNS, Kerberos, LDAP, RPC, SMB) between DC01-IRONBRIDGE and DC01-HAWTHORNE, matching rule sets are required on both the LAN tab (governing traffic originating from VLAN10-Corp) and the OPT2 tab (governing traffic originating from VLAN30-Hawth). This duplication is intentional pfSense behavior, not a documentation error.

**RPC dynamic range (49152–65535):** this is the widest rule in the set by port count. It reflects the standard Windows RPC dynamic port allocation and is required for AD replication and management traffic. Flagged as a candidate for future hardening — Windows supports restricting RPC to a smaller static range via registry configuration, which would allow this rule to be narrowed in a later phase.

**Reference screenshots:** GUI confirmation of the applied rule sets for each interface are archived at `pfsense-lan-rules-final.png`, `pfsense-opt1-rules-final.png`, and `pfsense-opt2-rules-final.png`.
