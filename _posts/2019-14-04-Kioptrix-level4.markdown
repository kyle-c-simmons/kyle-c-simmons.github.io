---
layout: post
title:  "Kioptrix Level 4 Walkthrough"
date:   2019-04-16 11:00:02 +0530
author: Kyle Simmons
categories: [vulnhub, vulnerable-machine]
---
Kioptrix level 4 vulnhub link below: <br>
[Kioptrix Level 4 download page]
<br><br>
The goal is to get root access and retrieve the flag in the 'root' directory.
<h2>1.1 Enumeration</h2>
A port scan reveals that 4 ports are open (22, 80, 139 and 445).
{% highlight shell %}
nmap -T4 -sV -A -oA nmap-scan 192.168.0.73
Nmap scan report for 192.168.0.73
Host is up (0.00027s latency).
Not shown: 566 closed ports, 430 filtered ports
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1.2 (protocol 2.0)
ssh-hostkey:
1024 9b:ad:4f:f2:1e:c5:f2:39:14:b9:d3:a0:0b:e8:41:71 (DSA)
2048 85:40:c6:d5:41:26:05:34:ad:f8:6e:f2:a7:6b:4f:0e (RSA)
80/tcp  open  http        Apache httpd 2.2.8 ((Ubuntu) PHP/5.2.4-2ubuntu5.6 with Suhosin-Patch)
Host script results
clock-skew: mean: 1h59m59s, deviation: 2h49m42s, median: 0s
nbstat: NetBIOS name: KIOPTRIX4, NetBIOS user: (unknown), NetBIOS MAC: (unknown)
| smb-os-discovery:
|   OS: Unix (Samba 3.0.28a)
|   Computer name: Kioptrix4
|   NetBIOS computer name:
|   Domain name: localdomain
|   FQDN: Kioptrix4.localdomain
|   System time: 2019-04-15T15:34:29-04:00
| smb-security-mode:
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|   message_signing: disabled (dangerous, but default)
|smb2-time: Protocol negotiation failed (SMB2)
{% endhighlight %}
<br>


<h3>1.2 SSH</h3>
SSH appears to not have any vulnerable versions. However, SSH could possibly have weak credentials.
<br><br>

<h3>1.3 HTTP Login with SQLI</h3>
The login page could potentially be vulnerbale to SQL Injection. To check for SQLI the following is entered:
<br><br>
Username: <font color="red">admin</font><br>
Password: <font color="red">1' or 1 = 1#</font>
<br>
<img src="/assets/images/Kioptrix/Kioptrix4-goat-login.png">
<br>
Login successful!

<h3>1.4 HTTP Testing for LFI</h3>
The URL has <font color="red">username=Admin</font>. LFI is tested here. However,
each file says 'permission denied'. This appears to be a dead end.<br><br>
<img src="/assets/images/Kioptrix/Kioptrix4-lfi-error.png">
<h3>1.5 SMB Enumeration</h3>
Enum4linux is run to attempt to enumerate users:
<br><br>
<img src="/assets/images/Kioptrix/Kioptrix4-enum-users.png">
<br><br>
Several users are revealed:
<font color="red">
-> nobody<br>
-> robert<br>
-> root<br>
-> john<br>
-> loneferret
</font>
<br>
The users can be further tested on the website and SSH.

<h2>2.1 Exploitation</h2>
An attempt is made to perform a dictonary attack over SSH against these usernames
but that did not work. After that an attempt is made to login with the users enumerated
on the login page. The 'john' user shows the password in plaintext! With a password of
'MyNameIsJohn'.
<br>
Since SSH is open a login attempt with SSH using the credientals collected is done
which is successful! However, the shell appears to be low privileged with few commands.
<br><br>
<img src="/assets/images/Kioptrix/kioptrix4-ssh-lowprivuser.png">
<br><br>
After some research, i found a way to escape a restriced shell using the echo command to execute the following:
<br>
<font color="red">echo os.system('/bin/bash')</font>
This successfully invokes a bash shell with the user id of 'john'.
<br><br>
Found on this link: [Escaping/Bypass From Jail/Restricted Linux Shells]
<br><br>

<img src="/assets/images/Kioptrix/kioptrix4-priv-escalated.png">

<h2>3.1 Post Exploitation</h2>
I then changed directory to the root and found a 'congrats.txt' file which
contained the flag!<br><br>
<img src="/assets/images/Kioptrix/kioptrix4-got-flag.png">
<br><br>
When looking around the file system a SQL username and password is found in
the '/var/www' web server files. The login php file contains the following:
<br><br>
<img src="/assets/images/Kioptrix/kioptrix4-sql-login.png">
<br>
Username: <font color="red">root</font><br>
There is no password for root.
<br><br>
When logging the mysql i was easily able to enumerate the databases.
<br><br>
<img src="/assets/images/Kioptrix/kioptrix4-mysql-login.png">
<br><br>
The databses are:
<br>
<font color="red">
-information_schema<br>
-members<br>
-mysql
</font>
<br>
After further enumeration of mysql, i was able to find the user accounts for the
usernames 'john' and 'robert'.<br><br>
<img src ="/assets/images/Kioptrix/kioptrix4-sql-enumeration.png">
<br><br>
<h4>Success!</h4>The flag has been retrived and the mysql database has been enumerated.



[Kioptrix Level 4 download page]: https://www.vulnhub.com/entry/kioptrix-level-13-4,25/
[Escaping/Bypass From Jail/Restricted Linux Shells]: https://ud64.com/ask/61/escaping-bypass-from-jail-restricted-linux-shells
