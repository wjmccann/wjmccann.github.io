---
layout: post
title: Baby Monitor Beat Up
tags: [Security, IOT]
---
Following on from [my previous](https://wjmccann.github.io/blog/2018/07/04/Baby-Monitor-Breakdown) breakdown of a Baby Monitor, I decided to give it another shot. Sadly, this time I did not have the success that I wanted, but it has brought to light some new attack pathways which I initially did not consider. Anyway, here are the details on the monitor:  

![](/img/bm3/ad.PNG)

This monitor was purchased through Groupon, its relatively cheap at $49 and claims it has a whole bunch of features.

## The Camera
The camera arrived in a plain brown box. Not branding at all. Luckily, once I opened up the box I could see that it was a Yoosee branded IP camera. The majority of the instructions were in Chinese and those that were written in English were poorly translated. I downloaded the Yoosee app and the app guided me through the installation process of getting it onto my network. 
![](/img/bm3/camera.jpg)

The first thing I noticed with the camera and the installation process is the basic device password. The device password is '123'. This is also written in the instructions so it doesn't appear that i just lucked out and got a device with the password '123'. It seems that all devices have the default password '123'. 
![](/img/bm3/password.jpg)

Like last time, I had to register an account to use the app and access the camera. 

## Scanning
I intially tried to scan this device with nmap but it simply returned that no host was available. I tried with the -Pn flag and again nothing registered. I confirmed I could ping the device but I could not scan. In some previous penetration tests where I have had issues with scanning hosts I knew were up I had some success using unicornscan. I hit the device with unicornscan and I returned ports 554, 5000, 10086. 

Port 554 is the RTSP stream, however I could not connect to this at all. I am unable to confirm if it is authenticated or not as it simply wouldn't connect. I had the same issues with ports 5000 and 10086. Looking at Wireshark captures reveals that the device will not communicate over the LAN and to access to video feed you need to go over the internet and through the App. 

## Wireshark
Looking over the Wireshark captures I had hoped to find some interesting API calls, however surprisingly the majority of the devices connections were over TLS. The only transmission I found that wasn't secured by TLS is the following: 
![](/img/bm3/wireshark.PNG)

From this capture you can see the device make a POST request with my User ID and session ID. The session ID is key as that indicates that I must be authenticated to make this post request. The capture returns details such as the Device that is registered to my User ID. 

So far, so good for security of this device until...

## Adding a new device
There is a feature in the App that allows you to add new devices. The options are: 'Add new device' and 'Add online device'. 'Add new device' allows you to connect a new device via AP mode, where the device broadcasts a Wifi Network and you connect to that to set up the device. 'Add online device' allows you to add a device by simply entering the device ID and the device password. Remember earlier when I stated that the default device password was '123'? Well, now it is simply a matter of guessing a correct device ID and having that device be online and you can gain access to someone elses camera. This issue is very similar to the recent flaw found in [Swan IP Cameras](https://www.helpnetsecurity.com/2018/07/26/swann-security-cameras-spying/).

## Conclusion
For the most part, this device is relatively secure - Although it connects over the internet and does not allow for a local connection. However, in saying this the two devices I have analysed that allow local connections have significant flaws. The key issue is the ability to add a device by its device serial number with the use of default passwords. This flaw has made me realise there is another more likely attack pathway that I had not considered. In my last two Baby Monitor breakdowns, I looked the the security vulnerabilties from the perspective that an attacker has already gained access to your home network. However, with this device, an attacker does not need to breach the home network to be able to access a monitor and interact with it. Perhaps these stories you occasionally hear on the news about hackers breaking into baby monitors are infact taking advantage of a flaw similar to this? 
