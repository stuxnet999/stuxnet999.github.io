---
title: EV3 Player - HITCON Quals 2019
categories: 
  - CTF-Writeups
tags: 
  - LEGO EV3 Robot
  - Wireshark
  - HITCON
author: Abhiram Kumar
date: 2019-10-14
draft: false
slug: hitcon-quals-2019-ev3-player
toc: true
---

Full solution of EV3 Player challenge from HITCON CTF Quals 2019.

<!--more-->

**tl;dr**

+ EV3 Robot pklg analysis
+ .RSF file recovery

**Challenge points**: 207

**No. of solves**: 58

**Solved by**: [stuxn3t](https://twitter.com/_abhiramkumar/), [f4lc0n](https://twitter.com/_f41c0n), [cr4ck3t](https://twitter.com/nambiar_kartuzz), [g4rud4](https://twitter.com/NihithNihi)

## Challenge Description

![Description](/images/CTF/HITCON/EV3Player/challenge-description.png)

## Initial analysis

We are provided with the EV3 packet capture. You can get the idea of working of the robot by viewing the YouTube video given in the description.[Link](https://www.youtube.com/watch?v=J5hUOzusb0E)

So basically, the robot is giving out audio, which in this case is the flag. So let us see what the **.pklg** looks like.

![Wireshark](/images/CTF/HITCON/EV3Player/Wireshark1.png)

There are 1742 packets as displayed in Wireshark, some of which probably contain the audio data which was sent to the EV3 robot. Then we found something quite interesting.

![File Paths](/images/CTF/HITCON/EV3Player/RSF-filepath.png)

As you can see, a file with extension - **'.rsf'** is being sent to the LEGO EV3 robot. So we didn't know what an **rsf** file was. So we googled the term.

![RSO-file](/images/CTF/HITCON/EV3Player/WhatIsRSF.png)

So now we know what an rsf file is. So let us try to extract the data from the pklg and I guess we just need to play it in some audio player and get the flag.

## So where is the data present?

For this, we needed to see how a **.rsf** file is seen in the hex dump. For this, we downloaded some sample **.rsf** files from the internet and loaded it in a hex editor.

![hex](/images/CTF/HITCON/EV3Player/RSF-DATA.png)

So now let us extract that data as hex streams and write them into a file. Next, we need to load them in an audio player. However, when we tried it in Audacity, it wouldn't support it. So we set out to look for solutions online. Then we found the EV3 toolkit.

This software helps us to load programs to the EV3 brick. So we thought it was possible to play the **.rsf** files using this. Indeed it worked! But we are not able to listen to the flag properly due to a frequency issue. So we thought to convert this audio into a **.WAV** format. So I found a tool which could convert it from '.rsf' to '.wav'.

In this process, I found the tool [WAV2RSO](http://bricxcc.sourceforge.net/utilities.html) which would convert a WAV to an RSO file and vice versa. So basically, from what we found out, an RSO file is very similar to RSF. So we changed the extension of RSF to RSO and then using the above tool to convert it to WAV.

In this way, we successfully converted the file to WAV and launched it in Audacity. Now we needed to change the frequency of the audio file so that we can listen clearly. After a few trials, we could listen to the audio at 6300-6500 Hz.

## Flag

The audio file readout,

Congratulations the flag contains no spaces and all lowercase. Here is flag hitcon{playsoundwithlegomindstormrobot} hello.
{:.warning}

Voila! We got the flag.

**FLAG**: hitcon{playsoundwithlegomindstormrobot}
{:.success}

## References

+ WAV2RSO
  + <http://bricxcc.sourceforge.net/utilities.html>
+ RSF
  + <https://fileinfo.com/extension/rsf>
+ EV3 software
  + <https://www.lego.com/en-us/themes/mindstorms/downloads>
