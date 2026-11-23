---
title: "Informe Técnico: Basterd (Hack The Box) — Pentesting & Exploit Analysis"
published: true
description: "Informe técnico profesional de auditoria sobre la maquina Basterd de Hack The Box: vector de entrada, PoC de explotacion, escalada y mitigaciones OffSec."
tags: cybersecurity, ctf, hackthebox, pentesting
series: "Hack The Box Walkthroughs"
cover_image: "https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/basterd/images/step_02.png"
---

# Informe Técnico de Seguridad Ofensiva: Basterd

![Hack The Box](https://img.shields.io/badge/Platform-Hack%20The%20Box-green?style=for-the-badge&logo=hackthebox)
![Difficulty](https://img.shields.io/badge/Difficulty-Medium-yellow?style=for-the-badge)
![OS](https://img.shields.io/badge/OS-Windows-orange?style=for-the-badge)
![Standards](https://img.shields.io/badge/Methodology-OffSec%20%2F%20PTES-purple?style=for-the-badge)
![Auditor](https://img.shields.io/badge/Assessor-araizcorp%20Offensive%20Security-blue?style=for-the-badge)

**Evaluación de Seguridad Ofensiva y Penetration Testing** realizada sobre el entorno de laboratorio **Basterd** de **Hack The Box**, siguiendo los estándares de ejecución metodológica **OffSec PEN-200 (OSCP)** y **PTES (Penetration Testing Execution Standard)**.

---

## 📑 1. Resumen Ejecutivo (Executive Summary)

* **Objetivo de la Evaluación:** Identificar, documentar y explotar debilidades de seguridad en el host objetivo para evaluar el impacto sobre la confidencialidad, integridad y disponibilidad del sistema.
* **Calificación de Riesgo:** `CRÍTICO` (Acceso no autenticado que deriva en compromiso completo de la máquina y obtención de máximos privilegios administrativos).
* **Vector de Acceso Inicial:** Drupal 7.54 RCE (Drupalgeddon)
* **Vector de Elevación de Privilegios:** CVE-2015-1701 Windows Kernel Exploit (APT28)
* **Estado Final:** Compromiso total del objetivo (`User Proof` y `Root/System Proof` obtenidas exitosamente).

### Ficha Técnica de la Evaluación

| Parámetro | Detalle Técnico |
| :--- | :--- |
| **Host Objetivo** | Basterd |
| **Plataforma** | Hack The Box |
| **Sistema Operativo** | Windows |
| **Nivel de Dificultad** | Medium |
| **CVE(s) Involucrados** | CVE-2015-1701 |
| **Vector de Entrada** | Drupal 7.54 RCE (Drupalgeddon) |
| **Mecanismo de Escalada** | CVE-2015-1701 Windows Kernel Exploit (APT28) |
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
         │─── Ejecución del vector: Drupal 7.54 RCE (Drupalgeddon)
         └─── Establecimiento de Reverse Shell / Acceso de usuario
         │
         ▼
[ Fase 4: Escalada de Privilegios Local ]
         │─── Auditoría interna: CVE-2015-1701 Windows Kernel Exploit (APT28)
         └─── Elevación a privilegios de superusuario (root / SYSTEM)
         │
         ▼
[ Fase 5: Evidencia & Banderas ]
         └─── Captura de User Flag y Root Flag
```

---

## 🔬 3. Procedimiento Técnico y Evidencias Paso a Paso

CVE(s): CVE-2015-1701
GTFOBins/Abusos (sudo/SUID): N/A
Initial Access Exploit (Ingreso): Drupal 7.54 RCE (Drupalgeddon)

Privilege Escalation (Escalada): CVE-2015-1701 Windows Kernel Exploit (APT28)



| Campo | Detalle |
| --- | --- |
| Nombre | Bastard |
| OS | Windows Server 2008 R2 |
| Dificultad | Medium |
| CVE | CVE-2015-1701 (Escalada) |
| Acceso Inicial | Drupal 7.54 RCE (Drupalgeddon) |
| Escalada | CVE-2015-1701 Kernel Exploit (APT28) |

---

## Reconocimiento

Se identificaron **3 puertos TCP abiertos**. El sitio web corre sobre IIS con el CMS Drupal:

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/basterd/images/step_02.png)

## Enumeracion

El CMS identificado es **Drupal**:

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/basterd/images/step_02.png)

La version de Drupal es **7.54**, vulnerable a multiples exploits de ejecucion remota de codigo (Drupalgeddon):

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/basterd/images/step_03.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/basterd/images/step_04.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/basterd/images/step_05.png)

## Explotacion

Al explotar Drupal 7.54, se obtuvo una shell como usuario **iusr** (cuenta de servicio web de IIS):

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/basterd/images/step_06.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/basterd/images/step_07.png)

![Captura de pantalla 2025-08-10 164914.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/basterd/images/step_08.png)

## Escalada de Privilegios

El sistema **no tiene hotfixes aplicados**, haciendolo vulnerable a multiples exploits de kernel:

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/basterd/images/step_09.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/basterd/images/step_10.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/basterd/images/step_11.png)

Se utilizo **CVE-2015-1701**, una vulnerabilidad en el kernel de Windows originalmente reportada por FireEye como utilizada por el grupo APT28, que permite escalada a SYSTEM:

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/basterd/images/step_12.png)

## Flags

### User Flag (dimitris)

Flag del usuario dimitris: `5cdd6e3d031d49c9a6e652d7a7c5214f`

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/basterd/images/step_13.png)

### Root Flag (Administrator)

Flag del administrador: `f160caa8a43ea8eb472fa12dbc66ddc8`

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/basterd/images/step_14.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/basterd/images/step_15.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/basterd/images/step_16.png)

---

## 🛡️ 4. Plan de Remediación y Recomendaciones de Hardening

### Acciones Tácticas Inmediatas
1. **Aplicación de Parches y Actualización:** Actualizar todos los servicios obsoletos y aplicar los parches correspondientes a las vulnerabilidades documentadas (CVE-2015-1701).
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
| **Initial Access** | `T1190` | Exploit Public-Facing Application | Explotación de vulnerabilidad en servicio de red expuesto (Drupal 7.54 RCE (Drupalgeddon)) |
| **Execution** | `T1059` | Command and Scripting Interpreter | Spawning de reverse shell interactiva para control del endpoint |
| **Privilege Escalation** | `T1068 / T1548` | Abuse Elevation Mechanism / Exploitation | Elevación de privilegios hacia superusuario (CVE-2015-1701 Windows Kernel Exploit (APT28)) |

---

© 2026 **araizcorp** — Documentación técnica desarrollada para preparación y consultoría de certificaciones **OffSec** (OSCP, OSEP, OSWE).
