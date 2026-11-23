---
title: "Informe Técnico: Passage (Hack The Box) — Pentesting & Exploit Analysis"
published: true
description: "Informe técnico profesional de auditoria sobre la maquina Passage de Hack The Box: vector de entrada, PoC de explotacion, escalada y mitigaciones OffSec."
tags: cybersecurity, ctf, hackthebox, pentesting
series: "Hack The Box Walkthroughs"
cover_image: "https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/passage/images/step_01.png"
---

# Informe Técnico de Seguridad Ofensiva: Passage

![Hack The Box](https://img.shields.io/badge/Platform-Hack%20The%20Box-green?style=for-the-badge&logo=hackthebox)
![Difficulty](https://img.shields.io/badge/Difficulty-Medium-yellow?style=for-the-badge)
![OS](https://img.shields.io/badge/OS-Linux-orange?style=for-the-badge)
![Standards](https://img.shields.io/badge/Methodology-OffSec%20%2F%20PTES-purple?style=for-the-badge)
![Auditor](https://img.shields.io/badge/Assessor-araizcorp%20Offensive%20Security-blue?style=for-the-badge)

**Evaluación de Seguridad Ofensiva y Penetration Testing** realizada sobre el entorno de laboratorio **Passage** de **Hack The Box**, siguiendo los estándares de ejecución metodológica **OffSec PEN-200 (OSCP)** y **PTES (Penetration Testing Execution Standard)**.

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
| **Host Objetivo** | Passage |
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

**How many open TCP ports are listening on Passage?**

R:2

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/passage/images/step_01.png)

**What anti-brute force software has been installed to protect the server?**

R: Fail2Ban

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/passage/images/step_02.png)

**What content management system (CMS) is being used to host the website?**

R: cutenews

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/passage/images/step_03.png)

**What version of CuteNews is running on Passage?**

R: 2.1.2

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/passage/images/step_04.png)

What is the 2019 CVE ID for a remote code execution vulnerability in this version of CuteNews?

R: CVE-[**2019-11447**](https://nvd.nist.gov/vuln/detail/CVE-2019-11447)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/passage/images/step_05.png)

What user is the webserver running as on Passage?

R: www-data

![Captura de pantalla 2025-08-13 144411.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/passage/images/step_06.png)

What is the paul user's password on Passage?

R: atlanta1

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/passage/images/step_07.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/passage/images/step_08.png)

**Submit the flag located in the paul user's home directory.**

R: ec8e717f0ea2ee16ebfdd1ffcc9c024a

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/passage/images/step_09.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/passage/images/step_10.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/passage/images/step_11.png)

**What is the value of the comment field in the public key in paul's `authorized_keys` file?**

R: nadav@passage

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/passage/images/step_12.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/passage/images/step_13.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/passage/images/step_14.png)

The nadav user has a bunch of groups. Which one jumps out as a potentially quick way to get root?

R: sudo

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/passage/images/step_15.png)

nadav can't run sudo without a password. However, there's a bug from 2019 discovered by Unit 42 that allows bypassing the password requirement for users in the sudoer group through the D-Bus interface of what software?

R: USBCreator

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/passage/images/step_16.png)

![Captura de pantalla 2025-08-13 153547.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/passage/images/step_17.png)

```jsx
Comprueba que estás en sudo y que expone la interfaz:

id
gdbus introspect --system --dest com.ubuntu.USBCreator --object-path /com/ubuntu/USBCreator | head
Copia la clave privada de root a /tmp usando D-Bus (arbitrary file read/write):

gdbus call --system \
  --dest com.ubuntu.USBCreator \
  --object-path /com/ubuntu/USBCreator \
  --method com.ubuntu.USBCreator.Image \
  /root/.ssh/id_rsa /tmp/root_id_rsa true
chmod 600 /tmp/root_id_rsa
```

**Submit the flag located in the root user's home directory.**

R:65db98d4d91bcdc2ff456d97a1e8f59f

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/passage/images/step_18.png)

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
