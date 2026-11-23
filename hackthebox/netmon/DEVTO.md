---
title: "Informe Técnico: Netmon (Hack The Box) — Pentesting & Exploit Analysis"
published: true
description: "Informe técnico profesional de auditoria sobre la maquina Netmon de Hack The Box: vector de entrada, PoC de explotacion, escalada y mitigaciones OffSec."
tags: cybersecurity, ctf, hackthebox, pentesting
series: "Hack The Box Walkthroughs"
cover_image: "https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/netmon/images/step_01.png"
---

# Informe Técnico de Seguridad Ofensiva: Netmon

![Hack The Box](https://img.shields.io/badge/Platform-Hack%20The%20Box-green?style=for-the-badge&logo=hackthebox)
![Difficulty](https://img.shields.io/badge/Difficulty-Easy-brightgreen?style=for-the-badge)
![OS](https://img.shields.io/badge/OS-Windows-orange?style=for-the-badge)
![Standards](https://img.shields.io/badge/Methodology-OffSec%20%2F%20PTES-purple?style=for-the-badge)
![Auditor](https://img.shields.io/badge/Assessor-araizcorp%20Offensive%20Security-blue?style=for-the-badge)

**Evaluación de Seguridad Ofensiva y Penetration Testing** realizada sobre el entorno de laboratorio **Netmon** de **Hack The Box**, siguiendo los estándares de ejecución metodológica **OffSec PEN-200 (OSCP)** y **PTES (Penetration Testing Execution Standard)**.

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
| **Host Objetivo** | Netmon |
| **Plataforma** | Hack The Box |
| **Sistema Operativo** | Windows |
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

**What is the name of the application running on port 80? Given the three words in the logo.**

R: PRTG Network Monitor

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/netmon/images/step_01.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/netmon/images/step_02.png)

**What service is running on TCP port 21?**

R: FTP

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/netmon/images/step_03.png)

**Submit the flag located on the Public user's desktop.**

R: 8a4c7dbf3c94f3bccb4cab6f64c08f07

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/netmon/images/step_04.png)

**What is the full path of the folder where PRTG Network Monitor saves its configuration files by default?**

R:C:\ProgramData\Paessler\PRTG Network Monitor

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/netmon/images/step_05.png)

**What is the name of the backup config file?**

R: PRTG Configuration.old.bak

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/netmon/images/step_06.png)

**What was the prtgadmin user's password according to that file?**

R: PrTg@dmin2018

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/netmon/images/step_07.png)

**What is the prtgadmin user's password on the website now?**

R: PrTg@dmin2019

**What version of PRTG is installed?**

R: 18.1.37.13946 

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/netmon/images/step_08.png)

![Captura de pantalla 2025-08-24 232117.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/netmon/images/step_09.png)

**Which user is this software running as by default? Don't include anything before a `\`.**

R: system

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/netmon/images/step_10.png)

**Submit the flag located on the administrator's desktop.**

R: 19c7accae74c6746dcd7dba271f2ed13

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/netmon/images/step_11.png)

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
