---
title: "OffSec Assessment: Bastion (Hack The Box) — Penetration Testing Report"
published: true
description: "Comprehensive offensive security assessment and penetration testing report for Bastion (Hack The Box). Methodological analysis covering initial access, local privilege escalation, and OffSec remediation."
tags: cybersecurity, hackthebox, pentesting, infosec
series: "HackTheBox Enterprise Assessments"
cover_image: "https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/bastion/images/step_01.png"
---

# Offensive Security Assessment Report: Bastion

![Hack The Box](https://img.shields.io/badge/Platform-Hack%20The%20Box-green?style=for-the-badge&logo=hackthebox)
![Difficulty](https://img.shields.io/badge/Difficulty-Easy-brightgreen?style=for-the-badge)
![OS](https://img.shields.io/badge/OS-Windows-orange?style=for-the-badge)
![Standards](https://img.shields.io/badge/Standards-OffSec%20PEN--200%20%2F%20PTES-purple?style=for-the-badge)
![Assessor](https://img.shields.io/badge/Assessor-araizcorp%20Offensive%20Security-blue?style=for-the-badge)

**Offensive Security Assessment & Penetration Testing Report** conducted on the target environment **Bastion** hosted on **Hack The Box**, executed in accordance with **OffSec PEN-200 (OSCP)** and **PTES (Penetration Testing Execution Standard)** methodologies.

---

## 📑 1. Executive Summary

* **Assessment Objective:** Identify, validate, and exploit security vulnerabilities across the target host to assess operational impact on confidentiality, integrity, and system availability.
* **Risk Rating:** `CRITICAL` (Unauthenticated remote exploit vector leading to complete host compromise and administrative takeover).
* **Initial Access Vector:** Public-Facing Network Service Exploitation
* **Privilege Escalation Vector:** Local Privilege Escalation
* **Engagement Outcome:** Complete Host Compromise (`User Proof` and `Root / SYSTEM Proof` successfully retrieved and verified).

### Target Technical Specifications

| Parameter | Technical Detail |
| :--- | :--- |
| **Target Host** | Bastion |
| **Platform** | Hack The Box |
| **Operating System** | Windows |
| **Difficulty Rating** | Easy |
| **Associated CVE(s)** | N/A |
| **Initial Foothold Vector** | Public-Facing Network Service Exploitation |
| **Privilege Escalation Vector** | Local Privilege Escalation |
| **Lead Penetration Tester** | Pedro Araiz (`araizcorp`) |

---

## 🎯 2. Attack Path & Kill Chain Workflow

```
[ Phase 1: Reconnaissance & Port Scanning ]
         │─── TCP SYN scanning and service fingerprinting (Nmap)
         └─── Attack surface mapping and daemon enumeration
         │
         ▼
[ Phase 2: Vulnerability Analysis & Service Enumeration ]
         │─── Web directory fuzzing, parameter discovery, and banner analysis
         └─── CVE identification and credential harvesting
         │
         ▼
[ Phase 3: Exploitation & Initial Foothold ]
         │─── Exploit delivery: Public-Facing Network Service Exploitation
         └─── Establishing an interactive reverse shell with unprivileged user context
         │
         ▼
[ Phase 4: Local Privilege Escalation ]
         │─── Internal audit: Local Privilege Escalation
         └─── Elevation to superuser / administrator privileges (root / SYSTEM)
         │
         ▼
[ Phase 5: Evidence & Flag Retrieval ]
         └─── Capture and cryptographic verification of user.txt and root.txt
```

---

## 🔬 3. Technical Walkthrough & Step-by-Step Proof of Concept

> **Assessment Query / Target Milestone:** How many TCP ports are open on the remote host?

* **Observed Evidence / Output:** 4


![step_01.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/bastion/images/step_01.png)
* **Technical Finding (Phase 1):** Initial network reconnaissance and service detection exposing active daemons and communication endpoints.



> **Assessment Query / Target Milestone:** What is the name of non-standard SMB share on Blocky?

* **Observed Evidence / Output:** backups


![step_02.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/bastion/images/step_02.png)
* **Technical Finding (Phase 2):** Attack surface enumeration, directory discovery, or parameter analysis identifying potential vulnerability vectors.



> **Assessment Query / Target Milestone:** What is the file extension of the two most interesting files on the share?

* **Observed Evidence / Output:** vhd


![step_03.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/bastion/images/step_03.png)
* **Assessment Step 03:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_04.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/bastion/images/step_04.png)
* **Assessment Step 04:** Command execution output, exploit delivery proof, or auxiliary evidence.


**Two files are needed from a Windows machine are needed to dump it's user hashes. One is `SAM`. What is the other?**

* **Observed Evidence / Output:** system


> **Assessment Query / Target Milestone:** What is the password for user L4mpje?

* **Observed Evidence / Output:** bureaulampje

```jsx
sudo mkdir -p /mnt/bastion
sudo mount -t cifs //10.10.10.134/Backups /mnt/bastion \
  -o 'ro,username=guest,password=,vers=2.1,uid='$(id -u)',gid='$(id -g)',nounix,noperm,iocharset=utf8'


sudo apt -y install libguestfs-tools
sudo mkdir -p /mnt/vhd
sudo guestmount --add "/mnt/bastion/WindowsImageBackup/L4mpje-PC/Backup 2019-02-22 124351/9b9cfbc4-369e-11e9-a17c-806e6f6e6963.vhd" --inspector --ro /mnt/vhd
ls /mnt/vhd/Windows/System32/config

```


![step_05.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/bastion/images/step_05.png)
* **Assessment Step 05:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_06.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/bastion/images/step_06.png)
* **Assessment Step 06:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_07.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/bastion/images/step_07.png)
* **Assessment Step 07:** Command execution output, exploit delivery proof, or auxiliary evidence.



> **Assessment Query / Target Milestone:** Submit the flag located on the L4mpje user's desktop.

* **Observed Evidence / Output:** 69cebb7feb9b6f6fef640af57af1f77d


![step_08.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/bastion/images/step_08.png)
* **Assessment Step 08:** Command execution output, exploit delivery proof, or auxiliary evidence.



> **Assessment Query / Target Milestone:** What is the name of the remote connection management tool installed on Bastion?

* **Observed Evidence / Output:** mRemoteNG


![step_09.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/bastion/images/step_09.png)
* **Assessment Step 09:** Command execution output, exploit delivery proof, or auxiliary evidence.



> **Assessment Query / Target Milestone:** What is the name of the file that mRemoteNG uses to store encrypted user passwords?

* **Observed Evidence / Output:** confCons.xml


![step_10.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/bastion/images/step_10.png)
* **Assessment Step 10:** Command execution output, exploit delivery proof, or auxiliary evidence.



> **Assessment Query / Target Milestone:** What is the administrator's password?

* **Observed Evidence / Output:** thXLHM96BeKL0ER2


![step_11.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/bastion/images/step_11.png)
* **Privilege Escalation PoC (Phase 4):** Exploitation of misconfiguration, SUID binary, or vulnerability leading to elevated privileges.



> **Assessment Query / Target Milestone:** Submit the flag located on the administrator's desktop.

* **Observed Evidence / Output:** 550225c2b7576c86b79bf125f400139f


![step_12.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/bastion/images/step_12.png)
* **Evidence Verification (Phase 5):** Final administrative validation confirming root/SYSTEM access and retrieval of system flags.

---

## 🛡️ 4. Strategic Remediation & Hardening Roadmap

### Immediate Tactical Actions (P1)
1. **Patch Management & Software Upgrades:** Immediately apply official security updates and patches for all vulnerable software components and exposed services (N/A).
2. **Access Control & SUID Audit:** Remove unnecessary SUID/SGID bits, audit `/etc/sudoers` rules, and enforce strict Access Control Lists (ACLs) on sensitive files and administrative scripts.
3. **Attack Surface Reduction:** Configure host-based firewalls (e.g., `iptables` / `ufw` / Windows Firewall) to restrict exposure of management services and internal ports to trusted administrative subnets only.

### Architectural & Defensive Hardening (P2)
1. **Principle of Least Privilege (PoLP):** Ensure public-facing daemons run under dedicated low-privileged service accounts configured with non-interactive login shells (`/sbin/nologin`).
2. **Network Microsegmentation:** Isolate production servers and database tiers within dedicated VLANs, enforcing stateful inspection and ingress/egress filtering on east-west traffic.
3. **Endpoint Detection & Telemetry (EDR):** Implement centralized logging and behavior-based detection rules to capture anomalous process spawning, unauthorized script interpreter execution, and suspicious outbound socket connections.

---

## 📊 5. MITRE ATT&CK Enterprise Framework Mapping

| Tactic | Technique ID | Technique Name | Technical Application in Engagement |
| :--- | :--- | :--- | :--- |
| **Reconnaissance** | `T1595.002` | Active Scanning: Vulnerability Scanning | Automated and manual port scanning, banner grabbing, and service discovery |
| **Initial Access** | `T1190` | Exploit Public-Facing Application | Exploitation of exposed network/web service (Public-Facing Network Service Exploitation) |
| **Execution** | `T1059` | Command and Scripting Interpreter | Interactive reverse shell execution and remote command dispatch |
| **Privilege Escalation** | `T1068 / T1548` | Abuse Elevation Control Mechanism / Exploitation | Local elevation to administrative/root privileges (Local Privilege Escalation) |

---

© 2026 **araizcorp** — Technical documentation and security assessment portfolio developed for **OffSec** certification preparation (OSCP, OSEP, OSWE) and commercial penetration testing enablement.
