---
title: "Informe Técnico: Frolic (Hack The Box) — Pentesting & Exploit Analysis"
published: true
description: "Informe técnico profesional de auditoria sobre la maquina Frolic de Hack The Box: vector de entrada, PoC de explotacion, escalada y mitigaciones OffSec."
tags: cybersecurity, ctf, hackthebox, pentesting
series: "Hack The Box Walkthroughs"
cover_image: "https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/frolic/images/step_01.png"
---

# Informe Técnico de Seguridad Ofensiva: Frolic

![Hack The Box](https://img.shields.io/badge/Platform-Hack%20The%20Box-green?style=for-the-badge&logo=hackthebox)
![Difficulty](https://img.shields.io/badge/Difficulty-Easy-brightgreen?style=for-the-badge)
![OS](https://img.shields.io/badge/OS-Linux-orange?style=for-the-badge)
![Standards](https://img.shields.io/badge/Methodology-OffSec%20%2F%20PTES-purple?style=for-the-badge)
![Auditor](https://img.shields.io/badge/Assessor-araizcorp%20Offensive%20Security-blue?style=for-the-badge)

**Evaluación de Seguridad Ofensiva y Penetration Testing** realizada sobre el entorno de laboratorio **Frolic** de **Hack The Box**, siguiendo los estándares de ejecución metodológica **OffSec PEN-200 (OSCP)** y **PTES (Penetration Testing Execution Standard)**.

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
| **Host Objetivo** | Frolic |
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

**How many TCP ports are open on Frolic?**

R: 5

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/frolic/images/step_01.png)

**What is the relative path on the webserver running on TCP port 9999 that returns the path of an instance of PlaySMS?**

R:  /dev/backup

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/frolic/images/step_04.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/frolic/images/step_03.png)

**Which relative path on the website running on port 9999 returns to a login form?**

R: /admin

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/frolic/images/step_04.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/frolic/images/step_05.png)

**What password is required to authenticate as the `admin` user on the login panel located at `/admin`?**

R: `superduperlooperpassword_lol`

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/frolic/images/step_06.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/frolic/images/step_07.png)

**In which programming language is the text displayed on the page that appears after authentication on the `/admin` page? Don't forget the trailing punctuation.**

R: Ook!

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/frolic/images/step_08.png)

What relative path is output by the Ook! program?

R:/asdiSIAJJ0QWE9JAS

![Captura de pantalla 2025-08-08 212333.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/frolic/images/step_09.png)

**What kind of file can be recovered from the data at `/asdiSIAJJ0QWE9JAS`?**

R: zip

![Captura de pantalla 2025-08-08 214727.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/frolic/images/step_10.png)

```jsx
echo 'UEsDBBQACQAIAMOJN00j/lsUsAAAAGkCAAAJABwAaW5kZXgucGhwVVQJAAOFfKdbhXynW3V4CwAB
BAAAAAAEAAAAAF5E5hBKn3OyaIopmhuVUPBuC6m/U3PkAkp3GhHcjuWgNOL22Y9r7nrQEopVyJbs
K1i6f+BQyOES4baHpOrQu+J4XxPATolb/Y2EU6rqOPKD8uIPkUoyU8cqgwNE0I19kzhkVA5RAmve
EMrX4+T7al+fi/kY6ZTAJ3h/Y5DCFt2PdL6yNzVRrAuaigMOlRBrAyw0tdliKb40RrXpBgn/uoTj
lurp78cmcTJviFfUnOM5UEsHCCP+WxSwAAAAaQIAAFBLAQIeAxQACQAIAMOJN00j/lsUsAAAAGkC
AAAJABgAAAAAAAEAAACkgQAAAABpbmRleC5waHBVVAUAA4V8p1t1eAsAAQQAAAAABAAAAABQSwUG
AAAAAAEAAQBPAAAAAwEAAAAA' > encoded.txt


base64 -d encoded.txt > archivo.zip


unzip archivo.zip

Archive:  archivo.zip
[archivo.zip] index.php password: 

zip2john archivo.zip > hash

john hash --wordlist=/usr/share/wordlists/rockyou.txt

unzip archivo.zip
```

**What is the password retrieved from the file that is obtained from the above subdirectory?**

R:idkwhatispass

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/frolic/images/step_11.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/frolic/images/step_12.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/frolic/images/step_13.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/frolic/images/step_14.png)

What is the 2017 CVE ID for a remote code execution vulnerability in PlaySMS?

R:CVE-2017-9101

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/frolic/images/step_15.png)

Submit the flag located in the ayush user's home directory.

R: bd7bf744e1ec61ef57a18ab3f5517d28

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/frolic/images/step_16.png)

![Captura de pantalla 2025-08-08 225009.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/frolic/images/step_17.png)

ayush

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/frolic/images/step_18.png)

**What is the full path to the SetUID binary that is owned by root?**

R: /home/ayush/.binary/rop

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/frolic/images/step_19.png)

**The `rop` executable is vulnerable to a buffer overflow. How many bytes of junk should be put in the user input before the return address to get control over the EIP (the instruction pointer)?**

R: 52

```jsx
PATTERN=$(/usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 600)
gdb -q --args /tmp/rop "$PATTERN"
(gdb) run
(gdb) i r rip

```

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/frolic/images/step_20.png)

**Submit the flag located in root's home directory.**

R: 278adea30dbd2368c0e366c57df18214

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/frolic/images/step_21.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/frolic/images/step_22.png)

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
