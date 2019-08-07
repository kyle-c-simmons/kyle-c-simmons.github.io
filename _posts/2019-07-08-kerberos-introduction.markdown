---
title: "An Introduction to Kerberos"
description: A brief introduction to Kerberos.
date:   2019-08-07 16:19:00 +0100
layout: page
---
Kerberos is an authentication protocol used natively in Active Directory to authenticate users, hosts and services to the network. This is used to tell the network what resources or services clients are allowed to access.

**In simple terms Kerberos is:**
* A protocol for authentication
* Uses tickets to authenticate
* Tickets allow clients to use resources  

##### **<span style="color:#009588">How does Kerberos work?</span>**

**Kerberos consists of the following 3 components:**

* **KDC (Key Distribution Centre)** - This is used to handles tickets in Kerberos which is installed on the Domain Controller in Active Directory. It is responsible for assigning tickets to users or clients.
* **Client** - Client that wants to authenticate to the network using a Kerberos ticket.
* **SPN (Server Principal Name)** - SPN is used to offer services to the user in the network, for example allowing access to files on a server.

##### **<span style="color:#009588">Kerberos Tickets</span>**

**Ticket Granting Ticket (TGT)** - Tickets in Kerberos grants users access to resources in the network which is done by the TGT. The KDC will issue TGT requests to clients in the network. For example, lets just say you want to access a server with some files. However, you do not have access. To get access you must request a ticket by sending a TGT to the KDC server.

**This TGT request contains the following:**
* Username/name
* Length of the ticket is valid for
* IP address / machine name

**Ticket Granting Server (TGS)** – The TGS will handle granting tickets to clients, this will be in the KDC. Once a client sends a TGT to the server the ticket will be verified and the client will be granted a ticket and session key.  

<img border="1" width="80%" src="/assets/images/kerberos-diagram.png">

**A link to [Image source](https://www.manageengine.com/products/active-directory-audit/kb/windows-security-log-event-id-4768.html)**


##### **<span style="color:#009588">Kerberos in pentesting</span>**

Since Kerberos plays a big part of an Active Directory network and is used in a lot of networks, there is a lot of
research conducted on how to attack Kerberos.

**The following attacks are some common attacks used against Kerberos:**

* **Kerberoasting** - Cracking Kerberos service tickets (TGT) to get access to a service.
* **LLMNR Poisioning** - Capture usernames and hashes using Responder whcih can be cracked or used in Pass the Hash attacks.
* **Pass the Ticket** - Steal a users ticket and use it to authenticate to the network without a password.
* **Silver Ticket** - Using a user account to create a fake authenticated ticket to access services.
* **Golden Ticket** - Use the KRBTGT account (Golden ticket) authentication token to pass-the-hash on any account on the network.
