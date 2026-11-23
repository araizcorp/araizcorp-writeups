---
title: "OffSec Assessment: Static (Hack The Box) — Penetration Testing Report"
published: true
description: "Comprehensive offensive security assessment and penetration testing report for Static (Hack The Box). Methodological analysis covering initial access, local privilege escalation, and OffSec remediation."
tags: cybersecurity, hackthebox, pentesting, infosec
series: "HackTheBox Enterprise Assessments"
cover_image: "https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/static/images/step_01.png"
---

# Offensive Security Assessment Report: Static

![Hack The Box](https://img.shields.io/badge/Platform-Hack%20The%20Box-green?style=for-the-badge&logo=hackthebox)
![Difficulty](https://img.shields.io/badge/Difficulty-Hard-red?style=for-the-badge)
![OS](https://img.shields.io/badge/OS-Linux-orange?style=for-the-badge)
![Standards](https://img.shields.io/badge/Standards-OffSec%20PEN--200%20%2F%20PTES-purple?style=for-the-badge)
![Assessor](https://img.shields.io/badge/Assessor-araizcorp%20Offensive%20Security-blue?style=for-the-badge)

**Offensive Security Assessment & Penetration Testing Report** conducted on the target environment **Static** hosted on **Hack The Box**, executed in accordance with **OffSec PEN-200 (OSCP)** and **PTES (Penetration Testing Execution Standard)** methodologies.

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
| **Target Host** | Static |
| **Platform** | Hack The Box |
| **Operating System** | Linux |
| **Difficulty Rating** | Hard |
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

![step_01.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/static/images/step_01.png)
* **Technical Finding (Phase 1):** Initial network reconnaissance and service detection exposing active daemons and communication endpoints.



![step_02.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/static/images/step_02.png)
* **Technical Finding (Phase 2):** Attack surface enumeration, directory discovery, or parameter analysis identifying potential vulnerability vectors.



![step_03.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/static/images/step_03.png)
* **Assessment Step 03:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_04.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/static/images/step_04.png)
* **Assessment Step 04:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_05.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/static/images/step_05.png)
* **Assessment Step 05:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_06.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/static/images/step_06.png)
* **Assessment Step 06:** Command execution output, exploit delivery proof, or auxiliary evidence.


```jsx
sudo apt -y install p7zip-full                                                         ─╯
7z x -so db.sql.gz > db.sql || true
file db.sql; wc -c db.sql
head -50 db.sql
```


![step_07.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/static/images/step_07.png)
* **Assessment Step 07:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_08.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/static/images/step_08.png)
* **Assessment Step 08:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_09.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/static/images/step_09.png)
* **Assessment Step 09:** Command execution output, exploit delivery proof, or auxiliary evidence.


