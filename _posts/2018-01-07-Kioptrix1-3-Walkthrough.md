---
layout: post
title: Vulnhub - Kioptrix 1.2 Walkthrough
tags: [Hacking, Vulnhub, CTF]
---
Kioptrix 1.2 is a Boot to Root CTF available [here](https://www.vulnhub.com/entry/kioptrix-level-12-3,24/) on Vulnhub. It's difficulty is rated as Beginner. This VM is the third in the Kioprtix series and the third VM in my OSCP preparation series based off [abatchy's blog post](http://www.abatchy.com/2017/02/oscp-like-vulnhub-vms.html).  

The VM and my Kali instance are set up with NAT networking, so to discover the IP address of the VM I run netdiscover

![](/img/kioptrix1-3/netdiscover.png)

Once I obtain the IP, I then run a nmap scan and find the following ports: 22 and 80. 
![](/img/kioptrix1-3/nmap.png)

As per the instructions from Vulnhub, I add the IP address and host name (kioptrix3.com) to my hosts file and then visit the web page. The webpage consists of a blog, a gallery and a login panel.
![](/img/kioptrix1-3/web.png)
![](/img/kioptrix1-3/web2.png)
![](/img/kioptrix1-3/web3.png)

I concurrently run a nikto scan against the site and discover that there is a phpmyadmin page being hosted.
![](/img/kioptrix1-3/nikto.png)

I visit the phpmyadmin page and attempt some default logins. I successfully get access using the username 'admin' and no password. However, I only have access to the information schema and therefore this is no use to me.
![](/img/kioptrix1-3/phpmyadmin.png)

Back on the login panel, I attempt a few SQLI techniques with no success so I go to research the CMS that is hosting the site - Lotus CMS. I find that there is a metasploit module for Lotus CMS so I give this a shot.

![](/img/kioptrix1-3/metasploit.png)

The metasploit module was successful and I gain a meterpreter shell. I drop into a standard shell and run 'uname -a' to get the version of linux that is running. 
![](/img/kioptrix1-3/uname.png)

I research this version of Linux and see that it is vulnerable to the Dirty Cow exploit. I had [previously used this exploit with success](https://wjmccann.github.io/blog/2017/11/15/Sedna-Walkthrough) so I decided I would try it again. Like last time, the exploit seemed to hang whilst running it, but it had created the new user with root level permissions and I was able to SSH in as the new user (The exploit did complete fully, I was just too impatient to wait - but still got the result through SSH).

![](/img/kioptrix1-3/access.png)

I navigated to the home directory for the Root user and found a file titled Congrats.txt. Success!

![](/img/kioptrix1-3/flag.png)

