---
title: "Internal.sh"
date: 2020-09-16T18:30:35-04:00
draft: true
toc: true
featured: true
summary: "You have been assigned to a client that wants a penetration test conducted on an environment due to be released to production in three weeks."
images:
  - "img/internal_cover.png"
cover: "img/internal_cover.gif"
tags:
  - hacking
  - tryHackme
  - writeups
---

## Internal - THM Room

This is another writeup, this time for TryhackMe's 'Internal' room that is part of the Offensive Pentesting path. This is a writeup detailing all my process to root this room, flaws and all. I embrace mistakes and try to learn from them, therefore this post also includes my failed attempts. You have been warned :)

Leave a comment, leave your feedback or suggestions. It's all welcome, Have a great day!

Enjoy!

- Please visit This room on TryHackMe by [clicking this link](https://tryhackme.com/room/internal).
- **PLEASE NOTE:** Passwords, flag values, or any kind of answers to the room questions were intentionally masked as required by THM writeups rules. The write-up follows my step by step solution to this box, errors, and all. They are usually long :laughing:

PS: I been really busy at work these past days and seems to it'll be the same for the next few weeks. That being said, I still owe you (me? yep) the pentesting report for the last room we solved called [Relevant](https://www.tzero86bits.tk/posts/relevant/). I'd like to also work on the report for this one, so in all fairness I'll owe you two. Will try to get those done relatively soon. I promise. :smile:

### Stage 1 - Pre-engagement Interactions

> "One over-looked step to penetration testing is pre-engagement interactions or scoping. During this pre-phase, a penetration testing company will outline the logistics of the test, expectations, legal implications, objectives and goals the customer would like to achieve."


The client requests that an engineer conducts an external, web app, and internal assessment of the provided virtual environment. The client has asked that minimal information be provided about the assessment, wanting the engagement conducted from the eyes of a malicious actor (black box penetration test).  The client has asked that you secure two flags (no location provided) as proof of exploitation:

  - User.txt
  - Root.txt

Additionally, the client has provided the following scope allowances:

  - Ensure that you modify your `hosts` file to reflect `internal.thm`
  - Any tools or techniques are permitted in this engagement
  - Locate and note all vulnerabilities found
  - Submit the flags discovered to the dashboard
  - Only the IP address assigned to your machine is in scope

### Stage 2 - Reconnaissance

> "Reconnaissance or Open Source Intelligence (OSINT) gathering is an important first step in penetration testing. A pentester works on gathering as much intelligence on your organization and the potential targets for exploit." _from [Cipher.com](https://cipher.com/blog/a-complete-guide-to-the-phases-of-penetration-testing/)_


### Stage 3 - Threat Modeling & Vulnerability Identification

> "During the threat modeling and vulnerability identification phase, the tester identifies targets and maps the attack vectors. Any information gathered during the Reconnaissance phase is used to inform the method of attack during the penetration test." _from [Cipher.com](https://cipher.com/blog/a-complete-guide-to-the-phases-of-penetration-testing/)_


### Stage 4 - Exploitation

> "With a map of all possible vulnerabilities and entry points, the pentester begins to test the exploits found within your network, applications, and data. The goal is for the ethical hacker is to see exactly how far they can get into your environment, identify high-value targets, and avoid any detection." _from [Cipher.com](https://cipher.com/blog/a-complete-guide-to-the-phases-of-penetration-testing/)_


### Stage 5 - Post-Exploitation, Risk Analysis & Recommendations

> "After the exploitation phase is complete, the goal is to document the methods used to gain access to your organization’s valuable information. The penetration tester should be able to determine the value of the compromised systems and any value associated with the sensitive data captured." _from [Cipher.com](https://cipher.com/blog/a-complete-guide-to-the-phases-of-penetration-testing/)_

### Stage 6 - Reporting

> "Reporting is often regarded as the most critical aspect of a pentest. It’s where you will obtain written recommendations from the penetration testing company and have an opportunity to review the findings from the report with the ethical hacker(s)." _from [Cipher.com](https://cipher.com/blog/a-complete-guide-to-the-phases-of-penetration-testing/)_

