---
title: Attacking Active Directory - SMB Relay Attack 
author: Dimitrios Tsarouchas
date: 2021-08-10 09:55:00 +0800
categories: [Windows, Active Directory]
tags: [active directory, windows]
pin: true
featured-image: /assets/img/SMB-Relay-Attack.png
featured-image-alt: SMB Relay Attack
---

## Server Message Block Relay Attack Overview
In an SMB Relay attack, an attacker instead of cracking the captured hashes offline relays those hashes to another machine on the same network. 

**For this attack to be effective, first, the SMB signing must be disabled on the target machine.**

SMB signing is a [packet-level protocol](https://en.wikipedia.org/wiki/Packet_Layer_Protocol){:target="_blank"}. If SMB signing is enabled, when an attacker tries to relay the credentials, the machine understands that these packets are not signed by him so it does not allow him access. 

Nevertheless, if SMB signing is disabled it never checks for the authenticity of where these packets are coming from. It accepts the username and the hash and let the attacker connect to that machine if he has the permission to do so. 

**Second, the attacker has to have administrator access on that machine as well.**

The attacker cannot relay the credentials captured from one machine back to the same machine. It has to relay those credentials on two separate machines. [1](#1), [2](#2)

## Attack Phases
**Step 1: Checking whether SMB is enabled or not.**

The attacker runs the 
``` nmap --script=smb2-security-mode.nse -p445 192.168.242.0/24
```
 command.

![SMB-Enabled-and-Required-on-the-DC-and-not-required-on-clients](/assets/img/SMB-Enabled-and-Required-on-the-DC-and-not-required-on-clients.png)*SMB Enabled and Required on the DC and not required on clients*

On the domain controller (**192.168.242.139**) the message signing is enabled and required. 

Message signing is disabled by default in any workstation but enabled and required on every single server (or Domain Controller) by default. Therefore, the attacker is not able to relay through the domain controller, as it is a server. 

However, in both client machines (**192.168.242.168** and **192.168.242.169**), the message signing is enabled but not required. That means the attacker can perform a relay attack because of that non-requirement. 

The attacker grabs both client IP addresses and saves them into the **targets.txt file**. The attacker now will relay the credentials from 192.168.242.169 to 192.168.242.168 using [Responder](https://tools.kali.org/sniffingspoofing/responder){:target="_blank"}.

**Step 2: Attacker modifies Responder**

The attacker will turn off both SMB and HTTP on the Responder's configuration file. In that way Responder plays the role of a listener and does not respond on these services.

![SMB-and-HTTP-disabled-on-Responder](/assets/img/SMB-and-HTTP-disabled-on-Responder.png)*SMB and HTTP disabled on Responder*

**Step 3: Attacker runs Responder**

The attacker runs Responder to listen for a response using the ``` responder -I eth0 -rwdv ``` command. Responder now is waiting for events to happen.

**Step 4: Attacker sets up the relay**

The attacker sets up a relay using the ```ntlmrelayx.py -tf targets.txt -smb2support``` command.

![Running-ntlmrelayx.py-and-waiting-for-connections](/assets/img/Running-ntlmrelayx.py-and-waiting-for-connections.png)*Running ntlmrelayx.py and waiting for connections*

Servers are now waiting for a connection to be established.

**Step 5: An event occurs**

User **dtsarouchas** wants to access the domain controller. Instead of the domain controller’s IP address, he types the attacker's IP address.

![DNS-failure-Wrong-address](/assets/img/DNS-failure-Wrong-address.png)*DNS failure: Wrong address*

**Step 6: SAM hashes have been dumped**

![Local-SAM-hashes-have-been-dumped](/assets/img/Local-SAM-hashes-have-been-dumped.png)*Local SAM hashes have been dumped*

The authentication succeeded because SMB signing is enabled but not required and because "dtsarouchas" is the Administrator on this computer. Therefore, the SAM hashes were able to be dumped. Now the attacker can copy the SAM hashes, try cracking them offline and move laterally or vertically.

The attacker now will gain access to SMB interactive shell by using the ```ntlmrelayx.py -tf targets.txt -smb2support -i``` command.

![Start-interactive-SMB-client-shell-via-TCP](/assets/img/Start-interactive-SMB-client-shell-via-TCP.png)*Start interactive SMB client shell via TCP*

Now the attacker has access to the interactive shell on 127.0.0.1:11000 and will access that SMB shell using the command ```nc 127.0.0.1 11000```.

![Run-netcat-to-access-the-SMB-shell](/assets/img/Run-netcat-to-access-the-SMB-shell.png)*Run netcat to access the SMB shell*

The attacker navigates to shares where he can find ADMIN$, C$, IPC$, and Share folders. Then he uses the C$ drive and lists the files this folder has in it.

![List-the-contents-of-C$-drive](/assets/img/List-the-contents-of-C$-drive.png)*List the contents of C$ drive*

Going a step further, he uses the ADMIN$ folder where he can add files, get files, having full control on the administrator folder and subsequently to this computer.

![Listing-the-contents-of-the-ADMIN$-folder](/assets/img/Listing-the-contents-of-the-ADMIN$-folder.png)*Listing the contents of the ADMIN$ folder*

## Mitigating SMB Relay Attack

In order to mitigate SMB relay attacks, organisations should first enable SMB signing on all devices, which will stop the attack completely. 

A downside is that it can cause performance issues with file copies. Alternatively, they could disable NTLM authentication on their network, which will completely stop the attack as well. However, if Kerberos stops working as the authentication method, Windows is going to default back to NTLM, so it is not a failsafe ultimately. 

Another useful measure is Account Tiering, which limits domain admins to specific tasks. Account Tiering means that the Domain Administrator is only logging into their domain accounts and not to their user accounts. However, enforcing this policy might not be easy for an organisation. 

Last but not least, organisations should restrict local administrators, which will prevent much lateral movement. However, there will be a potential increase in the amount of service desk tickets because users may complain and may want to have admin rights [3](#3), [4](#4), [5](#5).

### Evaluation of Mitigation Strategy's Effectiveness

The administrators should restrict NTLM authentication on the domain. To do that they should follow the steps below:

On the domain controller, by using **Win+R** type **gpedit.msc** that will bring on the **Local Group Policy Editor**. 

Under **Local Computer Policy**, navigate to ```Computer Configuration → Windows Settings → Security Settings → Security Options```. 

On the right side search to find, the policy called **Network Security: Restrict NTLM: NTLM authentication in this domain**. 

Double click on that policy and at the pop-up Window on the **Local Security Setting** tab using the dropdown menu select **Deny All**. [6](#6)

![Open-Group-Policy-Management-Editor](/assets/img/Open-Group-Policy-Management-Editor.png)*Open Group Policy Management Editor*

![Deny-NTLM-authentication-in-this-domain](/assets/img/Deny-NTLM-authentication-in-this-domain.png)*Deny NTLM authentication in this domain*

The attacker will scan against the whole network on port 445 to check if SMB signing is enabled and required using the ```nmap --script=smb2-security-mode.nse -p445 192.168.242.0/24``` command.

![Checking-where-SMB-is-enabled-or-disabled](/assets/img/Checking-where-SMB-is-enabled-or-disabled.png)*Checking where SMB is enabled or disabled*

On the domain controller, message signing is enabled and required, and on both client machines are enabled but not required, meaning that the attacker can relay and authenticate against the client machines. The attacker runs the ```ntlmrelayx.py -tf targets.txt -smb2support``` command but this time, the authentication of the user "dtsarouchas" on the domain failed on both machines.

![Authentication-as-dtsarouchas-user-failed-on-both-user-machines](/assets/img/Authentication-as-dtsarouchas-user-failed-on-both-user-machines.png)*Authentication as dtsarouchas user failed on both user machines*

The attacker goes a step further and tries to gain interactive shell access to connect to the client machine using the ```ntlmrelayx.py -tf targets.txt -smb2support -i``` command, but this failed as well.

![Authentication-as-dtsarouchas-user-failed-and-no-interactive-shell-gained](/assets/img/Authentication-as-dtsarouchas-user-failed-and-no-interactive-shell-gained.png)*Authentication as dtsarouchas user failed and no interactive shell gained*

It is now proved that by denying all NTLM authentication requests from the servers and accounts of the domain controller can effectively prevent the SMB Relay Attack.

In a later post we will learn another form of relaying that is more reliable than the previous attack because it utilises IPv6. Stay tuned!

**References**

[1]<a name="1"></a>  [SWikipedia Contributors. SMBRelay.](https://en.wikipedia.org/wiki/SMBRelay ){:target="_blank"}

[2]<a name="2"></a> [Baggett, M. (2013).  SMB Relay Demystified and NTLMv2 Pwnage with Python. Sans.org.](https://www.sans.org/blog/smb-relay-demystified-and-ntlmv2-pwnage-with-python/){:target="_blank"}


[3]<a name="3"></a> [Pollack, K. (2019).  Mitigating Relay NTLM Remote Code Execution. calcomsoftware.com.](https://calcomsoftware.com/mitigating-ntlm-remote-code-execution/){:target="_blank"}


[4]<a name="4"></a> [Bowes, R. (2008), ms08-068 - Preventing SMBRelay Attacks. Blog.skullsecurity.org.](https://blog.skullsecurity.org/2008/ms08-068-preventing-smbrelay-attacks){:target="_blank"}


[5]<a name="5"></a> [Intademly. (n.d.) Account Segmentation And It’s Importance. intademly.com.](https://www.intandemly.com/blog/tiered-approach-to-abm/){:target="_blank"}


[6]<a name="6"></a> [Microsoft Docs. (2017). Network security: Restrict NTLM: NTLM authentication in this domain. docs.microsoft.com.](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/network-security-restrict-ntlm-ntlm-authentication-in-this-domain){:target="_blank"}


