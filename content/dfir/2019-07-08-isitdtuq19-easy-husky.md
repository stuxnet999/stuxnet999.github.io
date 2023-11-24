---
title: Easy Husky - ISITDTU Quals 2019
categories: 
    - CTF-Writeups
tags: 
    - Windows Memory Analysis
    - ISITDTU-CTF
    - Volatility
author: Abhiram Kumar
date: 2019-07-08
draft: false
slug: isitdtu-quals-2019-easy-husky
toc: true
---

Full solution of Easy Husky challenge from ISITDTU Quals 2019.

<!--more-->

**tl;dr**

+ Volatility
+ Corrupted file analysis

**Challenge Points**: 534  

**Challenge Solves**: 37  

**Solved by**: [stuxn3t](https://twitter.com/_abhiramkumar) & [Nihith](https://twitter.com/NihithNihi)  

## Challenge Description

![Easy_husky.png](/images/CTF/ISITDTU/EasyHusky/Easy_husky.png)

## Full solution

Okay, let us take a look at the challenge file. It is a WindowsXP memory dump.

Let us see the command history using the **cmdscan** plugin.

![Cmdscan](/images/CTF/ISITDTU/EasyHusky/cmdscan.png)

They created a directory with the name **hu5ky_4nd_f0r3n51c**

Okay, let us have a look what files are present in the above-mentioned directory/folder.

![Filescan](/images/CTF/ISITDTU/EasyHusky/filescan.png)

The file present in the folder is **f149999**

So let us dump the file by using the **dumpfiles** plugin.

![ghex](/images/CTF/ISITDTU/EasyHusky/ghex.png)

As you can see it is reversed **RAR archive**. Just reverse the bytes to get the proper archive.

## Flag

So after obtaining the correct archive, we see that it is password protected. Luckily I guessed that the folder-name was in l33t, so it could be the password. Voila, and we got the flag.  

**Flag**: ISITDTU{1\_l0v3\_huskyyyyyyy<3}
{:.success}