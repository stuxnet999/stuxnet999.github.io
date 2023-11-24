---
title: Mr EvilPepo [series] - TrollCAT CTF 2021
categories:
  - CTF-Writeups
tags: 
  - Windows Memory Analysis
  - Trollcat-CTF
author: Abhiram Kumar
date: 2021-02-07
draft: false
slug: trollcat-ctf-2021-evilpepo
toc: true
---

Full solution of Mr Evilpepo (Part 1,2,3) challenge from TrollCAT CTF 2021.

<!--more-->

**tl;dr**

+ Part 1
  + Retrieving flag from command history
+ Part 2
  + Decrypting Encrypted Pastebin data
+ Part 3
  + Retreving MEGA drive URL from memory dump
  + Retreiving flag from Veracrypt drive

All of the challenges are quite simple hence I won't really go into great detail about them here. The profile after running **imageinfo** is `Win7SP1x64`.

## Part 1

![description](/images/CTF/trollcat/evil/description1.png)

The first flag was present in the command history. We can use the **cmdscan** plugin to get the flag.

![cmdscan](/images/CTF/trollcat/evil/cmdscan.gif)


FLAG: **trollcat{comands_4r3_important}**

## Part 2

![description](/images/CTF/trollcat/evil/description2.png)

From the description it is pretty clear that we need to fetch the browser history. As we see chrome running in the list of active processes, one can assume that it is the browser we need to look into.

![chromehistory](/images/CTF/trollcat/evil/history.gif)

And here, we find a link to encrypted pastebin webpage which requires us to give in a password to view the hidden data. For this we used the user's local system password and it worked!

The password can be obtained by using the mimikatz plugin.

![mimikatz](/images/CTF/trollcat/evil/mimikatz.gif)

Password: **abracadabra**
Pastebin URL: https://defuse.ca/b/sOOqp4UunTdD0oUjidJFlz

And voila! we get the flag.

![part2flag](/images/CTF/trollcat/evil/part2flag.png)

FLAG: **Trollcat{secret_hidden_0nn_th3_1ntern3t}**

## Part 3

![description](/images/CTF/trollcat/evil/description3.png)

Part 3 was also quite easy. Initally when I attempted the challenge, I looked at the clipboard contents using the **clipboard** plugin which revealed a MEGA drive link.

![megalink](/images/CTF/trollcat/evil/megalink.gif)

The same link could be obtained by just running `strings` on the memory dump as well or can be retrieved from a text file where it was originally stored.

![megaurl](/images/CTF/trollcat/evil/megaurl.gif)

From the Mega drive we download a file named **Secret** which is basically a Veracrypt drive. We just need to load it in Veracrypt and that's it, the challenge is done.

![final](/images/CTF/trollcat/evil/final.gif)

FLAG: **Trollcat{y0u_got_n1ce_Skills!!!}**

## Flags

+ **trollcat{comands_4r3_important}**
+ **Trollcat{secret_hidden_0nn_th3_1ntern3t}**
+ **Trollcat{y0u_got_n1ce_Skills!!!}**

## References

If you are new to memory forensics, I suggest you to go through my previous article - [Basics of Memory Forensics](https://stuxnet999.github.io/volatility/2020/08/18/Basics-of-Memory-Forensics.html)