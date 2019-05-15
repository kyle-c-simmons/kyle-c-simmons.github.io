---
layout: single
title:  "HackTheBox: Grandpa - Writeup"
date:   2019-05-15 20:30:02 +0530
author: Kyle Simmons
categories: [htb]
---
HackTheBox: Grandpa is a similar machine to Granny on HTB. Both the machines can be used for the same attack origin. However,
i've done this one different to Granny to practice metasploit more. Process migration was used in
this machine to migrate an exploit to another process.


<img src="/assets/images/htb/grandpa/grandpa-htb.png">


<h2>1.1 - Scanning</h2>
I first started by doing a port scan of Grandpa (10.10.10.14).
{% highlight shell %}
nmap -T4 -A -oA nmap -p- 10.10.10.14
Nmap scan report for 10.10.10.14
Host is up (0.034s latency).
Not shown: 65534 filtered ports
PORT   STATE SERVICE VERSION
80/tcp open  http    Microsoft IIS httpd 6.0
| http-methods:
|_  Potentially risky methods: TRACE COPY PROPFIND SEARCH LOCK UNLOCK DELETE PUT MOVE MKCOL PROPPATCH
|_http-server-header: Microsoft-IIS/6.0
|_http-title: Under Construction
| http-webdav-scan:
|   Allowed Methods: OPTIONS, TRACE, GET, HEAD, COPY, PROPFIND, SEARCH, LOCK, UNLOCK
|   Public Options: OPTIONS, TRACE, GET, HEAD, DELETE, PUT, POST, COPY, MOVE, MKCOL, PROPFIND, PROPPATCH, LOCK, UNLOCK, SEARCH
|   Server Date: Tue, 07 May 2019 09:51:49 GMT
|   Server Type: Microsoft-IIS/6.0
|_  WebDAV type: Unkown
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2003|2008|XP|2000 (91%)
OS CPE: cpe:/o:microsoft:windows_server_2003::sp1 cpe:/o:microsoft:windows_server_2003::sp2 cpe:/o:microsoft:windows_server_2008::sp2 cpe:/o:microsoft:windows_xp::sp3 cpe:/o:microsoft:windows_2000::sp4
Aggressive OS guesses: Microsoft Windows Server 2003 SP1 or SP2 (91%), Microsoft Windows Server 2008 Enterprise SP2 (91%), Microsoft Windows Server 2003 SP2 (91%), Microsoft Windows 2003 SP2 (90%), Microsoft Windows XP SP3 (89%), Microsoft Windows 2000 SP4 (86%), Microsoft Windows XP (86%), Microsoft Windows Server 2003 SP1 - SP2 (85%), Microsoft Windows XP SP2 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
{% endhighlight %}

The target only has port 80 running Microsoft IIS 6.0 which is an old version of IIS and is vulnerable to RCE. This machine did not require much enumeration since it is a known vulnerability.


<h2>2.1 - Exploitation</h2>
Metasploit is launched and `IIS` is searched for to find the exploit:

<img src="/assets/images/htb/grandpa/exploited.png">

The exploit is executed and a meterpreter shell is opened! Very easy start.

<h2>3.1 - Post Exploitation</h2>

Local exploit suggester is a metasploit recon module to find local exploits on an active session. It is executed on the meterpreter session opened:

<img src="/assets/images/htb/grandpa/local-exploit-suggester.png">

Several exploits returned, after trying several of them the `ppr_flatten_rec` exploit worked with some changes. When executing the exploit it returns a error:

<img src="/assets/images/htb/grandpa/local-exploit-error.png">


This error is a problem with the process. Meterpreter injects the code into active processes to stay hidden. However, in this case there was a problem with the process permissions. To fix this the exploit is then migrated to another process:

<img src="/assets/images/htb/grandpa/migrate-proccess.png">

Once migrated, the exploit is executed and works correctly:

<img src="/assets/images/htb/grandpa/exploited-local.png">

The root flag can then be retrieved!
