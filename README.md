![Logo](homelab_logo_maxime_belliard.svg)

# 🛡️ Active Directory Security Lab

> Home lab simulating a corporate Windows environment with Active Directory, GPO hardening, and SIEM integration using Wazuh — built as a portfolio project for GRC + Technical security roles in Belgium.

---

## 📋 Overview

This lab demonstrates end-to-end implementation of a Windows Active Directory environment with security hardening, identity management, and real-time threat detection via a SIEM. All components run as virtual machines in VirtualBox on a single host machine.

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────┐
│                  VirtualBox Host                    │
│              Internal Network: intnet               │
│                                                     │
│  ┌──────────────────┐    ┌──────────────────┐       │
│  │ Windows Server   │    │  Windows 10 Pro  │       │
│  │ 2022 — DC        │    │  Client          │       │
│  │ 192.168.100.1    │    │  192.168.100.2   │       │
│  │ lab.local        │    │  joined: lab.local│       │
│  └────────┬─────────┘    └────────┬─────────┘       │
│           │                       │                 │
│           └──────────┬────────────┘                 │
│                      │                              │
│             ┌────────▼─────────┐                    │
│             │   Wazuh 4.14.4   │                    │
│             │   SIEM / OVA     │                    │
│             │  192.168.100.10  │                    │
│             └──────────────────┘                    │
└─────────────────────────────────────────────────────┘
```

| VM | Role | IP | OS |
|---|---|---|---|
| WIN-6PF6FQBBP9F | Domain Controller | 192.168.100.1 | Windows Server 2022 Datacenter Eval |
| DESKTOP-DB2B9Q5 | Client workstation | 192.168.100.2 | Windows 10 Pro |
| wazuh-server | SIEM (all-in-one) | 192.168.100.10 | Wazuh OVA 4.14.4 |

---

## 🗂️ Active Directory Structure

**Domain:** `lab.local`

```
lab.local
├── OU_Users
│   ├── OU_IT
│   │   └── alice.it  →  GRP_Admins
│   ├── OU_Finance
│   │   └── bob.finance  →  GRP_ReadOnly
│   └── OU_Logistics
│       └── carol.logistics  →  GRP_ReadOnly
├── OU_Computers
│   ├── OU_Workstations
│   └── OU_Servers
└── OU_Groups
    ├── GRP_Admins
    └── GRP_ReadOnly
