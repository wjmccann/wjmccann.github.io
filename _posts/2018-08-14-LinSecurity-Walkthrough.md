---
layout: post
title: Vulnhub - Lin.Security Walkthrough
tags: [Hacking, Vulnhub, CTF]
---
Lin.Security is a Boot to Root CTF available [here](https://www.vulnhub.com/entry/linsecurity-1,244/) on Vulnhub. It's difficulty is rated as Easy/Intermediate. I could not get this VM to work in VMWare, however had success with VirtualBox. Additionally, this VM provides you with a set of credentials to begin with. I only used these credentials to check that my networking was configured correctly, otherwise I forgot they existed and attacked this box externally.

After using the credentials provided to check the networking, I knew that the box's IP was 10.0.0.97. So as usual, I kicked off with an nmap scan.
![](/img/lin/nmap.png)

We see that port 2049 is exposed, which is typically a NFS share. I use showmount to view what shares are available (note: if your on Kali can don't have showmount it can easily be installed by using apt-get install nfs-common)
![](/img/lin/showmount.png)

We see that the home directory for the user peter is shared. I go ahead and mount this to a folder titled 'lin' and view its conntents. There is nothing in here, except for a few hidden fiels (sadly no .ssh keys as I was hoping). However I do not have access to read some files and I cannot write to the drive.
![](/img/lin/mount.png)

After reading up on NFS share misconfigurations, I learn that you can get full access to a folder if you share the same UID. I check the UID of the NFS share and see that it is 1001.
![](/img/lin/permissions.png)

I create a new user called 'test' and give them the UID of 1001. This now gives me the ability to read all files and write to the share. Originally, I was disappointed in that there were no .ssh keys in the share, but now that I can write to the share I can put my own .ssh keys in and log in as the user peter. so I create a .ssh folder and generate a new set of .ssh keys for the test user and copy these accross to the authorized_keys file in .ssh. I then ssh into the box as peter and I successfully log in!
![](/img/lin/sshkeys.png)

I start looking to escalate my privileges to root and I see in the /etc/passwd file that the user insecurity has a password hash defined. I copy this across and then use john to crack the password using the rockyou.txt wordlist.
![](/img/lin/passwd.png)

john takes no time at all to crack the hash and reveal the password to me.
![](/img/lin/password.png)

I then use this pasword to su into the user insecurity. Amazingly this user is actually root! Job Complete!

![](/img/lin/root.png)

This a was fun VM that took hardly any time at all to complete. I learnt a bit about NFS share misconfiguration which I will store away for later use. Recommend this VM for anyone just starting out or if your looking for something quick and easy to do on a weeknight

