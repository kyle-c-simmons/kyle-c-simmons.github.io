---
layout: post
title:  "HackTheBox: Optimum - Writeup"
date:   2019-05-04 23:24:02 +0530
author: Kyle Simmons
categories: [htb]
---
HackTheBox: Optimum was a fairly easy machine which involved exploiting CVE's to get a
reverse shell and system privileges.

<h2>1.1 Enumeration</h2>
<br>
I first started by doing a port scan of Optimum (10.10.10.8).
{% highlight shell %}
Nmap scan report for 10.10.10.8
Host is up (0.42s latency).
Not shown: 999 filtered ports
PORT   STATE SERVICE VERSION
80/tcp open  http    HttpFileServer httpd 2.3
|_http-server-header: HFS 2.3
|_http-title: HFS /
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Microsoft Windows Server 2012 (91%), Microsoft Windows Server 2012 or Windows Server 2012 R2 (91%), Microsoft Windows Server 2012 R2 (91%), Microsoft Windows 7 Professional (87%), Microsoft Windows 8.1 Update 1 (86%), Microsoft Windows Phone 7.5 or 8.0 (86%), Microsoft Windows 7 or Windows Server 2008 R2 (85%), Microsoft Windows Server 2008 R2 (85%), Microsoft Windows Server 2008 R2 or Windows 8.1 (85%), Microsoft Windows Server 2008 R2 SP1 or Windows 8 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
{% endhighlight %}
<br>
The port scan reveals that only 1 port is open (port 80). Several useful information including the version of the web server
is displayed which can be enumerated to check for a CVE.
<br><br>
A quick look at website shows that it is a web server for browsing files. The version is displayed `httpFileServer 2.3`, confirming again the version which was also shown in the nmap scan.

<img src="/assets/images/htb/optimum/hfs-website.png">
<br><br>

<h2>2.1 Exploitation</h2>
<br>
Using `searchsploit` to search for a vulnerability on the version, one is found for `hfs` which is remote command execution (RCE),
this can be used to get a reverse shell on the web server. The exploit is in metasploit which is booted up and the exploit is
exploited successfully.

<img src="/assets/images/htb/optimum/found-user-txt.png">
<br><br>
The user is then found easily, however the shell does not have root privileges which means further enumeration on the system
has to be conducted to get system access.

<h2>3.1 Post Exploitation</h2>
<br>
The first thing to check for is the system information, including architecture, operating system and versions. This can be done
by running the command `systeminfo`. In addition to that looking around the file system to
check what files is there. A windows local exploit finder can be used to gather possible vulnerabilities
that the system contains.

`Sherlock` is a powershell script which searches for possible vulnerabilities locally. Sherlock is downloaded from the
GitHub repository:
`https://github.com/rasta-mouse/Sherlock`

Once downloaded, i then uploaded the powershell scrip to the target using `meterpreter's upload` command.

<img src="/assets/images/htb/optimum/upload-sherlock.png">
<br><br>
Success, the powershell script was uploaded.
<br>

<h3>3.2 Post Exploitation - Running Sherlock</h3>
<br>
The powershell scrip is executed by doing:

`powershell .\Sherlock.ps1`

Several vulnerabilities are found with Sherlock after running the script. The exploits `MS16-034` and `MS16-135`
did not work for me, even after some changes to the code. However, after looking through the output of the script,
i found it is also vulnerable to the `MS16-098` exploit, which works! Searching the exploit on google, the exploit is
then downloaded and ready for transfer onto the target.

<img src="/assets/images/htb/optimum/2-vulns-found-with-sherlock.png">

The exploit is uploaded and the privileges are escalated. Then by navigating to the `Administrator`
directory, the root flag is found!

<img src="/assets/images/htb/optimum/priv-escalated-doneeee.png">
