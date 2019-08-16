---
title: "An Introduction to Active Directory"
description: A brief introduction to Active Directory
date:   2019-08-05 19:45:00 +0100
layout: page
---
There is not many resources on Active Directory attacks online so i decided to make a blog series to contribute to the community. This will be a mini blog series on Active Directory penetration testing covering the basic to all the way to more advanced topics eventually.

##### **<span style="color:#009588">What is Active Directory?</span>**

Active Directory (AD) is a collection of services to manage a Windows network which runs on a Windows Server to manage a network, computers, services
and users. Active Directory stores data as objects such as services, users, groups and devices. This provides a way for administrators to grow and manage a large
network, typically used in large organisations.
<br>

##### **<span style="color:#009588">Active Directory services</span>**
**Active Directory provides several services to manage an Active Directory network which includes:**
* **AD DS: Active Directory Domain Services** - AD DS is a role used to manage information and resources on a network. This
includes the management of computers, users, services, network devices and applications.
<br>
* **AD CS: AD Certificate Services** - AD CS is Microsoft implementation of public key infrastructure (PKI) used to manage certificates
on an AD network. PKI is set of hardware, software, people, policies, and procedures needed to create manage, distribute, store and
revoke digital certificates.
<br>
• <b>AD FS: AD Federation Services</b> - AD FS is a way to shares resources with the outside network and allows users to
access resources on the outside network.
<br>
• <b>AD RMS: AD Rights Management Services</b> - AD RMS is a service used to protect data and restricted access to information.
<br>
• <b>AD LDS: AD Lightweight Directory Services</b> - AD LDS is a lightweight version of AD DS which provides flexible support for direcotry-enabled applications without the dependancy of domain restrictons of AD DS.
<br>
<h5>Active Directory Structure</H5>
A forest in AD is at the top of the hierarchy which contains domains, computers and users. Inside a forest the domains have
multiple Organisational Units (UO) which is used typically for group policies inside a domain. A domain may have 2 sub UO which are
the Admin team and HR. Both these UOs can have different group policies inside the domain.
<br>
<h5>Why learn Active Directory in Penetration testing?</h5>
A lot of organisations use AD to manage there internal network for organising the users, computers and files of a organisation. A majority
of large organisations use AD to run internal infrastructure. This is why AD penetration testing is important to understand
<br>
<h5>AD course and labs:</h5>
Several labs / courses that are recommended to learn AD pen testing are the following:<br>
• <b>HackTheBox:</b> Rasta and Offshore labs<br>
• <b>Pentester Academy:</b> Attack Defence and Red Team Labs:<br>
• <b>eLearn Security:</b> PTX course<br>
