# 04 — System Build Specifications

## VM inventory (Phase 1)

| Host | Role | OS | RAM | Disk |
|---|---|---|---|---|
| DC01-IRONBRIDGE | Root domain controller, DNS, GPO | Windows Server 2022/2025 | 2GB | 40GB |
| DC01-HAWTHORNE | Legacy domain controller | Windows Server 2016/2019 | 2GB | 40GB |
| SIEM01 | Loki + Grafana + Alloy log aggregation stack (self-managed, open source, container-based via Docker Compose) | Ubuntu 22.04.5 | 4GB | 60GB |
| FW01 | pfSense — inter-VLAN routing, WAN NAT | pfSense CE | 1GB | 10GB |
| WS01 | Clinical workstation | Windows 11 Enterprise | 2GB | 40GB |
| APP01 | Mock clinical application stand-in (EHR/imaging/pharmacy) | Linux | 1GB | 20GB |
| **Total** | | | **12GB** | **210GB** |

> **Note (2026-07-15, IHA Day 10):** SIEM01 was originally deployed running Wazuh, which required 8GB RAM — a deviation from this table's original 4GB spec that was never reconciled here at the time. Following the pivot to Loki/Grafana/Alloy (installer defect + hardware resource ceiling; see risk register), SIEM01 was resized back to 4GB. This restores the original Phase 1 hardware budget above rather than representing a new cut — the 12GB guest total is accurate again for the first time since Wazuh was stood up.

Kali Linux remains the bare-metal host and attacker platform — no RAM cost against the guest budget above.

## Hardware budget

- Host: Dell OptiPlex 3040, 16GB RAM, 500GB HDD, KVM/QEMU
- 12GB allocated to guests leaves a thin but workable margin for the hypervisor; VMs are not all run concurrently in every session

## Asset inventory — real vs. simulated

| System | Status |
|---|---|
| Active Directory (both forests) | Real, functional |
| Loki + Grafana SIEM | Real, functional |
| pfSense firewall | Real, functional |
| Windows 11 workstation | Real, functional |
| EHR / imaging / pharmacy application layer (APP01) | Simulated — generic application stand-in, positioned and segmented the way real clinical software would be. Real licensed EHR software (Epic/Cerner) is not available for lab use and is explicitly not represented as real. |

**Optional Phase 2+ enhancement:** deploy OpenEMR (a legitimate open-source EHR platform) on APP01 to generate more realistic clinical-application traffic patterns. Not required for Phase 1.

## If hardware pressure forces trims, in this priority order

1. Drop APP01 first — document the segment and its intended role on paper, add the VM later (saves 1GB)
2. Reduce SIEM01's RAM further (e.g., to 2GB) if log volume stays low and `docker stats` shows consistent headroom (functional but more constrained — re-validate against actual container usage before committing to this, per Day 9/10 sizing notes)
3. Continue running only the VMs needed per session, rather than the full fleet at once (primary mitigation, already assumed)
4. Last resort: retire WS01 and work from the DCs directly (saves 2GB, reduces realism — avoid if possible)
