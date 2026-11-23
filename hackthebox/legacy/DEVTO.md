---
title: "Informe Técnico: Legacy (Hack The Box) — Pentesting & Exploit Analysis"
published: true
description: "Informe técnico profesional de auditoria sobre la maquina Legacy de Hack The Box: vector de entrada, PoC de explotacion, escalada y mitigaciones OffSec."
tags: cybersecurity, ctf, hackthebox, pentesting
series: "Hack The Box Walkthroughs"
cover_image: "https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/legacy/images/step_01.png"
---

# Informe Técnico de Seguridad Ofensiva: Legacy

![Hack The Box](https://img.shields.io/badge/Platform-Hack%20The%20Box-green?style=for-the-badge&logo=hackthebox)
![Difficulty](https://img.shields.io/badge/Difficulty-Easy-brightgreen?style=for-the-badge)
![OS](https://img.shields.io/badge/OS-Windows-orange?style=for-the-badge)
![Standards](https://img.shields.io/badge/Methodology-OffSec%20%2F%20PTES-purple?style=for-the-badge)
![Auditor](https://img.shields.io/badge/Assessor-araizcorp%20Offensive%20Security-blue?style=for-the-badge)

**Evaluación de Seguridad Ofensiva y Penetration Testing** realizada sobre el entorno de laboratorio **Legacy** de **Hack The Box**, siguiendo los estándares de ejecución metodológica **OffSec PEN-200 (OSCP)** y **PTES (Penetration Testing Execution Standard)**.

---

## 📑 1. Resumen Ejecutivo (Executive Summary)

* **Objetivo de la Evaluación:** Identificar, documentar y explotar debilidades de seguridad en el host objetivo para evaluar el impacto sobre la confidencialidad, integridad y disponibilidad del sistema.
* **Calificación de Riesgo:** `CRÍTICO` (Acceso no autenticado que deriva en compromiso completo de la máquina y obtención de máximos privilegios administrativos).
* **Vector de Acceso Inicial:** MS08-067 NetAPI (SMB RCE via ms08_067_netapi)
* **Vector de Elevación de Privilegios:** N/A - Exploit otorga NT AUTHORITYSYSTEM directamente
* **Estado Final:** Compromiso total del objetivo (`User Proof` y `Root/System Proof` obtenidas exitosamente).

### Ficha Técnica de la Evaluación

| Parámetro | Detalle Técnico |
| :--- | :--- |
| **Host Objetivo** | Legacy |
| **Plataforma** | Hack The Box |
| **Sistema Operativo** | Windows |
| **Nivel de Dificultad** | Easy |
| **CVE(s) Involucrados** | CVE-2008-4250, CVE-2017-0143 |
| **Vector de Entrada** | MS08-067 NetAPI (SMB RCE via ms08_067_netapi) |
| **Mecanismo de Escalada** | N/A - Exploit otorga NT AUTHORITYSYSTEM directamente |
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
         │─── Ejecución del vector: MS08-067 NetAPI (SMB RCE via ms08_067_netapi)
         └─── Establecimiento de Reverse Shell / Acceso de usuario
         │
         ▼
[ Fase 4: Escalada de Privilegios Local ]
         │─── Auditoría interna: N/A - Exploit otorga NT AUTHORITYSYSTEM directamente
         └─── Elevación a privilegios de superusuario (root / SYSTEM)
         │
         ▼
[ Fase 5: Evidencia & Banderas ]
         └─── Captura de User Flag y Root Flag
```

---

## 🔬 3. Procedimiento Técnico y Evidencias Paso a Paso

CVE(s): CVE-2008-4250, CVE-2017-0143
GTFOBins/Abusos (sudo/SUID): N/A
Initial Access Exploit (Ingreso): MS08-067 NetAPI (SMB RCE via ms08_067_netapi)

Privilege Escalation (Escalada): N/A - Exploit otorga NT AUTHORITYSYSTEM directamente



| Campo | Detalle |
| --- | --- |
| Nombre | Legacy |
| OS | Windows XP |
| Dificultad | Easy |
| CVE | CVE-2008-4250, CVE-2017-0143 |
| Acceso Inicial | MS08-067 NetAPI (SMB RCE) |
| Escalada | N/A - Exploit otorga NT AUTHORITYSYSTEM directamente |

---

## Reconocimiento

Se realizo un escaneo de puertos con Nmap identificando **3 puertos TCP abiertos**. Entre ellos se encuentra el servicio SMB, que resulta ser el vector de ataque principal.

![Captura de pantalla 2025-07-28 124103.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/legacy/images/step_01.png)

## Enumeracion

Se identifico que el servicio SMB es vulnerable a **CVE-2008-4250** (MS08-067), una vulnerabilidad critica de ejecucion remota de codigo en el servicio Server de Windows XP/2003 a traves del protocolo SMB. El script de Nmap `smb-vuln-ms08-067` confirma la vulnerabilidad:

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/legacy/images/step_02.png)

## Explotacion

Se utilizo el modulo de Metasploit **exploit/windows/smb/ms08_067_netapi** para explotar la vulnerabilidad. Este modulo aprovecha un desbordamiento de buffer en la funcion `NetPathCanonicalize()` del servicio Server para obtener ejecucion remota de codigo:

![Captura de pantalla 2025-07-28 124521.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/legacy/images/step_03.png)

El exploit otorga acceso directo como **NT AUTHORITYSYSTEM**, el nivel de privilegios mas alto en Windows. No se requiere escalada de privilegios adicional ya que el exploit compromete el servicio SMB que corre con privilegios de SYSTEM:

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/legacy/images/step_04.png)

## Flags

### User Flag (john)

Con acceso SYSTEM, se navego al escritorio del usuario john para obtener la flag de usuario: `e69af0e4f443de7e36876fda4ec7644f`

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/legacy/images/step_05.png)

### Root Flag (Administrator)

La flag del administrador se encontro en el escritorio de Administrator: `993442d258b0e0ec917cae9e695d5713`

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/legacy/images/step_06.png)

## Notas Adicionales

Ademas de MS08-067, el servicio SMB de Legacy tambien es vulnerable a **CVE-2017-0143** (MS17-010 / EternalBlue), otra vulnerabilidad critica de ejecucion remota de codigo en SMBv1. Esto demuestra que la maquina tiene multiples vectores de ataque a traves de SMB, ambos explotables para obtener SYSTEM directamente.

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/legacy/images/step_07.png)

---

## 🛡️ 4. Plan de Remediación y Recomendaciones de Hardening

### Acciones Tácticas Inmediatas
1. **Aplicación de Parches y Actualización:** Actualizar todos los servicios obsoletos y aplicar los parches correspondientes a las vulnerabilidades documentadas (CVE-2008-4250, CVE-2017-0143).
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
| **Initial Access** | `T1190` | Exploit Public-Facing Application | Explotación de vulnerabilidad en servicio de red expuesto (MS08-067 NetAPI (SMB RCE via ms08_067_netapi)) |
| **Execution** | `T1059` | Command and Scripting Interpreter | Spawning de reverse shell interactiva para control del endpoint |
| **Privilege Escalation** | `T1068 / T1548` | Abuse Elevation Mechanism / Exploitation | Elevación de privilegios hacia superusuario (N/A - Exploit otorga NT AUTHORITYSYSTEM directamente) |

---

© 2026 **araizcorp** — Documentación técnica desarrollada para preparación y consultoría de certificaciones **OffSec** (OSCP, OSEP, OSWE).
