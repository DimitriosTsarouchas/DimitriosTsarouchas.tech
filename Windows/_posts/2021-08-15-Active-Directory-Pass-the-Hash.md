---
title: Attacking Active Directory - Pass-the-Hash / Pass-the-Password Attacks 
author: Dimitrios Tsarouchas
date: 2021-08-15 09:55:00 +0800
categories: [Windows, Active Directory]
tags: [active directory, windows]
pin: true
featured-image: /assets/img/AD/pth.jpg
featured-image-alt: Active Directory Pass-the-Hash / Pass-the-Password Attack
---

## Pass-the-Hash Attack Overview
**Pass-the-Hash (PtH)** belongs to the family of Credential Theft and Reuse attacks, which takes advantage of the authentication mechanisms that maintain copies of the credentials on a system. 

A copy of credentials can be used on behalf of the users to access certain services and applications. Single Sign-On (SSO) is a type of such an authentication mechanism. 

Organisations running an Active Directory service will be significantly impacted if PtH is successful. For PtH to be successful, attackers first must have gained local admin access on a computer that is in the organisation’s network. 

As a next step, attackers will attempt to steal more authentication credentials from the already compromised computer to move laterally by reusing them for accessing other computers as well as services on the network. If the PtH is successful, it can result in compromising high-privileged accounts such as Enterprise Admins or Domain Admins. Therefore, organisations need to strengthen their security posture against PtH attacks and implement mitigation strategies to reduce the risks of such attacks [1](#1).

## Attack Phases
**Step 1: Passing the Hash around the network**

We will pass the hash around the network and pawn both client machines and the domain cotroller by usingthe `crackmapexec smb 192.168.242.0/24 -u "dtsarouchas" -H <hash_value>` command. Even though it does not have the “**Pwn3d!**” note next to it, the green plus mark indicates that this account has been pawned. 

![Pawning the DC and both user machines](/assets/img/AD/Pawning_the_DC.png)*Pawning the DC and both user machines*

**Step 2: Gaining shell access**

We can gain shell access on the "**MSCADMIN**" computer machine using the `psexec.py "dtsarouchas":@192.168.242.169 - -hashes <hash_value>` command.

![Gain shell access and control the user's machine](/assets/img/AD/Gain_shell_access.png)*Gain shell access and control the user's machine*


## Pass-the-Password Attack Overview
**Pass-the-Password (PtP)** attack works similarly to the PtH attack, which means that attackers have already compromised a user account. They managed to dump the NTLM hash from the SAM database of the system. They cracked it and got the user password in plain-text format. They then pass the user password along with the username to the network and try to access other computers in the network.


## Attack Phases
**Step 1: Pass the password around the network**
Crackmapexec tool takes a username, domain and password and throws this password all-around against the subnet. The command used to achieve that is `cme smb 192.168.242.0/24 –d university.local –u dtsarouchas –p Password1` 

![Pawning the DC and both user machines](/assets/img/AD/Pawning_the_DC_2.png)*Pawning the DC and both user machines*

Cracmapexec attacked the domain controller "**DC01**", the "**John Doe**" computer machine (**MSCSTUDENT**) and the "**Dimitrios Tsarouchas**" computer machine (**MSCADMIN**). It tried the password on the DC01, but it did not work, meaning that this user does not have SMB access to the domain controller. It pawned "**jdoe**" and "**dtsarouchas**" computer machine, and now the attacker has access to both machines.

**Step 2: Dumping the SAM file**

By typing `cme smb 192.168.242.0/24 -d university.local -u dtsarouchas -p Password1 -sam`, we are trying to dump the SAM file. 

![Dumping the SAM file](/assets/img/AD/Dumping_the_SAM.png)*Dumping the SAM file*

Cracmapexec dumped the SAM file, and we now have the "**Administrator**", "**Dimitrios Tsarouchas**" and "**John Doe**" NTLM hashes.

**Step 3: Getting a shell on the user machine**

Using `psexec.py university/dtsarouchas:Password1@192.168.242.168`, we gained shell access on the "**jdoe**" machine (**MSCSTUDENT**) and we own it.

![Gain shell access and control the user's machine](/assets/img/AD/Gain_shell_access_2.png)*Gain shell access and control the user's machine*

**Step 4: Dumping hashes**

To dump the hashes in both user machines, we will use the `secretsdump.py university/dtsarouchas:Password1@192.168.242.169` (first figure below) and the `secretsdump.py university/dtsarouchas:Password1@192.168.242.169` (second figure) commands.

![Dump the SAM hashes on dtsarouchas machine](/assets/img/AD/Dump_the_SAM_hashes.png)*Dump the SAM hashes on dtsarouchas machine*

![Dump the SAM hashes on jdoe machine](/assets/img/AD/Dump_SAM_on_jdoe.png)*Dump the SAM hashes on jdoe machine*


**Step 5: Cracking NTLM hashes**

We saved both users’ NTLM hashes and administrator NTLM hash into the "**secretsdumpHases.txt**" file and tried to crack them using the `hashcat -m 1000 secretsdumpHashes.txt rockyou.txt --force` command.

![Crack the obtained NTLM hashes](/assets/img/AD/Crack_the_obtained_NTLM.png)*Crack the obtained NTLM hashes*

Hashcat managed to crack both users’ NTLM hashes and returned their passwords in a clear-text format ("**Password1**" and "**Password2**") as shown below.
![NTLM hashes cracked and passwords revealed](/assets/img/AD/NTLM_hashes_cracked.png)*NTLM hashes cracked and passwords revealed*


## Pass-the-Password and Pass-the-Hash – Mitigation

Organisations cannot wholly prevent these two attacks by simply fixing or updating their systems. For example, a change in how LSASS credentials are stored simply requires an update of the attack tools to support such modifications. The following strategies will make it difficult for attackers to access organisations’ systems. Organisations should limit administrative delegation, implement policies that prevent local passwords from being reused, restrict local administrator accounts to standard users, and apply the Least Privilege principle [1](#1). 

Organisations should implement strong password policies, such as the use of passwords longer than fourteen characters, avoid the use of common words such as the word password, and establish policies for using long sentences with special characters instead of passwords (Passphrases). 

Using Privileged Access Management (PAM) will make it even more difficult for intruders to access the organisations’ systems. Organisations can implement PAM by firstly having individual accountability to avoid everyone sharing the same account. To do that, they must use a password vault where accounts are locked away and used only on an as-needed basis. By doing this, organisations give Just-Enough-Administration to admins to do their day-to-day activities using the Least Privilege principle. Having control and watch on what these people do with their delegated rights using Session Audits. 

Multi-factor authentication is needed to secure access. That proves that the person knows something more than a password. 

Augmentation of controls using Identity Analytics allows the identification of wrong permissions. 

Behavioural Analytics should be added for watching what the administrators do with the permissions they have been provided. The combination of Identity Analytics, Behavioural Analytics, and augmentation of controls that users get with the vault, the delegation and the session audit is more effective. 

Finally, govern everything that has to do with the identity and access management is the key. Therefore, PAM is a holistic end-to-end approach that provides all the benefits to an organisation from an operational and security perspective. Mitigation of risks comes by knowing who has access to what, and what they have done with that access giving an organisation the ability to detect anomalies and achieving compliance with regulations (PCI DSS, HIPPA, SOX, FISMA) [2](#2), [3](#3), [4](#4). 


## Evaluation of Mitigation Strategy's Effectiveness

Following the steps that Microsoft provides for implementing least privilege, an administrator should configure a GPO to restrict admin accounts on domain-joined computers. [5](#5).To do that on the domain controller use `Win+R` and run `secpol.msc`.

![Open security policy](/assets/img/AD/Open_security_policy.png)*Open security policy*

Once the Local Security Policy window is opened, navigate to `Local Policies → User Right Assignments`. The policies that must be configured are the `“Deny log on through Remote Desktop Services”`, `“Deny access to this system from the network”`, `“Deny log on as a service”` and finally `“Deny log on as a batch job”`. By default, no user has been added to that rights.

![Before adding the compromised account](/assets/img/AD/Before_adding.png)*Before adding the compromised account*

Administrators should add the user account "**university\dtsarouchas**" from these rights in order to prevent attackers who might get access to that account to pass the hash around the network and subsequently to the domain controller.

![Adding the compromised account](/assets/img/AD/Adding_the_compromised.png)*Adding the compromised account*
 
After these mitigations strategies were applied on the domain controller, the attacker could not be able to Pass-the-Hash or Pass-the-Password.
Now the attacker tries to Pass-the-Hash against the network "**192.168.242.0/24**" with a username and a hash that he hs on his hands. To do that he will use the `crackmapexec smb 192.168.242.0/24 -u "dtasrouchas" -H <hash_value>` command.

![Logon failure on the DC and on both user machines](/assets/img/AD/Logon_failure.png)*Logon failure on the DC and on both user machines*

Sdaly for the attacker, Pass-the-Hass attack can not logon to the domain controller, and a "**STATUS_LOGON_TYPE_NOT_GRANTED**" is displayed, meaning that user "**dtsarouchas**" is not granted the privileges to have access to the domain controller. On top of that, Pass-the-Hash could not authenticate the user on both client machines where "**dtsarouchas**" is the administrator and displayed the message "**STATUS_NOT_SUPPORTED**". 
The attacker will try to gain shell access on "**dtsarouchas**" computer machine. To achieve that, he uses the `psexec.py "dtsarouchas":@192.168.242.169 -hashes <hash_value>` command.

![Cannot create SMB session](/assets/img/AD/Cannot_create_SMB.png)*Cannot create SMB session*

The mitigation strategies applied to the domain controller prevented the attacker from gaining shell access even though he had the hashes of the user account. The only thing that the attacker got back is an SMB session error that the request is not supported. 

The same mitigation strategies apply to defend against the Pass-the-Password attack.
The attacker has the password of the user "**dtsarouchas**". He tries to attack against the domain controller and both domain-joined computers by using the `cme smb 192.168.242.0/24 -d university.local -u dtsarouchas -p Password1` command.

![Logon failure on the DC and on both user machines](/assets/img/AD/Logon_the_DC.png)*Logon failure on the DC and on both user machines*


Sadly for him, he cannot compromise the domain controller and gets back a message of "**STATUS_LOGON_TYPE_NOT_GRANTED**" as he got previously from the failed PtH attack. He will try to dump the hashes from the SAM database but will still be unable to do it.

![Logon failure on the DC and on both user machines and no SAM file is dumped](/assets/img/AD/Logon_failure_on_the_DC.png)*Logon failure on the DC and on both user machines and no SAM file is dumped*

Next, he will try to gain shell access on the "**dtsarouchas**" computer machine by using the `psexec.py university/dtsarouchas:Password@192.168.242.168` command.

![Shell access on the user's machine failed](/assets/img/AD/Shell_access.png)*Shell access on the user's machine failed*

Shell access cannot be gained, and an SMB session error that the request is not supported is being displayed. Then the attacker will try to dump the hashes from the SAM file on both domain-joined machines, but he will not be able to that either.

![Dumping the SAM file failed on both user machines](/assets/img/AD/Dumping_SAM_file_failed.png)*Dumping the SAM file failed on both user machines*

He gets back the message "**The request is not supported**", meaning that the mitigation strategies that were followed are effective and prevented both PtH and PtP attacks.

<br>

**References**

[1]<a name="1"></a>  [Mitigating Pass-the-Hash(PtH)Attacks and Other Credential Theft.](https://download.microsoft.com/download/7/7/A/77ABC5BD-8320-41AF-863C-6ECFB10CB4B9/Mitigating-Pass-the-Hash-Attacks-and-Other-Credential-Theft-Version-2.pdf/){:target="_blank"}

[2]<a name="2"></a> [Privileged Access Management for Active Directory Domain Services.](https://docs.microsoft.com/en-us/microsoft-identity-manager/pam/privileged-identity-management-for-active-directory-domain-services/){:target="_blank"}

[3]<a name="3"></a> [Privileged Access Management (PAM).](https://www.beyondtrust.com/resources/glossary/privileged-access-management-pam/){:target="_blank"}


[4]<a name="4"></a> [Cyberark Official Website.](https://www.cyberark.com/products/privileged-account-security-solution/core-privileged-account-security/){:target="_blank"}


[5]<a name="5"></a> [Implementing Least-Privilege Administrative Models.](https://docs.microsoft.com/en-us/windows-server/identity/ad-ds/plan/security-best-practices/implementing-least-privilege-administrative-models/){:target="_blank"}
