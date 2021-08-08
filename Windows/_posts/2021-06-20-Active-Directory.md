---
title: Active Directory
author: Dimitrios Tsarouchas
date: 2021-06-20 08:55:00 +0800
categories: [Windows, Active Directory]
tags: [active directory, windows]
pin: true
featured-image: https://kevinholman.com/wp-content/uploads/2018/12/image-243.png
featured-image-alt: Microsoft Active Directory
---

## Introduction

**Active Directory** was built with convenience in mind, like Microsoft Windows, which is the most commonly used operating system due to its user-friendliness. However, that might lead to unintended exposure of its features to even non-experts at both sides, authorised and unauthorised users. Malware that exploits its weak spots is a prevalent technique. 

A malware named ["TrickBot"](https://www.itsnyc.com/2020/01/30/active-directorybeing-targeted-by-malware-called-trickbot/){:target="_blank"} is a good example. By updating this already known malware, attackers are creating new ways to exploit AD's features. TrickBot's new functions are a typical example of attackers finding a way to take advantage of AD's functionality. 

[LockerGoga](https://www.bankinfosecurity.com/hydro-hit-by-lockergoga-ransomware-via-active-directory-a-12207){:target="_blank"} ransomware is another type of malware pointing AD's weaknesses. The reason is that LockerGoga needs administrator privileges to run, and AD exploits make that possible. 

Organisations use network security measures, such as IPS/IDS, firewalls, antivirus, and more to combat malware. However, these techniques are not enough when it comes to AD and most of the times, organisations do not deepen the security of their systems, and as a result, they often fall victims to attacks. If adversaries can gain access to the AD system, they can move forward in accessing all connected user accounts, database systems, applications, and even compromising an entire Active Directory forest. 

Active Directory security is critical for organisations to protect their information systems, users' information, and data that is valuable for them. When the security of Active Directory is compromised, it can lead to potentially disastrous levels of data leakage and systems damage. Apart from that, the integrity and reputation of the organisations are also affected, which also play a significant role in an organisation’s survival. 

### An overview of Active Directory
**Active Directory** is a directory service that stores information about objects such as users, computers, groups, applications, printers and files in a multi-master database. This database is based on the **Joint Engine Technology (JET)** and is a file called **ntds.dit**. 

Objects stored in the AD database have a big number of attributes, discrete permissions and relations. IT administrators can manage objects' data on Domain Controllers (DCs), which can store almost 2 billion objects. This data is stored in a hierarchical organisational structure in AD, and users can use them to access resources from anywhere in the organisation's network. 

AD provides administrators with a centralised location for managing the authentication and authorisation of the organisational identities. The following sections introduce Active Directory's basic concepts, its structure and its role in an organisation. [1,2]

![Microsoft Active Directory](/assets/img/Active-Directory.jpg)*Microsoft Active Directory*

### Active Directory Functionality
Active Directory is built on top of Windows 2000 and is Microsoft's **Network Operating System (NOS)**. The term network operating system or "NOS" is used to describe a variety of resource types such as user, computer and group accounts. These resources are stored in a central repository, called Active Directory, which contains network operating system information. 

AD is controlled by administrators and is accessible to end-users.  Administrators and end-users can access this repository through a directory service called **Active Directory Domain Services (AD DS)**. 

AD DS and AD are often used interchangeably. Many different services, such as the Domain Name System (DNS), the Red Hat directory service or the email services, have different directory service characteristics. However, there are industry standards on how a directory service should be implemented and accessed. [1,2]

These standards are the **X.500 Directory Access Protocol (DAP)** developed by ITU Telecommunication Standardization Sector (ITU-T) in 1988, and its alternative, the **Lightweight Directory Access Protocol (LDAP)**, developed by the University of Michigan in 1993 as Request for Comments **(RFC) 1487**. 

In 1997, **LDAPv3 (RFC 2251)** was released as the final major update to the LDAP specification, and organisations such as Netscape, Sun, IBM and Microsoft started developing LDAP-based directory services. Microsoft released the first Active Directory version with windows 2000, and it is a part of the Windows Server OSs ever since. [1,2]

### Active Directory Logical Components
Every organisation has its hierarchical layout that can contain many different branch offices, groups of companies and departments, each of these having different operations. For the effective management of resources and security, the design of an organisation's identity infrastructure must match with its hierarchical layout.

The logical structure of AD consists of two types of objects: 
1. **Container objects** – can be associated with other objects.
2. **Leaf objects** – is the smallest component that may not contain any other child objects.

An object is just an entry stored in the AD file system. Data in AD is presented hierarchically. However, it is stored in a flat database made up of rows and columns. 

AD database uses a storage technology called [Extensible Storage Engine](https://en.wikipedia.org/wiki/Extensible_Storage_Engine){:target="_blank"} (ESE). It can index the data stored in the ntds.dit file extremely fast, due to the use of the record-oriented database architecture. According to [1], it can grow up to 16 Terabytes, holding over 2 billion records. 

Other Microsoft's applications such as Exchange, DHCP, FRS make use of the ESE, as well. JET Blue is another way to refer to ESE. During the installation process of a Domain Controller, the database is created under the **"C:\Windows\NTDS"** path by default. The ntds.dit file contains the schema table, the link table and the data table. One or more domains and domain trees are part of an AD forest, which represents a complete AD instance and is a security boundary for objects in AD [1, 2]. 

Domains can have their unique characteristics, boundaries and resources, but they share a common logical structure, schema and configuration inside a forest to comply with a two-way trust relationship.[1]

![Forest Trusts](https://docs.microsoft.com/en-us/project/images/adsyncforesttrust.jpg)*AD Forest Trust*

A domain is composed of the following components: 
1. A hierarchical structure of containers and objects
2. A unique identifier using the DNS domain name
3. A security service for authentication and authorisation to any resource access through accounts in the domain
4. Policies that dictate restricted functionality for users and machines 

Active Directory uses two [Naming Systems](https://docs.microsoft.com/en-US/troubleshoot/windows-server/identity/naming-conventions-for-computer-domain-site-ou){:target="_blank"}: the **DNS** (Domain Name System; also referred to as **Fully Qualified Domain Name or FQDN**) and the **NetBIOS (Network Basic Input/ Output System)**. Both can be used for computers, domains, sites and OUs.

A collection of domains in AD is called domain tree and reflects the hierarchical structure of an organisation. The domains that are inside a domain tree have a parent-child relationship, with the first domain inside the domain tree to be a parent (or root domain). The domain tree can have only one parent domain, and all other domains are child domains (or subdomains). A single domain can create a forest. Adding more domains in the domain tree does not create another forest. Domain trees and domains inside a tree that belong to a forest are utterly connected via a **two-way transitive trust**. 

[Forest trusts](https://docs.microsoft.com/en-us/azure/active-directory-domain-services/concepts-forest-trust){:target="_blank"} can be created and allow trust between domains that belong to different forests. In the two-way trust, the authentication requests between two domains can be processed in both directions. Transitive means that child domains (or subdomains) are also trusted without having a direct connection [1]. There is no trust connection between two separated forests by default, but it can be created in order to provide seamless resource access between them. 

Active Directory administrators are often called to apply similar security and administrative requirements onto objects. To do that, they use a particular type of container called [Organizational Unit](https://en.wikipedia.org/wiki/Organizational_unit_(computing)){:target="_blank"} or **OU**. There is also another type of container that is used for storing a hierarchy of objects called a "**Container**". 

One thing that makes OUs to be explicitly used for building object hierarchies is that an OU can have group policies applied to it. Every object assigned to an OU automatically inherits its security settings and permissions. 

Administrators also use OUs to delegate administrative control to objects according to the [Least Privilege Administrative Models](https://docs.microsoft.com/en-us/windows-server/identity/ad-ds/plan/security-best-practices/implementing-least-privilege-administrative-models){:target="_blank"}. OUs can contain other OUs, users, groups and computers.

### Active Directory Physical Components 

A **Domain Controller (DC)** is a machine that runs a **Windows Server** operating system and is the fundamental physical component in Active Directory. A DC holds the AD DS role as well as the ntds.dit database file that will be replicated across other DCs in the same domain using the **Microsoft Directory Replication Service (MS-DRS)** remote protocol. An organisation can have any number of DCs depending on its size, geographic area, and network, but only one DC is authoritative for one domain. [1,2]

The server that holds a full writable copy of objects in a forest and contains a subset of attributes for each of them is called **Global Catalog (GC)**. These attributes are members of the **Partial Attribute Set (PAS)**, and an AD administrator can add or remove them using the **AD Schema snap-in** or by the **attributeSchema** object. 

Administrators can access the Global Catalog via **LDAP** and **LDAP/SSL** over the ports **3268** and **3269** respectively. GC cannot be directly updated since it is read-only. 

Active Directory is a multi-master directory, which means that any writable DC can update database values in the domain. However, there are cases that only one DC (server) can perform certain functions for maintaining the integrity of the AD DS. [1,2]

**Flexible Single Master Operator (FSMO)** is the master server responsible for performing specific functions or roles. Five roles must be performed in an FSMO, where two of them apply to the whole forest, and the other three exist for each domain. [2]

These FSMO roles are presented below:
*	**Schema Master** – Applies to the entire forest and is necessary only when changes happened to the schema.
*	**Domain Naming Master** – Applies to the entire forest and is necessary only when domains and application partitions are added. 
*	**Primary Domain Controller emulator (PDC)** – Exists on the domain and is the most critical role holder since it is responsible for communication between legacy systems, the way trust paths are resolved, time sync, and password chain.
*	**Relative Identifier Master (RID)** – Each domain has a different RID Master as it is a domain level role. Its responsibility is to deal with RID pools. These pools include RIDs (Relative Identifiers) which are created for every object for security purposes.
*	**Infrastructure Master** – Domain-level. Maintains references to either deleted objects or linked attributes of objects living in other domains, also known as phantoms.

### Active Directory Identifiers

As explained in previous sections, almost two billion objects can be stored in the AD database. Therefore, there is a need to identify these objects uniquely. 
When an object is created in AD, depending on its type, it can be assigned with one or two values. In case the object is a User or a Group object, it will be assigned with a **Globally Unique Identifier** and a **Security Identifier**. [1] 

Using the PowerShell "**Get-ADUser username**" command, where the **username** is a user's real username, one can see the GUID and SID values, associated with the user account. 

![Get-ADUser](/assets/img/Get-ADUSer.png)*Object Identifiers*

The **GUID** is stored inside the attribute **ObjectGUID**. The **SID** is the attribute where the value of the SID is stored inside the object. Every object inside Active Directory will be assigned with the ObjectGUID, a 128-bit value, which is recognisable in the global scope as it is not restricted to a particular domain and stays there until the Administrator deletes the object. While administrators modify or move objects across the AD, the value of the GUID stays unaffected and is published to the GC server. Although there is a confusion that the GUID is a unique value, according to the documentation, the method used to create a GUID is complicated, so it is unlikely that there is a duplicate GUID. Unlike the GUID, the SID value of an object is unique to its domain. Its value for a given user object will change when that object is moved to another domain. A new SID value is generated, and the old SID value is stored in the **sIDHistory** attribute. [1,2]

SID is used for identifying a trustee or security principal and is a variable-length identifier. A SID ending in 500 represents the built-in administrator account in Active Directory. sIDHistory plays a vital role in AD restructuring because Access Control Lists (ACLs) rely on SID values to grant or deny access to domain resources to a user account. Therefore, a user object moved to a new domain without having sIDHistory values will lose its access to resources. However, when sIDHistory values moved with it, the user will be allowed to access their assigned resources. [1,2]

As mentioned earlier, the object's GUID is a value that uniquely identifies a user object. However, it is not based on the directory hierarchy, and it is difficult to remember its value. Therefore, there must be another way for uniquely identify objects within Active Directory, and that is called a **Distinguished Name (DN)** [1]. A DN can be compared to a postal address that uses a hierarchical route to locate a person, starting from the country, going to the county, then to the city, the street, and finally the house number. Distinguished Names use the syntax and rules of the LDAP standard to represent an AD object [1]. 

For example, in Object Identifiers figure, a path to the root is **DC= university**, **DC=local**. To generate DNs, three types of AD naming attributes are used [2]:
*	**organationalUnitName (OU)**, which represents the root domain. In other words, the OU where the object is located.
*	**domainComponent (DC)**, which represents the domain name and the DN attribute.
*	**commonName (CN)**, which refers to items and containers in the Directory.

According to Object Identifiers figure, when returning the query for the user account **dtsarouchas**, its distinctive name is "**CN = Dimitrios Tsarouchas, CN = Users, DC = university, DC = local". "DC = university, DC = local**" represents the domain name, "**CN = Users**" represents the user container, and finally "**CN = Dimitrios Tsarouchas**" represents the actual object name.

A **Relative Distinguished Name (RDN)** is a unique value referring to an object in its parent container. Taking the example in Object Identifiers figure, "**CN= Dimitrios Tsarouchas**" is the RDN for that user object. Many objects can have the same RDN under one condition; these items must be in separate containers. [1, 2]

In a later post, we will discuss Active Directory authentication mechanisms giving a deep explanation of how NTLM and Kerberos authentication protocols work. Stay tuned!

**References**

[1]  Desmond, B., Richards, J., Allen, R. and Lowe-Norris. (2013). Active Directory. 5th ed.
United States of America, CA: O’Reilly Media, chap. 1-2, 5, 9-10.

[2] Francis, D. (2017). Mastering Active Directory. Birmingham: Packt Publishing Limited,
chap. 1-2, 5, 10, 15.