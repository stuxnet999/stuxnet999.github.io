---
title: Deep Dive Into Windows Diagnostic Data & Telemetry (EventTranscript.db) - PART 2
date: 2023-08-25
draft: false
categories:
  - DFIR
tags:
  - Windows Diagnostic Data
author: Abhiram Kumar
slug: windows-diagnostic-data-part-2
toc: true
---

A small article detailing my recent experiments with Windows Diagnostic Data Telemetry (EventTranscript.db).

<!--more-->

## Where we left off

In my [last post](https://stuxnet999.github.io/2023/08/15/Deep-dive-into-windows-diagnostic-data.html), we explored a way to sort of **"confuse"** the Windows diagnostic telemetry to record the wrong hash of a binary in the **Win32k.TraceLogging.AppInteractivitySummary** events. In this article, I will continue a bit on that and explore some other scenarios as well.

## Scenario 1

### SHA1 hash in Amcache.hve vs EventTranscript.db

Just as I published my previous findings, a few folks from the DFIR community were intrigued about the hash format in EventTranscript.db being quite similar to Amcache.hve. This naturally brought up the question of whether EventTranscript.db also calculated the hash of the first **31457280 bytes** just as Amcache did. The answer is **YES!**

In the below pictures, I calculate the hash of a binary on my end and also observe the recordings of both Amcache and EventTranscript.db.

![](/images/windows-diag-data/stripped-hash.png)


![](/images/windows-diag-data/amcache.png)

## Scenario 2

### Does EventTranscript.db not record CLI binaries?

Let us create 2 binaries with similar code.

```c
// cli.c

#include <stdio.h>

int main()
{
    printf("Hello World");
    return 0;
}
```

```c
// gui.c

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

+ file_cli.exe - This prints "Hello World" to the console.
+ file_gui.exe - Which prints "Hello World" via MessageBox

Also, it is to be noted that when compiling, make sure to give the **-mwindows** switch (basically utilizing the GUI subsystem). However, when we run both binaries, something quite interesting happens. When the **file_cli.exe** is run, the recorded binary name is conhost.exe but when the **file_gui.exe** is run, we get the expected result. So does EventTranscript.db not record binaries with the command line interface or is it just because the **binary wasn't compiled with the GUI switch?**

Let us test it.

## Scenario 3

### Compiling the CLI code with the GUI switch

For this let us change the code in **cli.c** slightly by adding 1-2 lines. We modify it to write "Hello World" to a text file.

```c

//cli.c (modified)

#include <stdio.h>

int main()
{
    FILE *fp;
    fp = fopen("test.txt", "w+");
    fprintf(fp, "Hello World");
    fclose(fp);
    return 0;
}
```

Now we compile it with the **-mwindows** switch and run it in the VM and see what is recorded. This time it is even strange. The event is **not even recorded**. Please take a look at the below screen recording.

<figure class="video_container">
  <video allowfullscreen="false" width="100%" height="auto" controls autoplay>
    <source src="/images/windows-diag-data/cli-gui-switch.mp4" type="video/mp4">
  </video>
</figure>

From the screen recording we can see that the binary did run successfully as we confirmed it by checking the contents of the text file **test.txt**. Quite strangely we see that the execution of notepad.exe is recorded but we don't see the execution of our binary. Why is this the case? Honestly, I don't have any clue.

Let us check the system's Amcache.hve to see if the binary **file_cli_gui-switch.exe** was recorded. Quite interestingly, Amcache does record this file

![](/images/windows-diag-data/amcache-recorded.png)

### Compile the GUI code without the GUI switch

Since we tried compiling and running the CLI code with the GUI switch, let us try the other way around as well. Let us utilize the same **gui.c** code and compile it without the **-mwindows** switch.

![](/images/windows-diag-data/file-gui-no-gui-switch.PNG)

Let us run the binary in the VM and examine the telemetry recorded in EventTranscript.db.

![](/images/windows-diag-data/no-gui-run.PNG)

As from above, we see that running the binary launches MessageBox and also a console window which makes the binary use the console subsystem. However quite strangely, this time we see that EventTranscript.db records both our binary **file_gui-no-gui-switch.exe** and **conhost.exe** as well.

![](/images/windows-diag-data/file-no-gui-conhost.PNG)

![](/images/windows-diag-data/file-no-gui.PNG)

Now this is quite strange too. Previously our binary (cli.c -> file_cli.exe) was not recorded and I assumed that this was because of the absence of the GUI switch but now we see that the execution of binary (file_gui-no-gui-switch.exe) is recorded and along with that, we see another event of conhost.exe as well.

So on what basis is EventTranscript.db recording/not recording the execution of binaries? Remains a mystery to me as of now. I guess that it has a GUI component. Please let me know. Would love to discuss this.

Video recording of the same -

<figure class="video_container">
  <video allowfullscreen="false" width="100%" height="auto" controls autoplay>
    <source src="/images/windows-diag-data/no-gui-switch.mp4" type="video/mp4">
  </video>
</figure>

## Scenario 4

### Launching PowerShell from a custom binary

Us DFIR folk regularly see malware utilize PowerShell for various things - Downloading more files, recon etc.... So let us write a simple code to launch PowerShell using CreateProcess() and system() and see what events EventTranscript.db records. Simple code -

```c

// powershell-exe.c

#include <windows.h>
#include <stdio.h>

int main()
{
    STARTUPINFO stInfo;

    PROCESS_INFORMATION proInfo;

    ZeroMemory( &stInfo, sizeof(stInfo) );

    stInfo.cb = sizeof(stInfo);

    ZeroMemory( &proInfo, sizeof(proInfo) );

    char cmdArgs[] = "powershell -c net user TestUser >> createprocess_powershell.txt";
    if(!CreateProcess("C:\\Windows\\System32\\WindowsPowerShell\\v1.0\\powershell.exe", cmdArgs, NULL, NULL, FALSE, 0, NULL, NULL, &stInfo, &proInfo))
    {

        printf("CreateProcess failed (%d).\n", GetLastError());
        return 0;

    }

    WaitForSingleObject( proInfo.hProcess, INFINITE );

    CloseHandle(proInfo.hProcess);

    CloseHandle(proInfo.hThread);

    system("powershell -c net user TestUser >> system_powershell.txt");

    char buf[1024];

    snprintf(buf, 1024, "Hello World");

    MessageBox(0, buf, "GUI Testing", 0);

    return 0;
}

```

From the above code, we aim to launch PowerShell instances to collect user info and write it to 2 separate text files - **createprocess_powershell.txt** and **system_powershell.txt**.

Let us compile the code with **-mwindows** switch to utilize the GUI subsystem. Here we are expecting 2 events - one is our binary **powershell-test.exe** and the **powershell.exe** (as we are launching the PowerShell process).

![](/images/windows-diag-data/file-powershell-test.PNG)

Upon running the binary we only see the Win32kTraceLogging.AppInteractivitySummary event corresponding to **powershell-test.exe** but we don't see anything corresponding to the PowerShell processes we launched. We only get conhost.exe and I guess it was expected.

Screen recording corresponding to the above test - 

<figure class="video_container">
  <video allowfullscreen="false" width="100%" height="auto" controls autoplay>
    <source src="/images/windows-diag-data/powershell-test.mp4" type="video/mp4">
  </video>
</figure>

## Conclusion

It was pretty interesting to spend some time working on this artifact trying out different scenarios of how various events are recorded. I am sure there can be many more such scenarios - just a matter of permutations and combinations. I am quite to excited to discuss more on this. Reach out to me via [Twitter](https://twitter.com/_abhiramkumar) (Now X :P) or [LinkedIn](https://www.linkedin.com/in/abhiramkumarp/).