# wufuc [![](https://ci.appveyor.com/api/projects/status/0s2unkpokttyslf0?svg=true)](https://ci.appveyor.com/project/zeffy/wufuc) [![](https://isitmaintained.com/badge/resolution/zeffy/wufuc.svg)](https://isitmaintained.com/project/zeffy/wufuc)

**English** | [русский](../../wiki/README-(русский)) | [Français](../../wiki/README-(Français)) | [Deutsch](../../wiki/README-(Deutsch)) | [Magyar](../../wiki/README-(Magyar)) | [Português Brasileiro](../../wiki/README-(Português-Brasileiro)) | [Italiano](../../wiki/README-(Italiano)) | [Español](../../wiki/README-(Español))

Disables the "Unsupported Hardware" message in Windows Update, and allows you to continue installing updates on Windows 7 and 8.1 systems with Intel Kaby Lake, AMD Ryzen, or other unsupported processors.

## Downloads [![](https://img.shields.io/github/downloads/zeffy/wufuc/total.svg)](../../releases)

- [**Click here for the latest stable version**](../../releases/latest)

- [~~Unstable builds~~](https://ci.appveyor.com/project/zeffy/wufuc) **Discontinued until AppVeyor adds WDK support for Visual Studio 2017 ([appveyor/ci#1554](https://github.com/appveyor/ci/issues/1554))**

## Preface

The changelog for Windows updates KB4012218 and KB4012219 included the following:

> Enabled detection of processor generation and hardware support when PC tries to scan or download updates through Windows Update.

These updates marked the implementation of a [policy change](https://blogs.windows.com/windowsexperience/2016/01/15/windows-10-embracing-silicon-innovation/) they announced some time ago, where Microsoft stated that they would not be supporting Windows 7 or 8.1 on next-gen Intel, AMD and Qualcomm processors.
This was essentially a big middle finger to anyone who decides to not "upgrade" to the steaming pile of :shit: known as Windows 10, especially considering the extended support periods for Windows 7 and 8.1 won't be ending until January 4, 2020 and January 10, 2023 respectively.

## Some people with older Intel and AMD processors are also affected!

I've received user reports of the following CPUs all being blocked from receiving updates:

- [Intel Atom Z530](../../issues/7)
- [Intel Atom D525](../../issues/34)
- [Intel Core i5-M 560](../../issues/23)
- [Intel Core i5-4300M](../../issues/24)
- [Intel Pentium B940](../../issues/63)
- [AMD FX-8350](../../issues/32)
- [AMD Turion 64 Mobile Technology ML-34](../../issues/80)

## Bad Microsoft!

If you are interested, you can read my original write up on discovering the CPU check [here](../../tree/old-kb4012218-19).

## Features

- Enables Windows Update on PCs with unsupported processors.
- Written in C, the best programming language. :sunglasses:
- Completely free (as in freedom) software.
- Does not modify any system files.
- Byte pattern-based patching, which means it will usually keep working even after new updates come out.
- Absolutely zero dependencies.

## How it works

Basically, inside a system file called `wuaueng.dll` there are two functions responsible for the CPU check: `IsDeviceServiceable(void)` and `IsCPUSupported(void)`. 
`IsDeviceServiceable` simply calls `IsCPUSupported` once, and then re-uses the result that it receives on subsequent calls.
My patch takes advantage of this behavior by patching a couple of boolean values and basically making Windows Update think that it has already checked your processor, and the result was that it is indeed supported.

- The installer registers wufuc as a custom Application Verifier provider.
- When a `svchost.exe` process starts, the Windows PE loader automatically loads wufuc into its virtual address space.
- After that, wufuc will then check the command line of the process it was loaded into, then install some API hooks when appropriate:
    * `LoadLibraryExW` hook will automatically patch `wuaueng.dll` as soon as it is loaded.
    * `RegQueryValueExW` hook is necessary to provide compatibility with attempts by other third-parties at bypassing the CPU check. (see issue [#100](../../issues/100))
- If wufuc gets loaded by a `svchost.exe` process that isn't related to Windows Update, it goes into a dormant state and no hooks are applied.

## How to deploy wufuc using Group Policy

[There is a tutorial on the Advanced Installer website that explains how to do this](http://www.advancedinstaller.com/user-guide/tutorial-gpo.html).

## How to use unattended feature in the batch setup scripts

`install_wufuc.bat` and `uninstall_wufuc.bat` both support two command line parameters that can be used alone, or combined to change the behavior of the scripts:

- `/NORESTART` - Automatically declines rebooting after the setup finishes.
- `/UNATTENDED` - Skips all prompts for user interaction, and automatically restarts unless `/NORESTART` is also specified.

These must be used from an elevated command line prompt.

## What to do if you get stuck on a black screen with just a cursor after the Windows boot animation

This will happen if wufuc somehow manages to crash the `svchost.exe` process that is responsible for displaying the login screen.
Normally this should **never ever** happen, because wufuc goes dormant in `svchost.exe` processes that are unrelated to Windows Update.
I have only encountered this during development with very unstable code, or by causing it intentionally.

However, just in case this does happen to someone, here is how to fix it:

1. [Boot into Safe Mode with Command Prompt](https://support.microsoft.com/en-us/help/17419/windows-7-advanced-startup-options-safe-mode).
2. In the command prompt type `regedit` and press enter.
3. Navigate to the key `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options`
4. Expand the `Image File Execution Options` tree.
5. Locate the `svchost.exe` sub key, right-click it and press **Delete**.
6. Reboot.
7. You should be able to log in normally again.
8. **If this happens to you, please report it in the issues tab so I can try to figure out what is causing the crash!**

## Sponsors

### [Advanced Installer](http://www.advancedinstaller.com/)

The installer packages are created with Advanced Installer using an [open source license](http://www.advancedinstaller.com/free-license.html). 
Advanced Installer's intuitive and friendly user interface allowed me to quickly create a feature complete installer with minimal effort. [Check it out!](http://www.advancedinstaller.com/)

## Special thanks

- Alex Ionescu ([@ionescu007](https://github.com/ionescu007)) for his [_"Hooking Nirvana"_ presentation at REcon 2015](https://www.youtube.com/watch?v=bqU0y4FzvT0) and its corresponding [repository of example code](https://github.com/ionescu007/HookingNirvana).
- Wen Jia Liu ([@wj32](https://github.com/wj32)) for his awesome program [Process Hacker](https://github.com/processhacker2/processhacker) which has been absolutely instrumental in the development of wufuc, and also for his [`phnt`](https://github.com/processhacker2/processhacker/tree/master/phnt) headers.
- Duncan Ogilvie ([@mrexodia](https://github.com/mrexodia)) for his [`patternfind.cpp`](https://github.com/x64dbg/x64dbg/blob/development/src/dbg/patternfind.cpp) algorithm from [x64dbg](https://github.com/x64dbg/x64dbg).