```

This structure reflects the **principle of least privilege** — users are assigned to groups with the minimum permissions required for their role.

---

## 🔒 Security Controls Implemented

### GPO — Password Policy
| Setting | Value |
|---|---|
| Minimum password length | 12 characters |
| Password complexity | Enabled |
| Maximum password age | 90 days |
| Minimum password age | 30 days |

### GPO — Account Lockout
| Setting | Value |
|---|---|
| Lockout threshold | 5 invalid attempts |
| Lockout duration | 30 minutes |
| Reset counter after | 30 minutes |
| Administrator lockout | Enabled |

### GPO — Additional Controls
| Control | Configuration |
|---|---|
| Audit Policy | Logon/Logoff + Account Management (success & failure) |
| USB Storage | Read and write blocked via Computer Config |
| Screen Lock | Auto-lock after 10 minutes (600 seconds) |

---

## 📡 SIEM Integration — Wazuh 4.14.4

Wazuh is deployed as an all-in-one OVA on the same internal network. Both agents report in real time to the Wazuh manager.

### Agents
| ID | Name | IP | Status | Version |
|---|---|---|---|---|
| 001 | WIN-6PF6FQBBP9F | 192.168.100.1 | ✅ Active | v4.14.4 |
| 002 | DESKTOP-DB2B9Q5 | 192.168.100.2 | ✅ Active | v4.14.4 |

### Events Monitored
- **Event ID 4625** — Failed logon attempts (Logon Failure — Unknown user or bad password)
- **Event ID 4740** — Account lockout
- **Event ID 4728** — User added to privileged group (Domain Admins)
- **Rule 60204** — Multiple Windows Logon Failures (correlated brute-force detection, level 10)

### MITRE ATT&CK Coverage Detected
| Technique | ID | Description |
|---|---|---|
| Valid Accounts | T1078 | Legitimate credentials used or targeted |
| Brute Force | T1110 | Repeated failed authentication attempts |
| Password Guessing | T1110.001 | Low-and-slow credential attacks |
| Account Access Removal | T1531 | Account manipulation events |
| Domain Policy Modification | T1484 | GPO change detection |
| Defense Evasion | TA0005 | Tactic-level detection |
| Privilege Escalation | TA0004 | Tactic-level detection |

---

## 🖼️ Screenshots

All screenshots are located in the `/screenshots` directory.

| File | Content |
|---|---|
| WS1-aduc-arborescence.png | ADUC — full OU tree |
| WS2-ou-groups.png | OU_Groups with GRP_Admins and GRP_ReadOnly |
| WS3-grp-admins-membres.png | GRP_Admins members (Alice) |
| WS4-grp-readonly-membres.png | GRP_ReadOnly members (Bob, Carol) |
| WS5-console-gpo.png | GPO console — Password Policy linked to lab.local |
| WS6-gpo-password-policy.png | GPO editor — password settings (12 chars, complexity) |
| WS7-gpo-account-lockout.png | GPO editor — account lockout (5 attempts, 30 min) |
| WS8-wazuh-ip-config.png | Wazuh VM — static IP configuration |
| WS9-wazuh-login.png | Wazuh login page (192.168.100.10) |
| WS10-wazuh-dashboard-overview.png | Wazuh overview — 2 active agents, 1,334+ alerts |
| WS11-wazuh-agents-active.png | Wazuh agents list — both agents active |
| WS12-wazuh-threat-hunting-dashboard.png | Threat Hunting — 18 auth failures, MITRE ATT&CK mapped |
| WS13-wazuh-events-logon-failure.png | Events — Logon Failure events list |
| WS14-wazuh-events-4625-filtered.png | Events filtered on Event ID 4625 — 15 hits |
| WS15-wazuh-mitre-attack-dashboard.png | MITRE ATT&CK dashboard — techniques by agent |

---

## 🎯 What This Demonstrates

| Skill | Evidence |
|---|---|
| Windows Server administration | Domain Controller setup, AD structure, OU design |
| Identity & Access Management (IAM) | Users, groups, least privilege model |
| System hardening | GPO-enforced password policy, lockout, USB block, screen lock |
| SIEM deployment & configuration | Wazuh all-in-one, agent enrollment, rule tuning |
| Threat detection | Real-time 4625 alerts, brute-force correlation (rule 60204) |
| MITRE ATT&CK mapping | T1078, T1110, T1484, T1531 detected and mapped |

---

## 📚 Lessons Learned

See [`lessons-learned.md`](./lessons-learned.md) for a detailed account of challenges encountered and production differences.

Key takeaways:
- Wazuh OVA requires manual static IP assignment when no DHCP is present on the internal network (`ip addr add` — persistent config via `/etc/network/interfaces` or `nmcli` depending on the distro)
- The Windows screen lock GPO at the domain level doesn't override a local policy already set — force a `gpupdate /force` on the client after linking
- `net use` commands generate real network authentication events against the DC, making them the most reliable method to trigger Event ID 4625 in a lab context

---

## ⚖️ Regulatory Alignment

| Framework | Control |
|---|---|
| ISO 27001 | A.9 Access Control, A.9.4 System and application access control |
| ISO 27001 | A.12.4 Logging and monitoring |
| NIS2 Art. 21 | Technical security measures — authentication, access control, monitoring |
| CIS Controls | Control 5 — Account Management, Control 8 — Audit Log Management |

---

## 🛠️ Tools Used

- **VirtualBox** — Hypervisor
- **Windows Server 2022** — Domain Controller (180-day eval)
- **Windows 10 Pro** — Client workstation
- **Wazuh 4.14.4 OVA** — Open source SIEM / XDR
- **Active Directory Users and Computers (ADUC)** — AD management
- **Group Policy Management Console (GPMC)** — GPO configuration

---

*All configurations are fictional and used solely for educational and portfolio purposes.*
