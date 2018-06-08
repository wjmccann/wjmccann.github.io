---
layout: post
title: Vulnhub - Mr Robot 1 Walkthrough
tags: [Hacking, Vulnhub, CTF]
---

Mr Robot 1 is a Boot to Root CTF available [here](https://www.vulnhub.com/entry/mr-robot-1,151/) on Vulnhub. It's difficulty is rated as Intermediate. This VM is the sixth in my OSCP preparation series based off [abatchy's blog post](http://www.abatchy.com/2017/02/oscp-like-vulnhub-vms.html). 

The VM is set up to use bridged networking. The Mr Robot VM is at 10.0.0.48 and my Kali machine is at 10.0.0.228.

I an an initial nmap scan and discover that ports 22, 80 and 443 are open. 
![](/img/robot/.png)

Browsing to the HTTP sites, I find a Mr Robot terminal themed website. 

Checking for robots.txt it reveals two hidden files - key-1-of-3.txt and fsociety.dic


Key-1-of-3.txt is our first flag for the VM, while fsociety.dic appears to be a wordlist dictionary that may come in handy later on.


I interacted with the terminal on the web page and visited the other pages. Viewing the source of these pages reveals that there is a wordpress site assoicated. This was evident through links to wp-content and xmlrpc.php.


I an wpscan against the site, however I was unable to enumerate any users. Luckily, WordPress gives you clues to what users are available based on the error message you recieve when logging in. I decided to try a few different users from the TV Show Mr Robot (since that is the theme) and was successful in determining that the user 'elliot' was available.

I decide to put the wordlist dictionary that we found to good use and use wpscan to brute force the elliot login. 


Well done to the creators of this VM, i spent 4 hours waiting to get to the end of the list to finally find valid login credentials. 

I log in to the WordPress site and see that I have access to the Theme editor. This is a great way to get code execution on a WordPress site, as it allows you to enter php code into the sites templates. I copy over the php reverse shell from /usr/share/webshells/php in my kali box and copy it into the top of footer.php. I set up a netcat listener and execute the code by generating a 404 error which loads the footer.php page.


Once I have my shell I navigate to the home directory and find the user robot and the file key-2-of-3.txt, however I do not have permissions to view the file. I then decide to look at escalating my priveleges. To start I copy over the LinEnum.sh script into the tmp directory and run it. Looking through the results I see that nmap has the SUID bit set. I check the version of nmap and see that it is an old version that supports interactive mode. I enter interactive mode, drop into a shell and confirm that I am root. I browse over to /home/robot and grab key-2-of-3.txt and then over to /root to get key-3-of-3.txt.


