---
layout: post
title: Vulnhub - LazySysAdmin 1 Walkthrough
tags: [Hacking, Vulnhub, CTF]
---
LazySysAdmin 1 is a Boot to Root CTF available [here](https://www.vulnhub.com/entry/lazysysadmin-1,205/) on Vulnhub. It's difficulty is rated as Beginner/Intermediate.

## Reconnaissance 
I start by obtaining my IP Address
![IP Address](/img/LazySysAdmin/ifconfig.png)
I then run netdiscover to find the IP Address of the LazySysAdmin VM.
![Netdiscover](/img/LazySysAdmin/netdiscover.png)
The first thing I run after obtaining the IP address is a Nmap scan. I find Port 22 (SSH), 80 (Apache web server), 139 (Samba), 445 (Samba), 3306 (MySQL), and 6667 (InspIRCd).
![Netdiscover](/img/LazySysAdmin/nmap1.png)

## Enumeration
I concurrently run dirb on the web server, while I run some NSE scripts against the Samba ports. Using smb-enum-shares.nse I discover there are three shares on the Samba server - IPC$, print$ and share$. 
![Netdiscover](/img/LazySysAdmin/nsescan.png)
The dirb scan finishes and reveals a wordpress site on the share. I then WPScan on the wordpress site, enumerating for users.
![Netdiscover](/img/LazySysAdmin/dirb.png)
WPScan identifies that there is only one user - Admin. 
![Netdiscover](/img/LazySysAdmin/wpscan.png)
## Samba Server
I decide to try and connect to each of the Samba shares. As I do not have any passwords I use the -N flag. I am successful in getting access to share$.
![Netdiscover](/img/LazySysAdmin/smbconnect.png)
I browse the files and see that the share is pointing to the root directory of the web server. I access the two .txt files via my web browser and obtain a password '12345' 
![Netdiscover](/img/LazySysAdmin/deets.png)
I then navigate to the wordpress folder and download the wp-config.php. Viewing this file gives me the admin password that I use to log into the WordPress admin panel.
![Netdiscover](/img/LazySysAdmin/wpconfig.png)

## WordPress Site
After gaining access to the WordPress site, I fire up Metasploit to use the exploit/multi/script/web_delivery.
![Netdiscover](/img/LazySysAdmin/metasploit.png)

I copy the required code into a .php file on the WordPress site through the 'editor' feature, save the file and reload the WordPress site.... except I didn't get a shell. I try again and get nothing. 

![Netdiscover](/img/LazySysAdmin/edit_theme.png)

I try another php reverse shell from /usr/share/webshells/php/php-reverse-shell.php and fire up Netcat to catch it and again nothing. Frustrated, I was looking at the WordPress site and saw the text 'My name is togie.' written over and over. Perhaps this is a user on the system? I now have two passwords. Lets try.
![Netdiscover](/img/LazySysAdmin/togie.png)

## SSH
I use SSH to connect to the VM using the togie username and the password 12345 and I have success. 

![Netdiscover](/img/LazySysAdmin/togielogin.png)

I decide to see if I have sudo permissions which I do. I then simply sudo su and I have root access.
![Netdiscover](/img/LazySysAdmin/root.png)

I then navigate to root's home directory and view the proof.txt
![Netdiscover](/img/LazySysAdmin/proof.png)


