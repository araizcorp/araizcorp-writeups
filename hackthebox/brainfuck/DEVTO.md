---
title: "Informe Técnico: Brainfuck (Hack The Box) — Pentesting & Exploit Analysis"
published: true
description: "Informe técnico profesional de auditoria sobre la maquina Brainfuck de Hack The Box: vector de entrada, PoC de explotacion, escalada y mitigaciones OffSec."
tags: cybersecurity, ctf, hackthebox, pentesting
series: "Hack The Box Walkthroughs"
cover_image: "https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/brainfuck/images/step_01.png"
---

# Informe Técnico de Seguridad Ofensiva: Brainfuck

![Hack The Box](https://img.shields.io/badge/Platform-Hack%20The%20Box-green?style=for-the-badge&logo=hackthebox)
![Difficulty](https://img.shields.io/badge/Difficulty-Insane-red?style=for-the-badge)
![OS](https://img.shields.io/badge/OS-Linux-orange?style=for-the-badge)
![Standards](https://img.shields.io/badge/Methodology-OffSec%20%2F%20PTES-purple?style=for-the-badge)
![Auditor](https://img.shields.io/badge/Assessor-araizcorp%20Offensive%20Security-blue?style=for-the-badge)

**Evaluación de Seguridad Ofensiva y Penetration Testing** realizada sobre el entorno de laboratorio **Brainfuck** de **Hack The Box**, siguiendo los estándares de ejecución metodológica **OffSec PEN-200 (OSCP)** y **PTES (Penetration Testing Execution Standard)**.

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
| **Host Objetivo** | Brainfuck |
| **Plataforma** | Hack The Box |
| **Sistema Operativo** | Linux |
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

Enumeration

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/brainfuck/images/step_01.png)

Wordpress

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/brainfuck/images/step_02.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/brainfuck/images/step_03.png)

![Captura de pantalla 2025-08-29 122931.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/brainfuck/images/step_04.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/brainfuck/images/step_05.png)

```jsx
wpscan --url [https://brainfuck.htb](https://brainfuck.htb/) -e --disable-tls-checks 
```

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/brainfuck/images/step_06.png)

```jsx
wpscan --url [https://brainfuck.htb](https://brainfuck.htb/) \
--disable-tls-checks \
--random-user-agent \
--plugins-detection aggressive \
--enumerate ap,at,tt,cb,u
```

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/brainfuck/images/step_07.png)

![Captura de pantalla 2025-08-29 131941.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/brainfuck/images/step_08.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/brainfuck/images/step_09.png)

![Captura de pantalla 2025-08-29 144736.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/brainfuck/images/step_10.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/brainfuck/images/step_11.png)

![Captura de pantalla 2025-08-29 144800.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/brainfuck/images/step_12.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/brainfuck/images/step_13.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/brainfuck/images/step_14.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/brainfuck/images/step_15.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/brainfuck/images/step_16.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/brainfuck/images/step_17.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/brainfuck/images/step_18.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/brainfuck/images/step_19.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/brainfuck/images/step_20.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/brainfuck/images/step_21.png)

```jsx

p = int("7493025776465062819629921475535241674460826792785520881387158343265274170009282504884941039852933109163193651830303308312565580445669284847225535166520307")
q = int("7020854527787566735458858381555452648322845008266612906844847937070333480373963284146649074252278753696897245898433245929775591091774274652021374143174079")
e = int("30802007917952508422792869021689193927485016332713622527025219105154254472344627284947779726280995431947454292782426313255523137610532323813714483639434257536830062768286377920010841850346837238015571464755074669373110411870331706974573498912126641409821855678581804467608824177508976254759319210955977053997")
c = int("44641914821074071930297814589851746700593470770417111804648920018396305246956127337150936081144106405284134845851392541080862652386840869768622438038690803472550278042463029816028777378141217023336710545449512973950591755053735796799773369044083673911035030605581144977552865771395578778515514288930832915182")

n = p*q
phi = (p-1)*(q-1)


def egcd(a,b):
    x,y, u,v = 0,1, 1,0
    while a:
        q,r = b//a, b%a
        b,a, x,y, u,v = a,r, u,v, x-u*q, y-v*q
    return b,x,y

_, d, _ = egcd(e, phi)
d %= phi

pt = pow(c, d, n)                 # plaintext como entero
hx = f"{pt:x}"                    # a hex
print(bytes.fromhex(hx).decode()) # a ASCII -> root.txt
```

![Captura de pantalla 2025-08-29 170052.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/brainfuck/images/step_22.png)

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
