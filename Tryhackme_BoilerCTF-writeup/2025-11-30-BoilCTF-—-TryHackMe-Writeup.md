---
Title: "BoilCTF — TryHackMe Writeup"
Author: Cankun Wang
date: 2025-11-30
tags: [tryhackme, writeup]
---

#Task

Intermediate level CTF. Just enumerate, you'll get there.

You can complete this with manual enumeration, but do it as you wish

#Enumeration

We start with nmap scan for all ports.

![image-20251130163111408](./assets/image-20251130163111408.png)

We noticed two uncommon ports.  And now we are able to answer question 2 and 3.

Question 1 requires us to login through ftp in anonymous.  Then we will find the file extention.

With using dirsearch to exploit the directory, we are able to answer the rest questions. We then keep enumeration.

![image-20251130170332232](./assets/image-20251130170332232.png)

After this, we need to find the interesting file here according to the questions.

Let's use gobuster to do a deeper enumeration to the joomla.

![image-20251130173559459](./assets/image-20251130173559459.png)

We find _test here.

![image-20251130173649470](./assets/image-20251130173649470.png)

![image-20251130173908270](./assets/image-20251130173908270.png)

We find that plot= is the parameter here. Let's try whether we can inject or not. 

![image-20251130174028136](./assets/image-20251130174028136.png)

We tried to change the parameter and we see the changes here.

OS is changed to 123---we can inject.

And we remember sar2html has vulnerabilities. 

![image-20251130174454429](./assets/image-20251130174454429.png)

![image-20251130174529542](./assets/image-20251130174529542.png)

After we inject ls command, we find the interesting file here---log.txt

![image-20251130174657580](./assets/image-20251130174657580.png)

We view the content of the log.txt through cat.

Here we find possible credential.

One is basterd and its password, and another username without password called pentest

![image-20251130175240307](./assets/image-20251130175240307.png)

We successfully login through ssh.

![image-20251130175430458](./assets/image-20251130175430458.png)

We find a file backup.sh

![image-20251130175451751](./assets/image-20251130175451751.png)

Well, we don't have write access. However, when we view the content, we find that there is a credential inside the file. Let's try this.

After we use this credential to login, we are able to find the user.txt

![image-20251130180307695](./assets/image-20251130180307695.png)

#Escalation privilege

First, try sudo -l

![image-20251130180358740](./assets/image-20251130180358740.png)

Lucky! 

However, after running the ls -l and sudo. We find that this is a trick.

Let's try the suid.

![image-20251130180912001](./assets/image-20251130180912001.png)

We find several files here. Let's check GTFO bins one by one.

find.md can be used to escalation privlege according to GTFO bins.

![image-20251130181301896](./assets/image-20251130181301896.png)

![image-20251130181251694](./assets/image-20251130181251694.png)

Now we are root!

Thanks for reading!

