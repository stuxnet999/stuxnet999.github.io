---
title: Incident Response Essentials & Tips - My Two Cents
draft: false
categories:
  - DFIR
tags: 
  - Incident Response
author: Abhiram Kumar
date: 2023-04-17
slug: ir-essentials-tips
toc: true
---

This blog post contains my thoughts on the essential foundation needed in Incident Response based on my experience.

<!--more-->

## About My Background

Before going into a lengthy discussion about the core foundation and skills in IR, I feel it is important to disclose my background so the reader knows where I'm coming from. I must mention that most of the content mentioned here is **solely my personal opinion** and please don't consider it to be "THE RULE". I have been working in IR for the last 2 years alongside some of the best people in the industry. I honestly consider many of them to be my gurus/mentors. I have learnt a lot from them - work discipline, communication, expanding my self-skillset etc...

Before this, I was a CTF player since 2017 at my University and my work there was concentrated in DFIR. So I believe that I have pretty good experience to speak about this.

## Essential Skills

The list below is a TL;DR - 

+ Good Programming Background
  + Preferable languages - Python & C (optional)
  + Ability to read code and understand what it does
+ Good Idea on CMD and PowerShell Commands
  + Usage and ability to interpret
+ Strong Foundation in Windows Forensics
  + Focus on parsing various artefacts and the ability to properly interpret parsed-output
  + Correlate information from various artefacts to build a proper timeline of events
+ Always be Open to New Stuff
+ Trust but Verify!

### Good Programming Background

I come from an engineering background with a major in computer science. So programming for me is an indispensable skill set which I believe everyone must have. I know a few roles might not need programming but I have always felt I can understand a situation better because I know to program. There are so many powerful programming languages and all are not necessary. The ones I recommend are **Python** (py2 has been deprecated so prefer learning py3) and **C**. Okay, I know many people might not consider C but hear me out. C is one of those languages which gives you a great understanding of the system. However since very few folks in IR use it every day, you can skip it.

If you ask me if you can write small pieces of code and also can read and understand code, that itself is enough. The reason why I bring this up is that many open-source tools/scripts that we regularly use are usually written in Python. Having good knowledge of Python can help you troubleshoot errors better.

### Good Idea on CMD and PowerShell Commands

The Windows Command Prompt and PowerShell commands are regularly used by IR professionals when going live on infected systems and are also abused by Threat Actors to deploy, run, create persistence of malware and also to exfiltrate data.

I mean it is almost every day that we see obfuscated PowerShell commands and cmd batch files which execute malicious commands. Most of this can be learned on the go too. Google and MSFT Docs are your best friends.

### Strong Foundation in Windows Forensics

The reason I mention Windows here is because of the high consumer base. Most of the cases you will ever come across will all be Windows systems. We usually see very few Linux/Mac systems. Maybe Mac more because nowadays orgs are supplying those as workstations to employees. Still, Windows remains the overall leader in this. From servers to workstations, it is the most dominant.

Okay, there are way too many artefacts in Windows, some you need to have strong knowledge of before joining the role and some you learn along the way. SANS has made a great poster which contains most of the important artefacts that you may use every day. I recommend the **Windows Forensic Analysis** and **Hunt Evil** posters.

### Always be Open to New Stuff

The field of cybersecurity is ever-changing. I consider this to be a beautiful thing as it always challenges us and brings out innovation within the community. Therefore, constantly be willing to learn, use, and most importantly, accomplish the same old things using newer methods.

### Trust but Verify!

Tools developed by community or companies is always a great asset. However, sometimes bugs always creep in. Hence don't blindly rely on the tool's output. Always compare tools. **Validate!!!**.

If you find a bug, do your best to report it to the developer.

Another piece of advice is a tool can only take you so far. It is you the person that needs to analyze and interpret the information. So never be afraid to ask for help.

I hope this article is useful to anyone interested in DFIR. Please contact me via any of my socials regarding questions/corrections. I would try to respond ASAP.