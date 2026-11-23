---
title: "Informe Técnico: Safe (Hack The Box) — Pentesting & Exploit Analysis"
published: true
description: "Informe técnico profesional de auditoria sobre la maquina Safe de Hack The Box: vector de entrada, PoC de explotacion, escalada y mitigaciones OffSec."
tags: cybersecurity, ctf, hackthebox, pentesting
series: "Hack The Box Walkthroughs"
cover_image: "https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/safe/images/step_01.png"
---

# Informe Técnico de Seguridad Ofensiva: Safe

![Hack The Box](https://img.shields.io/badge/Platform-Hack%20The%20Box-green?style=for-the-badge&logo=hackthebox)
![Difficulty](https://img.shields.io/badge/Difficulty-Easy-brightgreen?style=for-the-badge)
![OS](https://img.shields.io/badge/OS-Linux-orange?style=for-the-badge)
![Standards](https://img.shields.io/badge/Methodology-OffSec%20%2F%20PTES-purple?style=for-the-badge)
![Auditor](https://img.shields.io/badge/Assessor-araizcorp%20Offensive%20Security-blue?style=for-the-badge)

**Evaluación de Seguridad Ofensiva y Penetration Testing** realizada sobre el entorno de laboratorio **Safe** de **Hack The Box**, siguiendo los estándares de ejecución metodológica **OffSec PEN-200 (OSCP)** y **PTES (Penetration Testing Execution Standard)**.

---

## 📑 1. Resumen Ejecutivo (Executive Summary)

* **Objetivo de la Evaluación:** Identificar, documentar y explotar debilidades de seguridad en el host objetivo para evaluar el impacto sobre la confidencialidad, integridad y disponibilidad del sistema.
* **Calificación de Riesgo:** `CRÍTICO` (Acceso no autenticado que deriva en compromiso completo de la máquina y obtención de máximos privilegios administrativos).
* **Vector de Acceso Inicial:** Explotación de servicio de red
* **Vector de Elevación de Privilegios:** Escalada local de privilegios
* **Estado Final:** Compromiso total del objetivo (`User Proof` y `Root/System Proof` obtenidas exitosamente).

### Ficha Técnica de la Evaluación

| Parámetro | Detalle Técnico |
| :--- | :--- |
| **Host Objetivo** | Safe |
| **Plataforma** | Hack The Box |
| **Sistema Operativo** | Linux |
| **Nivel de Dificultad** | Easy |
| **CVE(s) Involucrados** | N/A |
| **Vector de Entrada** | Explotación de servicio de red |
| **Mecanismo de Escalada** | Escalada local de privilegios |
| **Consultor / Auditor** | Pedro Araiz (`araizcorp`) |

---

## 🎯 2. Diagrama del Flujo de Compromiso (Attack Path)

```
[ Fase 1: Reconocimiento ]
         │
         ▼
[ Fase 2: Enumeración & Identificación de Vectores ]
         │─── Descubrimiento de servicios y análisis de superficie de ataque
         └─── Verificación de configuraciones erróneas y CVEs conocidos
         │
         ▼
[ Fase 3: Explotación Inicial (Foothold) ]
         │─── Ejecución del vector: Explotación de servicio de red
         └─── Establecimiento de Reverse Shell / Acceso de usuario
         │
         ▼
[ Fase 4: Escalada de Privilegios Local ]
         │─── Auditoría interna: Escalada local de privilegios
         └─── Elevación a privilegios de superusuario (root / SYSTEM)
         │
         ▼
[ Fase 5: Evidencia & Banderas ]
         └─── Captura de User Flag y Root Flag
```

---

## 🔬 3. Procedimiento Técnico y Evidencias Paso a Paso

**How many TCP ports are listening on Safe?** 

R: 3

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/safe/images/step_01.png)

**What is the path on the webserver that returns an ELF binary?**

R: /myapp

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/safe/images/step_02.png)

```jsx
wget 10.10.10.147/myapp
```

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/safe/images/step_03.png)

**Is Data Execution Prevention (DEP) enabled on this binary?**

R: yes

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/safe/images/step_04.png)

pwn cyclic 400 | ./myapp

gdb -q myapp

(gdb) set disassemble-next-line on
(gdb) set follow-fork-mode child
(gdb) run "$(python3 -c 'print("A"*200)')"

run < <(msf-pattern_create -l 400)

x/2gx $rsp

x/16bx $rsp

x/8cb $rsp

msf-pattern_offset -l 400 -q 0x6541316541306541

msf-pattern_offset -l 400 -q 0x4134654133654132

**'We're going to have to reverse engineer this binary. Binary analysis is one of the most intimidating topics in information security and hacking. For help getting started, check out the [Intro to Assembly Language](https://academy.hackthebox.com/module/details/85) and [Stack-Based Buffer Overflows on Linux x86](https://academy.hackthebox.com/module/details/31) modules in Academy. What is the offset in number (decimal) of bytes from the start of the input buffer to what ends up overwriting the return address?**

R: 120 

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/safe/images/step_05.png)

**'We're going to have to reverse engineer this binary. Binary analysis is one of the most intimidating topics in information security and hacking. For help getting started, check out the [Intro to Assembly Language](https://academy.hackthebox.com/module/details/85) and [Stack-Based Buffer Overflows on Linux x86](https://academy.hackthebox.com/module/details/31) modules in Academy. What is the offset in number (decimal) of bytes from the start of the input buffer to what ends up overwriting the return address?**

R: r13

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/safe/images/step_06.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/safe/images/step_07.png)

Because DEP / NX is enabled, we'll have to use bits of existing code to control the flow of execution. For example, we can find code in the binary that moves a value into r13 and then returns. This little bit of code is called a gadget. What is the three letter acronym for this technique of combining gadgets to get a desired execution?

R: ROP

**What user is the myapp service running as?**

R: user

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/safe/images/step_08.png)

