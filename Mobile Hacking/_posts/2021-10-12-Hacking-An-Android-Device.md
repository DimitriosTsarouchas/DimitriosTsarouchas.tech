---
title: Hacking An Android Device
author: Dimitrios Tsarouchas
date: 2021-10-12 20:55:00 +0800
categories: [Android Hacking, Mobile Hacking]
tags: [android, moblie hacking, information security]
pin: true
featured-image: assets/img/andriod-hacked.jpg
featured-image-alt: 
---

# Introduction
In today's tutorial, we are going to explore some ways on how to exploit an Android device. We will see what is a tunnelling service and how we connect to it. We will also see how to create an android payload using **msfvenom** to get a session on the device. Finally, once the device is hacked we will navigate on its system directories, download data on our attacker machine and by using its embedded camera will get a picture of…myself :P

***Note**: An MLS iQTab 3G tablet with Android 4.4.2 / Kernel version: 3.4.67 was used for this tutorial.*

## A little story about iOS vs Android

In order to install an application on an iOS device like an iPad or iPhone, we use App Store, which is the official app store where the apps can be downloaded. So, suppose we managed to create a backdoor. For an iOS device, we need to find a way to deliver this. Every App Store application is being reviewed by Apple before it gets released. Therefore, even if you managed to hide it from the developers of Apple, even if it gets released on the App Store, they will take it down after a couple of days. The restrictions of the App Store are getting stricter every day. Though, by using an Apple Business Developer account, and paying the price of $300 per year, you're allowed to publish your apps without uploading them to the App Store. It's actually for business to business applications. So, using that you can distribute this app to people you want to, and they can install this app on their iPhones or iPads. But, here comes the dissatisfaction; it will be noticed in a couple of days and will be taken down.

On the other hand, Android backdoors are applicable and most of the time effective in real-life examples. This happens because we can download and install Android apps without using Google Play at all! Of course, Google Play is a way of distributing an Android application, but it will get noticed and taken down eventually.
Therefore, what an attacker can do, is to create malicious apps and distribute them via other means like email, WhatsApp etc. A victim actually, after downloading the malicious app, has only to allow the unknown sources to be installed on their phones.

Let's find out more about it by reading through this article.


## ngrok - Tunnel Service

Tunnelling services lets us connect our local IP address to the Internet by using their own services. Therefore, using ngrok as our tunnelling service, we can actually gather the information with its IP address from the backdoor we will create in the Android device and forward that information to our Kali Linux machine.  

Once the user taps on the malicious file created by using msfvenom a session will be established between the Android device and the Kali Linux and this is how we will get access to files and folders inside of this device. All of this information will make more sense as we progress in this article. 

In order to use the ngrok we need to create an account on the [ngrok website](https://ngrok.com/){:target="_blank"} ]. ngrok is cross-platform, but because we are using Kali Linux we will follow the installation steps referred to that OS.


![Downloading-ngrok](/assets/img/Android_Hacking/1.registeranddownloadngrok.jpg)*Downloading ngrok*

Once the file is downloaded we will have to unzip it, run it and connect it with our account using the provided authentication token. All the commands and guidance on how you connect your account are provided on the ngrok website. Remember that all of the commands below need to run in the folder the ngrok.zip file has been downloaded (i.e Downloads folder). That's all on how we set up and install our environment for tunnelling services.

![Downloading-ngrok](/assets/img/Android_Hacking/2.uzipandrunngrok.jpg)*Unzipping ngrok and connect using the  provided API*

Now we are ready to create our backdoor using the ngrok tunnelling service. In order to create some kind of connection, all we have to do is to specify the connection type and the connection port that we want to use. We are going to use this for sending the connection from the Android device to the ngrok service. We will use the random port 4242 and since we do not have any kind of firewall in our attack machine, it should work fine and won't be blocked. 

![Downloading-ngrok](/assets/img/Android_Hacking/3.runngrokontcpport4242.jpg)*Start ngrok on TCP port 4242*

We can see in the image below that the session has now started and it’s doing a forwarding operation, which is what we are looking for! We are forwarding the tcp.ngrok.io with the port 11921 to our localhost. 

We are going to use this information as the localhost and the local port in **msfvenom**, because we want to direct the connection from the Android device to the ngrok, and then, from the ngrok to our own localhost (our own Kali Linux machine).  We will give the `8.tcp.ngrok.io` as an input to LHOST, and the port **11921** as the LPORT in the msfvenom.

***Note**: This address and port will be different for you*.

![Downloading-ngrok](/assets/img/Android_Hacking/4.ngrokisrunning.jpg)*ngrok session has been created*


## Creating a payload with MSFVenom
Without closing the window that's running the ngrok service (if it's closed it will stop the session), open a new tab and type the following msfvenom command to create the payload as depicted below. 

