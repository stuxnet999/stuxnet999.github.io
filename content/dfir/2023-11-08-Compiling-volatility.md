---
title: Compiling Volatility for Windows
date: 2023-11-08
draft: false
slug: 'compiling-volatility-windows'
author: 'Abhiram Kumar'
categories:
  - DFIR
tags:
  - Windows Memory Analysis
  - Volatility
toc: true
---

A detailed guide to compile your Volatility 2.6.1 and 3 binaries for Windows.

## Introduction

Volatility is a popular Python-based memory analysis framework which is used by almost everyone interested in memory forensics. I like many use the tool by directly running the script in Python but I have seen quite a few scenarios where having the tool as an executable binary is much preferred. Also though Volatility 2.6.1 is no longer being developed it is still being used by many so I am including how to compile it in this blogpost as well.

## Compiling Vol 2.6.1 For Windows

### Step 1 - Installing Python 2.7.18

Head over to <https://www.python.org/downloads/release/python-2718/> and download the Windows x86-64 MSI installer. Volatility 2 is built on Python 2 and Python 2.7.18 was the last version released. When Installing Python make sure to click on "Add Python to PATH". This helps you to call/run Python from any folder in the system.

![](/images/volatility-compile/python2-install.png)

Once that is done, open up the command prompt anywhere and just enter `python` and you should see the interpreter shell.

![](/images/volatility-compile/python2-running.png)

### Step 2 - Download/Clone Volatility

Go to <https://github.com/volatilityfoundation/volatility/> and download the repository to your local system. If you prefer to use git and clone it instead, you can do that too. Whatever works.

![](/images/volatility-compile/github-vol2.png)

### Step 3 - Resolving Dependency issues

Extract it to a preferred location (mine is Desktop) and open a Powershell window there.

![](/images/volatility-compile/powershell-vol2.png)

After running the command mentioned in the above screenshot `python vol.py -h` we can see volatility throw a few "warnings". This indicates missing dependencies. To resolve the issue enter the following commands - 

```bash
pip install distorm3==3.5.2
```

Now when you run that command, you should see the following error

![](/images/volatility-compile/vc9.png)

However, the suggested link does not work. We use the Wayback machine to get this done. Navigate to <https://web.archive.org/web/20190720195601/http://www.microsoft.com/en-us/download/confirmation.aspx?id=44266> and download the installer. After that execute the **VCForPython27.msi** binary. Once it is installed, run the above commands again. This time distorm should be successfully installed.

```bash
<--snip-->
Installed c:\python27\lib\site-packages\distorm3-3.5.2-py2.7-win-amd64.egg
Processing dependencies for distorm3==3.5.2
Finished processing dependencies for distorm3==3.5.2
```

Now we need to install other remaining dependencies. For this run

```bash
pip install pycrypto==2.6.1
pip install yara-python==3.11.0
pip install Pillow==2.7.0
pip install openpyxl
pip install ujson==1.35
```

After this is done, you can run `python vol.py -h` and we can see that we no longer come across any errors. Everything is sorted now. All we need now is the exe.

### Step 4 - Compiling EXE Using PyInstaller

PyInstaller is the Python module needed for this. To install the one stable for our needs run

```bash
pip install pyinstaller==3.5
```

Once it is installed, navigate to the volatility folder and run the following

```bash
pyinstaller --onefile pyinstaller.spec
```

You will now find a binary **volatility.exe** within the dist directory.

![](/images/volatility-compile/pyinstaller-vol2.png)

### Step 5 - Test Run

