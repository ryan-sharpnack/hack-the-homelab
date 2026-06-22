# Hack the Homelab

## Building Enterprise Security Architecture from the Ground Up

This repository documents the construction and operation of a fully 
enterprise-grade security homelab under the fictional organization 
**Nexus Dynamics Corporation (NDC)** — built to develop hands-on skills 
in systems administration, security engineering, and defensive 
architecture.

---

## Infrastructure Overview

| Asset | Role | IP |
|---|---|---|
| Kali Linux | Attack platform / Host OS | 192.168.123.10 |
| DC01 | Windows Server 2025 — Active Directory | 192.168.123.20 |
| NDC-FS01 | Windows File Server / IIS | 192.168.123.25 |
| SRV01 | Ubuntu Server 26.04 | 192.168.123.30 |
| SIEM01 | Ubuntu 22.04 — Wazuh 4.7.5 SIEM | 192.168.123.35 |
| RHEL01 | Rocky Linux 10 — SELinux enforcing | 192.168.123.40 |
| WS01 | Windows 11 Enterprise | DHCP |
| FW01 | pfSense CE Firewall | 192.168.123.2 |

---

## Virtual Networks

| Network | Subnet | Purpose |
|---|---|---|
| pentest-lab | 192.168.123.0/24 | Primary lab network — all VMs |
| nexus-dmz | 192.168.200.0/24 | DMZ — future hosts |
| default NAT | 192.168.122.0/24 | Temporary internet access lane |

---

## Completed Phases

**Phase 1 — Core Infrastructure**
- Kali Linux bare metal host with KVM/QEMU hypervisor
- Windows Server 2025 Active Directory domain (homelab.local)
- Organizational units, user accounts, and Kerberoastable service accounts
- Deliberate misconfigurations for attack simulation
- Seven GPOs configured including audit policy

**Phase 2 — Network Security**
- pfSense CE firewall deployment
- pentest-lab and nexus-dmz network segmentation
- Inter-network traffic control

**Phase 3 — Security Monitoring**
- Wazuh 4.7.5 SIEM deployed on dedicated Ubuntu 22.04 VM
- Agents enrolled: DC01 (Windows Server), WS01 (Windows 11), 
  RHEL01 (Rocky Linux)
- Advanced audit policy configured via GPO
- Custom brute force detection rule — MITRE ATT&CK T1110
- Compliance mapping: GDPR, HIPAA, PCI-DSS, NIST 800-53
- Nessus Essentials deployed for vulnerability management
- Rocky Linux 10 VM deployed with SELinux enforcing

---

## Repository Structure


/network          — Network diagrams and IP schema
/active-directory — OU structure, GPO configs, AD documentation
/wazuh            — Custom detection rules, agent configs
/findings         — Vulnerability findings register
/runbooks         — Standard operating procedures
/notes            — Daily session notes


---

## Skills Demonstrated

Active Directory · Windows Server · Linux Administration ·
pfSense · KVM Virtualization · Wazuh SIEM · Detection Engineering ·
MITRE ATT&CK · SELinux · Vulnerability Management ·
Network Segmentation · GPO Management · Incident Detection

---

## Connect

- YouTube: [Hack the Homelab](https://www.youtube.com/@hackthehomelab)
- LinkedIn: [Ryan Sharpnack](www.linkedin.com/in/ryansharpnack)

---

*This is an ongoing project. New phases and documentation added regularly.*