We also need to specify the folder where this .apk file (the payload) that we have been trying to create, so we can access it and forward it easily. This can be done by appending `R > /root/maliciousapp.apk` at the end of the msfvenom command. This will generate the malicious app into the root folder. 
![Downloading-ngrok](/assets/img/Android_Hacking/5.createamaliciousapkfileandexportitintorootfolder.jpg)*Creating a malicious app and exporting it into the root folder*

Let's analyse the parameters one by one so we understand what the payload is doing:

* **android**: The operating system we're going to be attacking.
* **meterpreter**: The session that manages the connection between the target device and the Kali Linux so we can send some commands to the android to be executed.  
* **reverse_tcp**: The way that we're trying to hack into. **reverse** means that connection will come from the Android device to our attack machine. That way it's much less detectable. **tcp** is the gate that we're trying to go in.
* **LHOST**: The IP address we're expecting the session to come.
* **LPORT**: The port number we're expecting this session to come.


![Downloading-ngrok](/assets/img/Android_Hacking/6.maliciousappintorootfolder.jpg)*Downloaded malicious app int the root folder*

The **maliciousapp.apk** is the exact file we're going to send to the victim, and once it is installed on his/her device it will give us access to it. 

## Listening for connections

The first step is to start a database service called PostgreSQL and the reason for doing this is because we're going to use Metasploit. With Metasploit we can actually listen for incoming connections, find a bunch of exploits that can work on some of the vulnerabilities that are commonly found in servers, computers or mobile devices.

We are particularly interested in the `multi/handler` module as we need to specify the module to listen for incoming connections. The first thing we need to do is to assign in the multi/handler the same payload we have written before by using the exact same command we used in msfvenom.

![Downloading-ngrok](/assets/img/Android_Hacking/7.startmsfconsole.jpg)*Setting the payload in Metasploit*

We need to specify the localhost which is `0.0.0.0` and the local port which is `4242` in order to receive the backdoor's incoming connection in our Kali machine.  

![Downloading-ngrok](/assets/img/Android_Hacking/8.setparameters.jpg)*Set the parameters*

Now we are ready to be listening to incoming connections and we do that by typing `exploit -j -z`, sending our listener to the background without locking our terminal. 

Once we have a connection back from ngrok we will get notified and we will interact with the session.

All we have to do right now is just transfer this **maliciousapp.apk** file to the victim machine. We can do this in various ways, like sending the file via WhatsApp, email, uploading it to a server or any kind of file transfer website and asking the victim to open this file.

![Downloading-ngrok](/assets/img/Android_Hacking/9.startexploiting.jpg)*Start Exploiting*

