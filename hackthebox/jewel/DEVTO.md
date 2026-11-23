---
title: "OffSec Assessment: Jewel (Hack The Box) — Penetration Testing Report"
published: true
description: "Comprehensive offensive security assessment and penetration testing report for Jewel (Hack The Box). Methodological analysis covering initial access, local privilege escalation, and OffSec remediation."
tags: cybersecurity, hackthebox, pentesting, infosec
series: "HackTheBox Enterprise Assessments"
cover_image: "https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/jewel/images/step_01.png"
---

# Offensive Security Assessment Report: Jewel

![Hack The Box](https://img.shields.io/badge/Platform-Hack%20The%20Box-green?style=for-the-badge&logo=hackthebox)
![Difficulty](https://img.shields.io/badge/Difficulty-Medium-yellow?style=for-the-badge)
![OS](https://img.shields.io/badge/OS-Linux-orange?style=for-the-badge)
![Standards](https://img.shields.io/badge/Standards-OffSec%20PEN--200%20%2F%20PTES-purple?style=for-the-badge)
![Assessor](https://img.shields.io/badge/Assessor-araizcorp%20Offensive%20Security-blue?style=for-the-badge)

**Offensive Security Assessment & Penetration Testing Report** conducted on the target environment **Jewel** hosted on **Hack The Box**, executed in accordance with **OffSec PEN-200 (OSCP)** and **PTES (Penetration Testing Execution Standard)** methodologies.

---

## 📑 1. Executive Summary

* **Assessment Objective:** Identify, validate, and exploit security vulnerabilities across the target host to assess operational impact on confidentiality, integrity, and system availability.
* **Risk Rating:** `CRITICAL` (Unauthenticated remote exploit vector leading to complete host compromise and administrative takeover).
* **Initial Access Vector:** Ruby on Rails Deserialization RCE (CVE-2020-8165)
* **Privilege Escalation Vector:** sudo gem + Google Authenticator 2FA bypass
* **Engagement Outcome:** Complete Host Compromise (`User Proof` and `Root / SYSTEM Proof` successfully retrieved and verified).

### Target Technical Specifications

| Parameter | Technical Detail |
| :--- | :--- |
| **Target Host** | Jewel |
| **Platform** | Hack The Box |
| **Operating System** | Linux |
| **Difficulty Rating** | Medium |
| **Associated CVE(s)** | CVE-2020-8165 |
| **Initial Foothold Vector** | Ruby on Rails Deserialization RCE (CVE-2020-8165) |
| **Privilege Escalation Vector** | sudo gem + Google Authenticator 2FA bypass |
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
         │─── Exploit delivery: Ruby on Rails Deserialization RCE (CVE-2020-8165)
         └─── Establishing an interactive reverse shell with unprivileged user context
         │
         ▼
[ Phase 4: Local Privilege Escalation ]
         │─── Internal audit: sudo gem + Google Authenticator 2FA bypass
         └─── Elevation to superuser / administrator privileges (root / SYSTEM)
         │
         ▼
[ Phase 5: Evidence & Flag Retrieval ]
         └─── Capture and cryptographic verification of user.txt and root.txt
```

---

## 🔬 3. Technical Walkthrough & Step-by-Step Proof of Concept

| Field | Detail |
| --- | --- |
| Name | Jewel |
| OS | Linux (Debian) |
| Difficulty | Medium |
| CVE | CVE-2020-8165 |
| Initial Access | Ruby on Rails Deserialization RCE (CVE-2020-8165) |
| Privilege Escalation | sudo gem + Google Authenticator 2FA bypass |

---

## Reconnaissance

An Nmap scan reveals 3 open TCP ports: SSH (22), HTTP (8080 -- Gitweb), and HTTP (8000 -- Blog application).


![step_01.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/jewel/images/step_01.png)
* **Technical Finding (Phase 1):** Initial network reconnaissance and service detection exposing active daemons and communication endpoints.


## Enumeration

Port 8080 hosts **Gitweb**, a web interface for browsing Git repositories. Examining the repository reveals the blog application's source code, including its Gemfile.


![step_02.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/jewel/images/step_02.png)
* **Technical Finding (Phase 2):** Attack surface enumeration, directory discovery, or parameter analysis identifying potential vulnerability vectors.


The blog application on port 8000 is written in **Ruby** using Ruby on Rails.


![step_03.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/jewel/images/step_03.png)
* **Assessment Step 03:** Command execution output, exploit delivery proof, or auxiliary evidence.


## Exploitation

The Gemfile reveals the application uses a version of Rails vulnerable to **CVE-2020-8165**, an insecure deserialization vulnerability in ActiveSupport's `MemCacheStore`. By crafting a malicious serialized Ruby object and injecting it through the user profile update functionality, remote code execution is achieved.


![step_04.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/jewel/images/step_04.png)
* **Assessment Step 04:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_05.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/jewel/images/step_05.png)
* **Assessment Step 05:** Command execution output, exploit delivery proof, or auxiliary evidence.


The web application runs as the `bill` user.


![step_06.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/jewel/images/step_06.png)
* **Assessment Step 06:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_07.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/jewel/images/step_07.png)
* **Assessment Step 07:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_08.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/jewel/images/step_08.png)
* **Assessment Step 08:** Command execution output, exploit delivery proof, or auxiliary evidence.


### Phase 5: Proof of Compromise & Flag Capture

#### User Proof Flag Verification


![step_09.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/jewel/images/step_09.png)
* **Assessment Step 09:** Command execution output, exploit delivery proof, or auxiliary evidence.


## Privilege Escalation

The bill user's password (`spongebob`) is recovered by extracting and cracking the bcrypt hash from the application's database backup.


![step_10.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/jewel/images/step_10.png)
* **Assessment Step 10:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_11.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/jewel/images/step_11.png)
* **Assessment Step 11:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_12.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/jewel/images/step_12.png)
* **Assessment Step 12:** Command execution output, exploit delivery proof, or auxiliary evidence.


Running `sudo -l` shows bill can run `/usr/bin/gem` as any user, but sudo requires Google Authenticator 2FA verification. The `.google_authenticator` file in bill's home directory contains the TOTP seed. Using this seed to generate a valid OTP allows authenticating with sudo.


![step_13.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/jewel/images/step_13.png)
* **Assessment Step 13:** Command execution output, exploit delivery proof, or auxiliary evidence.


With sudo access confirmed, `/usr/bin/gem` can be abused via GTFOBins to execute arbitrary Ruby code as root:

```
sudo gem open -e "/bin/bash -c '/bin/bash'" rdoc
```


![step_14.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/jewel/images/step_14.png)
* **Privilege Escalation PoC (Phase 4):** Exploitation of misconfiguration, SUID binary, or vulnerability leading to elevated privileges.


#### Root / System Proof Flag Verification


![step_15.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/jewel/images/step_15.png)
* **Evidence Verification (Phase 5):** Final administrative validation confirming root/SYSTEM access and retrieval of system flags.

---

## 🛡️ 4. Strategic Remediation & Hardening Roadmap

### Immediate Tactical Actions (P1)
1. **Patch Management & Software Upgrades:** Immediately apply official security updates and patches for all vulnerable software components and exposed services (CVE-2020-8165).
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
| **Initial Access** | `T1190` | Exploit Public-Facing Application | Exploitation of exposed network/web service (Ruby on Rails Deserialization RCE (CVE-2020-8165)) |
| **Execution** | `T1059` | Command and Scripting Interpreter | Interactive reverse shell execution and remote command dispatch |
| **Privilege Escalation** | `T1068 / T1548` | Abuse Elevation Control Mechanism / Exploitation | Local elevation to administrative/root privileges (sudo gem + Google Authenticator 2FA bypass) |

---

© 2026 **araizcorp** — Technical documentation and security assessment portfolio developed for **OffSec** certification preparation (OSCP, OSEP, OSWE) and commercial penetration testing enablement.
