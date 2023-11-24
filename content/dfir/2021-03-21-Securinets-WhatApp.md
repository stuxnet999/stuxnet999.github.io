---
title: What App Is On Fire? - Securinets Quals 2021
categories:
  - CTF-Writeups
tags: 
  - Windows Disk Analysis
  - Securinets-CTF
author: Abhiram Kumar
date: 2021-03-21
draft: false
slug: securinets-quals-2021-what-app
toc: true
---

Full solution of **WhatApp is on Fire challenge?** from Securinets Quals 2021.

<!--more-->

**tl;dr**

+ Investigate chats from WhatsApp messages DB.
+ Obtain saved password from Mozilla Firefox.

There are a few fake flags here and there in this challenge. I won't be covering them.

## Challenge Description

![description](/images/CTF/SecurinetsQuals/whatapp/description.png)

You can download the challenge file from [here](https://drive.google.com/file/d/13W2EP5G1ceFTMRLdaBU_6IiwhAgZlwLq/view?usp=sharing)

## Investigating WhatsApp chats

The challenge doesn't really have a lot of information which can help us right away, so we proceed with analyzing the disk image using Autopsy. However, I was fortunate enough to spot **WhatsApp** in `Users\Semah\AppData\Roaming`. Upon digging through the folders, I spotted the **msgstore.db** file which could be read using any DB viewer.

![messagestore](/images/CTF/SecurinetsQuals/whatapp/messagestore.png)

I used DB Browser to look through the SQLite database and found the **messages** section where we can clearly see chat contents.

![messages](/images/CTF/SecurinetsQuals/whatapp/messages.png)

The whole chat is as follows

```
Hello

Hello

who is with me ?

I'm from the Dev department, there is important things i have to send it to you

oh hello, what is it ?

**okay you can check this file below : https://drive.google.com/file/d/1L1xN6R-Za4W1ME2UZE8LXH45gfHSw66D/view?usp=sharing**

Okaay i got it, thank you for the information

Also don't forget to change the password

if you had any problem, don't hesitate to come back to me at anytime

yeah it works fine for now, but why the change?

sorry we can't reveal anything for now, we will discuss it irl, a lot of thing should be told

oh okay hope everything is good

it is no need to worry ! 

Have a nice day

thank you, you too !
```

Clicking on the google drive link takes us to this:

![readme](/images/CTF/SecurinetsQuals/whatapp/readme.png)

From the above link we obtain a username, the URL of a login page and first half of the flag.

```
username: Securinets_Quals2k21
URL: https://for1.q21.ctfsecurinets.com
first half: Securinets{whatsapp_and_
```

And now we need to get the password of the login page.

## Recovering Firefox Saved Password

Autopsy does a really good job in recovering details about saved passwords using its ingest modules. Though we are able to get the username and URL, Autopsy doesn't give us the password yet.

![autofill](/images/CTF/SecurinetsQuals/whatapp/autofill.png)

However, recovering firefox passwords has been known for a long time.

We can simply extract the user profile and then load it a few open-source tools like **PasswordFox**.

**Profile PATH**: Users\Semah\AppData\Roaming\Mozilla\Firefox\Profiles\pyb51x2n.default-release
{:.info}

Now we load the extracted folder into PasswordFox.

![passwordfox](/images/CTF/SecurinetsQuals/whatapp/passwordfox.png)

We use the password **GacsriicUZMY4xiAF4yl** in the login page to get the final part of the flag.

![final](/images/CTF/SecurinetsQuals/whatapp/final.gif)

**FLAG**: Securinets{whatsapp_and_saved_passwords_isn't_helpful_after_all_x)}
{:.success}

## Tools used

+ Autopsy
  + <https://www.autopsy.com/download/>
+ DB Browser
  + <https://sqlitebrowser.org/>
+ PasswordFox
  + <https://www.nirsoft.net/utils/passwordfox.html>