```jsx
#!/usr/bin/env python3
"""
AraizCorp LLC · TOTP helper with NTP sync
Requisitos: pip install pyotp ntplib qrcode[pil] (qrcode es opcional)
"""

import argparse
import hashlib
import sys
import time
from datetime import datetime, timezone

import pyotp


try:
    import ntplib
except Exception:  # pragma: no cover
    ntplib = None

def parse_args():
    p = argparse.ArgumentParser(
        description="Generate TOTP codes with NTP sync and extras."
    )
    p.add_argument(
        "-s", "--secret",
        default="orxxi4c7orxwwzlo",
        help="Base32 TOTP secret (default: example from dump)"
    )
    p.add_argument(
        "-n", "--ntp",
        action="append",
        default=["10.10.10.246", "pool.ntp.org", "time.google.com"],
        help="NTP server (can be used multiple times). Default: 10.10.10.246, pool.ntp.org, time.google.com"
    )
    p.add_argument(
        "-w", "--window",
        type=int, default=1,
        help="How many steps before/after to show (default: 1 => prev, current, next)"
    )
    p.add_argument(
        "-p", "--period",
        type=int, default=30,
        help="TOTP time step / period in seconds (default: 30)"
    )
    p.add_argument(
        "-d", "--digits",
        type=int, default=6,
        choices=(6, 7, 8),
        help="Digits in code (default: 6)"
    )
    p.add_argument(
        "-a", "--algo",
        default="SHA1",
        choices=("SHA1", "SHA256", "SHA512"),
        help="HMAC digest algorithm (default: SHA1)"
    )
    p.add_argument(
        "--issuer", default="Zetta",
        help="Issuer for otpauth URI (default: Zetta)"
    )
    p.add_argument(
        "--account", default="admin@zetta",
        help="Account label for otpauth URI (default: admin@zetta)"
    )
    p.add_argument(
        "--uri", action="store_true",
        help="Print otpauth:// URI"
    )
    p.add_argument(
        "--qr", action="store_true",
        help="Render ASCII QR (requires qrcode)"
    )
    p.add_argument(
        "--unix", action="store_true",
        help="Only print the current TOTP (for piping/automation)"
    )
    return p.parse_args()

def get_network_time(servers, timeout=2.0, retries=1):
    """
    Try a list of NTP servers; return (epoch_seconds, source_label).
    Fallback to local time if ntplib not available or all fail.
    """
    if ntplib is None:
        return time.time(), "local-clock"

    c = ntplib.NTPClient()
    last_err = None
    for srv in servers:
        for _ in range(retries + 1):
            try:
                resp = c.request(srv, version=4, timeout=timeout)
                return resp.tx_time, f"ntp:{srv}"
            except Exception as e:  # keep trying others
                last_err = e
                continue
    # fallback
    return time.time(), f"local-clock (ntp failed: {last_err})"

def digest_from_name(name: str):
    name = name.upper()
    if name == "SHA1":
        return hashlib.sha1
    if name == "SHA256":
        return hashlib.sha256
    if name == "SHA512":
        return hashlib.sha512
    raise ValueError(f"Unsupported algo: {name}")

def normalize_b32(secret: str) -> str:
    # Remove spaces, make uppercase; pyotp tolerates padding
    return "".join(secret.split()).upper()

def fmt_iso(ts: float) -> str:
    return datetime.fromtimestamp(ts, tz=timezone.utc).isoformat(timespec="seconds")

def main():
    args = parse_args()

    secret = normalize_b32(args.secret)
    algo = digest_from_name(args.algo)

    # Time sync
    now, source = get_network_time(args.ntp)

    totp = pyotp.TOTP(secret, digits=args.digits, interval=args.period, digest=algo)

    # If only raw code requested
    if args.unix:
        print(totp.at(now))
        return 0

    # Pretty header
    print("── TOTP (AraizCorp) ─────────────────────────────────────────")
    print(f"Time source    : {source}")
    print(f"Now (UTC)      : {fmt_iso(now)}")
    rem = args.period - (int(now) % args.period)
    print(f"Period/digits  : {args.period}s / {args.digits} (algo {args.algo})")
    print(f"Valid for      : {rem}s")
    print()

    # Window: previous..current..next
    t0 = int(now // args.period) * args.period
    start_step = -args.window
    end_step = args.window

    print("Step  | Timestamp (UTC)        | Code")
    print("------+-------------------------+----------")
    for offset in range(start_step, end_step + 1):
        ts = t0 + offset * args.period
        code = totp.at(ts)
        mark = "← now" if ts == t0 else ""
        print(f"{offset:+3d}  | {fmt_iso(ts):<23} | {code:<8} {mark}")

    # URI / QR
    if args.uri or args.qr:
        uri = pyotp.totp.TOTP(secret, digits=args.digits, interval=args.period, digest=algo)\
            .provisioning_uri(name=args.account, issuer_name=args.issuer)
        print("\notpauth URI:")
        print(uri)

        if args.qr:
            try:
                import qrcode
                qr = qrcode.QRCode(border=1)
                qr.add_data(uri)
                qr.make(fit=True)
                qr.print_ascii(invert=True)
            except Exception as e:
                print(f"(QR not available: {e})")

    return 0

if __name__ == "__main__":
    try:
        sys.exit(main())
    except KeyboardInterrupt:
        sys.exit(130)

```


![step_10.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/static/images/step_10.png)
* **Assessment Step 10:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_11.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/static/images/step_11.png)
* **Assessment Step 11:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_12.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/static/images/step_12.png)
* **Assessment Step 12:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_13.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/static/images/step_13.png)
* **Assessment Step 13:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_14.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/static/images/step_14.png)
* **Assessment Step 14:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_15.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/static/images/step_15.png)
* **Assessment Step 15:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_16.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/static/images/step_16.png)
* **Assessment Step 16:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_17.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/static/images/step_17.png)
* **Assessment Step 17:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_18.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/static/images/step_18.png)
* **Assessment Step 18:** Command execution output, exploit delivery proof, or auxiliary evidence.


7a66bf21c87b3ece73153af305397aa7


![step_19.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/static/images/step_19.png)
* **Assessment Step 19:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_20.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/static/images/step_20.png)
* **Assessment Step 20:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_21.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/static/images/step_21.png)
* **Assessment Step 21:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_22.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/static/images/step_22.png)
* **Assessment Step 22:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_23.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/static/images/step_23.png)
* **Assessment Step 23:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_24.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/static/images/step_24.png)
* **Assessment Step 24:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_25.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/static/images/step_25.png)
* **Assessment Step 25:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_26.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/static/images/step_26.png)
* **Assessment Step 26:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_27.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/static/images/step_27.png)
* **Assessment Step 27:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_28.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/static/images/step_28.png)
* **Assessment Step 28:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_29.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/static/images/step_29.png)
* **Assessment Step 29:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_30.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/static/images/step_30.png)
* **Assessment Step 30:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_31.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/static/images/step_31.png)
* **Assessment Step 31:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_32.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/static/images/step_32.png)
* **Assessment Step 32:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_33.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/static/images/step_33.png)
* **Assessment Step 33:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_34.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/static/images/step_34.png)
* **Assessment Step 34:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_35.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/static/images/step_35.png)
* **Assessment Step 35:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_36.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/static/images/step_36.png)
* **Assessment Step 36:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_37.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/static/images/step_37.png)
* **Assessment Step 37:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_38.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/static/images/step_38.png)
* **Assessment Step 38:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_39.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/static/images/step_39.png)
* **Assessment Step 39:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_40.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/static/images/step_40.png)
* **Privilege Escalation PoC (Phase 4):** Exploitation of misconfiguration, SUID binary, or vulnerability leading to elevated privileges.


4f2a465f241fe603624c29869d7e3c3d


![step_41.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/static/images/step_41.png)
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
