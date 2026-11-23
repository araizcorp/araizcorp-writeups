---
title: "OffSec Assessment: Seal (Hack The Box) — Penetration Testing Report"
published: true
description: "Comprehensive offensive security assessment and penetration testing report for Seal (Hack The Box). Methodological analysis covering initial access, local privilege escalation, and OffSec remediation."
tags: cybersecurity, hackthebox, pentesting, infosec
series: "HackTheBox Enterprise Assessments"
cover_image: "https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/seal/images/step_01.png"
---

# Offensive Security Assessment Report: Seal

![Hack The Box](https://img.shields.io/badge/Platform-Hack%20The%20Box-green?style=for-the-badge&logo=hackthebox)
![Difficulty](https://img.shields.io/badge/Difficulty-Medium-yellow?style=for-the-badge)
![OS](https://img.shields.io/badge/OS-Linux-orange?style=for-the-badge)
![Standards](https://img.shields.io/badge/Standards-OffSec%20PEN--200%20%2F%20PTES-purple?style=for-the-badge)
![Assessor](https://img.shields.io/badge/Assessor-araizcorp%20Offensive%20Security-blue?style=for-the-badge)

**Offensive Security Assessment & Penetration Testing Report** conducted on the target environment **Seal** hosted on **Hack The Box**, executed in accordance with **OffSec PEN-200 (OSCP)** and **PTES (Penetration Testing Execution Standard)** methodologies.

---

## 📑 1. Executive Summary

* **Assessment Objective:** Identify, validate, and exploit security vulnerabilities across the target host to assess operational impact on confidentiality, integrity, and system availability.
* **Risk Rating:** `CRITICAL` (Unauthenticated remote exploit vector leading to complete host compromise and administrative takeover).
* **Initial Access Vector:** Upload/deserialize en web → RCE
* **Privilege Escalation Vector:** sudo misconfig / SUID
* **Engagement Outcome:** Complete Host Compromise (`User Proof` and `Root / SYSTEM Proof` successfully retrieved and verified).

### Target Technical Specifications

| Parameter | Technical Detail |
| :--- | :--- |
| **Target Host** | Seal |
| **Platform** | Hack The Box |
| **Operating System** | Linux |
| **Difficulty Rating** | Medium |
| **Associated CVE(s)** | N/A |
| **Initial Foothold Vector** | Upload/deserialize en web → RCE |
| **Privilege Escalation Vector** | sudo misconfig / SUID |
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
         │─── Exploit delivery: Upload/deserialize en web → RCE
         └─── Establishing an interactive reverse shell with unprivileged user context
         │
         ▼
[ Phase 4: Local Privilege Escalation ]
         │─── Internal audit: sudo misconfig / SUID
         └─── Elevation to superuser / administrator privileges (root / SYSTEM)
         │
         ▼
[ Phase 5: Evidence & Flag Retrieval ]
         └─── Capture and cryptographic verification of user.txt and root.txt
```

---

## 🔬 3. Technical Walkthrough & Step-by-Step Proof of Concept

> **Assessment Query / Target Milestone:** How many open TCP ports are listening on Seal?

* **Observed Evidence / Output:** 3


![step_01.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/seal/images/step_01.png)
* **Technical Finding (Phase 1):** Initial network reconnaissance and service detection exposing active daemons and communication endpoints.



> **Assessment Query / Target Milestone:** What is the name of the webserver that's running on TCP 443?

* **Observed Evidence / Output:** tomcat


![step_02.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/seal/images/step_02.png)
* **Technical Finding (Phase 2):** Attack surface enumeration, directory discovery, or parameter analysis identifying potential vulnerability vectors.



> **Assessment Query / Target Milestone:** What application is listening on TCP 8080?

* **Observed Evidence / Output:** GitBucket


![step_03.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/seal/images/step_03.png)
* **Assessment Step 03:** Command execution output, exploit delivery proof, or auxiliary evidence.



> **Assessment Query / Target Milestone:** What is the name of the repo containing configuration files for the web application listening on TCP 443?

* **Observed Evidence / Output:** seal_market


![step_04.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/seal/images/step_04.png)
* **Assessment Step 04:** Command execution output, exploit delivery proof, or auxiliary evidence.



> **Assessment Query / Target Milestone:** What is the password for the tomcat user with access to the manager-gui?

* **Observed Evidence / Output:** `42MrHBf*z8{Z%`


![step_05.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/seal/images/step_05.png)
* **Assessment Step 05:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_06.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/seal/images/step_06.png)
* **Assessment Step 06:** Command execution output, exploit delivery proof, or auxiliary evidence.


**The `README.md` mentioned mutual authentication. What is the name of the variable checked in the nginx config to implement this?**

* **Observed Evidence / Output:** `$ssl_client_verify`


![step_07.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/seal/images/step_07.png)
* **Assessment Step 07:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_08.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/seal/images/step_08.png)
* **Assessment Step 08:** Command execution output, exploit delivery proof, or auxiliary evidence.



> **Assessment Query / Target Milestone:** What is the title at the top right of `/admin/dashboard`?

* **Observed Evidence / Output:** [Seal Market Admin Dashboard](https://10.10.10.250/admin;foo=bar/dashboard/index.html)


![step_09.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/seal/images/step_09.png)
* **Assessment Step 09:** Command execution output, exploit delivery proof, or auxiliary evidence.



> **Assessment Query / Target Milestone:** What type of file can be uploaded to the "Deploy" section of the Tomcat manager to get execution?

* **Observed Evidence / Output:** WAR


![step_10.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/seal/images/step_10.png)
* **Assessment Step 10:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_11.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/seal/images/step_11.png)
* **Assessment Step 11:** Command execution output, exploit delivery proof, or auxiliary evidence.



> **Assessment Query / Target Milestone:** What is the full path to the Ansible playbook that's being run every two minutes on Seal?

* **Observed Evidence / Output:** /opt/backups/playbook/run.yml


![step_12.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/seal/images/step_12.png)
* **Assessment Step 12:** Command execution output, exploit delivery proof, or auxiliary evidence.



> **Assessment Query / Target Milestone:** What dangerous option is set to yes in the synchronize task?

* **Observed Evidence / Output:** copy_links


![step_13.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/seal/images/step_13.png)
* **Assessment Step 13:** Command execution output, exploit delivery proof, or auxiliary evidence.


**Submit the flag located in the luis user's home directory.**

* **Observed Evidence / Output:** 91674fb0830a8e3612638d0135395b05


![step_14.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/seal/images/step_14.png)
* **Assessment Step 14:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_15.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/seal/images/step_15.png)
* **Assessment Step 15:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_16.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/seal/images/step_16.png)
* **Assessment Step 16:** Command execution output, exploit delivery proof, or auxiliary evidence.



> **Assessment Query / Target Milestone:** What is the full path of the binary that luis can run as root without a password?

* **Observed Evidence / Output:** /usr/bin/ansible-playbook


![step_17.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/seal/images/step_17.png)
* **Assessment Step 17:** Command execution output, exploit delivery proof, or auxiliary evidence.


**Submit the flag located in the root user's home directory.**

* **Observed Evidence / Output:** 90001f34963726c189f14c84958fcab7


![step_18.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/seal/images/step_18.png)
* **Privilege Escalation PoC (Phase 4):** Exploitation of misconfiguration, SUID binary, or vulnerability leading to elevated privileges.



![step_19.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/seal/images/step_19.png)
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
| **Initial Access** | `T1190` | Exploit Public-Facing Application | Exploitation of exposed network/web service (Upload/deserialize en web → RCE) |
| **Execution** | `T1059` | Command and Scripting Interpreter | Interactive reverse shell execution and remote command dispatch |
| **Privilege Escalation** | `T1068 / T1548` | Abuse Elevation Control Mechanism / Exploitation | Local elevation to administrative/root privileges (sudo misconfig / SUID) |

---

© 2026 **araizcorp** — Technical documentation and security assessment portfolio developed for **OffSec** certification preparation (OSCP, OSEP, OSWE) and commercial penetration testing enablement.
