---
title: Attacking Active Directory - Token Impersonation Attack 
author: Dimitrios Tsarouchas
date: 2021-08-16 09:55:00 +0800
categories: [Windows, Active Directory]
tags: [active directory, windows]
pin: true
featured-image: /assets/img/AD/token_impersonation.jpg
featured-image-alt: Active Directory Token Impersonation Attack 
---

## Token Impersonation Attack Overview
Tokens can be thought of as cookies for a computer. They are temporary keys allowing a user to have access to a system or network without having to provide his/her credentials each time they access a file. 

There are two types of tokens: the delegate and the impersonate token. The delegate token is used when a user logs into a machine, or they have a remote desktop session. The impersonate token (non-interactive) is used when a user has a network drive attached or a domain logon script [1](#1), [2](#2). 

Therefore, in a Token Impersonation attack, an attacker can use a local administrator to impersonate another user logged on to a system, such as a Domain Admin.

## Attack Phases
**Step 1: Run Metasploit and set payload to the target machine**

We will run Metasploit, load the `exploit/windows/smb/psexec` module [3](#3) and set the necessary options for attacking the "**MSCADMIN**" computer machine. Then we will load the exploit `windows/x64/meterpreter/reverse_tcp` to gain a reverse shell [4](#4) 

![Set the parameters to attack the MSCADMIN machine and get a reverse shell](/assets/img/AD/get_reverse_shell.png)*Set the parameters to attack the MSCADMIN machine and get a reverse shell*

**Step 2: Run the exploit and get shell access**

Everything is set up, the attacker runs the `exploit` command, and gains shell access on the "**MSCADMIN**" computer machine and dumps the hashes using the `hashdump` command.

![Gain shell access, own the MSCADMIN machine and dump the SAM hashes](/assets/img/AD/Gain_shell_access_3.png)*Gain shell access, own the MSCADMIN machine and dump the SAM hashes*

**Step 3: Loading Incognito**

The attacker loads Incognito [5](#5) to impersonate a user. Using the `help` command, Incognito shows the available commands that an attacker can utilise.

![Load Incognito](/assets/img/AD/Load_Incognito.png)*Load Incognito*

![Incognito commands](/assets/img/AD/Incognito_commands.png)*Incognito commands*

**Step 4: Listing tokens**

Using the `list_tokens -u` command, the attacker sees that "**university\Administrator**" and "**university\dtsarouchas**" users are in this computer machine.

![List tokens](/assets/img/AD/List_tokens.png)*List tokens*

**Step 5: Impersonating Administrator**

In order for the attacker to impersonate the Administrator user, he will use the `impersonate_token university\\Administrator` command (Figure 98). After the successful impersonation of the Administrator user, the attacker types the `shell` command to gain access to that computer machine, and he is now the "**university\administrator**" user.

![Successful impersonation of Administrator and shell access to his machine](/assets/img/AD/Successful_impersonation.png)*Successful impersonation of Administrator and shell access to his machine*


**Step 5: Impersonating a User**

Then the attacker impersonates the "**dtsarouchas**" user using the `impersonate_token university\\dtsarouchas` command, gains shell access using the `shell` command, and he is now the "**university\dtsarouchas**" user.

![Successful impersonation of dtsarouchas user and shell access to his machine](/assets/img/AD/impersonation_of_dtsarouchas.png)*Successful impersonation of dtsarouchas user and shell access to his machine*


## Token Impersonation Mitigation Strategies

Organisations should limit the permissions for a user or token creation, which will not entirely prevent the attack. 

A better strategy is by using Account Tiering [6](#6). Domain Admins should be logging into the machines they need to access, which should only be the domain controllers. If for some reason a Domain Admin logs into a user computer, or a server, and that user/server computer is compromised, an attacker can impersonate that token. In an Account Tiering, users typically have two accounts, one that will use as a regular user for everyday activities, and one that will be used when they need to access the domain controller. In that way, if the regular user account is compromised, the attacker will not be able to get into the domain controller. 

Typically, the administrator accounts have longer-password policies and are stricter with their permissions. Therefore, it should be repetition in policies, and if these policies are in place, they will prevent an attacker. Finally, by using the least privilege principle and removing admin privileges on local users, an attacker will be unable to get shell access in that computer with their account [7](#7). That prevents the attacker from getting into a computer and utilising this kind of attack. 


## Evaluation of Mitigation Strategy's Effectiveness

Since the machines used are in a lab environment, the mitigation strategy that will be used is by implementing the least privilege principle for the user "**dtsarouchas**". Since the user "**dtsarouchas**" does not have admin privileges, psexec module will not work. Therefore, the attacker could not gain shell access to the user’s machine. 
The following steps are to mitigate the Token Impersonation attack.
Following the steps, Microsoft provides for implementing least privilege an administrator should configure a GPO to restrict admin accounts on domain-joined computers. [7](#7).To do that on the domain controller use `Win+R` and run `secpol.msc`. 

![Open security policy](/assets/img/AD/Open_security_policy_1.png)*Open security policy*

Once the Local Security Policy window is opened, navigate to `Local Policies → User Right Assignments`. The policies that must be configured are the `“Deny log on through Remote Desktop Services”`, `“Deny access to this system from the network”`, `“Deny log on as a service”` and finally `“Deny log on as a batch job”`. By default, no user has been added to that rights.

![Before adding the compromised account](/assets/img/AD/Before_adding.png)*Before adding the compromised account*

Administrators should add the user account "**university\dtsarouchas**" from these rights in order to prevent attackers who might get access to that account to pass the hash around the network and subsequently to the domain controller.

![Adding the compromised account](/assets/img/AD/Adding_the_compromised.png)*Adding the compromised account*
 
The attacker will load Metasploit and use the psexec module to gain shell access to the user "**dtsarouchas**" machine. Then he will set a native payload to upload malware on the user’s machine in order to set up a listener.

![Set the parameters to attack the MSCADMIN machine and try to get a reverse shell](/assets/img/AD/get_reverse_shell_1.png)*Set the parameters to attack the MSCADMIN machine and try to get a reverse shell*

Then he will try to exploit the target machine, set up the reverse TCP listener and gain shell access.

![Connection to the user's machine failed](/assets/img/AD/Connection_failed.png)*Connection to the user's machine failed*

The exploit failed to connect to the user’s machine because this user account has not administrator privileges. Therefore, no session can be created with the user’s machine. A message of an SMB communication error has been returned, making the attacker unable to gain shell access and subsequently impersonate any user on the domain.

Coming up next Kerberoasting, don't miss it as it is the most popular Active Directory attack! 

<br>

**References**

[1]<a name="1"></a>  [Windows Privilege Abuse: Auditing, Detection, and Defense.](https://medium.com/palantir/windows-privilege-abuse-auditing-detection-and-defense-3078a403d74e/){:target="_blank"}

[2]<a name="2"></a> [Token Impersonation.](https://hunter2.gitbook.io/darthsidious/privilege-escalation/token-impersonation/){:target="_blank"}

[3]<a name="3"></a> [PSEXEC PASS THE HASH.](https://www.offensive-security.com/metasploit-unleashed/psexec-pass-hash/){:target="_blank"}

[4]<a name="4"></a> [WORKING WITH ACTIVE AND PASSIVE EXPLOITS IN METASPLOIT.](https://www.offensive-security.com/metasploit-unleashed/exploits/){:target="_blank"}

[5]<a name="5"></a> [FUN WITH INCOGNITO.](https://www.offensive-security.com/metasploit-unleashed/fun-incognito/){:target="_blank"}

[6]<a name="6"></a> [Account Segmentation And It’s Importance.](https://www.intandemly.com/blog/tiered-approach-to-abm/){:target="_blank"}

[7]<a name="7"></a> [Implementing Least-Privilege Administrative Models.](https://docs.microsoft.com/en-us/windows-server/identity/ad-ds/plan/security-best-practices/implementing-least-privilege-administrative-models/){:target="_blank"}