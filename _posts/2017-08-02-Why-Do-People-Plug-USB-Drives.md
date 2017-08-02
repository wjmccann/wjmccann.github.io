---
layout: post
title: Why do people keep plugging in USB Drives?
tags: [Social Engineering, Security]
---
This morning I read [this article](https://www.linkedin.com/pulse/so-how-do-you-build-out-100-usbs-drop-edward-farrell) by Edward Farrell on the USB drop exercise Mercury ISS conducted recently betweeen Canberra and Sydney. It got me thinking on the number of people that actually plug in devices they find. Now, Edward's USB Drop is not the first, nor will it be the last of this kind of research activity. 

Earlier this year I watched Hak5's AUSCERT2017 Keynote presentation titled - 'Of Hardware and Humans: Exploiting the Unviersal Attack Vector'. If you haven't seen the keynote it is worth checking out here [![Hak5 Keynote AUSCERT 2017](http://img.youtube.com/vi/m-hhNJLVLEE/0.jpg)](http://www.youtube.com/watch?v=m-hhNJLVLEE). 

Now, I could go on to detail a whole raft of other research in this area, but I will quickly summarise some of the results here:

### Hak5 USB Drop - RSA Conference 2016

 - 100 USB Drives used
 - 162 executions of code on the drive
 - 62 unique IP addresses
 - from 5 different nations
 - Data taken over 65 days from drop
 
### Google, University of Illinois Ubana-Champaign and University of Michigan, 2016
 - 297 USB Drives used
 - 48% success rate
 - [Link to article](https://www.theregister.co.uk/2016/04/11/half_plug_in_found_drives/)
 
### Department of Homeland Security - 2011
 - 60% success rate
 - 90% success rate if USB had an offical logo
 - [Link to article](https://www.forbes.com/sites/ygrauer/2015/10/30/usb-drive-malware-security-homeland-security-cybersecurity)
 
### Mercury ISS - 2017
 - 100 USB Drives used
 - 38 executions across 21 USB Drives
 
Now, one thing that I have seen here is that people are actually getting smarter when it comes to plugging in USB drives that they have just found laying on the ground. Although the Hak5 exercise is slightly concerning considering it was at a information security conference that this occured at. But the question still remains why do people still plug in drives?
 
# The Human Factor
 
 [This article](https://www.theregister.co.uk/2016/04/11/half_plug_in_found_drives/) provides some understanding as to why, and so does Hak5's keynote. That is trust and human nature. A USB drive found at a University is most likely going to belong to a student, it may contain their latest assignment or research and we would hate for them to lose that. So the only way to return a lost USB to someone is to plug it in and look at the files as it may afford us some clues. It's really no different than someone plugging in a SD CARD from a camera to view the photos in order to identify someone who has lost a camera (perhaps this is a new attack vector - the lost camera with a bad SD Card, albeit a more expensive one compared to the cheap USB). 
 
Then we come to the Hak5 exercise, in which people who attended a information security conference executed the code 162 times over 65 days. No again context is useful here. USB Drives may have been given out in swag bags and the like and therefore trusted. Also, you may even have security researchers executing code to see what it does (I guess playing with fire, you occasionally get burnt). But still there is a common theme to all this. 
 
# Curiosity killed the cat
 
I am sure no cats were harmed in any of the USB exercises but it is curiosity that resulted in all these code executions on bad USBs. Curiosity to help a fellow university student, curiosity to find out what Bad USB code did or simply curiosity as to what was on the drive. This trait of human nature is what is targetted with bad USB drops so there will always be a level of success. But, just perhaps people are now starting to get a little smarter before they plug in that drive they found.
