---
title: "Poster_writeup"
date: 2020-11-17T20:50:13-05:00
draft: false
featured: true
toc: true
summary: "The sys admin set up an RDBMS in a safe way, let's play with it and hack our way in!"
images:
  - "img/poster_cover.jpg"
cover: "img/poster_cover.gif"
tags:
  - hacking
  - tryHackme
  - writeups
  - poster
---



# Poster - THM Room

Welcome to another writeup, this time we'll be trying to hack a newly released room on TryHackMe called Poster created by `stuxnet`! This is gonna be a really quick writeup (compared to my previous ones), since I'm supposed to make dinner in 1 hour :joy: so that's the timne we have for this one, just an hour. Let's try to hack this, shall we?

- Please visit This room on TryHackMe by [clicking this link](https://tryhackme.com/room/poster).
- **PLEASE NOTE:** Passwords, flag values, or any kind of answers to the room questions were intentionally masked as required by THM writeups rules. 


## Getting initial access

Let's use once again, `rustscan` to get a very-fast lay of the land.

![](https://i.imgur.com/bMxpt3q.png)

A short explanation on that command:

- `--ulimit` is to avoid any kind of performance issues by setting the recommended value of `5000`.
- `--` let us pass the next switches to `nmap` instead of sending them to `rustcan`, this way we can customize which type of scan we want to run.
- `-sV` we pass this to `nmap` so we can try to get the versions of the services running on each open port.
- `-sC` we pass this to `nmap` so it automatically tries the different scripts that are available within nmap.

If instead we want to use plain nmap to get the version of the RDBMS running (first room question), we can do it like this (port number discovered by rustscan):

![](https://i.imgur.com/GdT4034.png)

## Metasploiting away!

Now we need to leverage `metasploit` to enumerate user credentials. Let's fire up metasploit with the command  `msfconsole -q` (-q is optional, just starts msfconsole quietly):

![](https://i.imgur.com/805Xb6x.png)


Once we have the module we need identified, we select it by running `use #` where `#` is the actual number listed to the left of the module name.

Once selected we run `show options` to see which required values we need to `set` before the module can be executed. In this case it seems we just need to set `RHOSTS` to the ip of the target machine. `set RHOSTS {Target_IP}`, once that is set. We simply `run` the script:

![](https://i.imgur.com/t9ZiNtW.png)

Once it runs we get a successful login back, with that we answer another room question.


![](https://i.imgur.com/FYkkwzh.png)

Again, we run `show options` and set the required values, remember that we found a set of credentials before, let's set those too before running the module. In this case I've set `PASSWORD` and `RHOSTS` since `USERNAME` was already set correctly:

![](https://i.imgur.com/YiTsjJx.png)

With that we get another answer for the room's questions. Let's now move to the other one:

![](https://i.imgur.com/sH7dkv0.png)

Again, we locate the right module, set its options and then run it:

![](https://i.imgur.com/WR39NRo.png)

With the results we can answer the room question.

For the next one we just simply run `search postgre` and by just looking at the module names you'll get the answer:

![](https://i.imgur.com/kmNhXyi.png)

From the same search we get the next module name to answer the room's question. Now we need to exploit this machine and get the flags.

## Getting the user flag

![](https://i.imgur.com/5BJMBLC.png)

Once we set all required options and the password we found, we can run the module:

![](https://i.imgur.com/MQnC5kn.png)

We managed to get the initial access to the machine, if we look around we see where the `user.txt` flag is located, but we lack permissions to read it:

![](https://i.imgur.com/DWFZQJx.png)

Since we know there is a module that would allow us to read files from the system. Let's use that to read the contents of the `/etc/passwd` file:

> Remember to set RHOSTS, PASSWORD before running the module.

![](https://i.imgur.com/TkHxB5Z.png)

Ok, the interesting part of that file is the mention to another file called `credentials.txt`. Let's use the same module to read that file, we need to set `RFILE` to that file path we found:

![](https://i.imgur.com/QtFVOj6.png)

Now we got dark's credentials, and if we recall the results from rustscan we now there is a port 22 open. Let's try to login there: dark:qwerty1234#!hackme

![](https://i.imgur.com/vICq8Oa.png)

Even though we are logged in as another user now, we still don't have access to that flag.

Let's fire up a local HTTP python server where we have the `linpeas.sh` file ready and let's upload it to the remote machine using `wget`:

![](https://i.imgur.com/vbZKvoE.png)

If we start checking the results from linpeas, there are a couple of interesting things. However, there is one that seems particularly simple to check:
![](https://i.imgur.com/OoCdG1y.png)


Let's check that file:

![](https://i.imgur.com/20H0xZI.png)

It seems we got the password for `Alison` account, which means we can get that user flag:

If we switch user to alison we get access to the first flag:

![](https://i.imgur.com/eIukDqU.png)

## Getting the root flag

If we run `sudo -l` for the current user we see that we can run all commands as sudo. Nice, we can get the root flag now:

![](https://i.imgur.com/4fM2Dg7.png)


That's it, a very quick and entertaining room, we went a bit over the hour mark but we also produced another writeup that could help back a fellow hacker solve this room in the future. Totally worth it! I hope you enjoyed it!

Happy hacking!


{{< thm_badge >}}