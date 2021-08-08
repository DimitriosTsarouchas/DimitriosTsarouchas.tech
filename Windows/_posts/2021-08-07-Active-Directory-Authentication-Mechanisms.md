---
title: Active Directory Authenitcation Mechanisms
author: Dimitrios Tsarouchas
date: 2021-08-07 08:55:00 +0800
categories: [Windows, Active Directory]
tags: [active directory, windows]
pin: true
featured-image: /assets/img/NTLM-vs-Kerberos.jpg
featured-image-alt: Active Directory Authentication Mechanisms
---
![NTLM vs Kerberos](/assets/img/NTLM-vs-Kerberos.jpg)*Windows New Technology LAN Manager vs Kerberos Authentication protocols*
## Introduction
Windows Authentication verifies that the information provided in the operating system comes from a trusted source such as a valid user or computer account, in order to provide access to local and network resources. Active Directory is a central location for storing identity information. Therefore, users need to be authenticated based on something only they know, such as their password. 

Credentials are stored locally in the [Windows Local Security Authority (LSA)](https://docs.microsoft.com/en-us/windows/win32/secauthn/lsa-authentication){:target="_blank"}. When a user logs into a domain, Windows authentication packages use the credentials transparently, providing a single sign-on to network resources. Windows also use authentication through smart cards and biometric characteristics in order to provide multi-factor authentication. [1](#1)

## Windows logon and sign-in scenarios
In order for users to access local and network resources, a valid user account is required to logon to a Windows computer. Windows computers perform user authentication using a login process to provide secure access. After successful user account authentication, the authorisation process is applied to determine the level of access that this user has to the requested resource. Users must sign-in when requesting access to application and service resources, which means valid accounts and credentials are required. It is similar to the logon (used for a hardware system when it starts up, like a computer) process with the difference that the logon information is stored in the [Security Account Management (SAM)](https://en.wikipedia.org/wiki/Security_Account_Manager#:~:text=The%20Security%20Account%20Manager%20(SAM,authenticate%20local%20and%20remote%20users.&text=SAM%20uses%20cryptographic%20measures%20to%20prevent%20unauthenticated%20users%20accessing%20the%20system.)){:target="_blank"} database on the local computer and in AD. In contrast, the sign-in information (account and credentials) is managed by the application or service. [2](#2). 

There are four types of Windows logon scenarios:
### Interactive logon
In an interactive logon, users are asked to enter their credentials into a credential input box, or by inserting a smart card into a smart card reader, or by interacting with a biometric device. The interactive logon can be made using a local or a domain logon process.  For a domain logon, credentials submitted by users contain their account name and password or a certificate, as well as information about the AD domain. It is followed by the process of comparing the user information provided with the stored data in the security database on the AD domain or user's local computer. An interactive logon can be performed either locally, where users have physical access to a system (or when that system is part of a computer network), or remotely via [Terminal Services](https://docs.microsoft.com/en-us/windows/win32/termserv/terminal-services-portal){:target="_blank"} or [Remote Desktop Services (RDS)](https://docs.microsoft.com/en-us/windows-server/remote/remote-desktop-services/welcome-to-rds){:target="_blank"}, where the logon is qualified as remote interactive. The difference between a local logon and a domain logon is that on the local logon users are granted permissions for accessing resources on local computers in the network, while on a domain logon users are granted permissions for accessing local and domain resources. [2](#2)
### Network Logon
In a network logon, there is no dialog box collecting credentials because the account has already been authenticated. Credentials were previously collected by another method, so authentication is transparent to users unless alternate credentials are required. [NTLM](https://docs.microsoft.com/en-us/windows/win32/secauthn/microsoft-ntlm){:target="_blank"}, [Digest](https://www.rfc-editor.org/rfc/rfc2617.txt){:target="_blank"} and [Digest as a SALS Mechanism](https://www.ietf.org/rfc/rfc2831.txt){:target="_blank"}, [SSL/TLS](https://en.wikipedia.org/wiki/Transport_Layer_Security){:target="_blank"}, [Public Key certificates](https://en.wikipedia.org/wiki/Public_key_certificate){:target="_blank"} and [Kerberos v5 protocol](https://docs.microsoft.com/en-us/windows/win32/com/kerberos-v5-protocol#:~:text=The%20Kerberos%20v5%20authentication%20protocol,1993%2C%20in%20document%20RFC%201510.){:target="_blank"} are the authentication mechanisms included in the security system for providing network logon authentication. [2](#2)
### Smart Cards
Smart cards are only used for connecting to a domain account, and their authentication is based on the Kerberos protocol. A user's smart card does not use passwords, but a pair of private/public keys are stored on the smart card. The shared secret key derives from the user's password and the private key is stored only to the user's smartcard. [2](#2)
### Biometric logon
A device captures physical characteristics such as a fingerprint, the retina or the voice of a person, creates digital characteristics, and saves them in the AD database. On logon, it captures the characteristic and compares it with the data in storage. If this comparison is successful, authentication is successful. [2](#2)
## Authentication protocols
As described above, Windows system uses five authentication mechanisms to provide network logon functionality. The two primary protocols Windows AD uses to authenticate user and computer accounts are the NTLM and Kerberos. An alternative for smartcard logon authentication is utilising the public key certificate and authentication within web services is mostly utilising Digest or SSL/TLS protocols.
### NTLM Authentication
![NTLM-Challenge/Response](/assets/img/server-during-ntlm-authentication.png)*NTLM Challenge/Response Process - Image by IONOS*

**NTLM (New Technology LAN Manager)** is a family of authentication protocols based on a challenge/response mechanism that proves to a DC that the users going to be authenticated, knows the passwords associated with their accounts. NTLM is used for local logon authentication on non-domain controllers. 

When NTLM is used, the verification of a user or computer account can be done from a resource server. The resource server contacts a domain authentication service on a DC for that account. If this is a domain account, or in the case the account is local, the DC looks this account up on the local account database. [3](#3) 

NTLM credentials are a combination of a domain name, a username and a one-way password hash which basically are data obtained during the interactive logon process. [4](#4)

A closer look at how NTLM authentication is working is provided below:
1.	A user provides a domain name, username, and password to a client machine, which rejects the actual password after calculating the password's cryptographic hash.
2.	The username is sent to the server in plain text by the client machine.
3.	The server generates a challenge that is a 16-byte random number and sends it back to the client.
4.	Using the user's password hash, the client encrypts the challenge and sends the result, also called the response, back to the server.
5.	The server sends to the DC the username as well as the client's challenge and response.
6.	The DC, using the username, retrieves the hash of the user's password stored in the SAM database, and uses this password hash for encrypting the challenge.
7.	Finally, DC makes a comparison between its computed encrypted challenge and the client's computed response, and in case of a match, the user is authenticated. [4](#4)

In an Active Directory environment, the preferred authentication method is Kerberos v5, although Microsoft or non-Microsoft applications may still use NTLM [3](#3). 

Windows systems use the Microsoft Negotiate security package to select between Kerberos and NTLM authentication. This security package selects the Kerberos protocol over NTLM unless the systems involved in the authentication do not support Kerberos or the calling application has not sufficient information about it. [5](#5)

### Kerberos Authentication

Client and server authentication in Active Directory is provided by Kerberos version 5, which first appeared in Windows Server 2003 OS and is used to protect the authentication between the client and server in an open network. Kerberos is an open standard described in RFC 4120. [6](#6) 

Any application or service that supports Kerberos can work with Active Directory. The main advantage of the Kerberos protocol is that instead of using secrets, which are at risk of being sniffed by malicious users, it uses a common symmetric cryptographic key. Kerberos took its name from the three-headed dog in Greek mythology, and the reason is that it has three main components, a client, a server and a trusted authority that issues secret keys. The trusted authority is called **Key Distribution Center (KDC)**. It is installed as part of the DC and is responsible for the **Authentication Service (AS)** as well as the **Ticket-Granting Service (TGS)**. [7](#7)

In a typical key exchange, when users log in to the system, they must prove to KDC that they are whom they claim to be. First, the user provides to the KDC the username along with a long-time key that is generated based on the user's password. The Kerberos client, in this case, the user's computer, accepts the user's password and generates the cryptographic key. A copy of that key is also maintained in KDC's database. By receiving the request, the KDC compares the username and the long-term key with its records. If this information is accurate, the KDC responds to the user with a session key. This session key is called **Ticket-Granting Ticket (TGT)**. [7](#7) 

A Ticket-Granting Ticket contains a copy of the session key that is encrypted with the KDC's long-term key and is used by the KDC to communicate with the user. The TGT also contains the session key. The user uses the TGT to communicate with the KDC, which is encrypted with the user's long-term key so that only the specific user can decrypt. This session key is stored in the volatile memory of the user's machine. Future communication between the KDC and the user will be based on that session key which has a **TTL** value for a temporary usage. 

When the user request access to the server needs to contact the KDC using the session key provided. TGT and timestamp, are encrypted by the session key and the **service ID** included in the user's request. In order for the KDC to retrieve the session key, once it receives that request from the user, it makes use of its long-term key to decrypt the TGT and decrypts the timestamp by using the session key. The time difference must be less than five minutes to make sure the request came from a specific user. Since KDC confirms the request as legitimate, creates a new ticket called **service ticket**. The service ticket contains one key for the server and one key for the user, both including the user's name, the server, the timestamp, the TTL value and a new session key. One of these keys is encrypted with the user's long-term key and the other one with the server's long-term key. Both are encrypted together with the session key between the user and the KDC. 

When the ticket is sent to the user, it decrypts it using the session key and decrypts his key by using his long-term key. The above process reveals a new session key that is shared between the user and the server. The user then creates a new request, including the server's key. Once that key is sent to the server, it decrypts its key with its long-term key and retrieves the session key. The server uses that session key to verify the request's authenticity by decrypting the timestamp. [7](#7), [8](#8)

![Kerberos Authentication](/assets/img/Kerberos-mechanism.png)*Kerberos Authentication. Source: https://adsecurity.org*

Let us see how **Kerberos** works [3](#3), [9](#9):
*	The user sends an **AS_REQ** message to the KDC. That message is encrypted with the user's NTLM hash. 
*	The KDC sends back the TGT as an **AS_REP** and encrypts it with the **Kerberos TGT (krbtgt) hash**.
*	The user sends a **TGS_REQ** message to the KDC to request the **TGS ticket** from the KDC presenting his TGT.
*	The KDC is not able to know if the user has access to the application server so is going to provide back to the user **the TGS with the application server's account hash**.
*	In order for the user to authenticate to the application server, he presents the TGS to the application server, and **the application server will decrypt it using its hash**.
*	The application server will send back a response if the user can access that service or not.

When the user requests **a Ticket Granting Service (TGS)** from the KDC, they use **a Service Principal Name (SPN)** and that SPN should be associated with at least one service logon account. [9](#9)

So far we have discussed the purpose of a Windows Active Directory and its main authentication mechanisms. In later posts we will learn where Active Directory is vulnerable and why and how these Active Directory attacks work. Stay tuned!


**References**

[1]<a name="1"></a>  [Microsoft Docs. (2016). Windows Authentication Concepts.](https://docs.microsoft.com/en-us/windows-server/security/windows-authentication/windows-authentication-concepts){:target="_blank"}

[2]<a name="2"></a> [Microsoft Docs. (2016). Windows Logon Scenarios.](https://docs.microsoft.com/en-us/windows-server/security/windows-authentication/windows-logon-scenarios){:target="_blank"}

[3]<a name="3"></a> [Microsoft Docs. (2016) NTLM Overview.](https://docs.microsoft.com/en-us/windows-server/security/kerberos/ntlm-overview){:target="_blank"}

[4]<a name="4"></a> [Microsoft Docs. (2018). Microsoft NTLM.](https://docs.microsoft.com/en-us/windows/win32/secauthn/microsoft-ntlm){:target="_blank"}

[5]<a name="5"></a> [Microsoft Docs. (2018) Microsoft Negotiate.](https://docs.microsoft.com/en-us/windows/win32/secauthn/microsoft-negotiate){:target="_blank"}

[6]<a name="6"></a> [Neuman, C., Hartman, S., Yu, T. and Raeburn, K. (2005). The Kerberos Network Authentication Service (V5).](https://tools.ietf.org/html/rfc4120){:target="_blank"}.

[7]<a name="7"></a> Desmond, B., Richards, J., Allen, R. and Lowe-Norris. (2013). Active Directory. 5th ed. United States of America, CA: O’Reilly Media, chap. 1-2, 5, 9-10.

[8]<a name="8"></a> Francis, D. (2017). Mastering Active Directory. Birmingham: Packt Publishing Limited, chap. 1-2, 5, 10, 15.

[9]<a name="9"></a> [Metcalf, S. (2015). Active Directory Security: How Attackers Use Kerberos Silver Tickets to Exploit Systems.](https://adsecurity.org/?p=2011){:target="_blank"}