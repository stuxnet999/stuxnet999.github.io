---
title: Just Do It - InCTF/bi0s Internationals 2019
categories: 
    - CTF-Writeups
tags: 
    - Windows Memory Analysis
    - Volatility
    - bi0sCTF
author: Abhiram Kumar
date: 2019-09-04
draft: false
slug: inctfi-2019-just-do-it
toc: true
---

Full solution of Just Do It challenge from InCTF Internationals 2019.

<!--more-->

**tl;dr**
+ Master File Table Analysis
+ Deleted file data recovery

**Challenge points**: 271

**No. of solves**: 28

**Challenge Author**: [stuxn3t](https://twitter.com/_abhiramkumar/)

## Challenge Description

![challenge-Description](/images/CTF/InCTFi/JustDoIt/description.png)

You can download the challenge from [**Mega**](https://mega.nz/#!A6pARabZ!yS8_6WxfCcC8o544wAK8VVte46E9sPNAgth52hPQVOQ) or [**G-Drive**](https://drive.google.com/file/d/125Dm-5u2LiVqlFWMmpMbZGgpOwop8-G3/view).

## Initial analysis

This is a fairly simple challenge.

We are provided with a **Windows 7** memory dump. Let us begin our initial level of analysis.

```bash
$ volatility -f Mem_Evidence.raw --profile=Win7SP1x64 pslist
```
![pslist](/images/CTF/InCTFi/JustDoIt/pslist.png)

There is nothing quite interesting in the **pslist** output except for the **Sticky Note** process. Hmm, perhaps there is something written in it.

Just to keep it short, there was nothing important written in the clipboard. It was a small rabbit hole.

## Enumertaing open files

Now let us proceed to the files present in the system.

```bash
$ volatility -f Mem_Evidence.raw --profile=Win7SP1x64 filescan
```

![filescan](/images/CTF/InCTFi/JustDoIt/filescan.png)

There are interesting files present on the desktop. The files Important.txt & galf.jpeg are of special interest. Let us try to dump them :)

```bash
$ volatility -f Mem_Evidence.raw --profile=Win7SP1x64 dumpfiles -Q 0x000000003e8ad250 -D .
```
Now we have dumped the file **galf.jpeg**. However, doing basic steg techniques on the file yield nothing. So it is useless.

## Extracting MFT

One important thing in the challenge and also the main exploit is to get the data present in the file **Important.txt**. However, dumpfiles will not be able to dump the required file as it has been deleted. However, its contents are still present in memory. If you fundamentally understand the Master File Table(MFT), you would know that we can access the data as long as the data blocks are overwritten.

For this, we take the help of the **mftparser** plugin.

```bash
$ volatility -f Mem_Evidence.raw --profile=Win7SP1x64 mftparser > mft_output.txt
```

So let us search for the data blocks of the file **Important.txt**

![mft](/images/CTF/InCTFi/JustDoIt/mft.png)

## Flag

Aha! Now we see the characters of the flag separated by irregular number of spaces(Done intentionally).

**Flag**: inctf{1\_is\_n0t\_EQu4l\_7o\_2\_bUt\_th1s\_d0s3nt\_m4ke\_s3ns3}
{:.success}
