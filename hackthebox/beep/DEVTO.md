---
title: "Informe Técnico: Beep (Hack The Box) — Pentesting & Exploit Analysis"
published: true
description: "Informe técnico profesional de auditoria sobre la maquina Beep de Hack The Box: vector de entrada, PoC de explotacion, escalada y mitigaciones OffSec."
tags: cybersecurity, ctf, hackthebox, pentesting
series: "Hack The Box Walkthroughs"
cover_image: "https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/beep/images/step_01.png"
---

# Informe Técnico de Seguridad Ofensiva: Beep

![Hack The Box](https://img.shields.io/badge/Platform-Hack%20The%20Box-green?style=for-the-badge&logo=hackthebox)
![Difficulty](https://img.shields.io/badge/Difficulty-Easy-brightgreen?style=for-the-badge)
![OS](https://img.shields.io/badge/OS-Linux-orange?style=for-the-badge)
![Standards](https://img.shields.io/badge/Methodology-OffSec%20%2F%20PTES-purple?style=for-the-badge)
![Auditor](https://img.shields.io/badge/Assessor-araizcorp%20Offensive%20Security-blue?style=for-the-badge)

**Evaluación de Seguridad Ofensiva y Penetration Testing** realizada sobre el entorno de laboratorio **Beep** de **Hack The Box**, siguiendo los estándares de ejecución metodológica **OffSec PEN-200 (OSCP)** y **PTES (Penetration Testing Execution Standard)**.

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
| **Host Objetivo** | Beep |
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

**Which Linux distribution is the target machine running?**

R: CentOs

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/beep/images/step_01.png)

**What version of TLS is the web application on TCP port 443 using?**

R: 1.0

**What is the name of the software that's hosting a webserver on 443?**

R: elastix

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/beep/images/step_02.png)

**Which Elastix endpoint is vulnerable to a Local File Inclusion?**

R: /vtigercrm/graph.php

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/beep/images/step_04.png)

**What is the name of the FreePBX configuration file that contains the database configuration?**

R:  `amportal.conf%00`

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/beep/images/step_04.png)

**What additional flag is needed when attempting to SSH as root to the target machine due to a "no matching key exchange method found" error? It starts with** `-o` **and ends with** `-sha1`**.**

R: -oKexAlgorithms=+diffie-hellman-group1-sha1

**Submit the flag located in the fanis user's home directory.**

R: 48f4497ed9f950906f40057ca08782d4

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/beep/images/step_05.png)

**Submit the flag located in root's home directory.**

R: 953fbe7d0315fab22cacffe3dee2b893

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/beep/images/step_06.png)

**There are many other ways to root Beep. These questions after the root flag are hints to help identify them. What is the 2012 CVE ID for pre-authentication remote code execution vulnerablity in FreePBX / Elastix?**

R: CVE-2012-4869

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/beep/images/step_07.png)

**What password from the PBX config also works to log in as root to the Webmin application listening on TCP 10000?**

R: jEhdIekWmdjE

![Captura de pantalla 2025-07-30 114917.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/beep/images/step_08.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/beep/images/step_09.png)

**What is the common name for the set of 2014 CVEs where this is a POC exploit:** `() { :; };sleep 10`**?**

R:  Shellshock

**What is the full path to the asterisk user's mail folder on Beep?**

R: /var/mail/asterisk

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/beep/images/step_10.png)

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
