---
title: Hacking An Android Device
author: Dimitrios Tsarouchas
date: 2021-10-12 20:55:00 +0800
categories: [Cybersecurity, Information Security, Android Hacking, Mobile Hacking]
tags: [android, moblie hacking, information security]
pin: true
featured-image: assets/img/andriod-hacked.jpg
featured-image-alt: 
---

## Introduction
In today's tutorial, we are going to explore some ways on how to exploit an Android device. We will see what is a tunnelling service and how we connect to it. We will also see how to create an android payload using **msfvenom** to get a session on the device. Finally, once the device is hacked we will navigate on it, download data on our attacker machine and by using the device’s camera will get a picture of…myself :P

***Note**: An MLS iQTab 3G tablet with Android 4.4.2 / Kernel version: 3.4.67 was used for this tutorial.*

##
![Downloading-ngrok](/assets/img/Android_Hacking/1.registeranddownloadngrok.jpg)*Downloading ngrok*
![Downloading-ngrok](/assets/img/Android_Hacking/2.uzipandrunngrok.jpg)*Downloading ngrok*
![Downloading-ngrok](/assets/img/Android_Hacking/3.runngrokontcpport4242.jpg)*Downloading ngrok*
![Downloading-ngrok](/assets/img/Android_Hacking/4.ngrokisrunning.jpg)*Downloading ngrok*
![Downloading-ngrok](/assets/img/Android_Hacking/5.createamaliciousapkfileandexportitintorootfolder.jpg)*Downloading ngrok*
![Downloading-ngrok](/assets/img/Android_Hacking/6.maliciousappintorootfolder.jpg)*Downloading ngrok*
![Downloading-ngrok](/assets/img/Android_Hacking/7.startmsfconsole.jpg)*Downloading ngrok*
![Downloading-ngrok](/assets/img/Android_Hacking/8.setparameters.jpg)*Downloading ngrok*
![Downloading-ngrok](/assets/img/Android_Hacking/9.startexploiting.jpg)*Downloading ngrok*
![Downloading-ngrok](/assets/img/Android_Hacking/10.uploadfileonfilesfm.jpg)*Downloading ngrok*
![Downloading-ngrok](/assets/img/Android_Hacking/23.MaliciousappAPKSigning-1.png)*Downloading ngrok*
![Downloading-ngrok](/assets/img/Android_Hacking/24.MaliciousappAPKSigning-2.png)*Downloading ngrok*
![Downloading-ngrok](/assets/img/Android_Hacking/11.downloadingthemaliciousfile.jpg)*Downloading ngrok*
![Downloading-ngrok](/assets/img/Android_Hacking/12.installingtheapp.jpg)*Downloading ngrok*
![Downloading-ngrok](/assets/img/Android_Hacking/13.appinstalled.jpg)*Downloading ngrok*
![Downloading-ngrok](/assets/img/Android_Hacking/14.sessionstarted.jpg)*Downloading ngrok*
![Downloading-ngrok](/assets/img/Android_Hacking/15.sysinfo.jpg)*Downloading ngrok*
![Downloading-ngrok](/assets/img/Android_Hacking/16.listingallthefilesintherootdirectory.jpg)*Downloading ngrok*
![Downloading-ngrok](/assets/img/Android_Hacking/17.gettingmoreinfo.jpg)*Downloading ngrok*
![Downloading-ngrok](/assets/img/Android_Hacking/18.accessingthecamerafolder.jpg)*Downloading ngrok*
![Downloading-ngrok](/assets/img/Android_Hacking/19.downloadinganimage.jpg)*Downloading ngrok*
![Downloading-ngrok](/assets/img/Android_Hacking/20.imagedownloaded.jpg)*Downloading ngrok*
![Downloading-ngrok](/assets/img/Android_Hacking/21.findingthemaliciousappunderdownloads.jpg)*Downloading ngrok*
![Downloading-ngrok](/assets/img/Android_Hacking/22.startingwebcak.jpg)*Downloading ngrok*
![Downloading-ngrok](/assets/img/Android_Hacking/23.capturingmyself.jpg)*Downloading ngrok*
