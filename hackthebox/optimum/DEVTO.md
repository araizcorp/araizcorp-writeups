---
title: "Informe Técnico: Optimum (Hack The Box) — Pentesting & Exploit Analysis"
published: true
description: "Informe técnico profesional de auditoria sobre la maquina Optimum de Hack The Box: vector de entrada, PoC de explotacion, escalada y mitigaciones OffSec."
tags: cybersecurity, ctf, hackthebox, pentesting
series: "Hack The Box Walkthroughs"
cover_image: "https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/optimum/images/step_01.png"
---

# Informe Técnico de Seguridad Ofensiva: Optimum

![Hack The Box](https://img.shields.io/badge/Platform-Hack%20The%20Box-green?style=for-the-badge&logo=hackthebox)
![Difficulty](https://img.shields.io/badge/Difficulty-Easy-brightgreen?style=for-the-badge)
![OS](https://img.shields.io/badge/OS-Windows-orange?style=for-the-badge)
![Standards](https://img.shields.io/badge/Methodology-OffSec%20%2F%20PTES-purple?style=for-the-badge)
![Auditor](https://img.shields.io/badge/Assessor-araizcorp%20Offensive%20Security-blue?style=for-the-badge)

**Evaluación de Seguridad Ofensiva y Penetration Testing** realizada sobre el entorno de laboratorio **Optimum** de **Hack The Box**, siguiendo los estándares de ejecución metodológica **OffSec PEN-200 (OSCP)** y **PTES (Penetration Testing Execution Standard)**.

---

## 📑 1. Resumen Ejecutivo (Executive Summary)

* **Objetivo de la Evaluación:** Identificar, documentar y explotar debilidades de seguridad en el host objetivo para evaluar el impacto sobre la confidencialidad, integridad y disponibilidad del sistema.
* **Calificación de Riesgo:** `CRÍTICO` (Acceso no autenticado que deriva en compromiso completo de la máquina y obtención de máximos privilegios administrativos).
* **Vector de Acceso Inicial:** Rejetto HFS 2.3 RCE (findMacroMarker)
* **Vector de Elevación de Privilegios:** local_exploit_suggester + Kernel Exploit
* **Estado Final:** Compromiso total del objetivo (`User Proof` y `Root/System Proof` obtenidas exitosamente).

### Ficha Técnica de la Evaluación

| Parámetro | Detalle Técnico |
| :--- | :--- |
| **Host Objetivo** | Optimum |
| **Plataforma** | Hack The Box |
| **Sistema Operativo** | Windows |
| **Nivel de Dificultad** | Easy |
| **CVE(s) Involucrados** | CVE-2014-6287 |
| **Vector de Entrada** | Rejetto HFS 2.3 RCE (findMacroMarker) |
| **Mecanismo de Escalada** | local_exploit_suggester + Kernel Exploit |
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
         │─── Ejecución del vector: Rejetto HFS 2.3 RCE (findMacroMarker)
         └─── Establecimiento de Reverse Shell / Acceso de usuario
         │
         ▼
[ Fase 4: Escalada de Privilegios Local ]
         │─── Auditoría interna: local_exploit_suggester + Kernel Exploit
         └─── Elevación a privilegios de superusuario (root / SYSTEM)
         │
         ▼
[ Fase 5: Evidencia & Banderas ]
         └─── Captura de User Flag y Root Flag
```

---

## 🔬 3. Procedimiento Técnico y Evidencias Paso a Paso

CVE(s): CVE-2014-6287
GTFOBins/Abusos (sudo/SUID): N/A
Initial Access Exploit (Ingreso): Rejetto HFS 2.3 RCE (findMacroMarker)

Privilege Escalation (Escalada): local_exploit_suggester + Kernel Exploit



| Campo | Detalle |
| --- | --- |
| Nombre | Optimum |
| OS | Windows Server 2012 R2 |
| Dificultad | Easy |
| CVE | CVE-2014-6287 |
| Acceso Inicial | Rejetto HFS 2.3 RCE (findMacroMarker) |
| Escalada | local_exploit_suggester (Metasploit) |

---

## Reconocimiento

Se identifico el servicio **HttpFileServer (HFS) version 2.3** corriendo en el puerto TCP 80. HFS es un servidor web ligero utilizado para compartir archivos, y esta version especifica contiene una vulnerabilidad critica de ejecucion remota de codigo:

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/optimum/images/step_01.png)

## Enumeracion

La version 2.3 de HFS es vulnerable a **CVE-2014-6287**, una vulnerabilidad de ejecucion remota de codigo en la funcion `findMacroMarker`. El fallo permite inyectar comandos del sistema operativo a traves de caracteres null byte (`%00`) en la busqueda del servidor:

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/optimum/images/step_02.png)

## Explotacion

Al explotar CVE-2014-6287, se obtuvo una shell como el usuario **kostas**, que es el usuario bajo el cual corre el servicio HFS. La flag de usuario se encontro en el escritorio de kostas: `eab5aa9ba0ed1d805556ed23acfcb7e0`

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/optimum/images/step_03.png)

Adicionalmente, se identifico la contrasena del usuario kostas: `kdeEjDowkS*`

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/optimum/images/step_04.png)

## Escalada de Privilegios

Para escalar privilegios, se utilizo el modulo de Metasploit **post/multi/recon/local_exploit_suggester**, que analiza el sistema comprometido y sugiere exploits locales aplicables. Este modulo identifico vulnerabilidades de kernel explotables para obtener NT AUTHORITYSYSTEM:

![Captura de pantalla 2025-07-30 174411.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/optimum/images/step_05.png)

![Captura de pantalla 2025-07-30 174555.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/optimum/images/step_06.png)

### Root Flag (Administrator)

Tras la escalada de privilegios exitosa, se obtuvo la flag del administrador: `109a117b89b316614742c21782d3c8e6`

![Captura de pantalla 2025-07-30 174522.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/optimum/images/step_07.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/optimum/images/step_08.png)

---

## 🛡️ 4. Plan de Remediación y Recomendaciones de Hardening

### Acciones Tácticas Inmediatas
1. **Aplicación de Parches y Actualización:** Actualizar todos los servicios obsoletos y aplicar los parches correspondientes a las vulnerabilidades documentadas (CVE-2014-6287).
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
| **Initial Access** | `T1190` | Exploit Public-Facing Application | Explotación de vulnerabilidad en servicio de red expuesto (Rejetto HFS 2.3 RCE (findMacroMarker)) |
| **Execution** | `T1059` | Command and Scripting Interpreter | Spawning de reverse shell interactiva para control del endpoint |
| **Privilege Escalation** | `T1068 / T1548` | Abuse Elevation Mechanism / Exploitation | Elevación de privilegios hacia superusuario (local_exploit_suggester + Kernel Exploit) |

---

© 2026 **araizcorp** — Documentación técnica desarrollada para preparación y consultoría de certificaciones **OffSec** (OSCP, OSEP, OSWE).
