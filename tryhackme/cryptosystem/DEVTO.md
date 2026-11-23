---
title: "Cryptosystem Walkthrough & Writeup — TryHackMe"
published: true
description: "Paso a paso detallado para resolver la maquina Cryptosystem en TryHackMe: reconocimiento, analisis, explotacion y escalada de privilegios."
tags: cybersecurity, ctf, tryhackme, pentesting
series: "TryHackMe Walkthroughs"
cover_image: "https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/cryptosystem/images/step_01.png"
---

```jsx
from Crypto.Util.number import *
from sympy import nextprime, isprime
from math import isqrt


n = 15956250162063169819282947443743274370048643274416742655348817823973383829364700573954709256391245826513107784713930378963551647706777479778285473302665664446406061485616884195924631582130633137574953293367927991283669562895956699807156958071540818023122362163066253240925121801013767660074748021238790391454429710804497432783852601549399523002968004989537717283440868312648042676103745061431799927120153523260328285953425136675794192604406865878795209326998767174918642599709728617452705492122243853548109914399185369813289827342294084203933615645390728890698153490318636544474714700796569746488209438597446475170891
c = 3591116664311986976882299385598135447435246460706500887241769555088416359682787844532414943573794993699976035504884662834956846849863199643104254423886040489307177240200877443325036469020737734735252009890203860703565467027494906178455257487560902599823364571072627673274663460167258994444999732164163413069705603918912918029341906731249618390560631294516460072060282096338188363218018310558256333502075481132593474784272529318141983016684762611853350058135420177436511646593703541994904632405891675848987355444490338162636360806437862679321612136147437578799696630631933277767263530526354532898655937702383789647510
e = 65537


print("[*] Buscando p y q...")
found = False
for k in range(100000):
    p = isqrt(n) - k
    if not isprime(p):
        continue
    q = nextprime(p)
    if p * q == n:
        print(f"[+] ¡Encontrado!\np = {p}\nq = {q}")
        found = True
        break

if not found:
    print("[-] No se encontraron factores.")
    exit()


phi = (p - 1) * (q - 1)
d = inverse(e, phi)


m = pow(c, d, n)
print(f"[+] Flag: {long_to_bytes(m).decode()}")

```

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/cryptosystem/images/step_01.png)

## 🧾 Writeup: RSA Crypto Challenge — TryHackMe

### 🧠 Descripción del reto

El reto consistía en recuperar la clave secreta y descifrar un mensaje cifrado con RSA. Nos entregaban únicamente el código fuente del sistema de cifrado, que contenía lo siguiente:

- Generación de `p` usando `getPrime(1024)`
- Generación de `q` como el **primo inmediatamente posterior a `p`**
- Cifrado de un mensaje `FLAG` con exponentes públicos `e = 65537`

También se nos proporcionaban los valores de `n` y `c` directamente en el script comentado.

---

### 🔍 Análisis del código

Fragmento crítico del código entregado:

```python
python
CopiarEditar
p = getPrime(1024)
q = primo(p)  # siguiente primo tras p
n = p * q
e = 65537
d = inverse(e, (p-1)*(q-1))
c = pow(bytes_to_long(FLAG.encode()), e, n)

```

Este esquema representa una implementación **vulnerable de RSA**, ya que `q` y `p` son **primos consecutivos**, lo que debilita significativamente el módulo `n`.

---

### 💥 Vulnerabilidad

Cuando `q = next_prime(p)`, `p` y `q` están extremadamente cerca, por lo que se puede realizar una **factorización eficiente** de `n` recorriendo hacia atrás desde `sqrt(n)` hasta encontrar un `p` tal que `q = next_prime(p)` y `p*q == n`.

Este es un tipo de ataque similar al de **Coppersmith** en idea, pero en este caso simplemente aprovechamos la cercanía entre los factores.

---

### 🧰 Script de ataque

```python
python
CopiarEditar
from Crypto.Util.number import *
from sympy import nextprime, isprime
from math import isqrt

n = <valor dado>
c = <valor dado>
e = 65537

for k in range(100000):
    p = isqrt(n) - k
    if not isprime(p):
        continue
    q = nextprime(p)
    if p * q == n:
        break

phi = (p - 1) * (q - 1)
d = inverse(e, phi)
m = pow(c, d, n)
print(long_to_bytes(m).decode())

```

---

### ✅ Resultado

El script encuentra correctamente `p` y `q`, calcula la clave privada `d`, y descifra el mensaje `c`, revelando la flag:

```
CopiarEditar
THM{Just_s0m3_small_amount_of_RSA!}

```

---

### 🧠 Lecciones aprendidas

- Nunca generes `q` como el siguiente primo tras `p`, ya que hace la factorización de `n` trivial.
- En RSA, la fortaleza se basa en la **dificultad de factorizar grandes números**, y este enfoque rompe esa premisa.
- Este tipo de ejercicios son típicos en retos de CTF y preparación para OSCP, especialmente en temas de **criptografía aplicada**.