---
layout: post
title: Flexibility of having multiple parallel instances of Linux using WSL
---

Working with Windows can be truly a nightmare in scientific computing projects. The main reason behind this is that most of the scientific computing libraries have developed natively for Linux, and in the case of cross-platform ones, the building and installation process on Linux is more straightforward and easy to accomplish. In addition to this, the freedom you have while working with the command line interface can never be experienced in Windows with PowerShell or the traditional Command Prompt.

So why are we talking about this when Linux is there and we don't need to stick to Windows? The problem arises when you work for a big organization that doesn't allow Linux to be installed on your machine because all the operational workflow, communication systems, and things like that are going to take place in Windows (which is indeed my case at KU Leuven, although I have installed an ILLEGAL dual boot version of Linux by bypassing the organizational Bitlocker, but it's not always feasible and convenient to reboot to Windows to join a meeting, sign something or print a document).

The solution to this problem was released by Microsoft several years ago, a feature of Windows called [Windows Subsystem for Linux (WSL)](https://en.wikipedia.org/wiki/Windows_Subsystem_for_Linux), providing you with an emulated Linux kernel on top of which you can install your favorite distribution. It gives you full terminal access as well as limited support for running graphical applications (with some tricks of course). Recently, they have also extended and upgraded this feature to a real Linux kernel in WSL 2 (which I haven't tested yet), which has also some cool and long awaited features such as GPU support.

But, beside this, WSL can be tweaked to bring even more productivity especially if you want to test multiple libraries and different configurations without touching your previous working instances of configured Linux (which indeed happens every now and then in scientific computing). Doing this in a native installation of Linux (like with `chroot`) requires some effort. Having multiple instances of the same Linux distribution in WSL is not supported out-of-the-box but can be accomplished using a less-known, yet fantastic tool called [LxRunOffline](https://github.com/DDoSolitary/). The simplest way to install LxRunOffline is by downloading the compiled binaries from GitHub. Adding the executable path to the "Path" environment variable is highly recommended. After doing that, a new instance can be installed as simple as:
```
> LxRunOffline i -n <name_of_the_instance> -d <isntallation_location> -f <path_of_the_downloaded_image> -s
```
in which the `i` argument indicates the installation action (to see all available operations, execute `LxRunOffline` without any action). Then, the installed instance can be run using:
```
> LxRunOffline r -n <name_of_the_instance>
```
You can encapsulate this command in a batch file, which can be placed on Desktop (or even better somewhere in "Path", so you can run it directly from Command Prompt in any directory) for an easier access.

And now, the great flexibility comes with the freedom of choosing whatever you want as the distribution. What I use most often for this purpose is [Ubuntu Core](https://ubuntu.com/core), a light-weight Linux distribution aimed for IoT devices, but with APT package manager installed, which enables me to start with a tiny Linux installation, and then configure it to fit my needs. By doing this, I can quickly have a fresh installation of Linux to test or run something without affecting the previous configurations I have. A wide variety of Ubuntu Core images can be downloaded [here](https://partner-images.canonical.com/core/). A typical command for such a purpose can be then something like this:
```
> LxRunOffline i -n ubuntu-4.6 -d c:\wsl\ubuntu-test46 -f ubuntu-bionic-core-cloudimg-amd64-root.tar.gz -s
```
