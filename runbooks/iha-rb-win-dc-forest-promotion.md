# Runbook: Domain Controller Build & New Forest Promotion

## Prerequisites
- Confirm target VLAN gateway IP (check pfSense console: Option 2 or GUI)
- Confirm assigned static IP from IP Addressing Scheme doc
- Windows Server ISO available

## 1. VM Creation (virt-manager)
- vCPU: 2 | RAM: 4096 MB | Disk: 60GB qcow2, SATA bus
- Firmware: UEFI (OVMF 4M.fd — no Secure Boot)
- Chipset: Q35
- NIC: attach to correct VLAN bridge (verify — do NOT leave on default NAT)
- Name VM to match documentation convention before Finish (renaming later requires undefine/redefine)

## 2. OS Install & Base Config
- Install Windows Server, log in as Administrator
- Rename computer to match host table, reboot, confirm
- Set static IP: [assigned IP]/24, gateway [VLAN gateway .254], DNS 127.0.0.1
- Verify via `ipconfig /all`

## 3. AD DS Role Install
- Server Manager > Manage > Add Roles and Features > AD DS > Install

## 4. Forest Promotion
- Promote this server to a domain controller > Add a new forest
- Root domain: [DOMAIN.LOCAL]
- Enable DNS Server + Global Catalog, set DSRM password
- Accept DNS delegation warning (expected, no parent zone)
- Default NetBIOS name, default paths
- Review Options > Prerequisites Check > Install (auto-reboots)

## 5. Post-Promotion Verification
- `Get-ADDomain` — confirm DNSRoot, DomainMode, PDCEmulator
- `Get-DnsServerZone` — confirm AD-integrated primary zone + _msdcs subzone
- `net share` — confirm SYSVOL + NETLOGON present
- `nslookup [domain]` — confirm self-resolution
- Event Viewer > DNS Server + Directory Service — check for Error-level entries; single transient DNS 4014 near boot is expected and self-resolves, monitor for recurrence

## Known Non-Issues (do not troubleshoot as failures)
- DNS delegation warning during promotion — expected for standalone root forests
- Single DNS Server Event 4014 shortly after reboot — NTDS/DNS service boot-order race, self-corrects within seconds
