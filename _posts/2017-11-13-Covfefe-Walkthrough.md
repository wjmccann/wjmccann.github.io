---
layout: post
title: Vulnhub - Covfefe Walkthrough
tags: [Hacking, Vulnhub, CTF]
---
Covfefe is a Boot to Root CTF available [here](https://www.vulnhub.com/entry/covfefe-1,199/) on Vulnhub. It's difficulty is rated as Beginner and there are three flags to capture.

I start by finding the IP address the VM is on by running netdiscover.

![Netdiscover](/img/covfefe/netdiscover.png)

I then run a nmap scan and find that ports 22, 80 and 31337 are open.

![Nmap](/img/covfefe/nmap.png)

I run dirb against port 80 and find nothing, but on port 31337 I find some interesting files. 

![](/img/covfefe/dirb.png)

Browsing to http://[IP Address]:31337/taxes, I find the first flag. I then download each of the files in the directory and extract the private key from the .ssh file.


![](/img/covfefe/flag1.png)
![](/img/covfefe/downloadkey.png)
![](/img/covfefe/key.png)

I use ssh2john to convert the private key to a hash and then run the hash file through john using the /usr/share/wordlists/rockyou.txt wordlist.

![](/img/covfefe/crackpassword.png)

John produces the password 'starwars'

I then view the public key which reveals the username for the ssh key - Simon.

![](/img/covfefe/username.png)

To use the SSH public key that I downloaded, I need to perform a chmod 600 on it, then SSH into the box using the username 'simon', the public key and the password 'starwars' and I am in.

![](/img/covfefe/sshlogin.png)

Having browsed the .bash_history file I downloaded I noted that the only command that had been run was 'read_message'. I run the command and it is a program that asks for a name. After a little trial and error I figure out it wants 'Simon' with a capital S. The program then spits out a message informing me that the source code is located in Charlie Root's home directory. 

![](/img/covfefe/readmessage1.png)

/etc/passwd doesn't have a user charlie, so I look in the /root directory and there is flag.txt and read_message.c. I do not have the permissions to view the flag.txt, but I can view the source code. This reveals the second flag.

![](/img/covfefe/source.png)

Reviewing the source code for the program I note that the program takes in a buffer of 20, but only reads the first 5 characters. I am now thinking buffer overflow as a way to exploit this. If the first 5 characters are 'Simon' the program will call execve and run another program which is the message.

I confirm the SUID of the program read_message and identify that it is run by root. So I need to overwrite the program at /usr/local/sbin/message which my own shellcode or identify if there is a buffer overflow. I first attempt to replace the message program at /usr/local/sbin/message but I do not have the permissions.

![](/img/covfefe/suid.png)

I download the source code to my own machine, compile it as a 32-bit ELF binary and run it through GDB. I set a breakpoint at 'execve' and run the program as normal. At the breakpoint I see that EAX is the value of the program that is going to be run by execve. 

![](/img/covfefe/gdb1.png)

Next, I attempt to overflow the buffer. I know that the buffer is 20, so I enter 'SimonAAAAAAAAAAAAAAABBBB'. At my breakpoint I note that EAX is now my 4 'B's. Perfect, anything I put after 20 characters (provided the first 5 are 'Simon') should run. 

![GDB 2](/img/covfefe/gdb2.png)

I create a bash reverse shell script and place it in the /tmp folder of the target machine and then write a C program that will call that shellscript. However, i run into the problem that GCC doesn't appear to be available on the target. To get around this, I simply compile the program on my machine, host it using 'python -M SimpleHTTPServer' and download the binary to the target into the /tmp directory.

![](/img/covfefe/shell_c.png)

I set up a netcat listener to catch the shell

![](/img/covfefe/netcat.png)

I make the program executable with 'chmod +x shell' and then run the program entering the values 'SimonAAAAAAAAAAAAAAA/tmp/shell', however I only get a shell as the user Simon. After a bit more research I discover that although read_message is being run as root, when the program executes the shellcode it is run as user simon. To get around this, I need to add [code] setuid(0) [/code] to my C program.

I recompile and download the binary to the target machine and run the commands again.

![](/img/covfefe/shell_c_good.png)

This time I get a shell as root.

![](/img/covfefe/root.png)

I navigate to the root directory and view the third and final flag.

![](/img/covfefe/flag3.png)

Overall, a great VM with a simple buffer overflow that was enjoyable to work through.
