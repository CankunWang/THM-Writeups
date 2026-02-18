---
Title: "Anonymous — TryHackMe Writeup"
Author: Cankun Wang
date: 2025-11-26
tags: [tryhackme, writeup]
---

#Task

Try to get the two flags! Root the machine and prove your understanding of the fundamentals! This is a virtual machine meant for beginners. Acquiring both flags will require some basic knowledge of Linux and privilege escalation methods.

#Enumeration

We start with a normal nmap scan for all ports.

![image-20251127174356073](./assets/image-20251127174356073.png)

Now we can answer most of the questions in the task. And let's try the smbclient since the task asked us for a share name.

![image-20251127180505911](./assets/image-20251127180505911.png)

 I noticed that ftp is opened in nmap scan result. Maybe we can try ftp?

![image-20251127174430855](./assets/image-20251127174430855.png)

Well done! I tried ftp with username and password anonymous and success.

![image-20251127174611239](./assets/image-20251127174611239.png)

Well, a surprise. I assume we can write and read and execute the file here.

![image-20251127174723740](./assets/image-20251127174723740.png)

We cd to the scripts directory to check the files here.

After get the file and view the contents, I think they may be cron jobs.

![image-20251127175729715](./assets/image-20251127175729715.png)

And I have read, write access to the directory. So let's put a reverse shell in clean.sh. 

![image-20251127175815377](./assets/image-20251127175815377.png)

I nano a clean.sh on my own machine, and then put this file into the target throug ftp.

And then start a listener.

![image-20251127175857783](./assets/image-20251127175857783.png)

Now we are in.

#Escalation privlege

![image-20251127181246328](./assets/image-20251127181246328.png)

We check the cronjob, but no cron is availble.

Then we check the sudo command with sudo -l. However, no sudo availble.

Let's check the suid.

![image-20251127181342693](./assets/image-20251127181342693.png)

![image-20251127181352850](./assets/image-20251127181352850.png)

We find an interesting thing. /usr/bin/env is here. We search on GTFO bins and now we know how to use it.

![image-20251127181648001](./assets/image-20251127181648001.png)

Now we are root.

Thanks for reading.



