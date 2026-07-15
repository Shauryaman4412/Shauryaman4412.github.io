---
title: "Vulnversity" 
date: 2026-07-15 00:00:00 0000+
tags: [WriteUp, Pentesting, Nmap, Directory Busting, Gobuster, Web Enumeration, Exploitation, Netcat, Reverse shell, Privilege Escalation, Ports, Root, TryHackMe, php, Systemctl, Offensive Security, Walkthrough ]
categories: [WriteUps, TryHackMe]
image:
    path: /assets/Imgs/Vulnversity/image.png
---
# Vulnversity - TryHackMe

![image.png]( /assets/Imgs/Vulnversity/image.png)

## Vulnversity TryHackMe

Vulnversity is a beginner-friendly TryHackMe room that walks through a full mini pentest: it starts with an Nmap scan that uncovers services like FTP, SSH, Samba, a Squid proxy, and an Apache web server on a custom port, then moves into web enumeration to discover a hidden internal directory containing a file upload form; from there, the challenge is bypassing the upload filter (using Burp Suite's Intruder to fuzz alternate PHP-executable extensions like .phtml) to upload a PHP reverse shell and gain initial access, and finally escalating privileges to root by finding a misconfigured SUID binary (commonly systemctl) — making it a great one-room introduction to reconnaissance, web app exploitation, and Linux privilege escalation all at once.

![image.png]( /assets/Imgs/Vulnversity/image%201.png)

## Reconnaissance

The first stage of Vulnversity is all about reconnaissance — gathering as much information about the target as possible before attempting anything else. Using Nmap with flags like -sV -sC for service/version detection, we scan the target machine and discover six open ports: FTP on port 21, SSH on port 22, Samba/NetBIOS on ports 139 and 445, a Squid HTTP proxy on port 3128, and an Apache web server running on port 3333 instead of the usual port 80. The operating system is identified as Ubuntu. 

```jsx
nmap -sC -sV -vv <Target IP>
```

![image.png]( /assets/Imgs/Vulnversity/image%202.png)

![image.png]( /assets/Imgs/Vulnversity/image%203.png)

![image.png]( /assets/Imgs/Vulnversity/image%204.png)

![image.png]( /assets/Imgs/Vulnversity/image%205.png)

## Web Enumeration

With the web server identified on port 3333, the next step is enumerating the website to find hidden directories and potential entry points. Using a tool like GoBuster or dirb with a common wordlist, we brute-force directory paths on the web server and discover a directory called **/internal** which isn't linked anywhere on the main site. Visiting this path reveals a file upload form,
Further enumeration also uncovers an **/internal/uploads** directory where uploaded files are stored and accessible, which is important because it means if you can get a malicious file past the filter, we can actually navigate to it and trigger its execution.

![image.png]( /assets/Imgs/Vulnversity/image%206.png)

```jsx
gobuster dir -u http://<Target IP><Port> -w /usr/share/word.........  -t 100 -b 404,403
```

-u is used for URL and -w for wordlist with -t for number of threads to be used and -b to bypass the code 404,403(error)

![image.png]( /assets/Imgs/Vulnversity/image%207.png)

![image.png]( /assets/Imgs/Vulnversity/image%208.png)

## Exploitation — File Upload Vulnerability

The upload form on **/internal** blocks standard **.php** files, so uploading a reverse shell straight away doesn't work. To figure out which extensions are allowed, we will start Burp Suite, intercept the upload request, and send it to Intruder where we can run a Sniper attack using a wordlist of alternative PHP-executable extensions like **.php3, .php4, .php5** and **.phtm**l . By comparing the response lengths or status codes, we find that **.phtml** slips through the filter while everything else gets blocked. With that knowledge, we grab the well-known pentestmonkey PHP reverse shell script or can use HackTools extension to download the file, change the IP and port inside it to point back to our attacking machine, rename it with a **.phtml** extension, and upload it through the form. 

![image.png]( /assets/Imgs/Vulnversity/image%209.png)

![image.png]( /assets/Imgs/Vulnversity/image%2010.png)

on the left side is the request where we add the **Silcrow** on the .php extension. Also adding the text file containing the extensions to be tried on the file.

![image.png]( /assets/Imgs/Vulnversity/image%2011.png)

On our attacker machine we start a Netcat listener on the chosen port, then navigate to **/internal/uploads**  in the browser and click on our uploaded file — the server executes it, connects back to our listener, and we now have a reverse shell as the **www-data** user.

```bash
nc -lvnp <port>
```

![image.png]( /assets/Imgs/Vulnversity/image%2012.png)

We have achieved the reverse shell but it is unstable, So we run some commands to make the shell stable which are as follows :

```jsx
python -c 'import pty; pty.spawn("/bin/bash")'
export TERM=xterm
stty raw -echo; fg
stty rows 38 columns 116
```

This should be done in beginning, but I implemented it a bit later. The room hints at looking for SUID binaries, which are files that run with the permissions of their owner (often root) regardless of who executes them. Running

```bash
find / -prem -u=s -type f 2>/dev/null
```

| Part | Meaning |
| --- | --- |
| `find` | the search command |
| `/` | start from root directory (search everywhere) |
| `-perm` | filter by permissions |
| `-u=s` | find files where **user (owner) has SUID bit set** |
| `-type f` | only find **files** (not directories) |
| `2>/dev/null` | hide all **"Permission denied"** errors |

which will provide with this as our output:

![image.png]( /assets/Imgs/Vulnversity/image%2013.png)

Among the usual system binaries in the results, we spot **/bin/systemctl,** which should not normally have SUID permissions and is a known privilege escalation vector. Because **systemctl** manages system services and is running with root privileges, we can craft a malicious service unit file called **root.service** in **/tmp**. The service file contains three sections: a **[Unit]** section with a description, a **[Service]** section with **Type=oneshot** so it runs once, and the critical line **ExecStart=/bin/sh -c “chmod +s /bin/bash”** which adds the SUID bit to **/bin/bash** and a **[Install]** section with **WantedBy=multi-user.target** so systemd knows when to hook it in. 

![image.png]( /assets/Imgs/Vulnversity/image%2014.png)

We then use **systemctl enable** and **systemctl start** on our crafted service, and because systemctl itself has the SUID bit, it executes our service as root, permanently making /bin/bash exploitable so we can simply run **bash -p** to get a root shell and capture the flag.

![image.png]( /assets/Imgs/Vulnversity/image%2015.png)

![image.png]( /assets/Imgs/Vulnversity/image%2016.png)

Rooted!

![image.png]( /assets/Imgs/Vulnversity/image%2017.png)

Thanks for reading.