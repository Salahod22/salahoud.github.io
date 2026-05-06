---
title: TryHackMe Include
date: 2026-05-06 01:00:00 +0100
categories: [Writeup]
tags: [LFI, SSRF, Log Poisoning] 
render_with_liquid: false
media_subpath: /assets/imgs/include/
image:
  path: 0.png
---

## Reconnaissance
    
Ran nmap scan, returned many open ports (forgot to screenshot) but I went straight to the web ports, there were two 4000 and 50000.

Port 4000 had a guest login, we login, after some fuzzing and looking around the only thing that looked intersting was this form : 
![](1.png){: width="2500" height="1300"}

I tried a lot of stuf here looking for ssti or xss or prototype pollution but the way to go was really simple just had to change the isAdmin to "true" :
![](2.png){: width="2500" height="1300"}

After this we had access to 2 more pages, API and Settings.

The API page had info on internal API :
![](3.png){: width="2500" height="1300"}

And the Settings page had an "Update Banner Image URL", this was clearly a SSRF case, and sure enough after updating image to http://127.0.0.1:5000/getAllAdmins101099991 we get :
![](4.png){: width="2500" height="1300"}

We decode the Base64 to get :
![](5.png){: width="2500" height="1300"}

We use these credentials to login to the 50000 service and we get the first flag there.

## Second Part
 
 There wasn't anything interesting in this app and after some fuzzing and looking around the only interesting thing was this /profile.php?img=profile.png :
 ![](6.png){: width="2500" height="1300"}

 This was clearly a LFI case, we just have to find the right payload path, so I ran burpsuite intruder with an LFI paths wordlist and found what we needed :
![](7.png){: width="2500" height="1300"}

We got some users with /etc/passwd but I had to find something else to exploit (I should have tried ssh bruteforce here). 
This was the hardest part of this challenge, I spent too much time here with no success and had to finally look for a clue online. Appparently the way to go was to use telnet to send an email on the SMTP server, this mail will be logged and we will use log poisoning to execute remote code :

![](8.png){: width="2500" height="1300"}

The mail.log file will execute our command and that's how we exploit this.


```console
$ telnet 10.129.129.180 25
MAIL FROM: SalahOud
RCPT TO: <?php system($_GET['cmd']); ?> 

```

![](9.png){: width="2500" height="1300"}

What's left is to find the flag and read it.

![](10.png){: width="2500" height="1300"}

![](11.png){: width="2500" height="1300"}


