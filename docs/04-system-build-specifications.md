# 04 — System Build Specifications

## VM inventory (Phase 1)

| Host | Role | OS | RAM | Disk |
|---|---|---|---|---|
| DC01-IRONBRIDGE | Root domain controller, DNS, GPO | Windows Server 2022/2025 | 2GB | 40GB |
| DC01-HAWTHORNE | Legacy domain controller | Windows Server 2016/2019 | 2GB | 40GB |
| SIEM01 | Wazuh manager + indexer + dashboard (self-managed, open source) | Ubuntu/Rocky Linux | 4GB | 60GB |
| FW01 | pfSense — inter-VLAN routing, WAN NAT | pfSense CE | 1GB | 10GB |
| WS01 | Clinical workstation | Windows 11 Enterprise | 2GB | 40GB |
| APP01 | Mock clinical application stand-in (EHR/imaging/pharmacy) | Linux | 1GB | 20GB |
| **Total** | | | **12GB** | **210GB** |

Kali Linux remains the bare-metal host and attacker platform — no RAM cost against the guest budget above.

## Hardware budget

- Host: Dell OptiPlex 3040, 16GB RAM, 500GB HDD, KVM/QEMU
- 12GB allocated to guests leaves a thin but workable margin for the hypervisor; VMs are not all run concurrently in every session

## Asset inventory — real vs. simulated

| System | Status |
|---|---|
| Active Directory (both forests) | Real, functional |
| Wazuh SIEM | Real, functional |
| pfSense firewall | Real, functional |
| Windows 11 workstation | Real, functional |
| EHR / imaging / pharmacy application layer (APP01) | Simulated — generic application stand-in, positioned and segmented the way real clinical software would be. Real licensed EHR software (Epic/Cerner) is not available for lab use and is explicitly not represented as real. |

**Optional Phase 2+ enhancement:** deploy OpenEMR (a legitimate open-source EHR platform) on APP01 to generate more realistic clinical-application traffic patterns. Not required for Phase 1.

## If hardware pressure forces trims, in this priority order

1. Drop APP01 first — document the segment and its intended role on paper, add the VM later (saves 1GB)
2. Reduce SIEM01's indexer allocation to 3GB if log volume stays low (functional but more constrained)
3. Continue running only the VMs needed per session, rather than the full fleet at once (primary mitigation, already assumed)
4. Last resort: retire WS01 and work from the DCs directly (saves 2GB, reduces realism — avoid if possible)
