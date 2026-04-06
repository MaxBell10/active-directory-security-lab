# Lessons Learned — Active Directory Security Lab

## Overview

This document reflects my honest experience building this lab from scratch, as someone transitioning into a GRC + Technical cybersecurity role. The goal is not to pretend everything went smoothly, but to document what I actually learned — including the friction points.

---

## Main Challenges

### Getting familiar with Windows Server 2022

The biggest challenge was simply navigating Windows Server 2022 for the first time. Server Manager, the role installation wizard, promoting the server to a Domain Controller, and understanding the difference between local and domain policies — none of this was intuitive at the start.

It took time to understand the relationship between the DC, the domain, and the client machine — and why certain settings apply at the domain level via GPO rather than locally on each machine.

This is exactly the kind of hands-on familiarity that no course can fully replace.

### Wazuh network connectivity

The Wazuh OVA did not receive an IP address automatically because the internal VirtualBox network (`intnet`) has no DHCP server. The VM had no IPv4 address at all — only an IPv6 link-local address — which made the dashboard unreachable.

**Fix applied:**
```bash
sudo ip addr add 192.168.100.10/24 dev eth0
sudo ip link set eth0 up
```

This is a temporary assignment that does not survive a reboot. In a persistent lab setup, the correct approach is to configure a static IP permanently via `/etc/network/interfaces` or a network manager tool.

**Production difference:** In a real environment, static IPs for security infrastructure (SIEM, log collectors) are always assigned either via DHCP reservation or static configuration in the OS — never left to chance.

### Triggering real Event ID 4625 alerts

The Windows lock screen does not generate a real Event ID 4625. Typing a wrong password on the lock screen is handled locally and does not result in a network authentication attempt against the Domain Controller.

To generate real 4625 events visible in Wazuh, a network authentication must be forced:

```cmd
net use \\192.168.100.1\sysvol /user:lab\bob.finance WrongPass999
```

This sends a real SMB authentication request to the DC, which logs the failure and forwards it to Wazuh via the agent.

**Why it matters:** Understanding the difference between local and network authentication is fundamental for log analysis and SIEM tuning. A misconfigured audit policy or a misunderstanding of authentication flows leads to blind spots in detection.

---

## Positive Surprises

### Active Directory structure is more approachable than expected

Creating the OU hierarchy, adding users, assigning them to groups, and linking GPOs was more straightforward than anticipated. The logic of the structure — OUs for organisation, groups for permissions, GPOs for enforcement — is clean once understood.

### Wazuh out-of-the-box detection capability

Without any custom rule configuration, Wazuh already detected and correlated multiple security events:
- Individual logon failures (rule 60122, level 5)
- Brute-force pattern correlation (rule 60204, level 10)
- MITRE ATT&CK technique mapping (T1110 Brute Force, T1078 Valid Accounts)

The depth of visibility available from a free, open-source SIEM on a home lab was genuinely surprising. That said, understanding the full rule logic, alert tuning, and dashboard navigation remains a work in progress.

---

## What I Would Do Differently in Production

I am currently in a learning phase — theoretical and lab-based — with the goal of moving into a hands-on GRC + Technical role involving real infrastructure. I do not yet have production experience, and I think it is important to say that clearly rather than pretend otherwise.

That said, based on what I have learned so far, I can identify some differences between this lab and a real environment:

- **Wazuh IP configuration** would be set permanently via static network configuration, not a temporary `ip addr add` command that resets on reboot
- **GPOs would be more granular** — separate GPOs per OU rather than a single policy linked at the domain root, to allow more targeted enforcement and easier troubleshooting
- **Wazuh would sit on a dedicated management network**, isolated from the monitored endpoints, to prevent an attacker from tampering with the SIEM if they compromise the LAN
- **Agent enrollment** would use certificate-based authentication rather than the default shared key, to ensure integrity of the agent-manager communication

---

## Next Steps

- Deepen understanding of Wazuh rule logic and custom correlation rules
- Configure a persistent static IP on the Wazuh OVA
- Add a custom rule for Event ID 4728 (user added to Domain Admins)
- Move on to Project 2 — pfSense network segmentation

---

*This lab is part of a broader portfolio designed to demonstrate practical cybersecurity skills aligned with both GRC and technical roles in real-world environments.*

