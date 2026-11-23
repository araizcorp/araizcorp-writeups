---
title: "Informe Técnico: Nibbles (Hack The Box) — Pentesting & Exploit Analysis"
published: true
description: "Informe técnico profesional de auditoria sobre la maquina Nibbles de Hack The Box: vector de entrada, PoC de explotacion, escalada y mitigaciones OffSec."
tags: cybersecurity, ctf, hackthebox, pentesting
series: "Hack The Box Walkthroughs"
cover_image: "https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/nibbles/images/step_01.png"
---

# Informe Técnico de Seguridad Ofensiva: Nibbles

![Hack The Box](https://img.shields.io/badge/Platform-Hack%20The%20Box-green?style=for-the-badge&logo=hackthebox)
![Difficulty](https://img.shields.io/badge/Difficulty-Easy-brightgreen?style=for-the-badge)
![OS](https://img.shields.io/badge/OS-Linux-orange?style=for-the-badge)
![Standards](https://img.shields.io/badge/Methodology-OffSec%20%2F%20PTES-purple?style=for-the-badge)
![Auditor](https://img.shields.io/badge/Assessor-araizcorp%20Offensive%20Security-blue?style=for-the-badge)

**Evaluación de Seguridad Ofensiva y Penetration Testing** realizada sobre el entorno de laboratorio **Nibbles** de **Hack The Box**, siguiendo los estándares de ejecución metodológica **OffSec PEN-200 (OSCP)** y **PTES (Penetration Testing Execution Standard)**.

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
| **Host Objetivo** | Nibbles |
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

**How many open TCP ports are listening on Nibbles?**

R: 2

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/nibbles/images/step_01.png)

What is the relative path on the webserver to a blog?

R: nibbleblog

![Captura de pantalla 2025-07-29 093821.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/nibbles/images/step_02.png)

**What content management system (CMS) is being used by the blog??**

R:`Nibbleblog`

**What is the relative path to an XML file that contains the admin username?**

R: /nibbleblog/content/private/users.xml

![Captura de pantalla 2025-07-29 094424.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/nibbles/images/step_03.png)

![Captura de pantalla 2025-07-29 094402.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/nibbles/images/step_04.png)

**What is the admin user's password to log into the blog?**

R: nibbles

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/nibbles/images/step_05.png)

**What version of nibble blog is running on the target machine? Do not include the "v".**

R: 4.0.3

![Captura de pantalla 2025-07-29 095924.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/nibbles/images/step_06.png)

**What is the 2015 CVE ID for an authenticated code execution by file upload vulnerability in this version of NibbleBlog.**

R: CVE-2015-6967

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/nibbles/images/step_07.png)

**Which user the Nibbleblog instance is running on the target machine?**

R: nibbler

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/nibbles/images/step_08.png)

![Captura de pantalla 2025-07-29 101250.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/nibbles/images/step_09.png)

**Submit the flag located in the nibbler user's home directory.**

R: 0250b3bc05dfc02856ef5172482b306a

![Captura de pantalla 2025-07-29 101453.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/nibbles/images/step_10.png)

**What is the name of the script that nibbler can run as root on Nibbles?**

R: monitor.sh

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/nibbles/images/step_11.png)

**Enter the permission set on** `monitor.sh`**? Use the Linux file permissions format, like** `-rw-rw-r--`**.**

R: -rwxrwxrwx

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/nibbles/images/step_12.png)

**Submit the flag located in root's home directory.**

R:f95fabd93167ed40fb18f97427337595

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/nibbles/images/step_13.png)

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
