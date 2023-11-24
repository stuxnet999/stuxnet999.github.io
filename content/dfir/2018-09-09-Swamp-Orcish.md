---
title: Orcish - Swamp CTF 2018
categories:
    - CTF-Writeups
tags: 
    - Network Forensics
    - Wireshark
    - Scapy
    - SwampCTF
author: Abhiram Kumar
date: 2018-09-09
draft: false
slug: swampctf-2018-orcish
toc: true
---

The full solution of Orcish challenge from Swamp CTF 2018.

<!--more-->

**tl;dr**

+ ICMP data bytes exfiltration from pcap

## **Challenge Description**

**An army of orcs was spotted not too far from town. We were listening in on some of their communications but we have not been able to find anything containing their battle plans. Maybe they are disguising the messages somehow...**
{:.info}

**Challenge file**: [data.pcap](https://mega.nz/file/zwhgGKgR#RQxpByNmu8w1p2dxyIstsG25mawVkcfRLYbPf05cj40)

## **Initial analysis**

We are provided with a network packet capture (.pcap) file. Loading into Wireshark, and checking the protocol hierarchy we see some common protocols stand out.

![protocols](/images/CTF/swamp/orcish/proto.png)

+ UDP
+ TCP
+ HTTP
+ ICMP

For the sake of making this post precise, I won't be delving too much into the unnecessary parts. If you are interested to know, please drop comments or feel free to contact me.

## Analysis of ICMP streams

Using the ICMP filter in Wireshark, as soon as we see the first ICMP packets, we see that the `Info` column shows some as **Obsolete or Malformed?**. This error here is of particular interest to us.

![ICMP-1](/images/CTF/swamp/orcish/icmp1.png)

We see that the error corresponds to the **ICMP Type** field. The **Type** field corresponds to determining what type of ICMP packet it is. E.g: ping-reply, ping-request, reserved etc...

Since we see that in the current case, it is malformed, that must mean the type field here has been configured manually or intentionally. So, that is the lead.

Let us see what data we can gather from the first few malformed packets.

### **Analysis of Packet number: 6971-6976**

![packet1](/images/CTF/swamp/orcish/pkt1.png)

ICMP Type: 71 --> 'G' (In ASCII)
{:.info}

![packet2](/images/CTF/swamp/orcish/pkt2.png)

ICMP Type: 73 --> 'I' (In ASCII)
{:.info}

![packet1](/images/CTF/swamp/orcish/pkt3.png)

ICMP Type: 70 --> 'F' (In ASCII)
{:.info}

![packet1](/images/CTF/swamp/orcish/pkt4.png)

ICMP Type: 56 --> '8' (In ASCII)
{:.info}

![packet1](/images/CTF/swamp/orcish/pkt5.png)

ICMP Type: 57 --> '9' (In ASCII)
{:.info}

![packet1](/images/CTF/swamp/orcish/pkt6.png)

ICMP Type: 97 --> 'a' (In ASCII)
{:.info}

So far, if we concatenate the collected types, we get **GIF89a**. This indicates the [file signature](https://en.wikipedia.org/wiki/GIF#File_format) of a GIF image.

## Writing script using scapy

So in order to ease our job, let us write the script to exfiltrate all the required data using [scapy]().

```python
from scapy.all import *

r = rdpcap("data.pcap")
list_type_bytes = [] #list to store all type bytes

for i in range(len(r)):
    if r[i].haslayer(ICMP) and r[i][IP].dst == '45.58.48.13': # filter to check destination IP and ICMP layer.
        type_byte = chr(r[6970][ICMP].type)
        list_type_bytes.append(type_byte)

concatenate = ''.join(list_type_bytes) # join all the list elements to form single string stream.

f = open('flag.gif','w')
f.write(concatenate)
f.close()
```
{: .copyable}

## Flag

So, let us open the file **flag.gif**

![flag](/images/CTF/swamp/orcish/flag.gif)

**Flag**: flag{we_ride_at_midnight}
{:.success}

## Resources

+ GIF file structure
    + <https://www.w3.org/Graphics/GIF/spec-gif89a.txt>
    + <https://en.wikipedia.org/wiki/GIF>
+ Scapy documentation
    + <https://scapy.readthedocs.io/en/latest/usage.html#starting-scapy>
    + <https://github.com/secdev/scapy>