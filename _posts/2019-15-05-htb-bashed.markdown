---
layout: post
title:  "HackTheBox: Bashed - Writeup"
date:   2019-05-15 23:34:00 +0530
author: Kyle Simmons
categories: [htb]
---
HackTheBox: Bashed was more of a CTF style machine and is ranked as easy on HTB.

<img src="/assets/images/htb/bashed/bashed-htb.png">


<h2>1.1 - Scanning</h2>
<br>
I first started by doing a port scan of Bashed (10.10.10.68).
{% highlight shell %}
nmap -sV -sC -A -oA bashed-scan 10.10.10.68
Nmap scan report for 10.10.10.68
Host is up (0.026s latency).
Not shown: 999 closed ports
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Arrexel's Development Site
{% endhighlight %}

The only port open is port 80 from the nmap scan. Firstly, dirb and Nikto is executed to enumerate the server:

{% highlight shell %}

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

OUTPUT_FILE: dirb
URL_BASE: http://10.10.10.68/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612

---- Scanning URL: http://10.10.10.68/ ----
==> DIRECTORY: http://10.10.10.68/css/
==> DIRECTORY: http://10.10.10.68/dev/
==> DIRECTORY: http://10.10.10.68/fonts/
==> DIRECTORY: http://10.10.10.68/images/
+ http://10.10.10.68/index.html (CODE:200|SIZE:7743)
==> DIRECTORY: http://10.10.10.68/js/
==> DIRECTORY: http://10.10.10.68/php/
+ http://10.10.10.68/server-status (CODE:403|SIZE:299)
==> DIRECTORY: http://10.10.10.68/uploads/
{% endhighlight %}



<h2>2.1 - Exploitation</h2>
<br>
Dirb reveals several directories including one called `/dev`. When browsing this directory there is a php file called `phpbash.php`.

<img src="/assets/images/htb/bashed/shell-found.png">

When clicking the `phpbash.php` file a shell is opened and user is found:


<img src="/assets/images/htb/bashed/found-user.png">


<h2>3.1 - Post Exploitation</h2>
<br>
When entering the command `sudo -l ` it reveals that scriptmanager does not have any password. An attempt is made to switch to the `scriptmanager` user:

<img src="/assets/images/htb/bashed/change-user.png">

At the top of the hierarchy in `/` there is a `scripts` directory which contains a python script and is owned by `scriptmanager`. The script is called `test.py` and outputs to a `test.txt` file. This appears to be a cronjob. A python shell can be inserted into the `test.py` file to get a reverse shell.
<br>

A python reverse shell is insetted into the `test.py` file:


<img src="/assets/images/htb/bashed/echo-shell-to-python.png">

A `netcat` listener is open in the background waiting for the cronjob to run:

<img src="/assets/images/htb/bashed/nc-connection.png">


After a few minutes a shell is opened and success! Root access gained. The root flag can then be retrieved.
