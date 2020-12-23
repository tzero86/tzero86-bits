---
title: "All_In_One_writeup"
date: 2020-12-22T19:37:05-05:00
draft: false
featured: true
toc: true
summary: "This is a fun box where you will get to exploit the system in several ways. Few intended and unintended paths to getting user and root access."
images:
  - "img/aio_cover.png"
cover: "img/aio_cover.gif"
tags:
  - hacking
  - tryHackme
  - writeups
  - poster
---

# THM - All in one

This time we'll hack through `All in One` a recently released THM room created by `i7m4d`. Check it out in the link below.


- Please visit This room on TryHackMe by [clicking this link](https://tryhackme.com/room/allinonemj).
- **PLEASE NOTE:** Passwords, flag values, or any kind of answers to the room questions were intentionally masked as required by THM writeups rules. 


## Enumeration
Let's start by running some scans on the target, I'm feeling like using `Rustscan` once again. To start things up we run the following command:

`
┌──(kali㉿kali)-[~/Documents/THM/allInOne]
└─$ rustscan -u 5000 -a 10.10.139.211 -- -A -sV -sC
`

![](https://i.imgur.com/CkY8SWV.png)

Let's also fire up `gobuster` to see if we can identify any directories on this server that we can access:

`
┌──(kali㉿kali)-[~/Documents/THM/allInOne]
└─$ gobuster dir -u http://10.10.139.211:80 -w /usr/share/wordlists/dirb/big.txt 
`

![](https://i.imgur.com/iM65FM3.png)

We can already see something insteresting, the server is running `Wordpress`. Let's see what we find at that URL:

![](https://i.imgur.com/jFavizB.png)

We get a home page so we can access that Wordpress site after all. Let's recap all the information we got so far:

* Open Ports: `21`, `22` and `80`.
	* Port 21: Accepts `anonymous login`. But there is nothing there to see, unless I missed something :joy:.
	* Port 22: `SSH` running `OpenSSH 7.6p1`.
	* Port 80: `Default Apache` page.
* CMS: `Wordpress` installation at `/wordpress`.
* Username Mentioned: If we look at the welcome page post we see a potential username mentioned, called `elyana`.


## WPscan

Now let's see what vulnerabilities we can potentially identify for that Wordpress installation. Let's run `WPscan` and see if it finds any. To do that, we run the command as follows:

`
┌──(kali㉿kali)-[~/Documents/THM/allInOne]
└─$ wpscan --url http://10.10.139.211:80/wordpress --no-banner --api-token {YOUR_WPSCAN_API_KEY_HERE}
`

![](https://i.imgur.com/xZke5jT.png)

Once WPScan finishes the scan, we get a long list of results. First we get 8 vulnerabilities for the version of Wordpress itself:

![](https://i.imgur.com/rjno7p1.png)

> NOTE: For the sake of space I won't be listing all vulnerabilities on the screenshots. WPScan results are usually very lenghty.

We also get a couple of vulnerabilities for a Wordpress Plugin called, `mail-masta`:

![](https://i.imgur.com/QgMb4Zq.png)

Having all these vulnerabilities listed clearly means we could potentially get access to this target in various ways. In the next section we'll start looking into how we can actually abuse some of these vulns to get us the initial access to the target.

## LFI (Local File Inclusion)

The first thing I would probably want to see if we somehow can confirm the username we gathered before is actually a user in the server. Lucky us, we do have a vulnerability which allow us to perform an LFI. Which will basically let us read files that are in the server. Could we even manage to get the `/etc/passwd` file where the users are listed?

First we need to know where to look for more details on this vulnerability in particular in order to get familiar with how it works. If we observe the results provided by WPScan, for the LFI vuln in particular. We see a series of links that could be useful, in particular the one from exploit-db.com. 

Let's check that link out:

![](https://i.imgur.com/Cd9TLLF.png)

There are several ways of exploiting this LFI, we'll try the very first one. According to the report it should let us perform it without any authentication needed. We just need to use the sample URL we got and replace the file we want to read. Let's see this working on the following screencapture:

![](https://i.imgur.com/Eze7xEl.png)

As you can see we were able to exploit this plugin vulnerability and confirm that indeed `elyana` is a user in the system. Shall we try to crack the password for this user? I think so.

## Attempting brute-forcing with Hydra

We'll try to use Hydra to crack the password of the user `elyana`, for that we'll need some things before we can perform the attack. In Wordpress, unless changed by the admin, the login panel can usually be accessed by navigating to the `/wp-admin/` URL, let's see if that's the case this time too:

![](https://i.imgur.com/3F2s6QI.png)

It works, awesome. So we'll by trying to log-in as `elyana` and using `burpSuite` or the browser `dev-tools` to inspect the login request to see which type of request we are sending and how it is formatted. We'll use this information along with the response from the server, to setup our attack in `hydra`.

So we just need to enter the username, anything as the password and with the dev-tools (or burp) open inspect the request and response:

![](https://i.imgur.com/0Yqhi2I.png)

From this request we need the following details to feed them to Hydra:

- Request Method: `POST`
- Request URL: `http://10.10.145.252/wordpress/wp-login.php`
- Username and Password parameters: `log` and `pwd`.
- Error Message: `Error: The password you entered for the username elyana is incorrect`.

Now let's use this to craft our attack command using hydra:

`
hydra -l elyana -P /usr/share/wordlists/rockyou.txt 10.10.145.252 http-post-form '/wordpress/wp-login.php:log=^USER^&pwd=^PASS^:F=incorrect'
`

![](https://i.imgur.com/QiojZxR.png)

This is taking too much time, and we don't even have any certainty that the password is indeed included in the wordlist we used. We need to find another approach.

If we go back the LFI vuln, we can try to get another file that could point us in the right direction. In Wordpress there is a general config file called `wp-config.php` where the Database password is stored. Rarely but it happens, users tend to reuse the same password for the user accounts. Let's see if we can get that file with LFI in a similar fashion as we did for the passwd file.

If we visit the IP of the target on the browser, we'll get the default apache page. This page mentions something useful for us: 

![](https://i.imgur.com/BbD7feJ.png)

If we think about it, we can assume the Wordpress installation should be at the following path: `/var/www/html/wordpress/` and the wp-config.php file it's usually in wordpress root folder. Let's see if we can read that out, for this let's try with another version of the LFI that we saw on exploit-db. After reviewing the exploit a while none of the LFI variants worked to get the file. I did a bit of googling around and found a mention to something called a PHP Filter that could help us get the encoded version of the file we want. The LFI command using the PHP filter will look like this:

`/wp-content/plugins/mail-masta/inc/campaign/count_of_send.php/?pl=php://filter/convert.base64-encode/resource=/var/www/html/wordpress/wp-config.php`

Let's try it out:

![](https://i.imgur.com/EinY5vw.png)

Now we need to decode this huge text we got back, we can use `CyberChef` for that, let's copy the text and decode it (base64) using cyberchef:

![](https://i.imgur.com/xJGnX9o.png)

As you can see the DB username and password are in the file, let's see if we can log in as `elyana` using the password mentioned in that file we just decoded:

![](https://i.imgur.com/JyaNBtC.png)


It worked! We have access to the Admin panel of wordpress, which means from now on we could try the other vulnerabilities to get ourselves access to the server and get the flags.


## Abusing WP Themes

One pretty common way of getting a reverse shell back to our box is by abusing the Theme Editor feature of wordpress to modify a theme file and have it include a PHP reverse shell. We've seen this already in another room that I can't quite remember, but should be in one of my previous writeups here in the blog. Let's quickly see if we can do that.

We need to navigate within the WP Admin panel to this location:

![](https://i.imgur.com/B4FBmJx.png)

Once there we need to add the reverse shell code, we'll use this varian that should be supported. You can get this reverse shell at the following [link](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php).

We just replace the code for the 404 template with the rev shell code:

![](https://i.imgur.com/5dcJhm1.png)

> NOTE: Make sure to replace the placeholder values for IP and PORT. To target your attack machine (Tryhackme VPN ip, usually tun0)

Then we just need to update the file we changed, and call it so we get our reverse shell connection:

> NOTE: Remember to fire up your netcat listener, in my case: `nc -nlvp 2112`

Now we just need to invoke the `404.php` page by navigating to for example, a non-existing post. Let's see if it works:

![](https://i.imgur.com/XsghWXM.png)

It works! We got the initial access, finally!

## Getting the user.txt flag

Even though we have an initial access into the target, we still don't have enough permissions to read the user.txt file. We need to find a way to escalate privileges or even get a hold on the password for elyana.

![](https://i.imgur.com/9Im50fy.png)

If we check Elyana's user directory, we see a file called hint.txt that we can view. It seems we are gonna have to find the password. Since the file could have any name, the only real piece of info we have is the username of `elyana`. Let's use the `find` command to hopefully get the files that contain that username:

![](https://i.imgur.com/AOm9LKh.png)

We got the password, let's see if we can SSH using that:

![](https://i.imgur.com/M5FuA1d.png)

We got the flag, however it seems encoded and looks like base64 again. Let's pipe the output of cat to base64 utility to decode it:

![](https://i.imgur.com/Fo0whRu.png)
That's it we got the first flag.


## Getting the root.txt flag

Now that we have the initial access we need to get root, let's fire up a local python http server in our machine where we have downloaded `linpeas.sh` (Check this [link](
https://raw.githubusercontent.com/carlospolop/privilege-escalation-awesome-scripts-suite/master/linPEAS/linpeas.sh) to get it). Having that ready, let's upload linpeas to the target machine and run it:

> NOTE: You need to `chmod +x linpeas.sh` before running it!

![](https://i.imgur.com/Sqi99qd.png)


Once we get the results, we are lucky linpeas did something we forget. To check if the user can run any command as sudo:

![](https://i.imgur.com/ZNsPOU5.png)

If we check GTFObins.github.io and search for that binary, we get this:

![](https://i.imgur.com/UVlA7Hl.png)

So it seems we can fire up a bash/sh shell that even though will be a bit broken since the page tells us it won't be a full TTY shell. We could still try to use it to read the root flag:

![](https://i.imgur.com/NWm1gAC.png)
We did it, the password was again encoded so we use the same command as before. We cat the file and pipe the output into the base64 utility.

This is a really awesome room, there are so many other ways we can get root on this box. I won't be covering them all since this is the path I took to solve it. I usually do my writeups at the same time I'm going though the lab so, that's it for this time. This is my way of solving this box.

I would still look into the other vulnerabilities to learn more, but won't be part of this writeup. I hope you enjoyed the room, kudos to `i7m4d` for creating it!

Happy hacking!

{{< thm_badge >}}