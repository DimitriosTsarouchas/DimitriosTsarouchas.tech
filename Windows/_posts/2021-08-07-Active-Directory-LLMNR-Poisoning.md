---
title: Active Directory - LLMNR and NBT-NS Poisoning 
author: Dimitrios Tsarouchas
date: 2021-08-07 09:55:00 +0800
categories: [Windows, Active Directory]
tags: [active directory, windows]
pin: true
featured-image: /assets/img/LLMNR-Poisoning.jpg
featured-image-alt: LLMNR Poisoning
---

## Attacking Active Directory
Active Directory plays an essential role in a corporate domain that makes it quite attractive to attackers. The service itself does not have to be the target as it can serve as a means of providing a route for breach of other systems connected to it. 

As Charles Kolodgy said, most organisations do not have adequate security measures on their Active Directory. This is based on the findings of a research that showed that 97% of the companies who use AD believe that it is a critical part of their company and should be safeguarded as possible. A high percentage of them also (71%) believes that a recovery after an AD attack would be very difficult if not impossible [1](#1), [2](#2). 

The following are the most common attacks used by adversaries to break into Active Directory. 

A prerequisite for the success of the following attacks is the deactivation of any antivirus software.

## LLMNR and NBT-NS Poisoning 
![LLMNR poisoning](/assets/img/LLMNR-Poisoning.jpg)*LLMNR poisoning*
### Overview
LLMNR stand for **Link-Local Multicast Name Resolution**, and NBT-NS stands for **NetBIOS Name Service**. LLMNR and NBT-NS used to identify hosts when DNS is unable to do it. 

The difference between them is that LLMNR is based on the DNS format allowing hosts to perform name resolution for other hosts that are on the same local link. At the same time, NBT-NS identifies systems by their NetBIOS name on a local network. [3](#3) 

When a user responds to this service, the service responds with a username and a password hash, giving the attacker the most valuable information to compromise a domain.

### Vulnerability
The attacker listens to the UDP ports **5355** or **135** that are related to LLMNR and NBT-NS respectively. He broadcasts and responds to them, pretending that he knows the location of the requested host. 
![LLMNR poisoning process](/assets/img/LLMNR-Poisoning-Process.png)*LLMNR poisoning process - Image by technologysolutions*

Let us analyse the steps from the above figure: [4](#4)
1.	The VICTIM wants to connect to the hackme server at **\\hackme**, but mistakenly typed in **\\hackm**
2.	The DNS server responds to VICTIM saying that it does not know that host.
3.	The VICTIM asks then if anyone in the network (multicasts) knows the location of \\hackm.
4.	The HACKER responds to the VICTIM saying that he knows where the \\hackm is, and that he needs the credentials to log in the VICTIM.
5.	The VICTIM believes the HACKER and sends its username and NTLMv2 hash to the HACKER.
6.	The HACKER can now crack the hash to discover the password.
Note: The HACKER can use various methods to make the VICTIM type his IP or domain name, instead of waiting for him to do it by mistake. Nevertheless, these methods will not be analysed here, as they are not related to AD's functionality.  

### Attack Phases
The attacker uses a tool (Responder) to perform a man in the middle attack. When an event occurs (someone mistyped the network drive), a DNS failure happens. Responder captures the IP address of the client machine, the username and the NTLMv2 hash. The attacker gets the hash and then he can try to crack it using a tool designed for this purpose, like **Hashcat**. [5](#5)
#### Step 1: The attacker runs Responder to capture hashes
The attacker uses the following command to capture hashes:
```terminal
root@kali:~# responder –I eth0 –rwdv
```

![Responder waiting for requests](/assets/img/Responder-waiting-for-requests.png)*Responder waiting for requests*

After Responder is loaded, it is in the middle waiting for any requests. It needs to be run when the network has high traffic.

#### Step 2: An event occurs (DNS failure)
The user tried to open a file share but instead of pointing to the domain controller’s IP address at 192.168.242.139, mistypes and points to the attacker's IP address at 192.168.242.138. The Windows Security dialog box pops up informing the user that access is denied to that domain.

![DNS failure - Access Denied](/assets/img/DNS-failure-Access-Denied.png)*DNS failure - Access Denied*

#### Step 3: Attacker grabs the hashes
While the event occurred, Responder captured the NTLMv2 user hash, the IP address of the user's machine, and the username.
![Responder captured user information](/assets/img/Responder-captured-user-information.png)*Responder captured user information*

#### Step 4: Attacker cracks the hashes using Hashcat.
Hashcat is a tool utilised to crack hashes. Since the attacker has to crack an NTLMv2 hash, he will use the module **5600**. Because the attacker's machine is hosted in a virtual environment, he needs to use the **--force**, forcing Hashcat to run over CPU instead of a GPU.
![NTLMv2 module](/assets/img/NTLM-Module.png)*NTLMv2 module*

The attacker runs the 

```terminal
root@kali:~# hashcat –m 5600 university-ntlmhashes.txt rockyou.txt –force
```
 command and tries to crack the hashes that saved in the **"university-ntlmhases.txt"** file against the wordlist **"rockyou.txt"**.
![NTLMv2 module](/assets/img/Cracking-the-captured-Hashes.png)*Cracking the captured hashes against a wordlist with Hashcat*

Hashcat cracked the NTLMv2 hash of the user and revealed the password that is **"Password1"**. That is precisely the password of that user, as shown in the figure below.

![NTLMv2 hash cracked and password has been revealed](/assets/img/NTLMv2-hash-cracked-and-password-revealed.png)*NTLMv2 hash cracked and password has been revealed*

If the passwords are weak and guessable, such as the above situation, the attacker will be able to crack those hashes very fast. 

A weak password is considered when it is less than fourteen characters long and has no complexity at the same time. When the attacker gets the password in plain text, it leverages the account to get to a machine. 

Most organisations do not have an effective password policy and are still using LLMNR and NBT-NS, which makes them an easy prey for an attacker who wants to get the initial foothold. 


### LLMNR Poisoning Mitigation
Administrators should disable both LLMNR and NBT-NS. Disabling just LLMNR is not enough because if DNS fails it goes to LLMNR and if LLMRN fails, it goes to NBT-NS.
#### Disabling LLMNR
To disable LLMNR navigate to **Local Group Policy Editor** using **Win + R** and run **gpedit.msc**. Under 
```
Local Computer Policy → Computer Configuration → Administrative Templates → Network → DNS Client
```
, double click on **"Turn Off Multicast Name Resolution"** on the popup window select **"Enabled"** then click **"Apply"** and "OK". [6](#6) 
![Disable LLMNR protocol](/assets/img/Disable-LLMNR-protocol.png)*Disable LLMNR protocol*

#### Disabling NBT-NS
To disable NBT-NS navigate to 

```
Network Connections → Network Adapter Properties → TCP/IPv4 Properties
```
click on the **"Advanced"** button go to **"WINS"** tab and select **"Disable NetBIOS over TCP/IP"** and click "OK". [7](#7)
![Disable NBT-NS protocol](/assets/img/Disable-NBT-NS.png)*Disable NBT-NS protocol*

In case an organisation cannot disable LLMNR or NBT-NS, it must first require Network Access Control [8](#8), meaning that no one can go and plugin to any port in the organisation's network and gain access. 

However, only MAC addresses that belong to the organisation's network should be allowed. If the MAC address does not belong to the network or it is not allowed, they must shut that port down. 

Second, the organisation must make use of a strong password policy, because the longer and complex the password is, the harder is for an attacker to crack the password hash. 

A strong password policy could be a password that avoids common word usage and is greater than 14 characters long. [9](#9)

In a later post we will learn how an attacker relays the obtained hashes to another machine on the same network. Stay tuned!


**References**

[1]<a name="1"></a>  [Semperis Tech. (2020). 84% Of Organizations Report That the Impact of an Active Directory Outage Would Be Significant, Severe, or Catastrophic in the Latest Semperis Study.](https://www.semperis.com/press-release/84-of-organizations-report-that-the-impact-of-an-active-directory-outage-would-be-significant-severe-or-catastrophic-in-the-latest-semperis-study/){:target="_blank"}

[2]<a name="2"></a> [Semperis Tech. (2020). New survey reveals dangerous gaps in crisis management plans. Semperis.com](https://www.semperis.com/new-survey-reveals-dangerous-gaps-in-crisis-management-plans%e2%80%af/){:target="_blank"}


[3]<a name="3"></a> [Kuehn, E., Demaske, M., Adaptforward, (2020). Man-in-the-Middle: LLMNR/NBT-NS Poisoning And SMB Relay.](https://attack.mitre.org/techniques/T1557/001/){:target="_blank"}


[4]<a name="4"></a> [Heath, A., (2018). Pen Testing Techniques for Internal Enterprise Network Attacks: LLMNR and NBT-NS Poisoning. technologysolutions.northstate.net.](https://technologysolutions.northstate.net/insights/pen-testing-techniques-for-internal-enterprise-network-attacks-llmnr-and-nbt-ns-poisoning/){:target="_blank"}


[5]<a name="5"></a> [Hashcat.net, (2020). Advanced Password Recovery. Hashcat’s Official Website.](https://hashcat.net/hashcat/){:target="_blank"}


[6]<a name="6"></a> [Vmware Docs. (2019). Disable LLMNR with Active Directory GPO. vmware.com.](https://docs.vmware.com/en/VMware-Workspace-ONE-UEM/2011/WS1_KCD_SEGV2/GUID-0B7E3C5E-CCFE-4A5E-B990-5C8196D0B084.html){:target="_blank"}


[7]<a name="7"></a> [LLMNR, NBT-NS, DNS and other acronyms I can own your network with ;) TECHGUARD BLOG.](https://blog.techguard.com/the-roi-of-security-awareness-training-0#:~:text=LLMNR%20and%20NBT%2DNS%20are,IP%20address%20of%20a%20resource.){:target="_blank"}

[8]<a name="8"></a> [Cisco. (n.d.) What Is Network Access Control. cisco.com.](https://www.cisco.com/c/en_uk/products/security/what-is-network-access-control-nac.html){:target="_blank"}

[9]<a name="9"></a> [Specops. (2020). Password length best practices. Specopssoft.com.](https://specopssoft.com/blog/password-length-best-practices/){:target="_blank"}

