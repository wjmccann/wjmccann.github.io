---
layout: post
title: Vulnhub - Sedna Walkthrough
tags: [Hacking, Vulnhub, CTF]
---
Sedna is a Boot to Root CTF available [here](https://www.vulnhub.com/entry/hackfest2016-sedna,181/) on Vulnhub. It's difficulty is rated as Medium and there are four flags to capture; obtaining a shell, obtaining root and two post exploitation flags.

The VM provides us with its IP address, so I start with an nmap scan. I find ports 22, 53, 80, 110, 111, 139, 143, 445, 993, 995 and 8080 open.

![](/img/sedna/nmap1.png)
![](/img/sedna/nmap2.png)

I start by going to the site hosted and Port 80 and realise that it is much like the VM I recently did titled [Quauar](https://wjmccann.github.io/blog/2017/11/13/Quaoar-Walkthrough) with the 'Hack the Planet' images.

![](/img/sedna/web1.png)
![](/img/sedna/web2.png)

I then run a dirb and nikto scan against the webserver. The dirb scan identifies a number of folders for me to explore and the nikto scan finds some interesing files such as the license.txt file. I have a look at this file as it provides a clue as to the CMS they are using - Builder Engine.

![](/img/sedna/dirb.png)
![](/img/sedna/nikto.png)

I browse through the folders that dirb found and eventually stumble upon a further clue - They are using Builder Engine v3.

![](/img/sedna/be.png)

I search through searchsploit and find an arbituary file upload exploit for Builder Engine v3. I copy the exploit to my home directory and save it as a .html file. I then modify the exploit to point it at my target and then upload a PHP Reverse Shell. 
![](/img/sedna/searchsploit.png)
![](/img/sedna/exploit1.png)
![](/img/sedna/exploit2.png)

The PHP Reverse Shell ends up in the files directory that I found using dirb.
![](/img/sedna/exploit3.png)

I set up a netcat listener and catch the shell. I gain a shell as the www-data user.
![](/img/sedna/shell.png)

I find the first flag located at /var/www/
![](/img/sedna/flag1.png)

I copy across the [LinEnum]() script that I use to automate Linux enumeration and run it. I take note of the Kernel version (3.13.0) and look to see if there are any privilege escalation vulnerabilities. 

![](/img/sedna/kernel.png

Through my research I identify the Dirty Cow exploit as being a potential candidate. Details of the Dirty Cow exploit can be [found here](https://dirtycow.ninja/), however it is basically a kernel exploit that can allow an unprivileged user to write to privileged files (Copy Over Write - C.O.W) and in the case of the exploit I used, it overwrites the /etc/passwd to add in a new user. The exploit I used was found [here](https://www.exploit-db.com/exploits/40839/). I compile the exploit as per the instructions and execute it (yes, I left the default user in the exploit, because I had a sensible chuckle at the username FireFarts).

![](/img/sedna/dirtycow.png)

The first time the exploit ran, it hung for quite some time and then the VM crashed. Not a great exploit to run if this was a live target! I reset the VM and tried again. This time when the program hung, I quickly ssh'd into the box in another terminal and I was successful at gaining accesses. I was quickly able to view the root flag prior to the box crashing again.

![](/img/sedna/flag2.png)

Although a new user was created that allowed me to access the box, this new account was not persisting during reboots. This meant I had to find another exploit if I was to find the remaining two post exploitation flags. However, I ran into some issues with my Kali instance which meant I had to rebuild my Kali box so I called it a day with Sedna, happy that I was able to at least get the Root flag. 

I did get a chance to view the /etc/passwd file whiles as www-dat and observed a user named - 'crackmeforpoints' which I believe is a big clue for one of the post exploitation flags.
