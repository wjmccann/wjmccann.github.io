---
layout: post
title: Vulnhub - DerpNStink Walkthrough
tags: [Hacking, Vulnhub, CTF]
---

DerpNStink is a Boot to Root CTF available [here](https://www.vulnhub.com/entry/derpnstink-1,221/) on Vulnhub. It's difficulty is rated as Beginner. The VM has four flags hidden throughout.

The VM is set up for bridged networking and the VM has th IP Address of 10.0.0.236, my attacking kali machine is 10.0.0.228.

I run an intial nmap scan and discover that ports 21, 22 and 80 are open.
![](/img/derp/nmap.png)

## Flag 1
I kick off a nikto scan, whilst I browse to the site with my browser
![](/img/derp/web.png)

I view the page source of the home page to find the first flag
![](/img/derp/flag1.png)

## Flag 3
I kick off a gobuster scan to see what other directories are hidden and see that there is a blog.
![](/img/derp/gobuster.png)

I got to browse to it, but it fails to load. My url reverts to derpnstink.local, so I modify my /etc/hosts file to include the VMs IP and name. Now browsing dernnstink.local/weblog I can see that there is a WordPress site. From here I kick off with a wpscan using:
{% highlight bash %}
wpscan -u http://derpnstink.local/weblog -e u
{% endhighlight %}
wpscan tells me there is an arbitary file upload vulnerability and a few XSS vulns as well. It reveals that the users are 'admin' and 'unclestinky'. 
![](/img/derp/wp-scan-vulns.png)
![](/img/derp/wp-scan-users.png)

I head over to http://derpnstink.local/weblog/wp-admin and test default passwords against the admin user. Success! I get access to the WordPress admin page, however I do not have full admin rights and I only have access to a Gallery Upload feature. This just happens to be what the arbitary file upload vulnerability was for. So I go back to my wpscan results and start researching into the vulnerability. There turns out to be a metasploit module for the vulnerability, so I decide to be lazy and fire up msfconsole.

I successfully get a shell and go and access the wp-config.php file and get mysql credentials. I look through mysql and find the hashes for wordpress.
![](/img/derp/wp-hashes.png)

I put the hash for unclestinky into john and then start enumerating the rest of the file system. I find that there are two users stinky and mrderp. Once john cracks the hash I use it to get a shell as stinky.
![](/img/derp/john.png)

Inside stinky's home directory I find a SSH private key, a .pcap file and finally flag 3.
![](/img/derp/privatekey.png)
![](/img/derp/flag3.png)

## Flag 4
Insides Stinky's home directory there is a message between him and Mr Derp which states Mr Derp is having some difficulty logging in and that Stinky will take a pcap to find the problem. So i fire up Wireshark and look at the pcap. I finally find the post request with the password.
![](/img/derp/wireshark.png)

This allows me to su into mrderp. Viewing mrderp's sudo permissions we see that he has permissions for files starting with derpy in /home/mrderp/binaries directory.
![](/img/derp/sudo.png)

I now just create a directory binaries inside mrderps home directory and then create a simple bash script. 
{% highlight bash %}
#!/bin/sh

/bin/bash
{% endhighlight %}

I then run the file with sudo permissions and get a root shell. I browse over to /root and find flag 4.
![](/img/derp/flag4.png)

## Flag 2
You may have noticed that I missed flag 2, well I did. Once I got root I started hunting around for it. Turns out it is well hidden and not necessarily on the path to getting all the flags. I logged back in to WordPress as unclestinky and found flag 2 hidden as a draft post.
![](/img/derp/flag2.png)

## Conclusion
This was a well put together VM that was quick and fun to complete. 
