# RootME — TryHackMe Walkthrough & Writeup

![TryHackMe](https://img.shields.io/badge/Platform-TryHackMe-blue?style=for-the-badge&logo=tryhackme)
![Difficulty](https://img.shields.io/badge/Difficulty-Walkthrough-yellow?style=for-the-badge)
![OS](https://img.shields.io/badge/OS-Linux-orange?style=for-the-badge&logo=linux)
![Author](https://img.shields.io/badge/Author-Pedro%20Araiz%20(araizcorp)-informational?style=for-the-badge)

Guía completa y walkthrough paso a paso para resolver la máquina **RootME** en la plataforma **TryHackMe**, incluyendo reconocimiento, explotación de vulnerabilidades y escalada de privilegios para obtener todas las flags.

---

Escanee la máquina, ¿cuántos puertos están abiertos?

¿Qué versión de Apache está ejecutándose?

¿Qué servicio se está ejecutando en el puerto 22?

![Captura de pantalla 2025-01-04 000057.png](./images/step_01.png)

Encuentre directorios en el servidor web utilizando la herramienta GoBuster

¿Qué es el directorio oculto?

![Captura de pantalla 2025-01-04 000211.png](./images/step_02.png)

usuario.txt

![Captura de pantalla 2025-01-04 000440.png](./images/step_03.png)

![image.png](./images/step_04.png)

![image.png](./images/step_05.png)

![image.png](./images/step_06.png)

![image.png](./images/step_07.png)

![image.png](./images/step_08.png)

Search for files with SUID permission, which file is weird?

Find a form to escalate your privileges.

![image.png](./images/step_09.png)

raíz.txt

![image.png](./images/step_10.png)

![image.png](./images/step_11.png)