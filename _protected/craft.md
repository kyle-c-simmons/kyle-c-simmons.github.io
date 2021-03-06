---
layout: protected
title:  "HackTheBox: Blue - Writeup"
date:   2019-04-23 23:24:02 +0530
author: Kyle Simmons
categories: [htb]
password: test
permalink: /crafty
---
HackTheBox: Blue was the simplest machine i have done, was easy to get into using a popular exploit.

<h2>1.1 Enumeration</h2>
<br>
I first started by doing a port scan of Blue (10.10.10.40).
```ruby
nmap -sV -A -sC -oA blue-scan 10.10.10.40
Nmap scan report for 10.10.10.40
Host is up (0.027s latency).
Not shown: 991 closed ports
PORT      STATE SERVICE      VERSION
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds Windows 7 Professional 7601 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
49152/tcp open  msrpc        Microsoft Windows RPC
49153/tcp open  msrpc        Microsoft Windows RPC
49154/tcp open  msrpc        Microsoft Windows RPC
49155/tcp open  msrpc        Microsoft Windows RPC
49156/tcp open  msrpc        Microsoft Windows RPC
49157/tcp open  msrpc        Microsoft Windows RPC
TCP/IP fingerprint:
OS:SCAN(V=7.70%E=4%D=4/22%OT=135%CT=1%CU=41398%PV=Y%DS=2%DC=T%G=Y%TM=5CBE15
OS:C7%P=x86_64-pc-linux-gnu)SEQ(SP=103%GCD=1%ISR=10D%TI=I%CI=I%II=I%SS=S%TS
OS:=7)OPS(O1=M54DNW8ST11%O2=M54DNW8ST11%O3=M54DNW8NNT11%O4=M54DNW8ST11%O5=M
OS:54DNW8ST11%O6=M54DST11)WIN(W1=2000%W2=2000%W3=2000%W4=2000%W5=2000%W6=20
OS:00)ECN(R=Y%DF=Y%T=80%W=2000%O=M54DNW8NNS%CC=N%Q=)T1(R=Y%DF=Y%T=80%S=O%A=
OS:S+%F=AS%RD=0%Q=)T2(R=Y%DF=Y%T=80%W=0%S=Z%A=S%F=AR%O=%RD=0%Q=)T3(R=Y%DF=Y
OS:%T=80%W=0%S=Z%A=O%F=AR%O=%RD=0%Q=)T4(R=Y%DF=Y%T=80%W=0%S=A%A=O%F=R%O=%RD
OS:=0%Q=)T5(R=Y%DF=Y%T=80%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=80%W=0
OS:%S=A%A=O%F=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=80%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1
OS:(R=Y%DF=N%T=80%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI
OS:=N%T=80%CD=Z)
```
Network Distance: 2 hops
Service Info: Host: HARIS-PC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: -2h37m23s, deviation: 34m36s, median: -2h17m24s
| smb-os-discovery:
|   OS: Windows 7 Professional 7601 Service Pack 1 (Windows 7 Professional 6.1)
|   OS CPE: cpe:/o:microsoft:windows_7::sp1:professional
|   Computer name: haris-PC
|   NetBIOS computer name: HARIS-PC\x00
|   Workgroup: WORKGROUP\x00
| smb-security-mode:
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode:
|   2.02:
|_    Message signing enabled but not required
| smb2-time:
<br>
The first interesting thing i see is that SMB on 445 is open with Windows 7 and service pack 1. This is likely vulnerable to
the NSA's EternalBlue exploit. Metasploit can be booted up to check if this service is vulnerable to EternalBlue or not.
<br><br>
<img src="/assets/images/htb/blue/scanner-eternalblue.png">
<br><br>

<h2>2.1 Exploitation</h2>
<br>
The scan revealed that MS17-010 can be used against the target. The exploit is loaded in Metasploit.
<br><br>
<img src="/assets/images/htb/blue/eternalblue-exploited.png">
<br><br>
The exploit is successful and a reverse shell is opened!

<h2>3.1 Getting the flags</h2>
<br>
A quick look and i have system privileges which means both the user and root flag can easily be retrieved.
<img src="/assets/images/htb/blue/system-priv.png">
<br><br>
Success! That was an easy machine to pwn.
<br><br>
