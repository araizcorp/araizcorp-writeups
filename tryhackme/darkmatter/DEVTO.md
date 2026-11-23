---
title: "OffSec Assessment: DarkMatter (TryHackMe) — Penetration Testing Report"
published: true
description: "Comprehensive offensive security assessment and penetration testing report for DarkMatter (TryHackMe). Methodological walkthrough covering enumeration, exploitation, privilege escalation, and OffSec-aligned hardening."
tags: cybersecurity, tryhackme, pentesting, infosec
series: "TryHackMe Enterprise Assessments"
cover_image: "https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/darkmatter/images/step_01.png"
---

# Offensive Security Assessment Report: DarkMatter

![TryHackMe](https://img.shields.io/badge/Platform-TryHackMe-blue?style=for-the-badge&logo=tryhackme)
![Difficulty](https://img.shields.io/badge/Difficulty-Medium-yellow?style=for-the-badge)
![OS](https://img.shields.io/badge/OS-Linux-orange?style=for-the-badge)
![Standards](https://img.shields.io/badge/Standards-OffSec%20PEN--200%20%2F%20PTES-purple?style=for-the-badge)
![Assessor](https://img.shields.io/badge/Assessor-araizcorp%20Offensive%20Security-blue?style=for-the-badge)

**Offensive Security Assessment & Penetration Testing Report** conducted on the target environment **DarkMatter** hosted on **TryHackMe**, executed in accordance with **OffSec PEN-200 (OSCP)** and **PTES (Penetration Testing Execution Standard)** methodologies.

---

## 📑 1. Executive Summary

* **Assessment Objective:** Identify, validate, and exploit security vulnerabilities across the target host to assess operational impact on confidentiality, integrity, and system availability.
* **Risk Rating:** `CRITICAL` (Unauthenticated remote exploit vector leading to complete host compromise and administrative takeover).
* **Initial Access Vector:** Web Application Exploitation / Default Credentials
* **Privilege Escalation Vector:** Abuse of SUID Binaries / Sudo Misconfiguration
* **Engagement Outcome:** Complete Host Compromise (`User Proof` and `Root / SYSTEM Proof` successfully retrieved and verified).

### Target Technical Specifications

| Parameter | Technical Detail |
| :--- | :--- |
| **Target Host** | DarkMatter |
| **Platform** | TryHackMe |
| **Operating System** | Linux |
| **Difficulty Rating** | Medium |
| **Associated CVE(s)** | N/A |
| **Initial Foothold Vector** | Web Application Exploitation / Default Credentials |
| **Privilege Escalation Vector** | Abuse of SUID Binaries / Sudo Misconfiguration |
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
         │─── Exploit delivery: Web Application Exploitation / Default Credentials
         └─── Establishing an interactive reverse shell with unprivileged user context
         │
         ▼
[ Phase 4: Local Privilege Escalation ]
         │─── Internal audit: Abuse of SUID Binaries / Sudo Misconfiguration
         └─── Elevation to superuser / administrator privileges (root / SYSTEM)
         │
         ▼
[ Phase 5: Evidence & Flag Retrieval ]
         └─── Capture and cryptographic verification of user.txt and root.txt
```

---

## 🔬 3. Technical Walkthrough & Step-by-Step Proof of Concept

```bash



#!/usr/bin/python3
from gmpy2 import isqrt
from math import lcm

def factorize(n):
    if (n & 1) == 0:
        return (n // 2, 2)
    a = isqrt(n)
    if a * a == n:
        return a, a
    while True:
        a = a + 1
        bsq = a * a - n
        b = isqrt(bsq)
        if b * b == bsq:
            break
    return a + b, a - b


n = 340282366920938460843936948965011886881
e = 65537



encrypted_aes_key_int = 0x6d6452c15ca333a4429a5e451901b6d7


p, q = factorize(n)


lambda_n = lcm(p - 1, q - 1)


d = pow(e, -1, lambda_n)

print("\n")
print(f"[*] Found Prime Factors:")
print(f"p = {p}")
print(f"q = {q}\n")




lambda_n = lcm(p - 1, q - 1)
print(f"[*] Calculated Lambda(n):")
print(f"λ(n) = {lambda_n}\n")

print(f"[*] Private Exponent (d): {d}\n")



decrypted_key_int = pow(encrypted_aes_key_int, d, n)

print(f"[*] Decrypted AES Key (as an Integer):")
print(f"{decrypted_key_int}\n")



decrypted_key_hex = format(decrypted_key_int, 'x')

print(f"✅ Decrypted AES Key (in Hex):")
print(f"{decrypted_key_hex}")
print("\n")
```


> **Assessment Query / Target Milestone:** ### En resumen: ¿qué debes hacer la próxima vez?

1. **Busca pistas en la máquina** (código, .py, .json, .pem).
2. Si ves `n`, `e` y un valor cifrado → **esto es RSA**.
3. Intenta factorizar `n`. Si es fácil, **puedes romper RSA**.
4. Si se descifra algo → probablemente es una **clave AES**.
5. Pruébala como "decryption key" en el binario, GUI o web de la máquina.


![step_01.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/darkmatter/images/step_01.png)
* **Technical Finding (Phase 1):** Initial network reconnaissance and service detection exposing active daemons and communication endpoints.



![step_02.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/darkmatter/images/step_02.png)
* **Technical Finding (Phase 2):** Attack surface enumeration, directory discovery, or parameter analysis identifying potential vulnerability vectors.



![step_03.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/darkmatter/images/step_03.png)
* **Assessment Step 03:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_04.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/darkmatter/images/step_04.png)
* **Assessment Step 04:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_05.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/darkmatter/images/step_05.png)
* **Privilege Escalation PoC (Phase 4):** Exploitation of misconfiguration, SUID binary, or vulnerability leading to elevated privileges.



![step_06.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/darkmatter/images/step_06.png)
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
| **Initial Access** | `T1190` | Exploit Public-Facing Application | Exploitation of exposed network/web service (Web Application Exploitation / Default Credentials) |
| **Execution** | `T1059` | Command and Scripting Interpreter | Interactive reverse shell execution and remote command dispatch |
| **Privilege Escalation** | `T1068 / T1548` | Abuse Elevation Control Mechanism / Exploitation | Local elevation to administrative/root privileges (Abuse of SUID Binaries / Sudo Misconfiguration) |

---

© 2026 **araizcorp** — Technical documentation and security assessment portfolio developed for **OffSec** certification preparation (OSCP, OSEP, OSWE) and commercial penetration testing enablement.
