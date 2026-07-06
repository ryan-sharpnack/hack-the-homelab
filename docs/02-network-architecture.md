# 02 — Network Architecture

## Approach

Ironbridge Health Alliance is modeled as a multi-site health system, implemented as a **logical multi-site** design — real network sites represented as segmented VLANs on a single physical host, routed through one pfSense firewall, rather than separate physical hardware. This is a legitimate and common way to represent a WAN in a lab environment, and the segmentation, routing, and trust-boundary work involved is the same skill set required in a real multi-site build.

## Sites

| Site | Description |
|---|---|
| Corporate HQ | Root domain, SOC/NOC, policy control |
| Ironbridge Regional Medical Center | Main hospital — clinical VLAN |
| Hawthorne Family Clinic | Acquired regional clinic — legacy AD domain |

## VLAN and IP scheme

| VLAN | Segment | Subnet | Hosts |
|---|---|---|---|
| 10 | Corporate | 10.50.10.0/24 | DC01-IRONBRIDGE, SIEM01 |
| 20 | Hospital clinical | 10.50.20.0/24 | WS01, APP01 |
| 30 | Hawthorne (legacy) | 10.50.30.0/24 | DC01-HAWTHORNE |

## Routing and segmentation

- FW01 (pfSense) carries one interface per VLAN plus a WAN leg, NAT'd to the host's real network for updates and threat intelligence feeds only.
- Default policy: deny between VLANs, explicit allow rules only where a real business/clinical reason exists (e.g., SIEM01 needs inbound log traffic from agents on all VLANs; Hawthorne's DC needs specific ports open to Ironbridge's DC to support the AD trust).
- **Intentional design flaw for Phase 1:** VLAN 20 (hospital clinical) is initially built flatter than best practice — minimal internal segmentation between WS01 and APP01 — to mirror the real-world "flat clinical network" pattern behind several major healthcare ransomware incidents. This gets tightened in the Phase 4/5 redesign, and the before/after is deliberate content, not an oversight.

## WAN handling

Internet access is limited to what's required for OS/agent updates and threat intel feeds. No inbound exposure from the real internet — this is a closed lab, not an internet-facing target.
