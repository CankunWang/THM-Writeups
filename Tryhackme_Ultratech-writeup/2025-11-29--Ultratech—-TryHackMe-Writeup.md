---
Title: " Ultratech— TryHackMe Writeup"
Author: Cankun Wang
date: 2025-11-29
tags: [tryhackme, writeup]
---

#Task

*This room is inspired from real-life vulnerabilities and misconfigurations* *I encountered during security assessments.*

*If you get stuck at some point, take some time to keep enumerating.*

*
*

**[ Your Mission ]**

You have been contracted by UltraTech to pentest their infrastructure.

It is a grey-box kind of assessment, the only information you have

is the company's name and their server's IP address.

**Start this room by hitting the "deploy" button on the right!**



Good luck and more importantly, have fun!

#Enumeration

We start with a normal nmap scan for all ports and service to answer the first several questions

![image-20251129122929764](./assets/image-20251129122929764.png)

Now we are able to answer the questions about services and ports.

Next, we use a nmap scan with -O to scan for the Linux distribution.

![image-20251129123033670](./assets/image-20251129123033670.png)

Next, we need to figure out the number of routes under REST api.

![image-20251129123117203](./assets/image-20251129123117203.png)

We do a dirsearch to find all the routes.

#Exploit

We noticed that ftp port 21 is opened, which means we can try the ftp anonymous login.

![image-20251129123419701](./assets/image-20251129123419701.png)

However, we can not.

According to the hint, let's focus on how api is called.

Let's use ffuf to fuzz the routes with common wordlists.

![image-20251129124611208](./assets/image-20251129124611208.png)

We find an interesting point here.

Ping is look like not filter.

![image-20251129132326138](./assets/image-20251129132326138.png)

We are correct. Ping is not filtered.

![image-20251129132351020](./assets/image-20251129132351020.png)

Success. It returned www(www-data).

![image-20251129132412707](./assets/image-20251129132412707.png)

utech.db.sqlite is returned.

![image-20251129132531513](./assets/image-20251129132531513.png)

We directly cat the target database, and looks like it returned two hash.

We need to crack the hash.

root:

 n100906

admin:

 mrsheafy

Now we have two credentials.

#Escalation privilege

I remembered ftp is opened, let's check for ssh.

After login as r00t, let's check our current status.

![image-20251129133809575](./assets/image-20251129133809575.png)

We are user r00t, however, there may exist docker.

![image-20251129133848876](./assets/image-20251129133848876.png)

Correct, there is a docker images.

We can try to escalation privilege through this.

![image-20251129133927975](./assets/image-20251129133927975.png)

We are root now.

We only need to use cat to view the ssh private key.

Thanks for reading!