We will be using [Files.fm](https://files.fm/){:target="_blank"}, which is essentially something like WeTransfer.com, and we use the specific service because it allows .apk files to be transferred without being blocked. 

Once the file is uploaded, we get a link which we will use to download the maliciousapp.apk onto our local machine in order to sign it before we send it to the victim.

![Downloading-ngrok](/assets/img/Android_Hacking/10.uploadfileonfilesfm.jpg)*Upload the malicious app onto Files.fm*

## Signing the maliciousapp.apk file

By signing the .apk file we will be identified as developers of this application. Skipping this step will cause most of the devices to not accept this .apk file and consequently don't run it. 

You may be able to install the file on your local machine but won't be able to run it. Therefore, signing the application is the only solution to overcome this obstacle.

Open a `cmd` and tyoe the following command 

```cmd
keytool -genkey -v -keystore my-release-key.keystore -alias alias_name -keyalg RSA -keysize 2048 -validity 10000
```

This command generates a keystore file that identifies the ID of the developer. Once it's executed it will ask for some information such as your name, your country, etc..., and a password. We need to type the password twice while skipping filling the following questions. When we are in the last question regarding the validation of correct provided information type `yes` and hit `enter`.  

![Downloading-ngrok](/assets/img/Android_Hacking/23.MaliciousappAPKSigning-1.png)*Maliciousapp.apk signing*

We have to have both the **maliciousapp.apk** and **my-release-key.keystore** files side by side. We're going to use the keystore to sign the .apk file using the [jarsigner](https://docs.oracle.com/javase/7/docs/technotes/tools/windows/jarsigner.html){:target="_blank"} command. Once executed, it will ask for a password and we have to **enter the same password** we used previously on the keystore file and hit `enter`. Now we have to just upload the signed maliciousapp.apk file on files.fm and send it to the victim. 

![Downloading-ngrok](/assets/img/Android_Hacking/24.MaliciousappAPKSigning-2.png)*Maliciousapp.apk signing*

## Downloading and installing the malicious application on the victim device

The signed maliciousapp.apk file has been sent to the victim using the files.fm service and with a nice convincing text we can make our victim believe that this is a benign app, so he/she can download and install on their system. 

![Downloading-ngrok](/assets/img/Android_Hacking/11.downloadingthemaliciousfile.jpg)*Victim downloads the malicious app on their system*

Once the unsuspecting victim clicks on the maliciousapp.apk file, they will be prompted to accept some kind of permissions in order to install the app.

![Downloading-ngrok](/assets/img/Android_Hacking/12.installingtheapp.jpg)*Accepting the  apps permissions*

Now the victim has accepted the permissions, installed the app and as soon as the app is opened in the victim device, we will have a meterpreter session in our Kali machine. There will be no sign of the application working on the victim device, therefore the victim could easily suppose that the app doesn't work. 

In fact, we have managed to hack into that Android device! 

![Downloading-ngrok](/assets/img/Android_Hacking/13.appinstalled.jpg)*Application successfully installed on the victim machine*


We will interact with the session using `sessions -1` and run `sysinfo` to get more information about the divice we have just hacked.

![Downloading-ngrok](/assets/img/Android_Hacking/14.sessionstarted.jpg)*Session started*


![Downloading-ngrok](/assets/img/Android_Hacking/15.sysinfo.jpg)*Getting system information*

We can navigate and list whatever the `/` directory hosts and we might find something that interests us.

![Downloading-ngrok](/assets/img/Android_Hacking/16.listingallthefilesintherootdirectory.jpg)*Lisitng the file in the root directory*

The storage folder looks interesting and we need to have a visit on it so we find out what type of information the victim has stored on his/her device.

![Downloading-ngrok](/assets/img/Android_Hacking/17.gettingmoreinfo.jpg)*Getting more info*
![Downloading-ngrok](/assets/img/Android_Hacking/18.accessingthecamerafolder.jpg)*Accessing the camera folder*

Since we found the Camera folder it's time to grab some of the pictures the victim has taken. We navigate into the `/sorage/emulated/legacy/DCIM/Camera` directory and use `download <name_of_the_image>.jpg` to download the picture on in our `/root` folder.

![Downloading-ngrok](/assets/img/Android_Hacking/19.downloadinganimage.jpg)*Downloading an image*
![Downloading-ngrok](/assets/img/Android_Hacking/20.imagedownloaded.jpg)*Image downloaded on the attack machine*

Up until this point, we managed to hack the victim's device and we're browsing into the system's folders (Cool, huh? :P). 

There are also other commands like sending SMS or getting the calls, but since we have hacked a tablet device with no SIM card we cannot perform any of these actions. 

**BUT**

**WAIT, HERE COMES THE COOL STUFF!!  ;)**

![Downloading-ngrok](/assets/img/Android_Hacking/21.findingthemaliciousappunderdownloads.jpg)*Maliciousapp.apk foun under device's dowloads folder*

## Getting access to tablet's camera

We will use something very cool; one of the most popular commands in Metasploit when it comes to mobile device hacking and that is `webcam_stream`!! 

This command, once fired, it will play a video stream for the specified webcam on the hacked device (it's scaaary). 

Once the service is started it gives us an HTML link (/root/OhKojwrd.html) and by open it on the web browser we can see the real-time recording on the victim's machine.

![Downloading-ngrok](/assets/img/Android_Hacking/22.startingwebcak.jpg)*Starting webcam_stream*

In that case my silly face :D That was scary :O

![Downloading-ngrok](/assets/img/Android_Hacking/23.capturingmyself.jpg)*Capturing myself in real time video*

**Lesson learned**: Be careful who you trust and what applications you allow to run on your system. Because an application claims to be benign and lets you run the latest and coolest features of GTA V doesn't mean that this is its purpose. Try to stay away from downloading untrusted applications and always go for the legit once hosted on Google Play when it comes to Android. Even if you know the person that suggested you run the application if you feel that something is wrong and you get suspicious over it DO NOT OPEN IT, because as we saw it's fairly easy to get hacked.  