---
layout: post
title: Vulnhub - Kioptrix 1.3 Walkthrough
tags: [Hacking, Vulnhub, CTF]
---
Kioptrix 1.3 is a Boot to Root CTF available [here](https://www.vulnhub.com/entry/kioptrix-level-13-4,25/) on Vulnhub. It's difficulty is rated as Beginner. This VM is the fourth in the Kioprtix series and the fourth VM in my OSCP preparation series based off [abatchy's blog post](http://www.abatchy.com/2017/02/oscp-like-vulnhub-vms.html).  

The VM and my Kali instance are set up with NAT networking, so to discover the IP address of the VM I run netdiscover

![](/img/kioptrix1-4/netdiscover.png)

Once I obtain the IP, I then run a nmap scan and find the following ports: 22, 80, 139 and 445. 
![](/img/kioptrix1-4/nmap.png)

I intitally browse to the website see that there is a login panel. My first action is to test for possible SQL Injection by entering an apostrophe (') in the user and password fields. 
![](/img/kioptrix1-4/web.png)

This generates an error message which confirms that the login is vulnerable to an SQLI attack. 
![](/img/kioptrix1-4/sql_error.png)

I try to use the username admin and the following as a password
{% highlight bash %}
1' or '1' = '1
{% endhighlight %}
However, I get the following error, which indicates I need to try and find some usernames.
![](/img/kioptrix1-4/web_error.png)

Noting that there is a SMB server running, I use enum4linux to see if I can discover any usernames. I find the usernames loneferret, john and robert. 
![](/img/kioptrix1-4/users.png)

I head back to the login panel and try these usernames. loneferret doesn't work, the username john takes me to a new page with a blank and username and password, however the user robert provides the following information
![](/img/kioptrix1-4/creds.png)

With these new credentials, I decide to try them against the SSH server and I am successful at logging in.
![](/img/kioptrix1-4/access.png)

I discover that I am inside a restrictive shell. To escape the restrictive shell I entered
{% highlight bash %}
echo os.system('/bin/bash')
{% endhighlight %}

![](/img/kioptrix1-4/uname.png)

Running 'uname -a' I see that the VM is running Linux 2.6.24, much like [Kioptrix 1.2](https://wjmccann.github.io/blog/2018/01/07/Kioptrix1-3-Walkthrough) I thought I could run the Dirty Cow exploit and be done. However, wget was not connecting and curl was not installed preventing me from downloading the exploit. Additionally, gcc was not installed so I could not compile an exploit anyway. So I started looking around and noticed that MySQL was running as root on the box and this may be a way in. Further investigation I find the MySQL database login details inside the web root.
![](/img/kioptrix1-4/sql_creds.png)

Further research led me to [this site](https://www.adampalmer.me/iodigitalsec/2013/08/13/mysql-root-to-system-root-with-udf-for-windows-and-linux/) which provided some guidance on how to exploit MySQL User Defined Functions. Interestingly on this VM the library file 'lib_mysqludf_sys.so' was already on the system and the user defined function 'sys_exec' was already created. This meant after logging into MySQL I only needed to enter:
{% highlight bash %}
mysql> use mysql;
mysql> select sys_exec('chmod u+s /bin/bash');
{% endhighlight %}

From here, I exited MySQL back into my bash shell and I had root.
![](/img/kioptrix1-4/root.png)

I now navigated to the Root home directory and viewed the congrats.txt
![](/img/kioptrix1-4/flag.png)
