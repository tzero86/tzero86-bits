---
title: "Get_SUID.sh"
date: 2020-08-15T18:13:59-04:00
draft: false
toc: false
images:
tags:
  - hacking
  - tools
---

#How to locate SUID Files using Find command#
--------------------------------------


>In Linux, **SUID** (set owner userId upon execution) is  a special type of >file permission given to a file. **_SUID gives temporary  permissions to a >user to run the program/file with the permission of  the file owner (rather >than the user who runs it)_**. **_{TryHackme}_**

For example,  the binary file to change your password has the SUID bit set on it  **_(/usr/bin/passwd)_**. This is because to change your password, it will need  to write to the shadowers file that you do not have access to, root  does, so it has root privileges to make the right changes.

{{< image src="../img/suid.png" alt="Hello Friend" position="center" style="border-radius: 1px; padding: 20px" >}}


Let's try to understand what SUID, SGID and the Stickly bits mean:

* __SUID bit__: 
  * On Files: User executes the file with permissions of the owner.
  * On Directories: nothing.

* __SGID bit__:
  * On Files: User executes the file with permissions of the group owner.
  * On Directories: Files created in directory get same permissions as group owner.

* __Sticky bit__:
  * On Files: No use.
  * On Directories: Users are blocked from deleting files from other users.


SUID  bits can be dangerous, some binaries such as passwd need to be run with elevated privileges (as its resetting your password on the system), however other custom files could that have the SUID bit can lead to all  sorts of issues.


To search the a system for these type of files, we can use the **find** command like this: 

```shell
find / -perm -u=s -type f 2>/dev/null
```

This will find all files that current user _**-u=s**_ has permissions _**-perm**_ to access, making sure they are file type _**-type file**_ and will not show any errors due to lack of access to any file _**2>/dev/null**_

Alternatively we can also do this:

```shell
find / -user root -perm /4000  2>&1 | grep -v "Permission denied"
```

* **find /**: we start the search on the root directory.
* **-user root**: files owned by root user.
* **-perm /4000**: files with octal format equivalent for SUID.
* **2>&1**: redirect any standar error messages as a standar output.
* **|**: The pipe character takes the output of previous command as input of the next command.
* **grep -v "Permission denied"**: We use grep to filter out any result including access errors. **-v** means inverse, so it will skip any files with the permissions denied message in it.


Happy hacking.

{{< thm_badge >}}
