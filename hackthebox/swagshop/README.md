# Informe Técnico de Seguridad Ofensiva: SwagShop

![Hack The Box](https://img.shields.io/badge/Platform-Hack%20The%20Box-green?style=for-the-badge&logo=hackthebox)
![Difficulty](https://img.shields.io/badge/Difficulty-Easy-brightgreen?style=for-the-badge)
![OS](https://img.shields.io/badge/OS-Linux-orange?style=for-the-badge)
![Standards](https://img.shields.io/badge/Methodology-OffSec%20%2F%20PTES-purple?style=for-the-badge)
![Auditor](https://img.shields.io/badge/Assessor-araizcorp%20Offensive%20Security-blue?style=for-the-badge)

**Evaluación de Seguridad Ofensiva y Penetration Testing** realizada sobre el entorno de laboratorio **SwagShop** de **Hack The Box**, siguiendo los estándares de ejecución metodológica **OffSec PEN-200 (OSCP)** y **PTES (Penetration Testing Execution Standard)**.

---

## 📑 1. Resumen Ejecutivo (Executive Summary)

* **Objetivo de la Evaluación:** Identificar, documentar y explotar debilidades de seguridad en el host objetivo para evaluar el impacto sobre la confidencialidad, integridad y disponibilidad del sistema.
* **Calificación de Riesgo:** `CRÍTICO` (Acceso no autenticado que deriva en compromiso completo de la máquina y obtención de máximos privilegios administrativos).
* **Vector de Acceso Inicial:** Magento 1.9.0.0 Shoplift SQLi + Admin Panel RCE
* **Vector de Elevación de Privilegios:** sudo vi (GTFOBins)
* **Estado Final:** Compromiso total del objetivo (`User Proof` y `Root/System Proof` obtenidas exitosamente).

### Ficha Técnica de la Evaluación

| Parámetro | Detalle Técnico |
| :--- | :--- |
| **Host Objetivo** | SwagShop |
| **Plataforma** | Hack The Box |
| **Sistema Operativo** | Linux |
| **Nivel de Dificultad** | Easy |
| **CVE(s) Involucrados** | Magento Shoplift (SQLi) |
| **Vector de Entrada** | Magento 1.9.0.0 Shoplift SQLi + Admin Panel RCE |
| **Mecanismo de Escalada** | sudo vi (GTFOBins) |
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
         │─── Ejecución del vector: Magento 1.9.0.0 Shoplift SQLi + Admin Panel RCE
         └─── Establecimiento de Reverse Shell / Acceso de usuario
         │
         ▼
[ Fase 4: Escalada de Privilegios Local ]
         │─── Auditoría interna: sudo vi (GTFOBins)
         └─── Elevación a privilegios de superusuario (root / SYSTEM)
         │
         ▼
[ Fase 5: Evidencia & Banderas ]
         └─── Captura de User Flag y Root Flag
```

---

## 🔬 3. Procedimiento Técnico y Evidencias Paso a Paso

CVE(s): Magento Shoplift (SQLi)
GTFOBins/Abusos (sudo/SUID): vi
Initial Access Exploit (Ingreso): Magento 1.9.0.0 Shoplift SQLi + Admin Panel RCE

Privilege Escalation (Escalada): sudo vi (GTFOBins)



| Campo | Detalle |
| --- | --- |
| Nombre | SwagShop |
| OS | Linux (Ubuntu) |
| Dificultad | Easy |
| CVE | Magento Shoplift (SQLi) |
| Acceso Inicial | Magento 1.9.0.0 Shoplift SQLi + Admin RCE |
| Escalada | sudo vi (GTFOBins) |

---

## Reconocimiento

Se identificaron **2 puertos TCP abiertos** (SSH y HTTP):

![image.png](./images/step_01.png)

## Enumeracion

El sitio web utiliza **Magento**, una plataforma de comercio electronico:

![image.png](./images/step_02.png)

La version en uso es **Magento 1.9.0.0**, vulnerable a la conocida vulnerabilidad Shoplift:

![image.png](./images/step_03.png)

## Explotacion

La vulnerabilidad Magento Shoplift (SQLi) permite modificar las credenciales del usuario **admin**, otorgando acceso al panel de administracion:

![image.png](./images/step_04.png)

![image.png](./images/step_05.png)

![Captura de pantalla 2025-08-06 103624.png](./images/step_06.png)

Desde el panel de administracion de Magento, se obtuvo ejecucion remota de codigo. El servidor web corre como **www-data**:

![image.png](./images/step_07.png)

## Flags

### User Flag (haris)

Flag del usuario haris: `5fad8e7069f06cf3b596f927fcee35f4`

![image.png](./images/step_08.png)

## Escalada de Privilegios

El usuario www-data puede ejecutar **vi** como root sin contrasena (`sudo -l`). Usando GTFOBins, se escala a root ejecutando `:!/bin/bash` dentro de vi:

![image.png](./images/step_09.png)

### Root Flag

Flag de root: `7ecfcf588fcc799dd5a1f5363fd0a166`

![Captura de pantalla 2025-08-06 125244.png](./images/step_10.png)

![image.png](./images/step_11.png)

---

## 🛡️ 4. Plan de Remediación y Recomendaciones de Hardening

### Acciones Tácticas Inmediatas
1. **Aplicación de Parches y Actualización:** Actualizar todos los servicios obsoletos y aplicar los parches correspondientes a las vulnerabilidades documentadas (Magento Shoplift (SQLi)).
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
| **Initial Access** | `T1190` | Exploit Public-Facing Application | Explotación de vulnerabilidad en servicio de red expuesto (Magento 1.9.0.0 Shoplift SQLi + Admin Panel RCE) |
| **Execution** | `T1059` | Command and Scripting Interpreter | Spawning de reverse shell interactiva para control del endpoint |
| **Privilege Escalation** | `T1068 / T1548` | Abuse Elevation Mechanism / Exploitation | Elevación de privilegios hacia superusuario (sudo vi (GTFOBins)) |

---

© 2026 **araizcorp** — Documentación técnica desarrollada para preparación y consultoría de certificaciones **OffSec** (OSCP, OSEP, OSWE).
