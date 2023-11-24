---
title: Notch It Up - InCTF/bi0s Internationals 2019
categories: 
    - CTF-Writeups
tags: 
    - Windows Memory Analysis
    - Volatility
    - bi0sCTF
author: Abhiram Kumar
date: 2019-09-04
draft: false
slug: inctfi-2019-notch-it-up
toc: true
---

Full solution of Notch It Up challenge from InCTF Internationals 2019.

<!--more-->

**tl;dr**
+ Chrome history analysis
+ File recovery from the memory dump
+ Raw analysis of email content
+ Environment variables analysis
+ RAR password cracking
+ Corrupted file analysis

**Challenge points**: 900

**No. of solves**: 11

**Challenge Authors**: [stuxn3t](https://twitter.com/_abhiramkumar/), [Sh4d0w](https://twitter.com/__Sh4d0w__) & [g4rud4](https://twitter.com/NihithNihi)

## Challenge Description

![Description](/images/CTF/InCTFi/NotchItUp/Notch_it_up_description.png)

You can download the challenge file at [**MEGA**](https://mega.nz/#!kypmTaLJ!cWChsh8CdTMTWt7Ae0oNiCFfrSXwK8vqEMGn0SO22JQ) or [**G-Drive**](https://drive.google.com/file/d/1bER4wmHP_LAMgdB52LGkb8x2Mf8hG3V6/view?usp=drivesdk)

## Writeup

We are provided with a **Windows 7** memory dump. Let us begin our initial level of analysis.

Let's start with the running processes.
```bash
$ volatility -f Challenge.raw --profile=Win7SP1x64 pslist
```

![NotchItUp-pslist](/images/CTF/InCTFi/NotchItUp/pslist1.png)
![NotchItUp-pslist2](/images/CTF/InCTFi/NotchItUp/pslist2.png)

## Retrieving browser history

As seen above, we see chrome and firefox as active running processes. Let us see the history of google chrome here.
I have trimmed most of the content and only focussing on the relevant part of the history
```bash
$ volatility --plugins=volatility-plugins/ -f Challenge.raw --profile=Win7SP1x64 chromehistory
```

![NotchItUp-chromehistory](/images/CTF/InCTFi/NotchItUp/pastebin-link.png)

Hmm, we find an interesting **PasteBin** link. The link is: **https://pastebin.com/RSGSi1hk**

![NotchItUp-PasteBin](/images/CTF/InCTFi/NotchItUp/pastebin.png)

The Pastebin link contains another Google Docs link, lets head there. The docs link is: [click here](https://www.google.com/url?q=https://docs.google.com/document/d/1lptcksPt1l_w7Y29V4o6vkEnHToAPqiCkgNNZfS9rCk/edit?usp%3Dsharing&sa=D&source=hangouts&ust=1566208765722000&usg=AFQjCNHXd6Ck6F22MNQEsxdZo21JayPKug)

![Google-Doc](/images/CTF/InCTFi/NotchItUp/docs_with_mega_link.png)

The doc contains a lot of spam but we find one interesting link which leads us to a mega drive: **https://mega.nz/#!SrxQxYTQ**.

However, to download the file present in the mega drive, we need to find the KEY. However, the text in the Pastebin link tells us that "David sent the key in mail".

## Retrieving screenshots of PC

Okay, let me use the **Screenshot** plugin. Maybe it'll help.

```bash
$ volatility -f Challenge.raw --profile=Win7SP1x64 screenshot -D .
```

![NotchItUp-Screenshots](/images/CTF/InCTFi/NotchItUp/screenshot.png)

We see that the browser window is open and also that GMail is open with the subject **Mega Drive Key**. Now it is the time to begin a little raw analysis. So a small intro. The data, when loaded into ram, is not encrypted, so basically, whatever you type in the browser window or load in it is saved as a sort of JSON data. So we just have to locate some JSON sort of data which contains our subject string "Mega Drive Key". Let us see if we can get the email data.

So what I did was use the command **strings**. Simple.
```bash
$ strings Challenge.raw | grep "Mega Drive Key"
```
![NotchItUp-strings](/images/CTF/InCTFi/NotchItUp/Mega-key.png)

So the key is **zyWxCjCYYSEMA-hZe552qWVXiPwa5TecODbjnsscMIU**.

So we find a PNG image in the drive. However, PNG is corrupted. Fixing the IHDR of the image gives us the 1st part of the flag.

![NotchItUp-1stpart](/images/CTF/InCTFi/NotchItUp/flag1.png)

The first part is: inctf{thi5\_cH4LL3Ng3\_!s\_g0nn4\_b3\_?\_
{:.success}

## Finding the other half

Now moving onto the second part,
Let us use the filescan plugin to find what kind of open-files are present in the system.
```bash
$ volatility -f Challenge.raw --profile=Win7SP1x64 filescan | grep Desktop
```

In the desktop of the system, we see a folder by the name **pr0t3ct3d**. It contains a RAR archive with the name **flag.rar**

![NotchItUp-filescan](/images/CTF/InCTFi/NotchItUp/filescan.png)

Let us dump the RAR archive with the help of the dumpfiles plugin.
```bash
$ volatility -f Challenge.raw --profile=Win7SP1x64 dumpfiles -Q 0x000000005fcfc4b0 -D .
```

However, the archive is password protected. Also, brute-forcing for the password is not at all intended. So let us look for some other clues which may help us to get the password of the archive.

Using the **cmdscan** plugin, we see that **env** command has been used but that is an invalid command in windows command prompt. So let us look at the state of the Environment variables.

```bash
$ volatility -f Challenge.raw --profile=Win7SP1x64 cmdscan
```

![cmdscan](/images/CTF/InCTFi/NotchItUp/cmdscan.png)

```bash
$ volatility -f Challenge.raw --profile=Win7SP1x64 envars
```

We observe a custom variable created named **RAR password**.

![NotchItUp-Env](/images/CTF/InCTFi/NotchItUp/envars.png)

So it gives out the password as **easypeasyvirus**. Now we get the last part of the flag.

![NotchItUp-flag2](/images/CTF/InCTFi/NotchItUp/flag2.png)

So now let us concatenate the 2 parts to finish off the challenge.

## Flag

**FLAG**: inctf{thi5\_cH4LL3Ng3\_!s\_g0nn4\_b3\_?\_aN\_Am4zINg\_!\_i\_gU3Ss???\_}
{:.success}