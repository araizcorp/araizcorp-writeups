---
title: "Informe Técnico: BountyHunter (Hack The Box) — Pentesting & Exploit Analysis"
published: true
description: "Informe técnico profesional de auditoria sobre la maquina BountyHunter de Hack The Box: vector de entrada, PoC de explotacion, escalada y mitigaciones OffSec."
tags: cybersecurity, ctf, hackthebox, pentesting
series: "Hack The Box Walkthroughs"
cover_image: "https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/bountyhunter/images/step_01.png"
---

# Informe Técnico de Seguridad Ofensiva: BountyHunter

![Hack The Box](https://img.shields.io/badge/Platform-Hack%20The%20Box-green?style=for-the-badge&logo=hackthebox)
![Difficulty](https://img.shields.io/badge/Difficulty-Easy-brightgreen?style=for-the-badge)
![OS](https://img.shields.io/badge/OS-Linux-orange?style=for-the-badge)
![Standards](https://img.shields.io/badge/Methodology-OffSec%20%2F%20PTES-purple?style=for-the-badge)
![Auditor](https://img.shields.io/badge/Assessor-araizcorp%20Offensive%20Security-blue?style=for-the-badge)

**Evaluación de Seguridad Ofensiva y Penetration Testing** realizada sobre el entorno de laboratorio **BountyHunter** de **Hack The Box**, siguiendo los estándares de ejecución metodológica **OffSec PEN-200 (OSCP)** y **PTES (Penetration Testing Execution Standard)**.

---

## 📑 1. Resumen Ejecutivo (Executive Summary)

* **Objetivo de la Evaluación:** Identificar, documentar y explotar debilidades de seguridad en el host objetivo para evaluar el impacto sobre la confidencialidad, integridad y disponibilidad del sistema.
* **Calificación de Riesgo:** `CRÍTICO` (Acceso no autenticado que deriva en compromiso completo de la máquina y obtención de máximos privilegios administrativos).
* **Vector de Acceso Inicial:** XXE en parser XML del formulario → exfiltrar ficheros → creds → RCE
* **Vector de Elevación de Privilegios:** sudo python / capabilities
* **Estado Final:** Compromiso total del objetivo (`User Proof` y `Root/System Proof` obtenidas exitosamente).

### Ficha Técnica de la Evaluación

| Parámetro | Detalle Técnico |
| :--- | :--- |
| **Host Objetivo** | BountyHunter |
| **Plataforma** | Hack The Box |
| **Sistema Operativo** | Linux |
| **Nivel de Dificultad** | Easy |
| **CVE(s) Involucrados** | N/A |
| **Vector de Entrada** | XXE en parser XML del formulario → exfiltrar ficheros → creds → RCE |
| **Mecanismo de Escalada** | sudo python / capabilities |
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
         │─── Ejecución del vector: XXE en parser XML del formulario → exfiltrar ficheros → creds → RCE
         └─── Establecimiento de Reverse Shell / Acceso de usuario
         │
         ▼
[ Fase 4: Escalada de Privilegios Local ]
         │─── Auditoría interna: sudo python / capabilities
         └─── Elevación a privilegios de superusuario (root / SYSTEM)
         │
         ▼
[ Fase 5: Evidencia & Banderas ]
         └─── Captura de User Flag y Root Flag
```

---

## 🔬 3. Procedimiento Técnico y Evidencias Paso a Paso

GTFOBins/Abusos (sudo/SUID): python
Initial Access Exploit (Ingreso): XXE en parser XML del formulario → exfiltrar ficheros → creds → RCE

Privilege Escalation (Escalada): sudo python / capabilities

**What is the path to the endpoint of the web application that data is submitted to via a POST request from the form on the Bounty Report System - Beta page?**

R:/tracker_diRbPr00f314.php

![Captura de pantalla 2025-08-22 124551.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/bountyhunter/images/step_01.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/bountyhunter/images/step_02.png)

**The data in the POST request is encoded with URL encoding and what other encoding?**

R: base64

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/bountyhunter/images/step_03.png)

**Looking at the data in the POST request, after URL-decoding and base64-decoding, what is the name of the data format of the data remains?**

R: xxe

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/bountyhunter/images/step_04.png)

**The XXE vulnerability can be validated by reading a known file on the target such as '/etc/passwd', but typically when viewing a PHP file the source code will be hidden. Can this be bypassed to view the source code of the web application PHP files using PHP filters?**

R: yes

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/bountyhunter/images/step_05.png)

**What is the 'dbpassword' value in the file 'db.php'?**

R: m19RoAU0hP41A1sTsq6K

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/bountyhunter/images/step_06.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/bountyhunter/images/step_07.png)

**Submit the flag located in the development user's home directory.**

R: eba1365b7732709ecc9d6a1f7b16c793

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/bountyhunter/images/step_08.png)

**What is the filename on the target that we have the ability to run as 'root' without requiring a password?**

R: [ticketValidator.py](http://ticketvalidator.py/)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/bountyhunter/images/step_09.png)

**Performing code analysis on the 'evaluate()' function in the Python script, which dangerous method is being used that could lead to arbitary code execution?**

R: 555c10c58c00ccb3f965a2b5a9a5a915

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/bountyhunter/images/step_10.png)

> 
> 
> 
> # Qué hace el script
> 
> 1. `load_file(loc)`: abre un fichero **.md**; si no termina en `.md` sale.
> 2. `evaluate(ticketFile)`: lee el Markdown línea a línea y valida:
>     - Línea 0 empieza por `# Skytrain Inc`
>     - Línea 1 empieza por `## Ticket to`  (imprime destino)
>     - Encuentra la línea `__Ticket Code:__` y toma **la siguiente** como el “código” (debe empezar por `*`).
>     - En esa línea:
>         - Calcula `ticketCode` = **primer término antes de un `+`**, quitando `*` (ej.: `*32+...` ⇒ `32`).
>         - Si `int(ticketCode) % 7 == 4`, entonces hace:
>             
>             ```python
>             validationNumber = eval(x.replace("**", ""))
>             
>             ```
>             
>             y devuelve “válido” si `validationNumber > 100`.
>             
> 3. `main()`: pide ruta del fichero, lo evalúa y dice si es válido. *(Tiene un bug menor: `ticket.close` sin `()` no cierra el fichero.)*
> 
> # Por qué `eval()` permite ejecución de código (RCE)
> 
> - **La línea del “ticket” es controlada por el usuario** (viene del .md que lee el programa).
> - El programa evalúa **toda la expresión Python** de esa línea (sin `*`). `eval` no solo suma números; **puede llamar funciones, importar módulos, acceder a atributos, etc.**
> - La única “barrera” es que el primer término (antes del `+`) cumpla `n % 7 == 4`. Eso se satisface con muchísimos valores (25, 32, 39, 46, …).
> - Resultado: basta poner una expresión que ejecute algo malicioso y que además dé un número > 100 para pasar la validación.
> 
> ## PoC (payload malicioso)
> 
> Contenido mínimo de un `.md` válido que ejecuta un comando y devuelve >100:
> 
> ```markdown
> # Skytrain Inc
> ## Ticket to Nowhere
> __Ticket Code:__
> **179+ 25 == 204 and __import__('os').system('/bin/bash') == True
> 
> ```
> 
> Explicación:
> 
> - `ticketCode = 32` → `32 % 7 == 4` ✔️
> - `__import__('os').system("id")` ejecuta `id`. Suele devolver `0` si todo bien.
> - En Python, `0 or 101` ⇒ `101`.
> - Así que la expresión total evalúa a `32 + 101 = 133` (>100) ⇒ “Valid ticket.”, **y además ejecutó tu comando** (RCE).
> 
> Puedes sustituir el `id` por cualquier comando, por ejemplo una reverse shell, exfiltración, etc. Incluso sin `os.system` hay vectores más creativos (atributos/`__subclasses__`), razón por la que **“eval pero con restricciones” no es seguro**.
> 
> # Cómo arreglarlo (seguro de verdad)
> 
> No uses `eval`. Si solo necesitas “suma de enteros separados por `+`”, **parsea tú y suma**:
> 
> ```python
> import re
> 
> # ...
> if code_line and i == code_line:
>     if not x.startswith("**"):
>         return False
>     expr = x.replace("**", "").strip()
> 
>     # Acepta SOLO "num (+ num)*" con espacios opcionales
>     if not re.fullmatch(r'\d+(?:\s*\+\s*\d+)*', expr):
>         return False
> 
>     ticketCode = int(expr.split('+', 1)[0])
>     if ticketCode % 7 == 4:
>         # Sumar números de forma segura
>         validationNumber = sum(int(n) for n in re.findall(r'\d+', expr))
>         return validationNumber > 100
> 
> ```
> 
> Y de paso:
> 
> - Cierra el fichero: `ticket.close()` **o** usa `with open(loc) as ticket:`.
> - Cambia `exit()` por `sys.exit(1)` y maneja excepciones de `open()`.
> - Si el destino solo se usa para imprimir, **no lo evalúes jamás**.
> 
> ### Si de verdad necesitas evaluar expresiones
> 
> - `ast.literal_eval` **no** sirve para `1+2` (solo literales).
> - Usa un parser de expresiones seguro (p. ej. eval “whitelist” de operadores/nodos con `ast.parse`), o una lib diseñada para ello (p. ej. *simpleeval* con entorno estrictamente limitado). Aun así, lo más seguro es **no evaluar nunca input del usuario**.

```jsx

## Ticket to Mars
__Ticket Code:__
**179+ 25 == 204 and __import__('os').system('/bin/bash') == True
```

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
| **Initial Access** | `T1190` | Exploit Public-Facing Application | Explotación de vulnerabilidad en servicio de red expuesto (XXE en parser XML del formulario → exfiltrar ficheros → creds → RCE) |
| **Execution** | `T1059` | Command and Scripting Interpreter | Spawning de reverse shell interactiva para control del endpoint |
| **Privilege Escalation** | `T1068 / T1548` | Abuse Elevation Mechanism / Exploitation | Elevación de privilegios hacia superusuario (sudo python / capabilities) |

---

© 2026 **araizcorp** — Documentación técnica desarrollada para preparación y consultoría de certificaciones **OffSec** (OSCP, OSEP, OSWE).
