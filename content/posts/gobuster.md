---
title: "Gobuster_basics.sh"
date: 2020-08-15T21:25:53-04:00
draft: false
toc: false
images:
cover: "img/go_buster_cover.png"
tags:
  - hacking
  - tools
---

#GoBuster Basic usage#
----------------------

GoBuster brute-force tool for URIs (directories and files), DNS subdomains and virtual host names. 

* **Get GoBuster**: If you need to download this tool to your system, [visit this link](https://github.com/OJ/gobuster) 

To start with the basic usage for this tool, let's review some of the flags we can specify to run gobuster.

{{< table >}}
| GoBuster flag         | Description                                    |
|:---------------------:|:----------------------------------------------:|
| **-e**                | Print the full URLs in your console            |
| **-u**                | The target URL                                 |
| **-w**                | Path to your wordlist                          |
| **-U** and -P         | Username and Password for Basic Auth           |
| **-p {proxy}**        | Proxy to use for requests                      |
| **-c {http cookies}** | Specify a cookie for simulating your auth      |
{{</table>}}



GoBuster requires us to provide a wordlist in order to brute force our way through. If you are using Kali linux you most likely already have several wordlists available in the _**/usr/share/wordlists/dirbuster/**_. If you are not, you'll be lucky just by googling for some.

Now assuming we already have a worlist available, we can start our basic brute-force attack:

```shell
 gobuster dir -u http://{target_IP}:{target_port} -w {wordlist_Path}
```

it is worth noting you can specify a host url and not only IPs. With this in mind a basic full example would look like this:


```bash
 gobuster dir -u http://127.0.0.1:80 -w /home/oj/wordlists/shortlist.txt
```

And it'll produce and output similar to this:

```bash
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Mode         : dir
[+] Url/Domain   : https://127.0.0.1/
[+] Threads      : 10
[+] Wordlist     : /home/oj/wordlists/shortlist.txt
[+] Status codes : 200,204,301,302,307,401,403
[+] User Agent   : gobuster/3.0.1
[+] Verbose      : true
[+] Timeout      : 10s
===============================================================
2019/06/21 11:50:51 Starting gobuster
===============================================================
Missed: /alsodoesnotexist (Status: 404)
Found: /index (Status: 200)
Missed: /doesnotexist (Status: 404)
Found: /categories (Status: 301)
Found: /posts (Status: 301)
Found: /contact (Status: 301)
===============================================================
2019/06/21 11:50:51 Finished
===============================================================
```

There you have it, a basic usage of GoBuster to fuzz some directories. If you want to explore additional flags to use some advanced features and fine tune your attack. Run gobuster with the **-h** flag.

Additionaly check out [this post](https://redteamtutorials.com/2018/11/19/gobuster-cheatsheet/) from RedTeamTutorials to see some advanced options.

Happy hacking.

{{< thm_badge >}}