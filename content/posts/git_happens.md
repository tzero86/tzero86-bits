---
title: "Git_happens"
date: 2020-10-08T21:39:14-04:00
draft: false
toc: true
featured: true
summary: "Boss wanted me to create a prototype, so here it is! We even used something called 'version control' that made deploying this really easy!"
images:
  - "img/git_happens_cover.png"
cover: "img/git_happens_cover.gif"
tags:
  - hacking
  - tryHackme
  - writeups
---

## Res - THM Room

Howdy! Welcome to `Git Happens` write-up. Let's see how we can compromise this room. I hope you enjoy it!


- Please visit This room on TryHackMe by [clicking this link](https://tryhackme.com/room/githappens).
- **PLEASE NOTE:** Passwords, flag values, or any kind of answers to the room questions were intentionally masked as required by THM writeups rules. 


### Scanning the machine

I start with `Autorecon` once again (referring to my last writeup). While it runs, let's check if we have any webpage or app being served at the IP address of the target machine.


We do get a login form.

![Login form](https://i.imgur.com/ujNWDPy.png)

Let's run `gobuster` as well.

![We run goBuster and Autorecon](https://i.imgur.com/yxpMN02.png)


While these two are working, let's see if we can find some information about git, since this room seems dedicated or based on it.

It is not crazy to assume this machine could have a git repository.

Turns out pubicly hosting your repository on a production website, by just cloning the repo, seems to be quite a bad idea. Long story short it seems git and other version control systems create a folder with a `copy` of the repository fo tracking. Meaning we can get full access to the code repository if the admin is not restricting access to that location.

> If this seems interesting, I suggest reading [this](https://en.internetwache.org/dont-publicly-expose-git-or-how-we-downloaded-your-websites-sourcecode-an-analysis-of-alexas-1m-28-07-2015/) article I consulted.

So let's see if we have access to a possible repository at `/.git` location:

![We do have access!](https://i.imgur.com/nMOPNSq.png)

Ok so far so good. Let's see what we can find by looking around the repo a bit. Mmm it seems we cannot just browse around this repo, since any link I click is downloading a file (that we can read though). 

The same article I pointed out earlier gives us a guide on what to try and do next. We need to try to download that repository so we can get our hands on it in a practical way.

Seems we can just use `wget` to get us a local copy of that repository. Let's see how we do that:

`wget --mirror -I .git {TARGET_IP}/.git/`

Let's run that:

![We start to download a copy of the repo](https://i.imgur.com/3akX2gJ.png)

If we navigate into the `cloned` repository in our local machine, now we can see all the information and browse around without issues:

![the copied repository folders](https://i.imgur.com/4Fl4oF0.png)

I think we just probably need to browse around for the `supersecret password` to solve the room. Let's give it a go then.

We can do a `git status` and we see some files were removed and can be recovered thanks to git version control.

![We can see deleted files in the repository and restore them](https://i.imgur.com/dOJM1v9.png)

### Let's compromise this repository


At this point we can keep manually looking for some clues or the supersecret key. But we need to do this in a better way, let's google a bit and see if we can use any tool to scan repositories for keys, hashes, etc.

I found [this](https://geekflare.com/github-credentials-scanner/) site that lists some tools that we can try out. As you might have noticed, I don't know yet how to exploit this room either. :laughing:

From the list two tools stand out to me, `Git Hound` and `Truffle Hog` I've seen mentioned around in some streams and videos but never really used them before.


I decided to try the `Truffle Hog`.

We run `pip install truffleHog` to install it, it seems.

Running it like this `trufflehog /Directory/.git` quickly returns quite a lot of results. Including a very interesting commit where a hash, username and algorithm used are mentioned:

![Intersting results](https://i.imgur.com/fniRgDE.png)


However at this `commit` in time, the password is already hashed. Let's see if we can find an older one.

There seems to be one that could be useful:

![](https://i.imgur.com/eOFL3CV.png)

Even though that commit is about a CSS file, we can assume it was styling the page we want to look into.

Let's `show` that commit by running `git show {Commit_hash_id}`. If you noticed each commit has this hash unique id. 

![we get the flag](https://i.imgur.com/1XrfFqr.png)

That's it if we look through that commit we can see the code where the password was hashed (in a later commit) it is unhashed in this one.

That's our flag!

That's it, I really hope you enjoyed this room like I did. Please leave a comment if you liked this writeup, or hated it. Or even if you ended up here by pure accident!

Happy hacking and thanks for stopping by!

{{< thm_badge >}}