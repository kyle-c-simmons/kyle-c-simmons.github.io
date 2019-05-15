---
layout: single
title:  "HackTheBox: Nibbles - Writeup"
date:   2019-05-15 15:12:02 +0530
author: Kyle Simmons
categories: [htb]
---
HackTheBox: Nibbles is a easy 20 point machine on HTB which involved mostly exploiting a web server. I found
this machine quite fun and is one of my favourite HTB machine with the methods used to exploit it.


<img src="/assets/images/htb/nibbles/nibbles-htb.png">


<h2>1.1 - Scanning</h2>
I first started by doing a port scan of Irked (10.10.10.75).
{% highlight shell %}
nmap -sC -sV -A -T4 -oA nmap 10.10.10.75
Nmap scan report for 10.10.10.75
Host is up (0.036s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
{% endhighlight %}

The target is only running SSH and HTTP which suggests that the target of origin is the web server.

<h3>1.2 - Web server enumeration</h3>
Firstly as always, Dirb and Nikto and executed in the background. However, they did not reveal anything. When opening the website
in the browser a blank page with `Hello world` is found. A look at the source code gives us a CTF style hint:
<br><br>
<img src="/assets/images/htb/nibbles/nibbles-blog-found.png">
<br><br>

<h3>1.3 - Nibbleblog enumeration</h3>

Using searchsploit to find possible nibbleblog exploits:

<img src="/assets/images/htb/nibbles/possible-exploit.png">

A SQLi and file upload exploit is found. However, the version of the nibbleblog has not been enumerated yet so the exploits
may not work. Research is conducted to find out what nibbleblog is. A open source CMS github repo is found from some research:

[nibbleblog open source CMS github]

When browsing through CMS file system, a interesting file is found called `98-constants.bit`. This file displays the version of the CMS. When attempting to find the file on the target, the file is found with a version!


<img src="/assets/images/htb/nibbles/found-version.png">

Version: `4.0.3`<br>
Possible exploit: File upload (found via searchsploit)<br>
Exploit information found: [Fie upload manual exploitation]

The exploit found uploads a file and the extension of the file is not checked and can be uploaded, which can be used to get an RCE. However, it requires credentials to login.

<h3>1.4 - Credentials Enumeration</h3>
A web page of `update.php` is found which shows hidden `xml` files:

<img src="/assets/images/htb/nibbles/installphp-private-files.png">

When browsing both the files an email / username is found in the `config.xml` file which could be used to guess the password.

<img src="/assets/images/htb/nibbles/username-found.png">

The `<notifcation_email_to>` tag contains `admin@nibbles.com`. This suggest that one of the usernames is `admin`. Using these credentials attempts were made to attempt to login to the server. After some attempts with guesses login was successful!
<br><br>
Username: `admin`<br>
Password: `nibbles`

<h2>2.1 - Exploitation</h2>
The exploit can now be used to get RCE. The exploit says to visit the URL:
<br>
URL: http://10.10.10.75/nibbleblog/admin.php?controller=plugins&action=install&plugin=my_image
<br><br>
Before running the exploit, a shell is generated with `msfvenom`:


<img src="/assets/images/htb/nibbles/generate-shell.png">
<br><br>
metasploit multi handler setup:

<img src="/assets/images/htb/nibbles/multi-handler-setup.png">

<h3> 2.2 - Exploitation file upload</h3>
The shell is then uploaded via the My image page:

<img src="/assets/images/htb/nibbles/shell-uploaded-warning.png">

The shell has been uploaded and a multi-handler is waiting for the shell connection. Some enumeration is done to find the shell, the `/private` folder found on the `/update.php` contains file paths to the `my_image` plugin:

<img src="/assets/images/htb/nibbles/shell-found.png">

A reverse shell is now open! The user.txt can be retrieved.

<img src="/assets/images/htb/nibbles/shell-worked.png">


<h2>3.1 - Post Exploitation</h2>
I first started by enumerating the kernel and OS information:

<img src="/assets/images/htb/nibbles/linux-version.png">

Searchsploit found a vulnerability for the kernel:

<img src="/assets/images/htb/nibbles/exploit-found.png">

The exploit is then downloaded and can tested on the target to attempt to get root.

<h3>3.2 - Executing the exploit</h3>

`wget` is used to download the exploit ti the `/tmp` directory and then the exploit is executed:

<img src="/assets/images/htb/nibbles/priv-escalated.png">

Root access gained!

[nibbleblog open source CMS github]: https://github.com/dignajar/nibbleblog
[Fie upload manual exploitation]: https://packetstormsecurity.com/files/133425/NibbleBlog-4.0.3-Shell-Upload.html