Here is a small video of me testing the newly compiled exe on a [MemLabs](https://github.com/stuxnet999/MemLabs) image.

<figure class="video_container">
  <video allowfullscreen="false" width="100%" height="auto" controls autoplay>
    <source src="/images/volatility-compile/vol2.mp4" type="video/mp4">
  </video>
</figure>

## Compiling Volatility 3 For Windows

### Step 1 - Install Python 3

**Note**: At the time of writing this article, Python 3.12 is the latest version but I am using Python 3.10.6

Just like what we did when installing Python 2, here also, make sure to select the "Add python.exe to PATH" option.

![](/images/volatility-compile/py3-install.png)

### Step 2 - Download/Clone Volatility 3

Download/Clone the Volatility 3 repository from here - <https://github.com/volatilityfoundation/volatility3>

![](/images/volatility-compile/vol3.png)

### Step 3 - Install Dependencies

Open a cmd/PowerShell window in the vol3 directory and run the following command

```bash
pip install -r .\requirements.txt
```

If you see the following error, go to <https://visualstudio.microsoft.com/visual-cpp-build-tools/> and install the latest version of Microsoft Visual C++.

![](/images/volatility-compile/yara-vol3-error.png)

After the required C++ version is installed re-run the command again and you should see this

![](/images/volatility-compile/yara-vol3-solved.png)

### Step 4 - Compiling EXE Using PyInstaller

Install pyInstaller using

```bash
pip install pyinstaller==6.1.0
```

Once that is done, navigate to the volatility3 repo and run

```bash
pyinstaller vol.spec
```

And you should see this

```bash
265 INFO: PyInstaller: 6.1.0
281 INFO: Python: 3.10.6
281 INFO: Platform: Windows-10-10.0.18362-SP0
3109 INFO: Extending PYTHONPATH with paths
['C:\\Users\\TestUser\\Desktop\\volatility3-develop']
3234 INFO: Appending 'binaries' from .spec
3234 INFO: Appending 'datas' from .spec
.
.
.
.
<snip>
.
.
.
15672 INFO: checking EXE
15687 INFO: Building EXE because EXE-00.toc is non existent
15687 INFO: Building EXE from EXE-00.toc
15687 INFO: Copying bootloader EXE to C:\Users\TestUser\Desktop\volatility3-develop\dist\vol.exe
15687 INFO: Copying icon to EXE
15687 INFO: Copying 0 resources to EXE
15687 INFO: Embedding manifest in EXE
15703 INFO: Appending PKG archive to EXE
15719 INFO: Fixing EXE headers
15859 INFO: Building EXE from EXE-00.toc completed successfully.
```

The compiled executable should be present in the dist folder.

## Optional - Volshell

Compiling volshell is very easy as well. Now that all the dependencies are satisfied, a simple one-line command would be enough

```bash
pyinstaller volshell.spec
```

```bash
296 INFO: PyInstaller: 6.1.0
296 INFO: Python: 3.10.6
296 INFO: Platform: Windows-10-10.0.18362-SP0
2281 INFO: Extending PYTHONPATH with paths
['C:\\Users\\TestUser\\Desktop\\volatility3-develop']
2405 INFO: Appending 'binaries' from .spec
2405 INFO: Appending 'datas' from .spec
2421 INFO: checking Analysis
2421 INFO: Building Analysis because Analysis-00.toc is non existent
2421 INFO: Initializing module dependency graph...
2421 INFO: Caching module graph hooks...
2453 INFO: Analyzing base_library.zip ...
.
.
.
.
.
<snip>
.
.
.
.
8062 INFO: Loading module hook 'hook-distutils.util.py' from 'C:\\Users\\TestUser\\AppData\\Local\\Programs\\Python\\Python310\\lib\\site-packages\\PyInstaller\\hooks'...
8234 INFO: Loading module hook 'hook-packaging.py' from 'C:\\Users\\TestUser\\AppData\\Local\\Programs\\Python\\Python310\\lib\\site-packages\\PyInstaller\\hooks'...
8484 INFO: Processing pre-safe import module hook win32com from 'C:\\Users\\TestUser\\AppData\\Local\\Programs\\Python\\Python310\\lib\\site-packages\\_pyinstaller_hooks_contrib\\hooks\\pre_safe_import_module\\hook-win32com.py'.
8937 INFO: Loading module hook 'hook-setuptools.msvc.py' from 'C:\\Users\\TestUser\\AppData\\Local\\Programs\\Python\\Python310\\lib\\site-packages\\PyInstaller\\hooks'...
9234 INFO: Loading module hook 'hook-difflib.py' from 'C:\\Users\\TestUser\\AppData\\Local\\Programs\\Python\\Python310\\lib\\site-packages\\PyInstaller\\hooks'...
9796 INFO: Looking for ctypes DLLs
9812 INFO: Analyzing run-time hooks ...
9812 INFO: Including run-time hook 'C:\\Users\\TestUser\\AppData\\Local\\Programs\\Python\\Python310\\lib\\site-packages\\PyInstaller\\hooks\\rthooks\\pyi_rth_inspect.py'
9812 INFO: Including run-time hook 'C:\\Users\\TestUser\\AppData\\Local\\Programs\\Python\\Python310\\lib\\site-packages\\PyInstaller\\hooks\\rthooks\\pyi_rth_pkgutil.py'
9812 INFO: Including run-time hook 'C:\\Users\\TestUser\\AppData\\Local\\Programs\\Python\\Python310\\lib\\site-packages\\PyInstaller\\hooks\\rthooks\\pyi_rth_multiprocessing.py'
9812 INFO: Including run-time hook 'C:\\Users\\TestUser\\AppData\\Local\\Programs\\Python\\Python310\\lib\\site-packages\\PyInstaller\\hooks\\rthooks\\pyi_rth_pkgres.py'
9827 INFO: Including run-time hook 'C:\\Users\\TestUser\\AppData\\Local\\Programs\\Python\\Python310\\lib\\site-packages\\PyInstaller\\hooks\\rthooks\\pyi_rth_setuptools.py'
9827 INFO: Looking for dynamic libraries
10296 INFO: Extra DLL search directories (AddDllDirectory): []
10296 INFO: Extra DLL search directories (PATH): []
10563 INFO: Warnings written to C:\Users\TestUser\Desktop\volatility3-develop\build\volshell\warn-volshell.txt
10624 INFO: Graph cross-reference written to C:\Users\TestUser\Desktop\volatility3-develop\build\volshell\xref-volshell.html
10656 INFO: checking PYZ
10656 INFO: Building PYZ because PYZ-00.toc is non existent
10656 INFO: Building PYZ (ZlibArchive) C:\Users\TestUser\Desktop\volatility3-develop\build\volshell\PYZ-00.pyz
11171 INFO: Building PYZ (ZlibArchive) C:\Users\TestUser\Desktop\volatility3-develop\build\volshell\PYZ-00.pyz completed successfully.
11187 INFO: checking PKG
11187 INFO: Building PKG because PKG-00.toc is non existent
11187 INFO: Building PKG (CArchive) volshell.pkg
13266 INFO: Building PKG (CArchive) volshell.pkg completed successfully.
13266 INFO: Bootloader C:\Users\TestUser\AppData\Local\Programs\Python\Python310\lib\site-packages\PyInstaller\bootloader\Windows-64bit-intel\run.exe
13266 INFO: checking EXE
13281 INFO: Building EXE because EXE-00.toc is non existent
13297 INFO: Building EXE from EXE-00.toc
13297 INFO: Copying bootloader EXE to C:\Users\TestUser\Desktop\volatility3-develop\dist\volshell.exe
13328 INFO: Copying icon to EXE
13328 INFO: Copying 0 resources to EXE
13328 INFO: Embedding manifest in EXE
13328 INFO: Appending PKG archive to EXE
13343 INFO: Fixing EXE headers
13421 INFO: Building EXE from EXE-00.toc completed successfully.
```
The compiled exe should be in the **dist** folder.


## Conclusion

This article aimed to make the process of compiling your volatility binary easy and I hope the objective was achieved. If in case you run into more errors, please ping me on Twitter at <https://twitter.com/_abhiramkumar>.

Also, I am now maintaining a repository which contains the volatility binaries - <https://github.com/stuxnet999/volatility-binaries>. If there are any errors here please let me know and I will fix them.