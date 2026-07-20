# Psycho Break - TryHackMe

![image.png](image.png)

## Psycho Break - TryHackMe

Psycho Break is a TryHackMe room themed around the psychological horror video game "The Evil Within," where we follow Sebastian and his team through a series of challenges that test a wide range of skills. It starts with an Nmap scan revealing FTP, SSH, and HTTP services, then dives into web exploration where GoBuster and manual source code inspection uncover hidden themed rooms with timed challenges and encoded clues. Along the way we encounter multiple cryptography challenges including Atbash and multitap phone ciphers, Morse code decoding from audio files, steganography to extract hidden credentials from images, OSINT through reverse image searching, and writing a custom Python script to brute-force an executable found on an FTP server. The final stretch involves SSH access, using LinPEAS to identify a crontab misconfiguration, and exploiting it to escalate privileges and capture both the user and root flags.

![image.png](image%201.png)

## Enumeration

The room kicks off with an Nmap scan to identify open ports, revealing FTP on port 21, SSH on port 22, and an Apache web server on port 80. Before moving to the web app, a quick check on FTP confirms anonymous login isn't allowed so credentials will be needed later.

```bash
nmap -sC -p- -sV -vv 10.48.151.69
```

![image.png](image%202.png)

![image.png](image%203.png)

![image.png](image%204.png)

![image.png](image%205.png)

Visiting the IP on port 80 in Firefox shows a themed page inspired by "The Evil Within," and running GoBuster against it returns nothing useful, so the next move is manually inspecting the page source code.

```bash
gobuster dir -u [http://10.48.151.69](http://10.48.151.69/) -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t100 -b 404,403
```

![image.png](image%206.png)

 Hidden in the source code is a comment pointing to **/sadistRoom**, and upon visiting that page the "here" link button provides a key for the Locker Room. But we check the source code for any other hints.

![image.png](image%207.png)

![image.png](image%208.png)

![image.png](image%209.png)

![image.png](image%2010.png)

![image.png](image%2011.png)

![image.png](image%2012.png)

After copying and entering the key into a prompt to escape, the page transitions to the Locker Room where Sebastian is hiding inside a locker.

![image.png](image%2013.png)

![image.png](image%2014.png)

The page presents an encoded string that needs to be decoded to access the map — running it through a cipher identifier reveals it's an Atbash cipher, a simple substitution where the alphabet is reversed.  

![image.png](image%2015.png)

![image.png](image%2016.png)

decoding it gives the key : **Grant_me_access_to_the_map_please** to unlock the map which reveals four locations: the Sadist Room, the Locker Room, Safe Haven, and the Abandoned Room.

![image.png](image%2017.png)

![image.png](image%2018.png)

![image.png](image%2019.png)

## Safe Haven

Exploring Safe Haven first

![image.png](image%2020.png)

The page source code contains a hint saying "search through me," signaling to run GoBuster again this time against the /Safeheaven directory.

![image.png](66007767-bab7-46cc-ba8a-bf392ac88db3.png)

```bash
gobuster dir -u http://10.48.151.69/SafeHeaven/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t100 -b 404,403
```

The scan uncovers a hidden directory called /keeper, and visiting it presents a timed challenge to escape the keeper by identifying the real-world location shown in an image before time runs out.

![image.png](image%2021.png)

![image.png](image%2022.png)

![image.png](image%2023.png)

Using Claude to help identify the image, the answer turns out to be St. Augustine Lighthouse, and entering it provides the keeper key for the next task.

![image.png](image%2024.png)

![image.png](image%2025.png)

## The Abandoned Room

Heading back to the map, the Keeper's key unlocks the Abandoned Room, where moving further into the page triggers another timer pressuring you to escape.

![image.png](image%2026.png)

![image.png](image%2027.png)

![image.png](image%2028.png)

Checking the source code reveals that we can use **“shell”** to get out of the page. 

![image.png](image%2029.png)

This means appending **?shell=** to the URL followed by a command, essentially giving us a limited command injection on the server.

```bash
http://10.48.151.69/abandonedRoom/be8bc662d1e36575a52da40beba38275/herecomeslara.php?shell=
```

![image.png](image%2030.png)

 After testing multiple commands like **pwd**, **cd**, and **whoami**, only **ls** works — so using **ls ..** to list the parent directory reveals two different keys.

```bash
http://10.48.151.69/abandonedRoom/be8bc662d1e36575a52da40beba38275/herecomeslara.php?shell=ls ..
```

![image.png](image%2031.png)

entering the first one successfully provides an escape leading to a page containing two files: a text file and a zip archive.

![image.png](image%2032.png)

