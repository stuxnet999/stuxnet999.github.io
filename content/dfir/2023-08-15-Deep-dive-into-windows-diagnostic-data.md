---
title: Deep Dive Into Windows Diagnostic Data & Telemetry (EventTranscript.db) - PART 1
draft: false
author: Abhiram Kumar
date: 2023-08-15
slug: windows-diagnostic-data-part-1
categories:
  - DFIR
tags: 
  - Windows Diagnostic Data
toc: true
---

A small article detailing my recent experiments with Windows Diagnostic Data Telemetry (EventTranscript.db).

<!--more-->

## What is this "EventTranscript.DB"?

As we all know Windows collects data from our computers. As per Windows - "Diagnostic Data is used to keep Windows secure and up to date, troubleshoot problems and make product improvements." Windows lets us choose (to some extent) what data we wish to share. It is normally "Required Diagnostic Data" and the more interesting one "Optional Diagnostic Data" where data includes Edge browsing history, app inventory, app usage, device activity etc.. are also shared.

Two years ago, my good friend Andrew Rathbun conducted an extensive research on this collected telemetry focused from a forensics perspective which resulted in a series of amazing [blog posts](https://www.kroll.com/en/insights/publications/cyber/forensically-unpacking-eventtranscript).

I too wanted to jump in on this and later released a small python tool to parse some forensically interesting data and produce some CSVs. The project was [EventTranscriptParser](https://github.com/stuxnet999/EventTranscriptParser).

## Renewed Interest

Since 2021, I did look again on this artifact until a few weeks ago when boredom led to some experimentation 😅.

This time I was however interest on a data sources relating to -
+ Application activity
 - This is of special interest for those of us in DFIR because we are always interested in different sources for evidence of execution.

## Application Activity

Few weeks ago, I published a twitter thread where I observed the SHA1 hash values were wrongly picked up by Windows Diagnostics. Here I publish the same while also testing a few more scenarios. Let's start.

As we know the "Win32kTraceLogging.AppInteractivitySummary" records the execution of binaries. The event looks like this - 

![](/images/windows-diag-data/Win32kTraceLogging.png)

Here in this JSON formatted data, a few interesting sections stand out -

+ **"time"**
  + As per [Andrew's research](https://www.kroll.com/en/insights/publications/cyber/forensically-unpacking-eventtranscript/forensic-quick-wins-with-eventtranscript), the timestamp here is an excellent forensic source for us. In his article he points out that he _**"observed no more than a four-second delta between the earliest timestamp recorded (Prefetch) and the latest timestamp recorded (MFT Last Access 0x10 and Sysmon Event 1)"**_

+ **"AppId"**
  + Contains the name of the binary and also the SHA1 hash of the same
  + As per above the hash is d05defe2c8efef10ed5f1361760fa0ae41fa79f5 and binary is notepad.exe.

The hash is stored in Amcache.hve style with four zeroes (0000) preceding the hash value.

While experimenting on this I found that sometimes the field can record an incorrect hash of a binary provided a few things. We delve into this now.

For this experiment, let us create a simple exe which uses MessageBox. Something like below. Compile this and run in the VM.

```python
#include <stdio.h>
#include <windows.h>

int main()
{
  char buf[1024];
  snprintf(buf, 1024, "Hello World");
  MessageBox(0, buf, "GUI Testing", 0);
  return 0;
}
```


Now lets follow some simple steps.

### Step 1

Place the binary in the Desktop and run it. As seen below, the hash recorded in the diagnostic data telemetry matches with the correct hash of the binary.

![](/images/windows-diag-data/Test-bin-original.PNG)

### Step 2

Delete the binary from the desktop and compile a new binary with the same name. Changed the code slightly. Copy this new binary to `Desktop` and also to `Documents` folders.

Old hash: 8888aec37665dd0f7bc0ff03a55ec048ec731076

New hash: 415fb677622142941af41a7d1c4dbd5d05339742


![](/images/windows-diag-data/powershell-window.png)

### Step 3

Execute both the binaries one after the other and wait for the event to be recorded. Now as we can see in the below screenshots, the hash recorded for the same binary present in Desktop and Documents is different.

![](/images/windows-diag-data/test-documents.png)
![](/images/windows-diag-data/test-desktop.png)

Here is a small recording of the above. This artifact is definitely quite interesting and I think there is a lot more to dig in to evaluate its forensic value. Will continue in the next blog.

<figure class="video_container">
  <video allowfullscreen="false" width="100%" height="auto" controls autoplay>
    <source src="/images/windows-diag-data/video.mp4" type="video/mp4">
  </video>
</figure>