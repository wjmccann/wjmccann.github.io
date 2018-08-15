---
layout: post
title: Vulnhub - Lampaio Walkthrough
tags: [Hacking, Vulnhub, CTF]
---
Lampaio is a Boot to Root CTF available [here](https://www.vulnhub.com/entry/lampiao-1,249/) on Vulnhub. It's difficulty is rated as Easy.

# Scanning
To begin with, I kicked off searching for the VM on my network using netdiscover. I found that the VM had the IP 192.168.206.136.
![](/img/lampaio/netdiscover.png)

This is then followed up with an nmap scan which reveals ports 22 and 80. 
![](/img/lampaio/nmap1.png)

I headover to port 80 with my web browser and I am greeted with some interesting ASCII art. 
![](/img/lampaio/port80.PNG)

I then kick off gobuster thinking that there must be something else hidden, but the scan fails. I sense something is up and through a few more web scans but they give me nothing. Stumped, I decide to run nmap again, but this time scanning all ports.
![](/img/lampaio/nmap2.png)

This time I have some more success and find that there is port 1898 open. I then run a more detailed scan using -sC (default scripts) and -sV (version detection) and find that this port his hosting a web site and the banners are informing me that it is hosting a Drupal 7 instance.
![](/img/lampaio/nmap3.png)

# Drupalgeddon 2
I browse to the site and it looks like a Drupal Blog. I check the robots.txt and it shows all the familiar entries of a Drupal site. I view the change log and confirm that this instance is version 7.54. This means there is a high chance that this will be vulnerable to the Drupalgeddon2 exploit. You can see a blog post I made about this exploit and running it against a Windows installation [here](https://wjmccann.github.io/blog/2018/06/02/Drupalgeddon2).
![](/img/lampaio/drupal.PNG)
![](/img/lampaio/changelog.png)

I then fire up metasploit and load exploit/unix/webapp/drupal_drupalgeddon2. Set my RHOST and RPORT and I successfully get a shell!
![](/img/lampaio/shell.png)

# Privilege Escalation
I started hunting around to find the avenue to exploit the box in order to gain root access, but I was starting to come up short. I tried a few kernel exploits with no success, so I decided to resort to a tool called [linux-exploit-suggestor](https://github.com/mzet-/linux-exploit-suggester). Interestinly it suggested the Dirty COW 2 exploit. I thought this was typically for Kernel versions below 3.9 and this machine was running 4.4.0-32-generic.
![](/img/lampaio/exploit.png)

I was getting desperate and I was starting to work my way through all exploits suggested by linux-exploit-suggestor. I tried the original .c version of the exploit, but it crashed the box. So I then tried the C++ version. Initially, it didn't work and that was because I wasn't using a proper tty shell. I quickily upgraded my shell and re-run the exploit. Success! I got Root!
![](/img/lampaio/root.png)

# Flag
Once I gained root and just jumped over to Root's home directory and viewed the flag.txt.
![](/img/lampaio/flag.txt)