Submit the flag located in the user user's home directory.

R: 37a89ad524ecee26a354e45ac34a6321

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/safe/images/step_09.png)

**What is the name of the file in user's home directory that is a password safe?**

R: MyPasswords.kdbx

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/safe/images/step_10.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/safe/images/step_11.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/safe/images/step_12.png)

**KeePass can be configure to use a file along with a password for unlocking access. What is the name of the image file used as part of the key for this database?**

R: IMG_0547.JPG

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/safe/images/step_13.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/safe/images/step_14.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/safe/images/step_15.png)

**What is the root password on Safe?**

R: u3v2249dl9ptv465cogl3cnpo3fyhk

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/safe/images/step_16.png)

**Submit the flag located in root's home directory.**

R: 9561f27122903c6438cb67f8eabc7c52

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/safe/images/step_17.png)

---

## 🛡️ 4. Plan de Remediación y Recomendaciones de Hardening

### Acciones Tácticas Inmediatas
1. **Aplicación de Parches y Actualización:** Actualizar todos los servicios obsoletos y aplicar los parches correspondientes a las vulnerabilidades documentadas (N/A).
2. **Revisión de Permisos Críticos:** Eliminar permisos SUID/sudo innecesarios y auditar listas de control de acceso (ACLs) en directorios y archivos sensibles.
3. **Cierre de Superficie de Exposición:** Configurar firewalls a nivel de host para denegar puertos de administración o de depuración que no deban estar expuestos públicamente.

### Acciones Estratégicas y de Arquitectura
1. **Principio de Mínimo Privilegio (PoLP):** Configurar servicios y aplicaciones para que se ejecuten bajo cuentas de servicio con privilegios limitados sin acceso a shells interactivas.
2. **Segmentación de Red:** Aislar componentes críticos y servidores de aplicaciones en VLANs dedicadas con inspección de tráfico Este-Oeste.
3. **Monitorización Continua y EDR:** Implementar reglas de detección basadas en comportamiento para identificar ejecuciones anómalas de comandos, spawning de shells y tráfico de red no autorizado.

---

## 📊 5. Mapeo con Matriz MITRE ATT&CK

| Táctica | ID de Técnica | Nombre de Técnica | Aplicación en el Ejercicio |
| :--- | :--- | :--- | :--- |
| **Reconnaissance** | `T1595.002` | Vulnerability Scanning | Detección automatizada y manual de servicios mediante escaneo de puertos |
| **Initial Access** | `T1190` | Exploit Public-Facing Application | Explotación de vulnerabilidad en servicio de red expuesto (Explotación de servicio de red) |
| **Execution** | `T1059` | Command and Scripting Interpreter | Spawning de reverse shell interactiva para control del endpoint |
| **Privilege Escalation** | `T1068 / T1548` | Abuse Elevation Mechanism / Exploitation | Elevación de privilegios hacia superusuario (Escalada local de privilegios) |

---

© 2026 **araizcorp** — Documentación técnica desarrollada para preparación y consultoría de certificaciones **OffSec** (OSCP, OSEP, OSWE).
