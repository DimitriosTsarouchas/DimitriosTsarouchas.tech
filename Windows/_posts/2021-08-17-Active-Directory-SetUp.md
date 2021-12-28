---
title: Active Directory - Domain Controller Setup
author: Dimitrios Tsarouchas
date: 2021-12-27 09:55:00 +0800
categories: [Windows, Active Directory]
tags: [active directory, windows]
pin: true
featured-image: /assets/img/Active-Directory.jpg
featured-image-alt: Active Directory 
---

## Setting up the Domain Controller

Once we have downloaded the required ISO files for the [Windows Server 2019](https://www.microsoft.com/en-gb/evalcenter/evaluate-windows-server-2019){:target="_blank"} and [Windows 10 Enterprise](https://www.microsoft.com/en-gb/evalcenter/evaluate-windows-10-enterprise){:target="_blank"}, and have installed them in the virtual machine of our choice, we are ready to set up the Domain Controller.

**Note**: First, when installing both iso files remember to remove entirely the **Floppy** device from the **Hardawre** tab on the **VM Settings**. Forgetting to do that step, the virtual machine will try to install the "**autoinst.flp**" file which will ask for a key and it won't let us proceed to the installation! Secondly, remember to use **NAT:Used to share host's IP address** (in VMWare) as your Network connection. Thirdly, during Windows Server setup we will select **"Windows Server 2019 Standard Evaluation (Desktop Experience)"** as our operating system.

That's it let's jump in to setting up the DC!

Once everything is set up and you enter your Windows Server desktop the first thing we need to do is to rename the PC. Click on the start button and type "**Computer**" this will show up the "**View your PC name**" click on it and it will send you to the **About** section of the PC settings.
![Change_AD_Name](/assets/img/ADSetup/1.Change_AD_Name.png)
Then click on the "**Rename this PC**" button.
![Rename_PC_Name](/assets/img/ADSetup/2.Rename_PC_Name.png)
I am gonna rename my PC as "**TEST-DC**" where the "**DC**" indicates that this is a domain controller. You can use whatever name you want your DC to be named, but keep in mind that the name should resemble a domain controller. After renaming your PC, click **Next** and restart your pc by clicking "**Restart Now**" in the next window that will pop up. Once the PC is restarted, sign in to it, and once you do that you will be ready to install the Domain Controller from the **Server Manager Dashboard**.
![Server_management_Dashboard](/assets/img/ADSetup/3.Server_management_Dashboard.png)

The next step is to click on the top right-hand side in the menu `Manage -> Add Roles and Features`.

![Add_RolesAndFeatures](/assets/img/ADSetup/4.Add_RolesAndFeatures.png)
Click `Next`.
![AddRolesAndFeaturesWizard](/assets/img/ADSetup/5.AddRolesAndFeaturesWizard.png)
Select "**Role-based or feature-based installation**" and click `Next`.
![RoleBasedAndFeatureBasedInstallation](/assets/img/ADSetup/6.RoleBasedAndFeatureBasedInstallation.png)
"**Select a server from a server pool**" and click `Next`.
![SelectSerrverFromTheServerPool](/assets/img/ADSetup/7.SelectSerrverFromTheServerPool.png)
On selecting server roles we will double click on the "**Active Directory Domain Services**" and in the new "**Add Roles and Features Wizard**" window, we hit `Add Features`.
![ADDSAddFeatures](/assets/img/ADSetup/8.ADDSAddFeatures.png)

Click `Next`.

![SekectFeatures_Next](/assets/img/ADSetup/9.SekectFeatures_Next.png)

Click `Next`.

![ADDS_Next](/assets/img/ADSetup/10.ADDS_Next.png)

Click `Install`.

![Confirmation_install](/assets/img/ADSetup/11.Confirmation_install.png)

After the installation is completed click `Close`.

![Installation_Close.png](/assets/img/ADSetup/12.Installation_Close.png) 

You will notice on the Dashboard that there is an alert button under the Flag icon, click on the flag, and then click on "**Promote this server to a domain controller**".

![Promote_this_server_to_A-DC](/assets/img/ADSetup/13.Promote_this_server_to_A-DC.png)

Then we will select "**Add a new forest**" and will give a Root domain, again I will name my domain as "**TESTING.local**", you can give whatever name you'd like and remember to include the "**.local**" after the name, and click `Next`.

![Adding_A_New_forest](/assets/img/ADSetup/14.Adding_A_New_forest.png)

Then we will specify a password for our Domain Services Restore Mode (DSRM), and click `Next`.

![DC_Options](/assets/img/ADSetup/15.DC_Options.png)

Click `Next`.

![DNS_OPtions](/assets/img/ADSetup/16.DNS_OPtions.png)

Next, you will see your domain name is automatically populated into the **NetBIOS domain name** input field. Click `Next`.

![Additoinal_Options](/assets/img/ADSetup/17.Additoinal_Options.png)

Click `Next` on the Paths.

![Paths](/assets/img/ADSetup/18.Paths.png)

Click `Next` on the Review Options.

![Reveiw_Options_Next](/assets/img/ADSetup/19.Reveiw_Options_Next.png)

Click `Install` and once the installation is done the PC will automatically reboot.

![Install](/assets/img/ADSetup/20.Install.png)

Once the machine is rebooted you will notice it will have a user of **TESTING\Administrator** (or whatever name you have chosen for your domain name), which means that we're logging in to the domain **TESTING** as the **Administrator**, type your password, and you will be logged in as the administrator of your domain.

## Configure the Domain Controller
![LoginToTheDomainAsAdmin](/assets/img/ADSetup/ConfigureDC/1.LoginToTheDomainAsAdmin.png)
![AD_UsersAndComputers](/assets/img/ADSetup/ConfigureDC/2.AD_UsersAndComputers.png)
![NewOU](/assets/img/ADSetup/ConfigureDC/3.NewOU.png)
![MoveSecurityGroups](/assets/img/ADSetup/ConfigureDC/4.MOveSecurityGroups.png)
![UsersOU](/assets/img/ADSetup/ConfigureDC/5.UsersOU.png)
![AdminProps](/assets/img/ADSetup/ConfigureDC/6.AdminProps.png)
![CreatingAUser](/assets/img/ADSetup/ConfigureDC/7.CreatingAUser.png)
![Userspassword](/assets/img/ADSetup/ConfigureDC/8.Userspassword.png)
![CopyADomainAdmin](/assets/img/ADSetup/ConfigureDC/9.CopyADomainAdmin.png)
![DifferenceBetrweenDA_And_DU](/assets/img/ADSetup/ConfigureDC/10.DifferenceBetrweenDA_And_DU.png)
![New_Shares](/assets/img/ADSetup/ConfigureDC/11.New_Shares.png)
![SMB_Share_Quick](/assets/img/ADSetup/ConfigureDC/12.SMB_Share_Quick.png)
![OathforShare](/assets/img/ADSetup/ConfigureDC/13.OathforShare.png)
![NameSharedFolder](/assets/img/ADSetup/14NameSharedFolder.png)
![CreateShare](/assets/img/ADSetup/ConfigureDC/15.CreateShare.png)
![ShareFolderCreated](/assets/img/ADSetup/ConfigureDC/16.ShareFolderCreated.png)
![SetSPN](/assets/img/ADSetup/ConfigureDC/17.SetSPN.png)
![CheckingSPNSetUp](/assets/img/ADSetup/ConfigureDC/18.CheckingSPNSetUp.png)

## Configure a Domain User Machine
![Enablesharing](/assets/img/ADSetup/ConfigureUser/1.Enablesharing.png)
![ADDCIpaddress](/assets/img/ADSetup/ConfigureUser/2.ADDCIpaddress.png)
![SettingDNStheDC](/assets/img/ADSetup/ConfigureUser/3.SettingDNStheDC.png)
![AccessWorkOrSchool](/assets/img/ADSetup/ConfigureUser/4.AccessWorkOrSchool.png)
![JoiningtheDomain](/assets/img/ADSetup/ConfigureUser/5.JoiningtheDomain.png)
![JoiningasAdmin](/assets/img/ADSetup/ConfigureUser/6.JoiningasAdmin.png)
![7Skip](/assets/img/ADSetup/ConfigureUser/7Skip.png)
![signingtothedomainasJohn](/assets/img/ADSetup/ConfigureUser/8.signingtothedomainasJohn.png)
![loggedinasJdoe](/assets/img/ADSetup/ConfigureUser/9.loggedinasJdoe.png)
![loginasadmin](/assets/img/ADSetup/ConfigureUser/10.loginasadmin.png)
![addingJdoeinthedomain](/assets/img/ADSetup/ConfigureUser/11.addingJdoeinthedomain.png)
![JohnPCjoinedinthisdomain](/assets/img/ADSetup/ConfigureUser/12.JohnPCjoinedinthisdomain.png)