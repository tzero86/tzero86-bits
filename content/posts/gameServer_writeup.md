---
title: "GamingServer_writeup"
date: 2020-10-10T22:07:40-04:00
draft: false
toc: true
featured: false
summary: "Can you gain access to this gaming server built by amateurs with no experience of web development and take advantage of the deployment system?"
images:
  - "img/gamingServer_cover.jpg"
cover: "img/gamingServer_cover.gif"
tags:
  - hacking
  - tryHackme
  - writeups
---

## GamingServer - THM Room

Welcome back! Today I have another writeup. This time I went for a just-released room on TryHackMe called `Gaming Server`. I hope you enjoy it!


- Please visit This room on TryHackMe by [clicking this link](https://tryhackme.com/room/gamingserver).
- **PLEASE NOTE:** Passwords, flag values, or any kind of answers to the room questions were intentionally masked as required by THM writeups rules. 


### Recon

I started a scan with `Autorecon` and while that is running let's look into what's at port `80`.

> `sudo autorecon 10.10.85.253`

If we browse to the IP address, we can see we do have a webpage being served about some game:

![](https://i.imgur.com/QOJf7kD.png)

And If we  open the console we see right away there is a mention to a possible username of `john`.

Let's see if we can find something else on that page. Autorecon is already running `gobuster` for us, so we should be able to look into those results in a bit.


If we look into `/about.php` again the console shows a commented piece of code for file uploads. 
Let's see if we can upload a `phpbash`. You can download a copy from [here](https://github.com/Arrexel/phpbash/blob/master/phpbash.php).


Uploading the `phpbash` file did not work, there was no error either. But when looking at `/uploads` the file wasn't there. It's very possible this is just a decoy. Let's keep looking.

There are some other interesting files: A hacker manifesto, a meme and a dictionary.

![](https://i.imgur.com/wp85FL4.png)


The mem is just an image of Beaker, who according to the internet is:

> Beaker is the hapless assistant to Dr. Bunsen Honeydew from The Muppets.

Gobuster found another interesting directory: `/secret`

![](https://i.imgur.com/lZ2z39X.png)

The secret key is an RSA Private one.
Let's see if we can ssh login as `John` using that key. To do that, we still need to crack the passphrase of the RSA key.


We need to use `ssh2john` so we can try to crack it with `john`:

```sh
┌──(kali㉿kali)-[~/Documents/THM/gamingServer]
└─$ python /usr/share/john/ssh2john.py secretKey > id_rsa.hash

```

Once that's done let's run john using the dictionary we found on the site as follows:

```
john --wordlist=dict.lst id_rsa.hash
```

That' works and we get the passphrase:

![](https://i.imgur.com/N7TVFqN.png)


Now we can try to ssh login as `john`:

![](https://i.imgur.com/rCHqiZp.png)

It works, we are in! yep! Let' me have this hacking cliche, just this time. :joy:

Once there we just do `cat user.txt` and we get the first flag.


### Getting root

Now let's see how we get root. First I fire up a simple http server with python to upload `linpeas.sh` to the target machine. And then run linpeas.


![](https://i.imgur.com/8TRAJcD.png)

> Remember to `chmod +x linpeas.sh` before trying to run it.


Linpeas results don't show anything that I know how to exploit, but while checking `id` command for the logged user something interesting comes up:


```sh
john@exploitable:/tmp$ id
uid=1000(john) gid=1000(john) groups=1000(john),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),108(lxd)

```

It seems John is part of the `ldx` group. And if I remember correctly I read something about getting root if you are part of that group. I think it was on `hacktricks`. Let's check [that](https://book.hacktricks.xyz/linux-unix/privilege-escalation/interesting-groups-linux-pe/lxd-privilege-escalation) out.



> Also check out this post about LXD which puts a nice context on this vulnerability. Follow [this Link](https://reboare.github.io/lxd/lxd-escape.html) to read it.


I tried the `With Internet` method first, by that didn't work. So let's try the `Method 2` and see if we get lucky this time:

We first need to create ourselves an image of Alpine, using the `lxd-alpine-builder` script located [here](https://github.com/saghul/lxd-alpine-builder).


We just need to run the builder script as sudo and it'll start making the image for us:

![](https://i.imgur.com/q9m8LRM.png)

Once done you'll see a compressed file on the directory where you built the image:

![](https://i.imgur.com/6s79Ctf.png)

now we need to upload the image to the target machine.

![](https://i.imgur.com/UFS2ZFC.png)

Now we need to let LDX know we have an image, import that image, add a device to it that mounts the file system with privileges and gives us root when we execute it:

![](https://i.imgur.com/LFhL5bO.png)


And since we mounted our files at `/mnt/root` we head that way to get the root flag:

![](https://i.imgur.com/D3YsaYS.png)

That's it for this room, quite an interesting one. First time for me doing anything related with `LXD` and Alpine. It was a good lesson and I really enjoyed it.

Hope you find this useful!


Happy hacking and thanks for stopping by!

{{< thm_badge >}}