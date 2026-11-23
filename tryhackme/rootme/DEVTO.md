---
title: "RootME Walkthrough & Writeup — TryHackMe"
published: true
description: "Paso a paso detallado para resolver la maquina RootME en TryHackMe: reconocimiento, analisis, explotacion y escalada de privilegios."
tags: cybersecurity, ctf, tryhackme, pentesting
series: "TryHackMe Walkthroughs"
cover_image: "https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/rootme/images/step_01.png"
---

Escanee la máquina, ¿cuántos puertos están abiertos?

¿Qué versión de Apache está ejecutándose?

¿Qué servicio se está ejecutando en el puerto 22?

![Captura de pantalla 2025-01-04 000057.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/rootme/images/step_01.png)

Encuentre directorios en el servidor web utilizando la herramienta GoBuster

¿Qué es el directorio oculto?

![Captura de pantalla 2025-01-04 000211.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/rootme/images/step_02.png)

usuario.txt

![Captura de pantalla 2025-01-04 000440.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/rootme/images/step_03.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/rootme/images/step_04.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/rootme/images/step_05.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/rootme/images/step_06.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/rootme/images/step_07.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/rootme/images/step_08.png)

Search for files with SUID permission, which file is weird?

Find a form to escalate your privileges.

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/rootme/images/step_09.png)

raíz.txt

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/rootme/images/step_10.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/rootme/images/step_11.png)