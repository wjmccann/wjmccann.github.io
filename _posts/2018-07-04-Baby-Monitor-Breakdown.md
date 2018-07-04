---
layout: post
title: Baby Monitor Breakdown
tags: [Security, IOT]
---
Whilst preparing for a presentation on Wireless Security; where I analyse two wireless security systems and demonstrate their flaws I realised how much I enjoy taking a piece of tech and breaking it down and looking at it through a security lense. After tearing down a [Baby Monitor last year](https://wjmccann.github.io/blog/2017/11/11/The-Scary-World-Of-Baby-Monitor-Hacking) I decided I would purchase another cheap baby monitor on the market and break it down. The Model I chose was [this one](https://www.amazon.com.au/Detection-UnionCam-Pro-Wireless-Security/dp/B079NH3C3N/ref=sr_1_28?ie=UTF8&qid=1530611815&sr=8-28&keywords=baby+monitor) from Amazon. The reason I chose this baby monitor was A) the price and B) the 'crying detection'. It was clear that this device send some kind of alerts and I was hoping I could be able to exploit this.

![](/img/bm2/ad.png)

## The Camera
The Camera arrived in fairly plain packaging and it was quite noticable that there was no branding at all on the device. I am seeing that this is quite common with cheap chinese IOT devices that they are generically mass produced with no branding. This is probably to avoid attributing the actual product to who designed and created it! 
![](/img/bm2/packaging.jpg)

Opening the packaging and we can see that the camera itself is quite small. Inside I find three pages of instructions written poorly in english, but I am able to fumble my way through. Essentially, I need to download the [CamKeeper]() app from the playstore and connect the app and the camera to my Wifi. It is simple enough to set the Camera up in AP Mode, where it generates its own WiFi Access Point for me to connect to and then add the Camera to my own WiFi. 

![](/img/bm2/camera.jpg)

After downloading the App and registering an Account (God knows where or what I registered to) I am able to view and hear what the camera sees. There is no paning or zooming, but I can send two way voice. Inside the app I can change settings such as the WiFi network, Recording Settings (if an SD Card is added) and also set the Alerts. With the alerts, I have a choice of motion alerts or voice alerts. I also can have the Alerts push to my phone as a notication or sent as an email. 

![](/img/bm2/camkeeper.jpg)

![](/img/bm2/camkeeper2.jpg)

## NMAP
The first thing I did was obtain the IP address of the Camera and conduct an nmap scan against it. This produced the following result:
{% highlight bash %}
Starting Nmap 7.70 ( https://nmap.org ) at 2018-07-03 14:52 AEST
Nmap scan report for CK-0145.gateway (10.0.0.25)
Host is up (0.022s latency).
Not shown: 997 closed ports
PORT     STATE SERVICE       VERSION
80/tcp   open  http          luozewen-Webs
| fingerprint-strings: 
|   FourOhFourRequest: 
|     HTTP/1.1 404 Site or Page Not Found
|     Server: luozewen-Webs
|     Date: Tue Jul 3 15:52:42 2018
|     Pragma: no-cache
|     Cache-Control: no-cache
|     Content-Type: text/html
|     <html><head><title>Document Error: Site or Page Not Found</title></head>
|     <body><h2>Access Error: Site or Page Not Found</h2>
|     <p>Cannot open URL</p></body></html>
|   GetRequest: 
|     HTTP/1.0 302 Redirect
|     Server: luozewen-Webs
|     Date: Tue Jul 3 15:52:37 2018
|     Pragma: no-cache
|     Cache-Control: no-cache
|     Content-Type: text/html
|     Location: http://\xfc
|     /index.html
|     <html><head></head><body>
|     This document has moved to a new <a href="http://\xfc
|     /index.html">location</a>.
|     Please update your documents to reflect the new location.
|     </body></html>
|   HTTPOptions, RTSPRequest: 
|     HTTP/1.1 400 Page not found
|     Server: luozewen-Webs
|     Date: Tue Jul 3 15:52:37 2018
|     Pragma: no-cache
|     Cache-Control: no-cache
|     Content-Type: text/html
|     <html><head><title>Document Error: Page not found</title></head>
|     <body><h2>Access Error: Page not found</h2>
|     <p>Bad request type</p></body></html>
|   Help: 
|     HTTP/1.1 400 Page not found
|     Server: luozewen-Webs
|     Date: Tue Jul 3 15:53:02 2018
|     Pragma: no-cache
|     Cache-Control: no-cache
|     Content-Type: text/html
|     <html><head><title>Document Error: Page not found</title></head>
|     <body><h2>Access Error: Page not found</h2>
|_    <p>Bad request type</p></body></html>
|_http-server-header: luozewen-Webs
| http-title: CamKeeper Web Console
|_Requested resource was http://CK-0145.gateway/index.html
554/tcp  open  rtsp
| fingerprint-strings: 
|   HTTPOptions, RTSPRequest: 
|     RTSP/1.0 200 OK
|     CSeq: 0
|     Public: OPTIONS, DESCRIBE, SETUP, TEARDOWN, PLAY, PAUSE, SET_PARAMETER
|   SIPOptions: 
|     RTSP/1.0 200 OK
|     CSeq: 42
|_    Public: OPTIONS, DESCRIBE, SETUP, TEARDOWN, PLAY, PAUSE, SET_PARAMETER
|_rtsp-methods: OPTIONS, DESCRIBE, SETUP, TEARDOWN, PLAY, PAUSE, SET_PARAMETER
8090/tcp open  opsmessaging?
2 services unrecognized despite returning data. If you know the service/version, please submit the following fingerprints at https://nmap.org/cgi-bin/submit.cgi
{% endhighlight %}

From the scan we can see that it refers to a CK-0145.gateway host. I add this with the device's IP address to my /etc/hosts file and I am able to view the web page hosted on port 80. Along with Port 80, we have RTSP on port 554 and something on Port 8090.

## Web Server
The presence of a Web Server for the device was not advertised when I purchased the device or whilst looking through the intruction manual. I attempted some basic credentials, I finally get access using the username 'admin' and the password I set for the device whilst setting it up with the CamKeeper app. Note, this is a different password from the account that I registered. This appears to be more of a device password than anything else.

![](/img/bm2/web.png)

Inside the web application I have access to settings to change the device password, change the WiFi/Network settings and perform updates on the device. I played around with the update feature and it appears to allow you to upload any file you want to the device. Once uploaded the web application will tell you that the firmware was installed and that the device was rebooting. No matter what kind of file I uploaded the same would occur, I was actually suprised I didn't brick the device doing this. Nevertheless, I was unable to get a foothold on the device as the device would always reboot and there was no way to access an uploaded file once it was 'uploaded'. 

![](/img/bm2/web2.png)

## RTSP
The first thing I did when looking at the RTSP stream data is to try accessing it via VLC Media Player. This time it worked, meaning the device has an unauthenticated RTSP stream. What this means is that anyone on the network with the device can access the video feed without the need to enter any credentials. This could be handy or some people, but it also means that a malicious actor can view your feed if they have compromised your home network. 

Further investigation into the settings of the CamKeeper app reveals that you can set authentication on the RTSP stream using the same credentials that were used to access the web application. But the App and the Camera default to no authentication meaning users need to delve into the settings to activate RTSP (if they even know what that means). 

## Packet Analysis
I was hoping to be able to observe how the device generates alerts in the hope of being able to replay them. In order to do this, I fired up Wireshark and analysed how the device was communicating. I initially tried to have the device connect to a WiFi Access Point I had created using hostapd in Kali so that I could view the packets easily, however it refused to connect. Looking into the error messages on the CamKeeper app reveals that it suprisingly has some strict rules on the kind of WiFi network it can connect to. It must be WPA2-PSK, not a guest network and channel allocation must be auto. So, thats a positive I guess. 

After getting traffic passed through Wireshark I could see how the device was communicating. Whilst the CamKeeper application was open on my phone the Camera communicated directly to my phone's IP via UDP. It was when I closed the CamKeeper App on the phone or the Camera could not ping my phone that things started to get interesting. The Camera would start communicating out to a bunch of different (well... shady chinese) IP addresses. 

![](/img/bm2/packet.png)

I further tested this with alerts. Whilst the camera could ping my phone, when an alert was triggered it would commence sending UDP packets to the phone's IP address. Once it couldn't ping the phone it would send a sequence of TCP packets to port 8083 on a chinese owned server. It is assessed that this server is actually where I registered an account, as once the TCP packets are sent to that server, I receive the notication on my phone regardless if I am on the same network as the camera. Now, for a young family when they discover that their camera can alert them to things when they are outside the range of their camera they may be pleased with the feature. However, there is no where in the advertisement or the instructions that informs you that this device pings out to the internet. 

![](/img/bm2/alert.png)

Further, when you are outside your WiFi network and you recieve an alert, if you click on the 'view camera' option one of these shady servers will start communicating with the Camera in order to pull images and sound from the camera (only if they were recorded). I was hoping that the majority of the communication between the device and these servers would be via API calls however it is all done via TCP or UDP packet streams. This makes it more difficult to record and replay. 

## Issues
The first significant issue I indentified with this particular Baby Monitor is the default unauthenticated RTSP stream. In order to determine how significant of an issue this is, I turned to [Shodan](https://www.shodan.io/) to see what devices like this are out there. Using the unqiue web server 'luozewen-Webs' as a search string I was able to identify 33 IP Cameras. Of these Cameras that have port 554 exposed all of them had unauthenticated RTSP streams. 

![](/img/bm2/cam-store.png)

The automatic connection to the outside world servers by the Camera is not a significant security issue, however it is difficult to know what these servers are and what data they are storing. The issue is that the device does that anyway without informing the user. As the product is marketted as a Baby Monitor, security is a concern to the user. If the device is going to send data over the internet the user should be informed of this fact when making the decision to purchase the camera. 

## Conclusion
As I write this, I have already purchased another style of Baby Monitor to analyse. I find this an interesting topic with real world considerations and implications. However, the more I delve into the analysis of IOT devices, particular cheap IOT devices it is hard to not have a pessimistic outlook on the world of IOT. There does not appear to be a solution to the issues that I have highlighted. Consumers can choose to not purchase these devices (and they should) but this only drives the price lower, making them an attractive purchase for general consumers, which then leads to more of them being made. Additionally, there will be people that simply do not care about the risk and happy to go with a cheaper option. At the end of the day (generally) going with a reputable brand and paying a bit extra will find you a more secure product and the best we can do is to educate consumers to this fact.
