---
title: Attacking Active Directory - IPv6 Attack 
author: Dimitrios Tsarouchas
date: 2021-08-12 09:55:00 +0800
categories: [Windows, Active Directory]
tags: [active directory, windows]
pin: true
featured-image: /assets/img/active-directory-ipv6.jpg
featured-image-alt: Active Directory IPv6 Attack
---

## Internet Protocol version 6 Attack Overview
IPv6 attack is another form of relaying and is more reliable than the <a href="https://dimitrios-tsarouchas.tech/posts/Active-Directory-SMB-Relay-Attack/">previous attack</a> because it utilises IPv6. 

A machine running on a Windows network typically runs on IPv4 and chances are the network is not even utilising IPv6. If the machine utilises IPv4, but IPv6 is turned on, then there is no DNS service for IPv6. 

As a result, the attacker listens for all these IPv6 messages pretending to be the DNS server. Then, the client machine will send all the IPv6 traffic to him. When this happens, the attacker can get authentication to the domain controller via LDAP or SMB. 

When users log into the network, their credentials which are in an NTLM form are delivered to the attacker's machine. The attacker relays the NTLM credentials to the domain controller, and this is called an LDAP relay. 

In case the user is a Domain Admin, the attacker using LDAP logs in and creates a new account on the domain controller, which he can use later to login into the domain controller [1](#1), [2](#2). 

For this attack to be successful, the domain controller must be configured to have installed a certificate to run LDAP secure and second the attacker machine must have installed a tool called mitm6 [3](#3).

## Attack Phases
**Step 1: Attacker runs mitm6 and ntlmrelayx tools.**

So to begin with this attack we need spin mitm6 up using `mitm6 -d university.local` and we are getting replies from different devices in our network. From here we also need to setup a relay attack using `ntlmrelayx.py -6 -t ldaps://192.168.242.139 -wh maliciouswpad -l loot`.

![The attacker as DNS assigns IPv6 address to the client and waits for connections to happen](/assets/img/AD/AssigningDNS.png)*SMB Enabled and Required on the DC and not required on clients*

**Step 2: Client computer restarted**

The **dtsarouchas** client computer is restarted in order to speed up the process of assigning an IPv6 address from the attacker DNS to the client (IPv6 is sending out a reply looking for a DNS every 30 minutes). When the client machine is restarted it will try to authenticate.

**Step 3:Information is stored in the loot folder**

Information about the domain is stored in the **loot** folder, and we can check deeper to find any useful information. 

![Information about the target is stored in the loot folder](/assets/img/AD/AssigningDNS.png)*Information about the target is stored in the loot folder*

We are going to check the **domain_users_by_group.html** file as we're after the Domain Admins. The administrator in the description field of the SQLService used the password (**Password123@**), and now it is visible to us!

![The attacker checked the Domain Admins and found the password for the SQLService in the description field](/assets/img/AD/domain_admins.jpg)*The attacker checked the Domain Admins and found the password for the SQLService in the description field*

Administrators think that the descriptions are not visible, but the attacker managed to see them. The computer was capable of accessing the domain controller via LDAPS, logging into it and dumping out any useful information to the attacker. Now the attacker has valuable information in his hands. He can see who the Domain Admins,  Enterprise Admins, and Domain Users are and generally, and who he needs to attack in this environment.

**Step 4: An administrator logs into the client machine**

It is possible for the attack to take effect and target LDAPS when an administrator logs into the client machine. We can get in through the command and we're able to create a new user account. First, we set up an access control list **ACL** (Figure 1) and then create a new user (Figure 2). Now as attackers we own this domain.

![Access the DC via LDAPS and create ACLs](/assets/img/AD/LDAPS_ACLS.png)*Access the DC via LDAPS and create ACLs*

![Creating a new user in Users OU](/assets/img/AD/Users_OU.png)*Creating a new user in Users OU*

Checking the Users OU in the AD's Users and Computers, the administrator can find there the new user with the name of **"epIz0yDyAX"**.

![User is created in the AD's Users OU](/assets/img/AD/AD's_Users_OU.png)*User is created in the AD's Users OU*


## IPv6 Attack – Mitigation

As it turned out, even in a Windows IPv4 network, this attack can be useful because Windows still query for an IPv6 address. For that reason, organisations must block DHCPv6 traffic as well as incoming router ads through a group policy that should be implemented on the Windows Firewall. Nevertheless, disabling IPv6 can cause unwanted side effects [4](#4).

* The following rules shown in the figures below must be applied to the Windows Firewall to prevent this attack.

    * Block the connection on Core Networking – Dynamic Host Configuration Protocol for IPv6 (DHCPV6-In)
    * Block the connection on Core Networking – Dynamic Host Configuration Protocol for IPv6 (DHCPV6-Out)
    * Block the connection on Core Networking – Router Advertisement (ICMPv6-In)

![Block the connection for DHCPV6-In](/assets/img/AD/DHCPV6-In.png)*Block the connection for DHCPV6-In*
![Block the connection for DHCPV6-Out](/assets/img/AD/DHCPV6-Out.png)*Block the connection for DHCPV6-Out*
![Block the connection for DHCPV6-Out](/assets/img/AD/Router_Advertisement.png)*Block the connection for Router Advertisement*

* Disable the **"WinHttpAutoProxySvc"** service in case WPAD is not used internally and set a registry change via command prompt: REG add `"HKLM\SYSTEM\CurrentControlSet\services\WinHttpAutoProxySvc" /v Start /t REG_DWORD /d 4 /f` [5](#5). 

![Disable the WinHttpAutoProxySvc service](/assets/img/AD/WinHttpAutoProxySvc.png)*Disable the WinHttpAutoProxySvc service*

* LDAP signing and channel binding must be enabled for avoiding LDAP/LDAPS relaying [6](#6). 
    To enable LDAP singing, open the Registry Editor and navigate to `"Computer\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\NTDS\Parameters"` create the `REG_DWORD: "ldapserverintegrity"` and set the `"Value data"` field to `"2"`.

![Enable LDAP singing in the registry](/assets/img/AD/Enable_LDAP_singing.png)*Enable LDAP singing in the registry*

Another way to enable LDAP signing is by using Group Policy, which is described in Microsoft's Support website [7](#7). 

![Enable LDAP signing via group policy](/assets/img/AD/LDAP_signing.png)*Enable LDAP signing via group policy*

To enable LDAP channel binding, open the Registry Editor and navigate to `"Computer\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\NTDS\Parameters"` and create the `REG_DWORD: "LdapEnforceChannelBinding"` and set the `"Value data"` field to `"2"`. For compatibility with older OSs (Windows server 2008 and earlier) set the value to "1". Note that the <a href="https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-8563" target="_blank">CVE-2017-8563</a> update must be installed to avoid any compatibility issues. [8](#8)

![Enable LDAP channel binding in the registry](/assets/img/AD/LDAP_channel.png)*Enable LDAP channel binding in the registry*


* User impersonation through delegation can be mitigated by putting administrators to the **Protected Users Group**. That will prevent credentials being abused when users log in to the devices (Windows 8.1 and Windows Server 2012 or higher) [9](#9). 

    Another way is marking these accounts as **"Account is sensitive and cannot be delegated"** [10](#10). 

![Prevent delegation to sensitive accounts](/assets/img/AD/Prevent_delegation.png)*Prevent delegation to sensitive accounts*

* Restrict NTLM authentication [11](#11).

![Deny NTLM authentication via group policy](/assets/img/AD/Deny_NTLM.png)*Deny NTLM authentication via group policy*

* Microsoft suggests in implementing a registry fix to **"Prefer IPv4 over IPv6"** [12](#12).

    Open the Registry Editor. Navigate to `"Computer\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Tcpip6\Parameters"` double-click on `REG_DWORD: "DisabledComponents"` and set the `"Value data"` field to `HEX 0x20` or `DEC 32`.

![Prefer IPv4 over IPv6 in the registry](/assets/img/AD/Prefer_IPv4.png)*Prefer IPv4 over IPv6 in the registry*

Another way to prefer IPv4 over IPv6 is by using the command prompt and run the following 
    REG ADD 
    `"HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Tcpip6\Parameters" /v DisabledComponents /t  REG_DWORD /d 32 /f` 


## Evaluation of Mitigation Strategy's Effectiveness

The attacker uses the `mitm6 -d university.local` and the `ntlmrelayx.py -6 -t ldaps://192.168.242.139 -wh maliciouswpad -l LOOTME` commands.

![Assign IPv6 address as DNS and wait for connections to happen](/assets/img/AD/Assign_IPv6.png)*Assign IPv6 address as DNS and wait for connections to happen*

The user reboots his machine, and the attacker waits for mitm6 to assign a new IPv6 address as he pretends to be the DNS server. After the client machine rebooted, mitm6 assigned an IPv6 address to it. Sadly, for the attacker, no server connections started meaning that he cannot relay via LDAPS.

![IPv6 address assigned by DNS but no connection started](/assets/img/AD/IPv6_address_assigned.png)*IPv6 address assigned by DNS but no connection started*

The attacker searches for the "**LOOTME**" folder inside "**/opt/mitm6**" folder where he set up the relay, but nothing comes up which proves that the attack was unsuccessful.

![No LOOTME folder is created](/assets/img/AD/No_LOOTME.png)*No LOOTME folder is created*
 
When an administrator logs into the client machine, creating a new user is enabled, and the attacker can use that for his benefit as it happened before the mitigation took place.

![Administrator logs into the user's machine](/assets/img/AD/Administrator_logs.png)*Administrator logs into the user's machine*

Now, when the Administrator logs into the client machine, no new user is created. The attacker is now unable to get into the domain controller and that consequently renders the attack impossible.

![No connection started with the target machine](/assets/img/AD/No_connection.png)*No connection started with the target machine*

Checking the AD Users and Computers, one can now see that there is not any new user account. Therefore, the attacker has no account to access the machine.

![No new user is created in the AD's Users OU](/assets/img/AD/No_new_user.png)*No new user is created in the AD's Users OU*


**References**

[1]<a name="1"></a>  [Fox IT. (2018). mitm6 – compromising IPv4 networks via IPv6. blog.fox-it.com.](https://blog.fox-it.com/2018/01/11/mitm6-compromising-ipv4-networks-via-ipv6/){:target="_blank"}

[2]<a name="2"></a> [A Complete Guide on IPv6 Attack and Defense. 1st ed.](https://www.sans.org/reading-room/whitepapers/detection/complete-guide-ipv6-attack-defense-33904){:target="_blank"}


[3]<a name="3"></a> [Man In The Middle 6 Tool.](https://github.com/fox-it/mitm6){:target="_blank"}


[4]<a name="4"></a> [The worst of both worlds: Combining NTLM Relaying and Kerberos delegation.](https://dirkjanm.io/worst-of-both-worlds-ntlm-relaying-and-kerberos-delegation/){:target="_blank"}


[5]<a name="5"></a> [Completely remove WPAD (Use of Windows Proxy Auto Discovery) - Windows from client systems.](https://social.technet.microsoft.com/Forums/en-US/bb0d5d52-085c-4d63-a6ce-9ca7807f01fa/completely-remove-wpad-use-of-windows-proxy-auto-discovery-windows-from-client-systems?forum=win10itprogeneral#:~:text=Create%20a%20GPO%20and%20set,to%20reboot%20the%20workstation%20TWICE){:target="_blank"}


[6]<a name="6"></a> [2020 LDAP channel binding and LDAP signing requirements for Windows.](https://support.microsoft.com/en-gb/help/4520412/2020-ldap-channel-binding-and-ldap-signing-requirements-for-windows){:target="_blank"}

[7]<a name="7"></a> [How to enable LDAP signing in Windows Server.](https://support.microsoft.com/en-us/help/935834/how-to-enable-ldap-signing-in-windows-server){:target="_blank"}

[8]<a name="8"></a> [Use the LdapEnforceChannelBinding registry entry to make LDAP authentication over SSL/TLS more secure.](https://support.microsoft.com/en-gb/help/4034879/how-to-add-the-ldapenforcechannelbinding-registry-entry){:target="_blank"}

[9]<a name="9"></a> [Protected Privileged Accounts.](https://petri.com/windows-server-protected-privileged-accounts){:target="_blank"}

[10]<a name="10"></a> [Protecting Privileged Domain Accounts: Safeguarding Access Tokens.](https://www.sans.org/blog/protecting-privileged-domain-accounts-safeguarding-access-tokens/){:target="_blank"}

[11]<a name="11"></a> [Network security: Restrict NTLM: NTLM authentication in this domain.](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/network-security-restrict-ntlm-ntlm-authentication-in-this-domain){:target="_blank"}

[12]<a name="12"></a> [Guidance for configuring IPv6 in Windows for advanced users.](https://support.microsoft.com/en-gb/help/929852/guidance-for-configuring-ipv6-in-windows-for-advanced-users){:target="_blank"}

