---
title: "Nmap_basics.sh"
date: 2020-08-15T16:56:33-04:00
draft: false
toc: false
images:
tags:
  - hacking
  - tools
---

#NMAP - Basic Usage#
------------------

> **nmap** is an free, open-source and powerful tool used to discover hosts and services on a computer network. In our example, we are 
> using nmap to scan this machine  to identify all services that are running on a particular port. nmap  has many capabilities, below > is a table summarizing some of the functionality it provides. 
> _{from TryHackme}_


Here are some of the basic switches/flags that your can use when running an nmap scan:


| nmap flag       | Description                                                 |
|:----------------|:-----------------------------------------------------------:|
| -sV             | *Attempts to determine the version of the services running* |
| -p {port} or -p-| *Port scan for port or scan all ports*                      |
| -Pn             | *Disable host discovery and just scan for open ports*       |
| -A              | *OS and version detection, runs integrated scripts*         |
| -sC             | *Scan with the default nmap scripts*                        |
| -v              | *Verbose mode*                                              |
| -sU             | *UDP port scan*                                             |
| -sS             | *TCP SYN port scan*                                         |
|                 |                                                             |




#How to Scan Nmap Ports#
------------------------

To scan Nmap ports on a  remote system, enter the following in the terminal:


```shell
  sudo nmap {target_IP}
```


Replace the {target_IP} with the IP address of the system you’re scanning. This is the most basic format for Nmap, and it will return information about the ports on that system. While is this a quick scan, **it wont scan all ports by default**. To specify all ports look at the **-p-** flag in the previous table.

In addition to scanning by IP address, you can also use the following commands to specify a target:


```shell
#To scan a host:
nmap www.hostname.com

#To scan a range of IP addresses (.1 – .10):
nmap 192.168.0.1-10

#To run Nmap on a subnet:
nmap 192.168.0.1/13

#To scan targets from a text file:
nmap –iL textlist.txt
```

~~~~~~~~~~~~~~
  Happy hacking.
  tzero86
~~~~~~~~~~~~~~

