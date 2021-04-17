---
title: "Mr Robot"
date: 2021-04-14T22:08:08-03:00
draft: false
featured: true
toc: true
summary: "A trip down Mr Robot's themed room, can we root this one? join me and let's find out!"
images:
  - "img/mr_robot.jpg"
cover: "img/mrrobot_cover.gif"
tags:
  - hacking
  - tryHackme
  - writeups
  - mrrobot
  - robot
---

# THM - Mr Robot

After a long time, I'm back!. This time we'll hack through `Mr Robot` a THM room created by [Leon Johnson](https://twitter.com/@sho_luv) BTW he did an AWESOME job with this room, it is full of details and I loved it!. Check it out in the link below.


- Please visit This room on TryHackMe by [clicking this link](https://tryhackme.com/room/mrrobot).
- **PLEASE NOTE:** Passwords, flag values, or any kind of answers to the room questions were intentionally masked as required by THM writeups rules. 


## Starting off with some Scans

Last weekend we tried solving this one with my friends from Trojan Wave and we didn't get far due to lack of time mostly, but I promised I will look into this room and give it my best to solve it. And Today after a long time of not doing any CTF-style rooms and got back at it. Join me on the quest of hacking this awesome room!

Let's start by running an nmap scan and see what ports we have open to start hacking through this room.

`nmap -Pn -A -sV `

![](https://i.imgur.com/NXZubCo.png)

It seems we have ports 80 and 443 open, so let's take a look and see what website might be there:

![](https://i.imgur.com/4PVfKcm.png)

After loading the IP in the browser we can see there is some sort of terminal-like application that we can interact with. The different options available will display a set of videos and images, which for not don't seem to be of much use for us:

![](https://i.imgur.com/r9ifHKV.jpg)

The screenshot above is just an image from a clip that gets played when typing in the command `prepare`. Let's move on for now, let's run a GoBuster scan and see what other directories might this target contain:

`gobuster dir -u http://{target_ip} -w /usr/share/wordlist/dirbuster/directory-list-2.3-medium-txt -t 30`

![](https://i.imgur.com/51PDthn.png)

This scan is going to take a while, but right from the start some interesting directories are found. And we find out that we are dealing with a Worpress site as we can see some familiar directories: `/wp-content`, `/wp-login` and we also see `/robots` among several others.

Let's take a look at some of those while gobuster does its thing:

Starting off with `/wp-login` and see what we can see there:

![](https://i.imgur.com/JehAq0F.png)

Indeed we have a seemingly functional Wordpress login form, this is particularly interesting since we can try to attempt to get access into the worpress panel if we manage to login. Let's save this for later and continue exploring the other directories found by gobuster.

If we look at `/robots` we get something interesting:

![](https://i.imgur.com/WT83Ufg.png)

It seems that without much work we apparently got the first flag and one other file as well. If we load one of those files, we'll get the first flag for this room:

![](https://i.imgur.com/2VGQlkf.png)

Let's take a look at the second file we found. For that we can either download it through the browser itself or like I did using Wget:

![](https://i.imgur.com/0YdzpIp.png)

If we take a look at the contents of this second file we see a lot of words, it is a quite big wordlist it seems. However at this point we don't have any way of knowing if these are usernames, passwords or just a nice decoy setup by the room creator. However, it is our best lead so far and considering we do have a Wordpress login we might just have to try to use this list and brute-force our way into that Wordpress site.

But before we do that let's see if we can get any other lead, let's check if gobuster found anything else for us. Indeed it found a `/blog` that we can inspect. However, when we try to acess that we get an access denied error. There is also a `/dashboard` that redirects us to the same login form we saw before and there is a `/readme` with a nice message from the room creator:

![](https://i.imgur.com/XWk0pYG.png)

Also gobuster found another directory `/license` and we get the sense that we are being played XD:

![](https://i.imgur.com/Qu19LNt.png)

Ok, so I don't think we'll get much more out of Gobuster. But, there is one directory that seems out of place `/0000` when we load that we see this:

![](https://i.imgur.com/1mR8Ksd.png)

So accessing that we managed to get a partial access to the blog, let's see if we can browse around a bit. The idea behind this is to try to locate a username since we do have a wordlist already, but there is no sign of a username besides what we can probably guess based on the show itself.


Looking at the RSS feed is a dead end, there are no entries published apparently. Same for the comments RSS feed. And the blog title brings us back to the same terminal app we saw before.

Gobuster did found another path `/rdf` that when we access it, reveals something we might find useful, the Wordpress version:

![](https://i.imgur.com/pc17lpT.png)

Let's fire up `searchsploit Wordpress 4.3.1` and see if we get anything useful:

![](https://i.imgur.com/9E5wSZK.png)

Out of all the results, the one I'm interested right now is the first one, the user enumeration vulnerability. Let's see if we manage to run that exploit and get a hold of some usernames existing in this Wordpress installation:

To get a copy of that exploit script to our working directory we can just do `searchsploit -m 41497 ./`.

Now we open that exploit in nano and see what we need to change in order to run it:

![](https://i.imgur.com/s1hDQpb.png)

It seems we just need to specify the URL of the target. We make the change and save the file.

Let's run it and see if it works:

![](https://i.imgur.com/IkKIOSw.png)

It failed, so let's try another approach. Let's see if we can enumerate a valid user by using burp suite and the error messages we get when trying to log in into the site:

Wordpress used to have a bit of revealing error messages, if we try to log in with a valid username but we use an incorrect password we'll get an error along the lines of "The password you entered for the user XXXX is invalid". Meaning that the username we entered is correct but not the password. Let's see if we can exploit this using Burp intruder.

First we need to attempt a login and capture the request so we can use it for our enumeration attack:

Once we attempt a login and send that request to intruder, we clear the positions automatically selected by burp and just add a position to the username value:

![](https://i.imgur.com/RkFONqv.png)

For the payload list we'll use the file we found before, the worlist. Using Burp community will take a while, but we might have luck. The idea is to monitor the response length. We can assume that for all incorrect logins the same response message will be returned, therefore those will always have the same length. Whoever when one of the usernames is correct the message returned by the site will be different, thus its lenght.

To verify our theory we can inspect a couple of the requests sent by intruder and see that all of them so far, expose the same response:

![](https://i.imgur.com/7fxquiA.png)

After a bit we see the light at the end of the tunnel!

![](https://i.imgur.com/7fxquiA.png)

We found ourselves a username! with this we can now try to get access to the server. We can use a variety of tools for this, we could try Hydra, or we can leave Burp doing the same. If we wanted to use Burp or ZAP Fuzzer This time we'll have to modify the request so the username is fixed to the one we found and the password is the payload position that will be interating over the wordlist we also have!

Instead I'm gonna use my friend Hydra which it's usually faster than the approach mentioned above. Before running Hydra, it is wise to remove any possible duplicates the wordlist might have and that will also increase the speed of the attack.

To do that we simply run:

`sort fsocity.dic | uniq > sanitizedList`

With that ready we can fire up Hydra:

`hydra -l Elliot -P sanitizedList 10.10.72.42 http-post-form "/wp-login/:log=^USER^&pwd=^PASS^:The password you entered" -t 30`

Here we are using the file we found as the password wordlist but with all the duplicates removed, the username it's fixed to the one we found with Burp while enumerating through the login error message and we also use the error we get when trying to log with that username and an incorrect password. We also set the parallel tasks to 30 to speed things up, since the wordlist is still pretty huge.

![](https://i.imgur.com/SAQUXzK.png)


And that's it after a while we managed to get the password. Let's see if we can login with those:

![](https://i.imgur.com/ESWwsny.png)

Indeed we got access. Now let's get ourselves a revershe shell, or try to. From other Wordpress related rooms I've solved before I remember a rather quick way of getting a reverse shell by modifying the 404 template file. So let's do that.

First we need a netcat listener in our machine: `nc -nlvp 2112` using my favorite port (Go listen to Rush's 2112 album while you are at it).

Once that is running, I'll use [revshell.com](revshell.com) to quickly generate a PHP reverse shell. I've used this in the past and the one that works best for me is the `PHP PentestMonkey` reverse shell.

![](https://i.imgur.com/Rk2o5Y2.png)

With that revshell copied, we go into the Theme editor in wordpress:

Then we just simply select the 404 Template and replace the content for the reverse shell script:

![](https://i.imgur.com/r6unSMB.png)

Then we just click on the `Update File` button. Now we can navigate to any incorret/inexistent URL in the blog and that template will be loaded and our payload will be executed (hopefully):

![](https://i.imgur.com/6sscFzC.png)

and we get our shell:

![](https://i.imgur.com/fZz4Yag.png)

Finally we get a foothold in this machine. Now its time to go flag hunting. Let's see if we can get ourselves a better shell using python TTY:

`python -c 'import pty; pty.spawn("/bin/bash")'`

That works and now we have a more stable TTY shell. Let's browse around a bit and see what we can find:

After a bit of searching we do have two interesting files in the `home/robot` folder:

![](https://i.imgur.com/pMiRQ2H.png)

There is the second flag and a hashed password. The first one is out of reach with the current access level, but the hashed file we can access. So let's try to crack that one.

We copy the hashed file contents into our machine, and try to crack it using hashcat:

![](https://i.imgur.com/Gol64iy.png)

And that's it we got the password for Robot's account, So back to our reverse shell we swicht users:

![](https://i.imgur.com/Nd6lrIC.png)

With that we got the second flag. Let's hunt the last one now. If we check the sudoers permission of this user we see that it cannot run sudo at all. So Let's take the Linpeas approach. 

We download Linpeas in case we don't have it with: `wget https://raw.githubusercontent.com/carlospolop/privilege-escalation-awesome-scripts-suite/master/linPEAS/linpeas.sh`

Then we fire up a local HTTP server with Python on our attack machine: `sudo python3 -m http.server`

![](https://i.imgur.com/CK8Gk7V.png)

and now from the reverse shell session we get linpeas:

![](https://i.imgur.com/4xk4dKj.png)

now we do `chmod +x linpeas.sh` so we can execute it as a program and then run it:

![](https://i.imgur.com/4xk4dKj.png)

This will take a bit to run, so let's wait and see what we can find to escalate our privileges. After some minutes we get a nice lead:

![](https://i.imgur.com/QtG44Dy.png)

It seems `nmap` can run as root, and if we look into GTFObins.github.io there is a simple way to get root:

![](https://i.imgur.com/9EcOA9F.png)

So let's give it a try:

![](https://i.imgur.com/OYhMdvH.png)

it works!, now we have root access. Let's get ourselves the last flag and complete this room and writeup!

![](https://i.imgur.com/jUIKzOs.png)

And That's it for this room, the most time consuming part was trying to figure out the password for Elliot's account but once again Hydra prove faster than burp and ZAP for the task.

I hope you enjoy this room as much as I did, this was a really awesome room and I had a lot of fun while trying to hack it.

Happy hacking!

tzero86.


{{< thm_badge >}}