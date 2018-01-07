---
layout: post
title: Vulnhub - Stapler 1 Walkthrough
tags: [Hacking, Vulnhub, CTF]
---

Stapler:1 is a Boot to Root CTF available [here](https://www.vulnhub.com/entry/stapler-1,150/) on Vulnhub. It's difficulty is rated as Beginner/Intermediate. This VM is the fifth in my OSCP preparation series based off [abatchy's blog post](http://www.abatchy.com/2017/02/oscp-like-vulnhub-vms.html).  

The VM and my Kali instance are set up with NAT networking, so to discover the IP address of the VM I run netdiscover

![](/img/stapler/netdiscover.png)

Once I obtain the IP, I then run a nmap scan and initially find the following ports: 21, 22, 53, 80, 139, 666 and 3306. 
![](/img/stapler/nmap.png)
![](/img/stapler/nmap2.png)

I intitally browse to the website see that there is a 404 error message. I run a nikto scan against the site and see that it is mapped to a home directory as I have access to the .bashrc and .profile files.
![](/img/kioptrix1-4/nikto.png)

I also run dirb against the website and find nothing. I then move onto the FTP server as my nmap scanned informed me that it allows anonymous login. I browse the FTP server and only find a single file - note. However it doesn't provide much information.
![](/img/stapler/note.png)

I next look at the SMB server as I knew from my nmap scan that it allowed guest access. I run enum4linux and identify a share titled 'kathy'. I connect to this share using smbclient and see that there are two folders - 'kathy_stuff' and 'backup'. 
![](/img/stapler/smb.png)

Inside the folder kathy_stuff there is a file titled 'todo-list.txt'. However  this doesn't provide much info other than there are backups. Inside the backup folder there is a vsftpd.conf file and a wordpress-4.tar.gz file. The vsftpd.conf doesn't tell me anything I already knew (that FTP allowed anonymous access) and the wordpress-4 archive contained a standard wordpress installation backup. No wp-config.php with details. 
![](/img/stapler/backup.png)

However, the WordPress backup did reveal a clue. That there may be a WordPress installation on this box. I decide to fully scan all TCP ports using unicornscan just incase I missed something initially. I see port 12380 open. I nmap this port and see that it is a web server.
![](/img/stapler/unicorn.png)
![](/img/stapler/12380.png)

I visit the site and there is nothing revealed other than 'a web site is coming soon' message.
![](/img/stapler/12380-web.png)

I run nikto against the web server and find that there are three subdirectories - /admin112233/, /blogblog/ and /phpmyadmin/
![](/img/stapler/nikto2.png)

The /admin112233/ directory redirects me to xss-payloads.com, however blogblog lands me at the WordPress blog that I knew was running somewhere.
![](/img/stapler/blog.png)

I run wpscan against the WordPress Instance and enumerate all users. From the wpscan I see that /wp-content/ and /wp-includes/ have directory listing enabled. I viewed the installed plugins and saw that advanced-video-embed-embed-videos-or-playlists v1.0 is installed. A little bit of research reveals that this plugin is vulnerable to a [LFI exploit](https://www.exploit-db.com/exploits/39646/). 
![](/img/stapler/plugins.png)

I run the exploit by pointing my browser to: 
{% highlight bash %}
https://192.168.5.134/blogblog/wp-admin/admin-ajax.php?action=ave_publishPost&title=random&short=1&term=1&thumb=../wp-config.php
{% endhighlight %}
This creates a new post on the page, which points to a .jpg file.
![](/img/stapler/file.png)

I navigate to the /wp-content/uploads/ folder and download the .jpg that is there. The .jpg file is not a vailed .jpg file, it contains the contents of the wp-config.php file that I requested through the LFI exploit. I simply cat the contents of the .jpg file and view the database username and password
![](/img/stapler/wpconfig.png)

I then use these credentials to access the phpmyadmin interface. I then navigate to the wordpress directory and view the WordPress users and their passwords. I copy all the password hashes into a file and then run hashcat to attempt to crack them.
{% highlight bash %}
hashcat -m 400 -a 0 hashes /usr/share/wordlists/rockyou.txt
{% endhighlight %}

As the hashes start to come in, I start accessing the WordPress admin panel with the credentials. Most users were simply standard users, it wasn't until I got the password for the user john (who was the first user in the database) that I finally got admin access. 
![](/img/stapler/wpadmin.png)

I was unable to modify any pages through the editor function, but I was able to upload a php reverse shell via the install plugins function. I modified and uploaded the php reverse shell found in /usr/share/webshells/php/ in Kali. I navigated to the uploads folder, started a netcat listener and executed the reverse shell.
![](/img/stapler/access.png)

Getting access via this method meant that I was the www-data user. I ran a uname -a and saw that the Kernel version was 4.4.0-21. I then proceeded to try a number of privilege escalation exploits without much success. Finally, I came across [this exploit](https://www.exploit-db.com/exploits/39772/) on Exploit-DB which was successful in gaining root.  The exploit was extremely easy to run, you simply run the 'compile.sh' file which will create the doubleput binary. You run that binary and root is gained.

I then navigated to Root's home directory and viewed the flag.
![](/img/stapler/flag.png)

This was a fun, but frustrating VM, where I went down a number of rabbit holes before finally getting access. Privilege Escalation took multiple attempts with multiple exploits before arriving at the right one. It was a great feeling once I finally got that flag!




