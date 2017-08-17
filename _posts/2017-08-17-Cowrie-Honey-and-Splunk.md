---
layout: post
title: Cowrie Honeypot and Splunk Anaylsis
tags: [Security, Honeypot]
---

As part of my Masters of Cyber Security, I have an assignment where I am required to review the logs from a Honeypot and analyse what has occured. This seems simple enough, but the logs are old and are probably the same logs that have been used for the last few courses, so I have decided that I will deploy my own Honeypot and conduct some fresh anaylsis.

I have decided that I will use [Cowrie Honeypot](https://github.com/micheloosterhof/cowrie) for no other reason than that is the Honeypot that is being used in my assignment, but also because Cowrie offers some nice features over say Kippo (which Cowrie is based on). I chose to host my Honeypot on Digital Ocean. This is the first time I have used their service and it was very nice, smooth, easy and quick to set up my own VPS. Also, since I do not need a lot of computing power to run Cowrie I was able to purchase two months worth of their smallest droplet for $13AUD running Ubuntu 16.04 

### Cowrie Installation
Once set up with my Digital Ocean account, I SSH into my box and go about installing Cowrie. The first step is to install all of the dependencies.
~~~~
sudo apt-get install git python-virtualenv libssl-dev libffi-dev build-essential libpython-dev python2.7-minimal authbind
~~~~

I then go about creating a seperate user account, as Cowrie will not run as root. 

~~~~
sudo adduser --disabled-password cowrie
~~~~

Once created, I then su into the new account

~~~~
su cowrie
~~~~

To install Cowrie I need to clone the respository from GitHub

~~~~
git clone http://github.com/micheloosterhof/cowrie
~~~~

Then move into the newly created directory

~~~~
cd cowrie
~~~~

The next step is to create a virtual environment for Cowrie to run. Cowrie is essentially a python program that provides a linux shell like interaction for attackers to interact with. The virtual environment keeps everything nice and need on our VPS. We need to make sure we are in the right directory. enter 'pwd' to confirm that you are in /home/cowrie/cowrie before creating the virtual environment

~~~~
virtualenv cowrie-env
~~~~

Once our virtual environment is created we need to activate it.

~~~~
source cowrie-env/bin/activate
~~~~

And then install the required packages

~~~~
pip install -r requirements.txt
~~~~

Cowrie is now installed. Next we will go onto to configuration.

### Cowrie Configuration
The configuration of Cowrie is stored within cowrie.cfg.dist. Its best to leave cowrie.cfg.dist as is and create a copy titled cowrie.cfg. Both files live happily within cowrie with cowrie.cfg taking precendence over cowrie.cfg.dist.

~~~~
cp cowrie.cfg.dist cowrie.cfg
~~~~

Once done, we will open up cowrie.cfg and change the hostname. This makes our honeypot just a little harder to spot. 

~~~~ 
nano cowrie.cfg
~~~~

In here we will change the parameter hostname from srv04 to anything you like - I changed mine to prod-08-sf0-01. But it can be anything you like to make it look like a real system.

~~~~
hostname = prod-08-sf0-01
~~~~

In the instructions for installing cowrie it mentions the creation of a DSA Key. The instructions states that this shouldn't be necessarily but doing it anyway can prevent issues later on. This is due to one of Cowries dependant packages Twisted. Some versions of the package do not create this automatically so to prevent issues we create it manually.

~~~~
cd data
ssh-keygen -t dsa -b 1024 -f ssh_host_dsa_key
cd ..
~~~~

#### Setting up SSH
Cowrie is a SSH honeypot, however it runs on port 2222 leaving port 22 to allow us to ssh normally into our VPS. ideally, we want would be attackers accessing port 22 thinking this is a real system so we need to move SSH over to a different port and port forward all traffic going to port 22 to port 2222.

First we will configure SSH for ourselves.

~~~~
sudo nano /etc/ssh/sshd_config
~~~~

You will see that Port 22 is set as our port for SSH, change this to anything you like

~~~~
Port 8724
~~~~

Save and exit and restart the SSH service for this to take effect.

~~~~
service ssh restart
~~~~

Now that we have SSH running on port 8724 we want to redirect all traffic on port 22 to port 2222

~~~~
sudo iptables -t nat -A PREROUTING -p tcp --dport 22 -j REDIRECT --to-port 2222
~~~~

### Starting Cowrie

Before starting cowrie we will need to make sure that cowrie is explicitly in pythons os.path. This step shouldn't be necessary but it helps prevents any issues that may arise later when starting cowrie. 

~~~~
export PYTHONPATH=/home/cowrie/cowrie
~~~~

Now we are ready to start cowrie. From within /home/cowrie/cowrie/ simply run the following command

~~~~
bin/cowrie start
~~~~

#### Getting Logs with Splunk

I decided I would create a second Digital Ocean Droplet and run Splunk to process the logs captured by my Cowrie Honeypot. To download the free version of Splunk, you need to sign up for a Splunk account and go to their downloads page. For my Ubuntu Digital Ocean VPS I simply ran the following command to download the latest version of Splunk.

~~~~
wget -O splunk-6.6.2-4b804538c686-linux-2.6-amd64.deb 'https://www.splunk.com/bin/splunk/DownloadActivityServlet?architecture=x86_64&platform=linux&version=6.6.2&product=splunk&filename=splunk-6.6.2-4b804538c686-linux-2.6-amd64.deb&wget=true'
~~~~

Once downloaded I simply installed Splunk using

~~~~
sudo dpkg -i splunk-6.6.2-4b804538c686-linux-2.6-amd64.deb
~~~~

I then started Splunk using

~~~~
sudo /opt/splunk/bin/splunk start
~~~~

After going through the EULA you will be informed that Splunk is running on http://localhost:8000. 

To easily process Cowrie logs in Splunk I used a Splunk App called [Tango Honeypot Intelligence](https://splunkbase.splunk.com/app/2666/). Simply download the app and install in Splunk using the 'install app from file' feature.

Once installed in Splunk we need to set up a Splunk Universal Forwarder in our Cowrie Honeypot. To do this, SSH over into the Cowrie VPS and run 

~~~~
git clone https://github.com/aplura/Tango.git /tmp/tango; chmod +x /tmp/tango/uf_only.sh
cd /tmp/tango/
./uf_only.sh
~~~~

This will set up and download the Splunk Universal Forwarder on our Cowrie VPS. Following the prompts to point the Forwarder to our Splunk installation. When asked for a port for the Splunk reciever use port 9997. All the scripts for Tango Honeypot Intelligence are configured for 9997 so it is best to keep it as is. 

Now that we have our Honeypot sending out logs, we need to create a listener in Splunk. To do this you need to navigate to $SPLUNK_HOME/etc/system/default/ on your Splunk VPS and open the inputs.conf file

~~~~
cd /opt/splunk/etc/system/default/
sudo nano inputs.conf
~~~~

Inside your inputs.conf add the following lines

~~~~
[splunktcp://9997]
disabled = 0
~~~~

Restart Splunk for the changes to take effect and then open up the Tango Honeypot Intelligence app in Splunk. You should now start to see the data coming in from your Honeypot. 

You can take this concept further and create a Honeynet with multiple Honeypots. Simply following these instructions to create other Honeypots all pointing back to the one Splunk installation. 

I intend to run the Honeypot for approximately two weeks to see what data I gather. I will post back here with the results.

Cheers,
