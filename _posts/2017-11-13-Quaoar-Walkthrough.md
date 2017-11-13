---
layout: post
title: Vulnhub - Quaoar Walkthrough
tags: [Hacking, Vulnhub, CTF]
---
Quaoar is a Boot to Root CTF available [here](https://www.vulnhub.com/entry/hackfest2016-quaoar,180/) on Vulnhub. It's difficulty is rated as Very Easy and there are three flags to capture.

I start with using netdiscover to identify the IP address of the VM.

![](/img/quaoar/netdiscover.png)

I then run an nmap scan and find the open ports are: 22, 53, 80, 110, 139, 143, 993 and 995. 

![](/img/quaoar/nmap1.png)
![](/img/quaoar/nmap2.png)

I browse to the web server on port 80 and find very little other than the site instructing me to 'hack the planet'. 

![](/img/quaoar/web1.png)
![](/img/quaoar/web2.png)

I run dirb against the site and find that there is a wordpress site, I go to the wp-admin login page and try Admin/Admin first up to see if default credentials are used and it is successful!

![](/img/quaoar/dirb.png)
![](/img/quaoar/wplogin.png)

I go to Appearance > Editor to modify the .php files of the wordpress site. In the footer.php I copy in the PHP reverse shell found at /usr/share/webshells/php/php-reverse-shell.php on Kali. 

![](/img/quaoar/edittheme.png)

I fire up a Netcat listener and catch the reverse shell when I re-load a WordPress page.

![](/img/quaoar/shell.png)

The shell I have is as user www-data and I upgrade it by running *python -c 'import pty; pty.spawn("/bin/sh")'*. I browse through the files in the web directory and go to view wp-config.php in the wordpress folder. I find the username and password in the file as root/rootpassword!. 

![](/img/quaoar/rootpassword.png)

I try to su using the password rootpassword! and it is successful at gaining root privileges.

![](/img/quaoar/rootlogin.png)

I then navigate to the home directory of root and view the flag.

![](/img/quaoar/rootflag.png)

I then navigate to the /home directory and see there is a user - wpadmin. Inside their home directory is the next flag.

![](/img/quaoar/userflag.png)

The description of the VM states that there is a third flag after gaining root. So I start to look around. I finally find the final flag tucked away in a file at /etc/cron.d/php5.

![](/img/quaoar/extraflag.png)
