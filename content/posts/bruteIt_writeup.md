---
title: "BruteIt_writeup"
date: 2020-11-14T22:26:26-05:00
draft: false
featured: false
toc: true
summary: "Learn how to brute, hash cracking and how to escalate privileges in this box!"
images:
  - "img/bruteIt_cover.jpg"
cover: "img/bruteIt_cover.gif"
tags:
  - hacking
  - tryHackme
  - writeups
  - bruteit
---

# Brute It - THM Room

Welcome to another writeup, this time we'll be trying to hack a newly released room on TryHackMe called Brute It created by `ReddyyZ`!

- Please visit This room on TryHackMe by [clicking this link](https://tryhackme.com/room/bruteit).
- **PLEASE NOTE:** Passwords, flag values, or any kind of answers to the room questions were intentionally masked as required by THM writeups rules. 


## Getting initial access

Let's run a `rustcan` to start things off:

![](https://i.imgur.com/OBpWIBs.png)


Right away, from rustscan results we get the answers to the first part of the room, specifically these questions:

> How many ports are open?
> What version of SSH is running?
> Which Linux distributions is running?

Now let's see if we can answer the following one:

> What is the hidden directory?

### Running GoBuster

Let's fire up gobuster and see if we can find that hidden directory:

![](https://i.imgur.com/yjWT9Ag.png)

Very quickly we get the hidden directory.

If we open the machine in our browser (I use Burp's suite integrated browser just in case I need Burp later) and navigate to the directory we found using `gobuster`, we see a login form.

![](https://i.imgur.com/JhF1O0K.png)

If we open the dev tools we see there are two users mentioned: `john` and `admin`. Let's kpep those in mind.

### Brute-forcing the login form

Let's see if we can bruteforce this form. First, we need to check how it responds when we attempt to login:

![](https://i.imgur.com/UOFHGOF.png)

From that attempt we get all we need to try to brute-force our way in. 

Let's ready our `hydra` command:

`hydra -l <username> -P <wordlist> 10.10.0.255 http-post-form "/admin/:user=^USER^&pass=^PASS^:F=Username or password invalid" -V`

NOTE: we could also do -L and provide a list of usernames with both `admin` and `john`. As for the wordlist I'll try with `rockyou.txt`. 

Once ready we launch the command:

![](https://i.imgur.com/5MPHfCJ.png)

This might take a while but we'll get the password. In `hydra` we trust :joy:

Let's log in with the credentials we now have:

![](https://i.imgur.com/ngiXffJ.png)

We get the web flag for the room and a link to a RSA private key:

![](https://i.imgur.com/xV47qZJ.png)

### Cracking the RSA key

Now we need to crack that key so we can hopefully get the initial access to this machine.

We need to grab that plain text key and save it into a file so we can use `john` to try and crack it. Before that we need to use `ssh2john` to generate a file that can be cracked by `john`.

![](https://i.imgur.com/c6iqz8X.png)

Now we can run john:

![](https://i.imgur.com/drW5YLk.png)

and we get the hash cracked.

Now let's try to connect as `john` with that key we've got:

> *NOTE:* Remember we need proper permissions set for the key to work, so do `chmod 600 id_rsa` before tryin to log in with the key.

![](https://i.imgur.com/lJwjB65.png)

Ok we managed to get the initial access. Off to get the flags now.

## Getting the user.txt flag

For this flag we just need to list the files in the directory, and then cat the user.txt file:

![](https://i.imgur.com/jOCr5ju.png)

That's the user flag.

## Getting the root.txt flag

Now we need to find a way to scalate privileges to root so we can get the last flag. Let's fire up a python  HTTP local server in a folder where we get the `linpeas.sh` ready. Then from the target machine we'll `wget` that file so we can do some enumeration.

![](https://i.imgur.com/HP86rN7.png)


> NOTE: Remmeber to run `chmod +x linpeas.sh` before running it.

Right away we see the user john seems to have permissions to run as sudo, let's check that out:

![](https://i.imgur.com/jpUIqBB.png)


If we run `sudo -l` we can see which commands we can run as sudo:

![](https://i.imgur.com/fSH27zC.png)

It seems we can run `cat` as sudo, we can probably just try to cat the root.txt flag if it is in the usual location `/root/root.txt`:

![](https://i.imgur.com/V594hIO.png)

That's it, we got the flag. However, we are not done yet. We need to find the password for the `root` account. So let's see how we can get root from `cat` by searching in `GTFObins` page:

![](https://i.imgur.com/uTQ8Q0J.png)

So let's try to get the `/etc/shadow` file so we can potentially get the root password:

![](https://i.imgur.com/AhoK6xj.png)

Ok we have the password hashes, let's try to crack them now:

It seems the password are using `SHA512crypt`. But instead of trying to crack that we can leverage the `cat` command again and get the `/etc/passwd` file two, so we can then use the `ushadow` tool to get the password:

![](https://i.imgur.com/7Fhx3h4.png)

Now having both files we can just use `unshadow` to get the password:

![](https://i.imgur.com/2XUHJZQ.png)

Now we need to run `john` again on that passwords file and we should be golden:

![](https://i.imgur.com/h5plnF0.png)


That's it we got the last answer to fully solve this room!

I hope you enjoy the room, props to `ReddyyZ` for creating it!


Happy hacking!



{{< thm_badge >}}



