Where the **you_made_it.txt** file contains: You made it. Escaping from Laura is not easy, good job ....

## Steganography, Cryptography & FTP Access

Downloading the zip file and extracting it reveals two files a helpme.txt file and a Table.jpg image. The text file tells us that there is another user named Ruvik and also states that the key is on the table maybe it has something to do with the JPG file named Table.jpg .

![image.png](image%2033.png)

However, running **exiftool** on **Table.jpg** shows it's not actually an image but another zip archive in disguise. 

![image.png](image%2034.png)

So unzipping it produces two more files: a real image this time and a .wav audio file.

![image.png](image%2035.png)

![image.png](image%2036.png)

Listening to the audio, it's clearly Morse code, so using an online Morse code decoder translates it into the word "SHOWME.”

![image.png](image%2037.png)

![image.png](image%2038.png)

![image.png](image%2039.png)

With that passphrase, running **steghide** on the image extracts hidden data containing FTP credentials for the user Joseph.

![image.png](image%2040.png)

```bash
steghide --extract -sf Joseph_Oda.jpg
```

![image.png](image%2041.png)

Logging into the FTP server as Joseph reveals two files: an executable called program and a dictionary file called random.dic .

![image.png](image%2042.png)

After downloading both and giving the program execute permissions with **chmod +x**, running it with ./program shows it expects a word as an argument — presumably one from random.dic .

![image.png](image%2043.png)

![image.png](image%2044.png)

Since the dictionary contains too many words to try manually, a Python script is written using Claude for time efficiency but it's important to understand what the script actually does: it opens **random.dic**, reads every word line by line, strips the newline characters, and uses **subprocess.run()** to execute **./program** with each word as an argument, printing every attempt so we can track which word produces a different output.

```bash
import os
import subprocess
import sys

f = open("random.dic", "r")

keys = f.readlines()

for key in keys:
    key = str(key.replace("\n", ""))
    print (key)
    subprocess.run(["./program", key])
```

![image.png](image%2045.png)

The word "kidman" turns out to be the correct one,

![image.png](image%2046.png)

and the program outputs a cipher which is identified as a multitap phone cipher using an online tool.

![image.png](image%2047.png)

![image.png](image%2048.png)

Decoding it reveals the message "kidmanpasswordissostrange" the SSH password for the user kidman.

![image.png](image%2049.png)

## SSH Access & Privilege Escalation to Root

With the decoded password **"kidmanpasswordissostrange"** and the username **"kidman"** an SSH connection is established using **ssh kidman@<Target IP>** and entering the password to gain a shell as user kidman.

![image.png](image%2050.png)

 Running ls immediately reveals a **user.txt** file, and using **cat user.txt** gives the first flag.

![image.png](image%2051.png)

Now the hunt for root begins:
checking hidden files with **ls -la** shows nothing particularly useful.

![image.png](image%2052.png)

so to get a better picture of potential escalation vectors, LinPEAS is transferred to the target machine by hosting it on the attacker machine using a Python HTTP server and downloading it on the victim using **wget**.

![image.png](image%2053.png)

![image.png](image%2054.png)

Running LinPEAS highlights a crontab job that executes a Python script called **.the_eye_of_ruvik.py** every two minutes without proper permission restrictions. This script updates a file called **the_eye.tx**t, which is how we can confirm it's actively running on a schedule and more importantly, kidman has write access to it meaning we can modify a file that runs as root.

![image.png](image%2055.png)

![image.png](image%2056.png)

We move to the var where the file .the_eye_of_ruvik.py is present.

![image.png](image%2057.png)

![image.png](image%2058.png)

The first attempt is adding a reverse shell payload into the script, but it doesn't work, possibly a syntax issue or connection problem.

![image.png](image%2059.png)

So a different approach is taken: overwriting the script with the command below:

```bash
echo "import subprocess;subprocess.call('chmod +s /bin/bash', shell=True)" > .the_eye_of_ruvik.py
```

which uses the same SUID trick learned from Vulnversity when the cron job fires as root, it adds the SUID bit to /bin/bash.

![image.png](image%2060.png)

After waiting two minutes for the cron to execute, running **bash -p** spawns a root shell with preserved privileges since the **-p** flag tells bash to skip dropping the elevated permissions.

![image.png](image%2061.png)

Using the **find** command to locate **root.txt** reveals the final flag.

```bash
find / -name root.txt
```

As a bonus task, the room asks to remove the user ruvik from the system, which is accomplished with: 

```bash
deluser --remove-all-files ruvik
```

to completely wipe the user and all their associated files from the machine.

![image.png](image%2062.png)

Rooted!

![image.png](image%2063.png)

Thanks for reading.