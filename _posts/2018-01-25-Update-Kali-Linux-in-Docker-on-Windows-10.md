---
layout: post
title: Update: Kali Linux in Docker on Windows 10
tags: [Kali, Security]
---
Since my original post on running [Kali Linux in Docker on Windows 10](https://wjmccann.github.io/blog/2017/12/03/Kali-Linux-in-Docker-on-Windows-10) I have been able to robustly test and refine this set up. I have used this setup to undertake challenges on Hack the Box and Vulnhub, but perhaps what gave these setup a real test was using it on real penetration testing engagements. This post is designed to complement the original to demonstrate the improvements I have made.

## Improvement 1: I Was Stupid
In my orignal post, I talked about using [Cmder](https://wjmccann.github.io/blog/2017/12/03/cmder.net/) to allow me to use multiple windows. What I originally was doing was creating up to four containers at a time to run applications or access files. There may be some benefits to this such as seperation and allocation of resources, however it was a pain to share files etc.. So I began running a single container and using [tmux](https://github.com/tmux/tmux/wiki). I ended up moving away from user Cmder as it could not handle enabling the mouse in tmux. I moved to powershell.exe and had similar issues, not to mention clashes with the colours of my bash shell and the colours of powershell. In the end I have ended up using trusty old cmd.exe which has been great! 

## Improve 2: GUI Applications
I was typically getting by not needing too many of Kali's GUI applications. The ones I did need, were typically .jar files and I was able to run these on Windows. The Applications I was using heavily were - Burp Suite, Dirbuster and OWASP ZAP. However, I still wanted to get GUI applications working and I tried a wide number of guides I found on the internet. In the end, what ending up working was the following:

1. Install [MobaXTerm](https://mobaxterm.mobatek.net/)
2. Start MobaXTerm
3. Click the Settings button (not the drop down menu)
4. On the X11 tab change 'X11 remote access' to Full

Then start your container with the -e parameter to give the BASH variable the value of your IP Address. (This should be the IP address of your host machine on the Hyper V NAT network). To do this add the following

{% highlight bash %} docker run --it -rm -e DISPLAY=172.20.128.1:0 kali tmux {% endhighlight %}

Now, you will be able to run GUI applications by calling them from the terminal.

## Improvement 3: Shared Folders
Whilst doing penetration testing engagements I found I often needed to get my files from the container onto the host machine so I can store or submit them. I started doing this with the Python SimpleHTTPServer and navigating with my Host's browser to download, but if I forgot to open the ports to do this, I would need to spin up another container and start sharing across multiple containers. It was a pain. The easiest way I have found was to simply create a folder titled 'DockerFiles' and then everytime I spin up a container I map this folder on the host to a folder in my containers home directory and I have instaneous access to files from the container. To run this, I simply enter the following:

{% highlight bash %} docker run --it -rm -e DISPLAY=172.20.128.1:0 -v C:\DockerFiles:/root/DockerFiles kali tmux {% endhighlight %}

## Improvement 4: .bat files
So, as you can see from above, typing all that in can become a bit of a pain. So I found that writing a simple .bat file to execute the docker commands was the way to go. Since, docker needs to be run as an administrator I found after I created the .bat file, I made a shortcut of it and set it to run as Administrator. This way I was prompted for my Administrator password everytime I ran the file to create a container. My .bat file looks like this:

{% highlight bash %} start cmd.exe /K docker run -it --rm -v C:\DockerFiles:/root/DockerFiles -e DISPLAY=172.20.128.1:0 kali tmux {% endhighlight %}

## Conclusion
I really enjoy running my Kali instance like this. The biggest reason for me is that since I am on a Surface Pro 4, I always have issues with scaling of some applications. So running Kali in VMWare was always tiny and a strain on the eyes. This way my command prompt is native and I can run all my command line applications as I want and when I run GUI applications they are scalled perfectly for my display. tmux has been amazing and has boosted my productivity immensely, shared folders are fantastic so I can simply open up files as they pop into the folder. If you are going to run Kali in docker, the improvements I have detailed here are must haves for your set up.
