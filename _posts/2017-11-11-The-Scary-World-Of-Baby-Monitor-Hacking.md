---
layout: post
title: The Scary World of Baby Monitor Hacking
tags: [Security, Hacking]
---
About two months ago, there were a number of news stories floating around about people hacking into Baby Monitors. The news stores detailed two occurances where Mothers had heard a strange voice talking to them through the monitor or the monitor moving on its own following people around the room. Scary stuff for people with children.

When my wife and I had our first child she asked me to research baby monitors and find something that couldn't be hacked. She was quite concerned to make sure it was safe. I looked at and considered options that included WiFi connected cameras (Cameras secured safely by your own WiFi) and I looked at Bluetooth cameras. I was concerned about WiFi connected cameras as they would be placed on my home network and if it was to be compromised someone could potentially access my camera. I finally decided on a Bluetooth camera, as it had to be connected directly to a hand held monitor which needed to be paired with a security code to the monitor. I have looked into hacking our own monitor and it turns out to be quite seecure. The signal is encrypted and it is hard to man in the middle due to the Bluetooth pairing code. I don't want to say that it is hack-proof, but I am comfortable that with where I live, the range of the device and the security features I mentioned, I wont having any issues.

## The Camera
So, the other day whilst browsing my junk email prior to deleting it I saw an Ad for a cheap $49 Wifi Baby Monitor from [Groupon](https://www.groupon.com.au/). Now thinking about the recent news stories, I decided I would purchase the monitor an actually test it out. The hypothesis was - "How easy is to to access a baby monitor". I thought the Groupon camera being offered was a great test case. It is cheap and has a great list of features that you can see below. You can easily see an expectant mother picking up this camera.

![Groupon Ad](/img/Baby%20Monitor/grouponad.png)

So the Camera arrived and the first thing that I noticed that concerned me was that there was no branding on the camera. I could not determine who made it, other than it was a 'WiFi Smart Camera - Easy to achieve real-time remote viewing'. 

![Packaging](/img/Baby%20Monitor/packaging.png) ![The Camera](/img/Baby%20Monitor/camera.png)

So I opened it up, connected it to my network and downloaded the App from the Google Play Store. Again, alarm bells were raised when I saw the 'Whats New' section of the App, it looks as if somewhat had hacked the page with Jibberish. Upon immediately connecting the camera up and getting it activated with the App, the device was attempting to connect to the Internet. Luckily, the firewall rules on my network prevented any remote access. 

![App Screenshot](/img/Baby%20Monitor/Appscreenshot.png)

## Network Scans
I first ran a nmap scan against the device and discovered that port 80, 554, 1935 and 8080. Upon further research I discovered the following: 

* Port 80: IP Camera Web Portal which is password protected. Should be noted that the Web Portal is not mentioned anywhere in any documentation. All documentation clearly dicates that access and management of the camera is via the Mobile phone app.
* Port 554: RTSP Hip Camera RealServer v1.0 service. This the actually video feed playing over the network
* Port 1935: Adobe Connect. I believe this is used in part with the Web Portal using flash to view the video feed.
* Port 8080: GSoap 2.8. A web service for generic XML data bindings. I believe this is used for passing information regarding the cameras data across the network.

![Nmap Scan 1](/img/Baby%20Monitor/nmap.png) ![Nmap Scan 2](/img/Baby%20Monitor/nmap2.png)

I connected to ports 554 and 1935 via Netcat. Only 554 was open, however the syntax to interact with the console, but it didn't require any authentication which is useful for later. Researching GSoap 2.8 I found that there was recently a [buffer overflow vulnerability](https://vuldb.com/?id.104323) in versions 2.7 to 2.8.47. There was no further information on the vulnerability and POC has been released.

## The Web Portal
I decided to run a dirb and nikto scan against the web server on Port 80. Nikto obtained some extremely useful information such as default credentials for users Admin and Guest. dirb discovered a number of folders and most interesting was the /sd folder.

![Nikto Scan](/img/Baby%20Monitor/nikto.png) ![Dirb scan](/img/Baby%20Monitor/dirb.png)

Now, that I had some credentials I visited the web page and was greated with a page that defaulted to Chinese. There was a javascript link that allowed me to change it to English and I logged in with admin/admin. From here I was able  to view the camera and control its movement. As admin I was able to set alarms and alerts change how images and video were saved (i.e direct that time images be sent to my own FTP server). Also as Admin I was able to access the /sd folder and view the contents of all images and video that were saved to the SD Card. I was also able to view all users that had access to the web port. The users consisted of Admin/Admin, User/User and Guest/Guest (already obtained by nikto). I logged out and tested each of these accounts to see what I could access. As both User and Guest, I could view the camera and move it as I wished. I did not have any access to the system settings or SD card. 

![Web Login Page](/img/Baby%20Monitor/weblogin.png) ![Camera controls](/img/Baby%20Monitor/webcontrols.png)

Now this is using default credentials, and the instructions provided with the device do say that for seecurity purposes I should change the password. So I went to the App and discovered that I can only change the Admin password. If I was a general consumer, I would never know that these additional accounts existed. 

![Password Changing](/img/Baby%20Monitor/Passwordscreenshot.png)

## Port 554
I decided to explore the port 554 a little further and discovered that you can access network streams of RTSP feeds via VLC player. So I connected my VLC player to rtsp://[IP Address]/1 (the 1 indicating the first stream) and straight away I had access to the camera feed with no need to enter any credentials.

I tried to exploit the device further (hoping to get console access to the camera's Linux sub system) but I was unable to progress any further than the standard web interface. However, I had achieved my original aim and that was to view and control a Baby Monitor. 

## The Real Risk
Now, you may be thinking to yourself - "But this can only occur if you have access to the network". And yes, you are correct. However, these devices connect to the internet and are intended to be used as remote viewing cameras. So I browsed [shodan.io](https://www.shodan.io/) and searched for "hipcam realserver". Shodan produced 169, 810 results of addresses that had port 554 exposed to the internet. As I browsed these I also discovered a number of addresses that also had Port 80 exposed, hosting the same 'IP Camera' front page with login. With what I have discovered it is possible for each of these devices to be accessed via default credentials, or if the admin credentials have been changed using the hidden User and Guest accounts. Using VLC player, I could easily connect to each of these camera streams without the need to authenticate.

## Outcomes and Legal
After making this discovery I was filled with a mix of fear and anger. How, can a device be sold as a 'Baby Monitor' that is so poorly secured and exposes so many to the risk of their privacy being breached. Prior to publishing this blog post I had intended to responsibly disclose what I had discovered, however, I have no one to disclose to. The closest I could get is the details I obtained from the MAC address of the device (Shenzhen Bilian Electronic Co Ltd), however they appear to be a mysterious internet entity and there are no details of IP Cameras being sold by them. Sadly, there appears to be no way to inform people who are using these IP Cameras that they are vulnerable.

## What Can you Do?
So, what can you do to protect yourself? The first thing is to conduct your research into the products you buy. Understand the benefits and risks of using a WiFi connected Baby Monitor over a Bluetooth enabled device. Consider the risks of having it connect to your PC or phone versus its own paired monitor. Know what you are exposing to the internet, consider these risks when enabling remote access. 

Baby monitors vary is cost from dirt cheap ($30-$50) to expensive ($200). As this experiment has shown, the cheap no name monitors have significant risks to them. Sometimes spending a bit more is an investment in your security. 

## Final thoughts.
After conducting this experiment it has shown me first hand how horribly broken the Internet of Things can really be. Cheap consumer products are being pushed out into the marketplace that are exposing people to risks they may not even know about. The decice attempts to direct people to being secure, such as recommending that you change the default password. But the fact that it exposes other user accounts with default credentials, along with the video feed over an unauthenticated port is dangerous. In fact, my thoughts were that it was 'criminal'. Ideas and Concepts such as [Warning Labels for IOT decices](https://www.troyhunt.com/what-would-it-look-like-if-we-put-warnings-on-iot-devices-like-we-do-cigarette-packets/) or the [Cyber Kangaroo](http://www.zdnet.com/article/how-the-cyber-kangaroo-can-help-defend-the-internet-of-things/) are not fool proof. You can only assure an device so far, but if these warning labels were in place, It would go a long way to preventing people from purchasing devices like this camera.
