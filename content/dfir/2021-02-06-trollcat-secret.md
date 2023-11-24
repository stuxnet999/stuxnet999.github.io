---
title: S3cr3t - TrollCAT CTF 2021
categories:
    - CTF-Writeups
tags: 
    - Windows Disk Analysis
    - Trollcat-CTF
author: Abhiram Kumar
date: 2021-02-06
draft: false
slug: trollcat-2021-secret
toc: true
---

Full solution of s3cr3t challenge from Trollcat CTF.

<!--more-->

**tl;dr**

+ Recover Bitlocker encrypted vhdx
+ Bruteforce using John the ripper
+ Recover deleted file from decrypted drive

## Challenge description

![Description](/images/CTF/trollcat/description.png)

You can download the challenge file from here: [Mega drive](https://mega.nz/file/PtsFHYzY#tKDykxlC1Uj5FniYU947AMRFJubc8OL11l0jmMbxmbA)

## Initial analysis

We are provided with a Encase forensic image (.E01). All the common forensics tools like Autopsy and FTK Imager are capable of analyzing this file. In my case, I prefer to stick with Autopsy.

![Autopsy](/images/CTF/trollcat/autopsy1.gif)

By having a brief look at the various folders, we can say that the image was acquired from a Windows system.

## Recovering vhdx

Using Autopsy, we can see that in the **Deleted Files** section, a **topsecret.vhdx** file. Let us extract it. When I tried to mount it, the mount was unsuccessful and Windows issued a notification saying is **Bitlocker encrypted drive**.

### Bruteforcing Bitlocker drive

To bruteforce the bitlocker drive, we can use John the ripper and the wordlist rockyou.txt. Just simply use **bitlocker2john** to create the hashes and load them to john. You can use this [website](https://hashes.com/en/johntheripper/bitlocker2john) to create the hashes as well.

```bash
$ cat hash
$bitlocker$1$16$f69baf5d4226828d3bfa2cc373630ec8$1048576$12$1025632abafad60103000000$60$04465c3433f92c243ff384e34dc7c23f8d2ff94b3b2cfd7544aa2aff8da10de3a68ce356d5ab4d9cc9f83c07225ec72f04bd01f46bd2d9fbb61a0981
$bitlocker$2$16$c1552ea6135b71ae62b1e2a5a21d9b75$1048576$12$1025632abafad60106000000$60$2177d1b2640f86122bcfd4dc1736a502c914cef56393400e6622dcf0ae7149d35a0f8d7a9ec8024cecf0390d9670dfe95a300d7e5c478e9f709a0a4b

$ john-the-ripper --format=bitlocker --wordlist=rockyou.txt hash
Using default input encoding: UTF-8
Loaded 1 password hash (BitLocker, BitLocker [SHA-256 AES 32/64])
Cost 1 (iteration count) is 1048576 for all loaded hashes
Note: This format may emit false positives, so it will keep trying even after
finding a possible candidate.
Press 'q' or Ctrl-C to abort, almost any other key for status
0g 0:00:00:04 0.00% (ETA: 2021-04-05 01:21) 0g/s 3.209p/s 3.209c/s 3.209C/s monkey
0g 0:00:00:08 0.00% (ETA: 2021-04-08 07:56) 0g/s 3.205p/s 3.205c/s 3.205C/s chocolate
0g 0:00:00:11 0.00% (ETA: 2021-04-08 05:25) 0g/s 3.208p/s 3.208c/s 3.208C/s justin
0g 0:00:00:14 0.00% (ETA: 2021-04-10 04:45) 0g/s 3.209p/s 3.209c/s 3.209C/s joshua
0g 0:00:00:17 0.00% (ETA: 2021-04-09 12:08) 0g/s 3.214p/s 3.214c/s 3.214C/s angels
0g 0:00:00:59 0.00% (ETA: 2021-04-14 14:25) 0g/s 3.039p/s 3.039c/s 3.039C/s myspace
0g 0:00:01:01 0.00% (ETA: 2021-04-16 03:44) 0g/s 2.999p/s 2.999c/s 2.999C/s angel1
**johncena**         (?)
1g 0:00:03:06 0.00% (ETA: 2021-04-14 13:05) 0.005348g/s 3.070p/s 3.070c/s 3.070C/s sweets
Session aborted
```

So the password is **johncena**.

![bitlocker](/images/CTF/trollcat/bitlocker.gif)

Hmm, we seem to be able open it but don't see the flag in there. That's when I thought maybe even the flag file is a deleted one.

## Recovering flag

I load the mounted drive into Autopsy and quickly found the flag present in the recycle bin.

![autopsy2](/images/CTF/trollcat/autopsy2.gif)

For those who are new to recycle bin forensics, $R file has the data of the deleted file whereas the $I file stores the original PATH and timestamps associated.

## Flag

**FLAG**: Trollcat{finallly_y0u_f0und_mY_s3ret!!!}
{:.success}

PS: I also got the first blood on this challenge.