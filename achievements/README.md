# NDC Achievements

A gamified record of milestones unlocked on the road to becoming a Blue Team Security Architect.

Every achievement represents a real technical accomplishment completed inside the Nexus Dynamics Corporation homelab environment. No simulations. No coursework. Real work, documented publicly.

---

## Why this exists

Security Architecture is built through repetition, depth, and documented evidence of thinking — not certifications alone. This achievement system creates a visible record of that progression, one unlocked milestone at a time.

---

## Achievement log

| Achievement | Unlocked | XP | Stage |
|---|---|---|---|
| 🏆 Homelab Architect | 2026-06-14 | +500 XP | Beginner |
| 🛡️ Network Sentinel | 2026-06-18 | +750 XP | Beginner |
| 📡 SIEM Operator | 2026-06-20 | +750 XP | Beginner |
| 🩸 Attack Path Hunter | 2026-06-24 | +850 XP | Intermediate |
| 📊 Risk Translator | 2026-06-25 | +900 XP | Intermediate |
| 👁️ Visibility Architect | 2026-06-26 | +750 XP | Intermediate |
| 🔐 Detection Engineer | 2026-06-26 | +1000 XP | Intermediate |

**Total XP earned: 5,500**

---

## Achievement descriptions

### 🏆 Homelab Architect
Built and configured a fully operational enterprise simulation environment from scratch on bare metal hardware. 8 VMs across Windows Server 2025, Windows 11 Enterprise, Rocky Linux 10, Ubuntu Server, pfSense CE, and Kali Linux. KVM/QEMU hypervisor. Active Directory domain HOMELAB.LOCAL.

### 🛡️ Network Sentinel
Deployed pfSense CE as a segmentation firewall creating isolated network zones for NDC. Two network segments — pentest-lab (192.168.123.0/24) and nexus-dmz (192.168.200.0/24). Firewall rules, VLAN management, and ZFS filesystem.

### 📡 SIEM Operator
Deployed Wazuh 4.7.5 as enterprise SIEM across the NDC domain. 5 agents enrolled across Windows and Linux assets. Custom brute force detection rule written and loaded. Events mapped to MITRE ATT&CK, PCI-DSS, and GDPR automatically.

### 🩸 Attack Path Hunter
Deployed BloodHound CE v9.3.0 via Docker and ran SharpHound v2.13 against the NDC Active Directory environment. Mapped the full domain attack surface — 4 Kerberoastable accounts identified, shortest path to Domain Admin confirmed at 5 hops.

### 📊 Risk Translator
Developed the TERM framework — Technical Enterprise Risk Management — a five-step cycle for translating technical security findings into business risk language. Published as a foundational blog post on Beehiiv and long-form LinkedIn article. 7 risk register entries with ATT&CK mappings and business impact statements.

### 👁️ Visibility Architect
Extended Wazuh agent coverage from 3 to 5 enrolled assets in a single session. NDC-FS01 (Windows Server 2025 — File Services / IIS) and SRV01 (Ubuntu Server) enrolled as Agent 004 and Agent 005. Two documented architecture gaps closed.

### 🔐 Detection Engineer
Identified a critical detection gap — Kerberos Service Ticket Operations audit policy not enabled on DC01 — meaning Kerberoasting attempts generated zero log evidence. Root cause identified, auditpol fix applied, detection verified end-to-end. 3 Wazuh alerts fired within 1 second of re-simulation. Event ID 4769 captured with source IP, encryption type, and ticket hash. Mapped to HIPAA, PCI-DSS, NIST 800-53, and GDPR automatically.

---

## What comes next

Achievements are unlocked as significant milestones are reached — not on a schedule. Future achievements will cover areas including EDR deployment, internal CA / PKI, OpenSSH remediation, SUID privilege escalation closure, email infrastructure, and NIST CSF full alignment.

---

## Path progression

```
Beginner Stage  ████████░░░░░░░░░░░░  Complete
Intermediate    ████████████░░░░░░░░  In progress
Advanced        ░░░░░░░░░░░░░░░░░░░░  Locked
Architect       ░░░░░░░░░░░░░░░░░░░░  Locked
```

---

*Nexus Dynamics Corporation · Hack the Homelab · Ryan Sharpnack*
*github.com/ryan-sharpnack/hack-the-homelab · linkedin.com/in/ryansharpnack*
