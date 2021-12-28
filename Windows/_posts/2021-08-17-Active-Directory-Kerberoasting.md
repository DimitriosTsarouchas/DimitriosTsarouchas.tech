---
title: Attacking Active Directory - Kerberoast Attack 
author: Dimitrios Tsarouchas
date: 2021-08-17 09:55:00 +0800
categories: [Windows, Active Directory]
tags: [active directory, windows]
pin: true
featured-image: /assets/img/AD/kerveros.jpg
featured-image-alt: Active Directory Kerberoast Attack 
---

## Kerberoast Attack Overview
Kerberoast is an Active Directory attack that first presented by [Tim Medin in 2014 at Derbycon 4.0](https://www.youtube.com/watch?v=PUyhlN-E5MU&ab_channel=AdrianCrenshaw){:target="_blank"}. This attack cracks Kerberos service tickets and rewrites them to gain access to a targeted service. 

Kerberoast does not require any interaction with the service. The export of the service ticket can be requested as legitimate AD access. Then the attacker holding the service ticket can go and crack it offline and retrieve the service password in a plain-text format. This happens because a service ticket is encrypted using the service account hash. 

Therefore, any domain user, without getting shell access into the system that is running the service is able to dump the service’s hash. If an attacker cracks a service ticket hash, he cannot only get access to that service, more dengerously, he can achieve a full domain compromise. Full domain compromise can happen because often service accounts have been assigned with elevated privileges. 

The identification of these tickets can be made by considering the SPN of the domain user account, the passwords last set, the password expiration and the last logon. The five steps involved in this attack are, the SPN discovery, the service ticket request, the service ticket export, cracking the service ticket offline and finally rewriting the service ticket and RAM injection [1](#1), [2](#2).

## Attack Phases
**Step 1: Requesting a service ticket**

The attacker requests a service ticket using the following command `GetUserSPNs.py university.local/dtsarouchas:Password1 -dc-ip 192.168.242.139 -request`. 

![Request a service ticket](/assets/img/AD/Request_service_ticket.png)*Request a service ticket*

Then copies the TGS and saves it into the **serviceticket.txt** file for cracking it later using Hashcat. In order to crack the TGS ticket, he first needs to find the right module, and he does it by using the `hashcat –help | grep Kerberos` command.

![TGS ticket module](/assets/img/AD/TGS_ticket_module.png)*TGS ticket module*

**Step 2: Cracking the hash**

The attacker tries to crack the TGS hash using the `hashcat –m 13100 serviceticket.txt rockyou.txt -O --force` command.

![Gain shell access, own the MSCADMIN machine and dump the SAM hashes](/assets/img/AD/Cracking_the_TGS_hash.png)*Gain shell access, own the MSCADMIN machine and dump the SAM hashes*

Hashcat successfully cracks the TGS and reveals the password ("**Password1@**") of the SQLService.

![TGS hash is successfully cracked and the password for the SQLService revealed](/assets/img/AD/TGS_hash_cracked.png)*TGS hash is successfully cracked and the password for the SQLService revealed*


## Kerberoast Mitigations
Since Kerberos is a Windows feature, there is nothing that administrators can do to defend against Kerberoasting, except have strong passwords for their service accounts along with a password policy and make sure that this policy is been applied. The implementation of the Least Privilege principle [3](#3) must also be applied as well as avoiding the assignment of service accounts as Domain Administrators.

## Evaluation of Mitigation Strategy's Effectiveness

With the implementation of the least privilege (applied in previous mitigation strategies) for the user "**dtsarouchas**" the attacker could not get a TGS. Instead of a TGS, he got back an error that invalid credentials have been used. 

![Request of the TGS ticket failed](/assets/img/AD/TGS_ticket_failed.png)*Request of the TGS ticket failed*

The administrator has changed the password from the SQLService to a complex one that is 30 characters long. Since it is a long and complex password there is a way to remember it as passphrase. 

![Generate a 30-character password with complexity for the SQLService](/assets/img/AD/30-character.png)*Generate a 30-character password with complexity for the SQLService*

Next, the administrator resets the password for the SQLService on "**Users and Computers**" in AD.

![Reset the password of the SQLService](/assets/img/AD/SQLService.png)*Reset the password of the SQLService*
 
Hashcat fails to crack the password and gives a status of "Exhausted", thanks to the complexity and the number of characters has been set to the SQLService account.

![Requesting the TGS ticket](/assets/img/AD/Requesting_TGS_ticket.png)*Requesting the TGS ticket*

![Failed to crack the TGS hash after the mitigation strategies were applied](/assets/img/AD/Failed_to_crack_TGS.png)*Failed to crack the TGS hash after the mitigation strategies were applied*

The SQLService is a member of Domain Admins, which is a security risk.

![SQLService is member of Domain Admins](/assets/img/AD/SQLService_DA.png)*SQLService is member of Domain Admins*

The administrator removes the SQLService from the Domain Admins group.

![Removing the SQLService from Domain Admins](/assets/img/AD/Removing_the_SQLService.png)*Removing the SQLService from Domain Admins*

The SQLService has been removed from the Domain Admins group, and the administrator has implemented all the above mitigation strategies that were successful against the Kerberoasting attack.

![SQLService is removed from Domain Admins](/assets/img/AD/SQLService_removed.png)*SQLService is removed from Domain Admins*

Coming up next Group Policy Preferences/cpassword attack aka MS14-025!

<br>

**References**

[1]<a name="1"></a>  [Penetration Testing Lab. (2018). Kerberoast.](https://pentestlab.blog/2018/06/12/kerberoast/?fbclid=IwAR14Ec9XoTnZVj3lDFmnDVgcoG0vA20xbqnVnmx8U8RwZ1WjUQX1h0UPtdg){:target="_blank"}

[2]<a name="2"></a> [Pen Test Partners. (2019). How to: Kerberoast like a boss.](https://www.pentestpartners.com/security-blog/how-to-kerberoast-like-a-boss/?fbclid=IwAR14Ec9XoTnZVj3lDFmnDVgcoG0vA20xbqnVnmx8U8RwZ1WjUQX1h0UPtdg){:target="_blank"}

[3]<a name="3"></a> [Implementing Least-Privilege Administrative Models.](https://docs.microsoft.com/en-us/windows-server/identity/ad-ds/plan/security-best-practices/implementing-least-privilege-administrative-models/){:target="_blank"}
