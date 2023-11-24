---
title: G1bs0n - SEC-T CTF 2017
categories: 
    - CTF-Writeups
tags: 
    - Windows Memory Analysis
    - Volatility
    - SEC-T-CTF
author: Abhiram Kumar
date: 2020-02-11
draft: false
slug: sect-ctf-2017-gibson
toc: true
---

Full solution of G1bs0n challenge from SEC-T CTF 2017.

<!--more-->

## Challenge Description

Agent Gill called, we have until tomorrow at 15:00 UTC to fix some virus problem.
{:.info}

## Writeup

The challenge description doesn't give out any clues. So let us try to dig into the challenge file.

My first bet was to try out the pslist and the psscan plugins but there was nothing suspicious there. Eventually, I began the process of using the filescan plugin and check if there are any suspicious files open.

Though filescan is an extremely useful plugin, the hitch is in using the right filter to get the desired view of the files as the general output is very large. When I used the plugin without any filters, I noticed that there were two users: **acidburn** and **plauge**.

So I decided to apply every filter I could possibly think of and like anybody, I tried Desktop as the filter. Looking through the files present in the desktop, I found a rather suspicious file called **g4rb4g3.txt**. So I dumped the file and looked at its contents.

![garbage](/images/CTF/SECT/gibson/garbage.png)

Looks like the flag is in 2 parts for this challenge. Maybe this is the first half and it has been reversed. And again I was searching for some files using the filescan plugin when I accidentally stumbled upon a **.bat** file. So I tried to search for something suspicious in that and Voila!!

Batch files are in a way similar to .sh files in linux
{:.info}

![bat-filescan](/images/CTF/SECT/gibson/bat.png)

We see a very suspicious file with the name **hack.bat**. Let us dump this file and examine its contents.

![hackbat](/images/CTF/SECT/gibson/hackbat.png)

We see a suspicious file named **gibson**. I thought to search for this file in the memory dump and if the file was present in the dump, try to recover and examine its contents.

Luckily the file was available as **gibson.jpg** in the memory dump. So I recovered it and its contents were encoded in base64.

![gibson](/images/CTF/SECT/gibson/gibson.png)

I knew that decoded bytes belonged to a certain file, so I ran the file command.

![realgibson](/images/CTF/SECT/gibson/realgibson.png)

So yeah! It was a **zip-compressed** file. Let us try to extract its contents. We get three files from the zip compressed file:

+ run.bat
+ run.ps1
+ run.reg

Let us try to look at the contents of all these files. **run.ps1** did not hold anything suspicious so I’m cutting it out in this post.

![runbat](/images/CTF/SECT/gibson/runbat.png)

Hmm…The directory `T3MP` looks suspicious, doesn’t it? So we are on the right track! Okay, moving on to **run.reg** file.

![runreg](/images/CTF/SECT/gibson/runreg.png)


So Yeah!! Looks like we got the 2 halves of the flag. Let us reverse them and concatenate and see what we get. Looks like it was rot13 so let us decode this.

## Flag

![final](/images/CTF/SECT/gibson/finalscript.png)

And that is how we solved this challenge!!

**FLAG**: SECT{PL46U3_PHR34K_M3_0UT_K4T3_FTW}
{:.success}