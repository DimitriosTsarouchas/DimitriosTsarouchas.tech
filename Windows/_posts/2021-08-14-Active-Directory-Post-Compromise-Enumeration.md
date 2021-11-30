---
title: Active Directory - Post-Compromise Enumeration 
author: Dimitrios Tsarouchas
date: 2021-08-14 09:55:00 +0800
categories: [Windows, Active Directory]
tags: [active directory, windows]
pin: true
featured-image: /assets/img/AD/pcenum.jpg
featured-image-alt: Post-Compromise Enumeration 
---

## Active Directory Post-Compromise Enumeration Overview
The attacker has compromised a user account and captured its password hash for later use. By using  offline cracking tools he managed to crack the password hash and got the password in plain-text. Taking advantage of SMB relay, he got into a machine, dumped the SAM file and collected the hashes that were stored stored in the file. He also managed to create an account in the AD using the mitm6 tool and owned the whole domain. 

Next step is the enumeratation of the network in order to see what actions he can do on the domain. Active Directory enumeration can be done by using two tools, "**PowerView**" [1](#1) and "**BloodHound**" [2](#2). 

PowerView is a PowerShell typescript that allows the attacker to enumerate the domain controller, domain policy, domain user groups and many more. 

BloodHound allows the attacker to visualise in a graph form what is going on in the domain and where he can find the user that might be logged in, or what is the shortest path to access Domain Admins.

## Domain Enumeration using PowerView

PowerView needs to be installed and run directly in one of the client machines since the domain enumeration takes place in a virtual environment. In a real-life scenario, an attacker will have access to a Windows shell and will use that shell to load PowerView. 

**Step 1: Bypassing PowerShell Execution Policy**

Our first step in the AD enumeration begins with bypassing the PowerShell execution policy, which exists for preventing users from accidentally executing scripts that might cause side effects on a Windows system. To bypass the execution policy we run `powershell –ExecutionPolicy bypass`.

![Bypassing PowerShell's execution policy](/assets/img/AD/Bypassing.png)*Bypassing PowerShell's execution policy*

**Step 2: Loading PowerView**

We will load PowerView by running `. .\PowerView.ps1`.

![Loading PowerView](/assets/img/AD/Loading_PowerView.png)*Loading PowerView*

**Step 3: Getting information about the domain**

It is time to collect information about the domain by running the `Get-NetDomain` command.

![Get information about the domain](/assets/img/AD/Get_information.png)*Get information about the domain*

Now we have valuable information about the domain like the forest name(**university.local**) and the domain controller(**DC01.university.local**) 

**Step 4: Getting information about domain controllers**

Let's dig more and get information about the domain controller using the `Get-NetDomainController` command.

![Get information about the DC](/assets/img/AD/info_about_DC.png)*Get information about the DC*

On our hands we have information about the domain controller, such as its IP address (**192.168.242.139**) and its name (**DC01.university.local**). Some networks have multiple domain controllers, and using this command will provide all that information for the domain controllers, as shown above. If other scanning tools fail to identify the domain controller in the network, the `Get-NetDomainController` provides the domain controller's IP address and that adds extra value in this command. 

**Step 5: Getting the domain policy**

Once we got what we need regarding the domain controller we need to learn more about the domain policy by using the `Get-DomainPolicy` command.

![Get information about the domain policy](/assets/img/AD/info_about_dp.png)*Get information about the domain policy*

This command displays all the different policies in the domain, such as the System Access policy and the Kerberos policy. We go deeper and searching for information about the System Access policy using the `(Get-DomainPolicy)." systemaccess"` command.

![Get information about the system’s policies](/assets/img/AD/info_about_sp.png)*Get information about the system’s policies*

The execution of this command gives the attacker valuable information about system policy such as the "**Minimum Password Age**", the "**Maximum Password Age**" and more. The exciting part here is the "**Minimum Password Length**" policy, which in this case it is seven. Why is this valuable to us? The answers is that now we know that the minimum password is seven and we can go ahead, spray a seven-character password across the network, and probably gain access to some user accounts.


**Step 6: Getting information about users**

Let's move further and grab some information about users in the domain which can be done using the 
`"Get-NetUser | select samaccountname, lastlogon, description, memberof"` command.

![Get information about the users](/assets/img/AD/info_about_users.png)*Get information about the users*

Now that we got some information about the users in the domain we will use filters to narrow down the results. The above figure displays all the users in the "**university.local**" domain, using filters like the SAM account name, the last logon, the description and which group the users are members of. The exciting thing here is that in the description of the SQLService account, it is shown that its password is "**Password123@**", allowing the attacker to compromise that account by just viewing the description.

**Step 7: Getting users' properties**

Now let's learn more about user properties, such as when the password was last set by using the `Get-DomainUser -Properties samaccountname,pwdlastset` command.

![Get the users' properties](/assets/img/AD/get_user_prop.png)*Get the users' properties*

We got information about the users' password last set property, which is useful for clarifying whether an account is old or new in the network. For example, the Administrator's password was last changed on 23th of August, and the new user from the man-in-the-middle attack was changed on the 25th of August.

**Step 8: Getting logon information**
In the same manner, the logon count can be obtained using the `Get-DomainUser -Properties samaccountname, logoncount` command.

![Get Logon information](/assets/img/AD/get_logon_info.png)*Get Logon information*

Using the above command, an attacker can identify whether or not there are honeypot accounts in the network by checking how many times a user has logged in the network. If an account has never logged in before that might be a honeypot account, and the attacker will avoid messing up with this account entirely because it is sitting there in the network on purpose for an attacker to capture it and once he captures it, it will alert the organisation's systems.


**Step 9: Getting bad password count**

Bad password count for the user accounts on the domain can be obtained using the `Get-DomainUser –Properties samaccountname, badpwdcount` command 

![Get information about wrong passwords](/assets/img/AD/wrong_pass.png)*Get information about wrong passwords*

It provides valuable information in case an attacker sees, for example, 100 bad passwords for an account, meaning that this account might be under a brute force attack.


**Step 10: Getting all computers in the network**

Let's check the operating systems that computer systems run on the domain by using the 
`"Get-DomainComputer | select samaccountname, operatingsystem"` command.

![Get information about the computers on the network](/assets/img/AD/get_computer_info.png)*Get information about the computers on the network*


Using this command filtered with the SAM account name and the operating system, an attacker can see the number of computers as well as if this computer is a server or a workstation on that network.


**Step 11: Getting group members**

Getting group members that are Domain Admins can be done by using the `Get-DomainGroupMember 'Domain Admins'` command.

![Get the group members](/assets/img/AD/group_members.png)*Get the group members*

Using this command, the attacker can identify who are domain administrators in that domain. According to the figure above, "**SQLService**", "**Jack Smith**" and "**Administrator**" belong to the Domain Admins group.


**Step 12: Getting Group Policies**

Getting group policies on the domain can be done by using the `Get-DomainGPO | select displayname, whenchanged` command.

![Get information about the group policies](/assets/img/AD/group_policies.png)*Get information about the group policies*

This command is useful to an attacker because it shows up all the group policies that have been specified in the domain. According to the above figure, the attacker gets information that the Windows Defender has been disabled, which will help him gain time from trying to bypass it.

**Step 13: Getting file shares**

File shares on the domain can be obtained by using the `Invoke-ShareFinder` command.

![Get information about file shares on the network](/assets/img/AD/file_shares.png)*Get information about file shares on the network*

Using the `Invoke-ShareFinder` command, we found out all the SMB shares in the network, and we can see what files are being shared and where they are being shared. For example, we found out the "**MScShare**" folder that lives on the domain controller as well as two "**Share**" folders that live on the users’ machines.


## Domain Enumeration using BloodHound

Once an attacker is in a machine in a network, BloodHound will download the data of Active Directory and visualise that data in a graph. It gives the attacker much information about the network very quickly. Otherwise, it would be very timing consuming to figure out all the intricate paths that exist on the network, like to get into a domain admin.


**Step 1: Collect data using SharpHound**

We will utilise SharpHound on the "**dtsarouchas**" machine to collect data from the domain that will then analyse with BloodHound. In order to do that we use the `Invoke-BloodHound –CollectionMethod All –Domain university.local –ZipFilename university.zip` command.

![Collect domain data via SharpHound](/assets/img/AD/SharpHound.png)*Collect domain data via SharpHound*

**Step 2: Uploading data to BloodHound**

After we get the data in the "**university.zip**" file, we then upload it to the BloodHound using the upload icon in the right-hand side on the BloodHound’s interface.

**Step 3: Database info**

After uploading the zip file in the BloodHound, the information about the database is available to us by clicking the hamburger icon on the left-hand side of the BloodHound’s interface. There one can see the number of users, computers, groups, active sessions, ACLs and relationships that exist on the network.

![Get database information](/assets/img/AD/database_information.png)*Get database information*

**Step 4: Pre-built queries**

BloodHound provides a list of pre-built queries that an attacker can make use of, as shown below.

![Pre-Built Analytics Queries](/assets/img/AD/Pre-Built.png)*Pre-Built Analytics Queries*

**Step 5: Finding all Domain Admins**

According to the image below, "**ADMINISTRATOR@UNIVERSITY.LOCAL**", "**JSMITH@UNIVERSITY.LOCAL**" and "**SQLSERVICE@UNIVERSITY.LOCAL**" are Domain Admins on the "**university.local**" domain. That way we know all the Domain Admins that exist on the domain.

![Find all Domain Admins](/assets/img/AD/Find_all.png)*Find all Domain Admins*

**Step 6: Finding the shortest path to Domain Admins**

![Find the shortest path to Domain Admins](/assets/img/AD/shortest_path.png)*Find the shortest path to Domain Admins*

As shown in the figure above, "**DTSAROUCHAS@UNIVERSITY.LOCAL**" user is an administrator on the "**MSCADMIN.UNIVERSITY.LOCAL**" computer, which in turn has a session to "**ADMINISTRATOR@UNIVERSITY.LOCAL**" account. It gives to the attacker the information that he can impersonate that user with the Token Impersonation attack, which will be covered on a later article. The attacker’s goal is to target machines where a Domain Admin is logged in. For example, if the attacker targets the "MSCADMIN" machine and owns it, he can own the Administrator account, and subsequently, he is the Domain Admin, and that is the shortest path for him to get on that level of access.

**Step 6: Finding the shortest path to Domain Admins**

![Find paths for Kerberoastable users](/assets/img/AD/Kerberoastable_users.png)*Find paths for Kerberoastable users*

According to the above figure, the "**SQLSERVICE@UNIVERISTY.LOCAL**" is part of the Domain Admins, "**DTSAROUCHAS (MSCADMIN.UNIVERSITY.LOCAL)**" and "**JDOE (MSCSTUDENT.UNIVERSITY.LOCAL)**" users have access to the Domain Admins as well.


**Step 7: Finding high-targeted values**

![Find high-targeted accounts](/assets/img/AD/high-targeted_accounts.png)*Find high-targeted accounts*

According to the above figure, "**Administrators**", "**Enterprise Admins**", and "**Domain Admins**" are high-level targets. "**DTSAROUCHAS@UNIVERSITY.LOCAL**" is an administrator on the "**MSCADMIN.UNIVERSITY.LOCAL**" computer machine, which has a session with "**ADMINISTRATOR@UNIVERSITY.LOCAL**", who is part of the "**ADMINISTRATORS@UNIVERSITY.LOCAL**" group, which is part of the "**DOMAN ADMINS@UNIVERSITY.LOCAL**" group, which is part of the "**ENTERPRISE ADMINS@UNIVERSITY.LOCAL**" group.


**References**

[1]<a name="1"></a>  [PowerSploit Tool.](https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1){:target="_blank"}

[2]<a name="2"></a> [BloodHound Tool.](https://github.com/BloodHoundAD/BloodHound){:target="_blank"}


