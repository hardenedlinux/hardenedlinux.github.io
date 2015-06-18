---
layout: post
title:  "PaX/Grsecurity for Nexus 7 2013"
date:   2015-05-11 22:49:45
categories: jekyll update
---

Update( May 28 2015)

The porting work of the PaX patch already done. We tested it with Towel & KINGROOT. The result as expected: they all failed to root the Android 5.0.2 with kernel code base from 2014. Perhaps, we might try to make GRSEC & RBAC into the Android in the future.........

![KINGROOT1](/images/mr1.jpg)
![KINGROOT2](/images/mr2.jpg)
![KINGROOT3](/images/pr3.jpg)
![TOWELROOT](/images/pr2.jpg)

# [armv7-nexus7-grsec](https://github.com/hardenedlinux/armv7-nexus7-grsec)

PaX/Grsecurity patch for Nexus7, which the original version is 3.4
kernel based with a bunch of backport features and fixes. In some
particular cases, TrustZone is useless if the Android kernel were
compromised. I don\'t think we need another rootkit-friendly solution
like SELinux always does. Get rid of one entire class of vulns in
kernel would be an inevitable ways to make your device secure.


# Credit for PaX/Grsecurity

PaX/Grsecurity is the most respected 0ld sch00l community and they
have been creating the best defense-in-depth kernel hardening solution
for 14 years. What PaX/Grsecurity brings to us, is amazing and
incridble work. Unfortunately, there are a lot of reasons that
PaX/Grsecurity don\'t get the credit what they deserves. Let me make
this short: To love those who are hatred by BIG BROTHER. That\'s the 
fuc\*ing point.


# What makes us ticks

The age of [IoT( Internet of things)](https://www.iotivity.org/) is coming soon...There will be
huge numbers of devices running with diverse communication
protocols. For the simply classify, I\'ll only treat these devices as
two types: One with TCP/IP stack, or not. The one with TCP/IP stack
might have high probablity run with GNU/Linux. The one without TCP/IP
stack may be just a simple MCU stuff. The heterougenous network need
to be protected in various ways. These devices may be running on our
cars, refrigrator, or everywhere around us, which could be a risk to
our money-shitty property and even lives. That's one of most important
reasons we need to \"H A R D E N E N I N G   E V E R Y T H I N G\" by free
software.


### [PaX/Grsecurity](https://grsecurity.net/)

  NAME                        | DESCRIPTION                | AUTHORS
------------------------------|----------------------------|----------------------------
[PAGEEXEC](https://pax.grsecurity.net/docs/pageexec.txt)  |  paging based non-executable pages  |  The PaX team, Mar 15 2003
[SEGMEXEC](https://pax.grsecurity.net/docs/segmexec.txt) | segmentation based non-executable pages | The PaX team, May 1 2003
[ASLR](https://pax.grsecurity.net/docs/aslr.txt) |      address space layout randomization | The PaX team, Mar 15 2003
[MPROTECT](https://pax.grsecurity.net/docs/mprotect.txt) | mmap() and mprotect() restrictions | The PaX team, Nov 4 2003
[RANDUSTACK](https://pax.grsecurity.net/docs/randustack.txt) | userland stack randomization  | The PaX team, Feb 12 2003
[RANDKSTACK](https://pax.grsecurity.net/docs/randkstack.txt) | kernel stack randomization | The PaX team, Jan 24 2003
[RANDMMAP](https://pax.grsecurity.net/docs/randmmap.txt) | mmap() randomization  | The PaX team, Jan 24 2003
[RANDEXEC](https://pax.grsecurity.net/docs/randexec.txt) | non-relocatable executable file randomization | The PaX team, Feb 19 2003
[VMMIRROR](https://pax.grsecurity.net/docs/vmmirror.txt) | vma mirroring, the core of SEGMEXEC and RANDEXEC  | The PaX team, Oct 6 2003
[EMUTRAMP](https://pax.grsecurity.net/docs/emutramp.txt) | gcc nested function and kernel sigreturn trampolines emulation | The PaX team, May 1 2003
[EMUSIGRT](https://pax.grsecurity.net/docs/emusigrt.txt) | automatic kernel sigreturn trampoline emulation |The PaX team, Feb 19 2003
[UDEREF](https://grsecurity.net/~spender/uderef.txt) | Prevent improper userland code/date access by the kernel | The PaX team, May 15 2007

[UDEREF/amd64](http://grsecurity.net/pipermail/grsecurity/2010-April/001024.html)


### [PaX/Grsecurity](https://forums.grsecurity.net/viewforum.php?f=7&sid=0c5e947c94d1dc30e3ea8a0daa6683bd) writings

[intro](https://forums.grsecurity.net/viewtopic.php?f=7&t=2520) | Dec 30 2010

[Assorted Notes on Defense and Exploitation](https://forums.grsecurity.net/viewtopic.php?f=7&t=2521) | Dec 31 2010

[False Boundaries and Arbitrary Code Execution](https://forums.grsecurity.net/viewtopic.php?f=7&t=2522) | Jan 2 2011

[The Dangers of Copy and Paste](https://forums.grsecurity.net/viewtopic.php?f=7&t=2551) | Feb 1 2011

[The Unseen Benefits of a Security Mindset](https://forums.grsecurity.net/viewtopic.php?f=7&t=2574) | Mar 12 2011

[Much Ado About Nothing: A Response in Text and Code](https://forums.grsecurity.net/viewtopic.php?f=7&t=2596) | Apr 16 2011

[Recent Advances: How We Learn From Exploits](https://forums.grsecurity.net/viewtopic.php?f=7&t=2939) | Feb 15 2012

[Supervisor Mode Access Prevention](https://forums.grsecurity.net/viewtopic.php?f=7&t=3046) | Sep 7 2012

[Inside the Size Overflow Plugin](https://forums.grsecurity.net/viewtopic.php?f=7&t=3043) | Aug 28 2012

[Recent ARM Security Improvements](https://forums.grsecurity.net/viewtopic.php?f=7&t=3292) | Feb 18 2013

[KASLR: An Exercise in Cargo Cult Security](https://forums.grsecurity.net/viewtopic.php?f=7&t=3367) | Mar 20 2013

[Guest Blog by Rodrigo Branco: PAX\_REFCOUNT Documentation](https://forums.grsecurity.net/viewtopic.php?f=7&t=4173) | Mar 21 2015


### *GCC plugins*


[Better kernels with GCC plugins](https://lwn.net/Articles/461696/)


# History

## 2005

[grsecurity 2.1.0 and kernel vulnerabilities](http://lwn.net/Articles/118251/)

[the "Turing Attack" (was: Sabotaged PaXtest)](https://lkml.org/lkml/2005/2/8/93)


## 2009

[The future for grsecurity](https://lwn.net/Articles/313621/)


## 2010

[Brad Spengler (PaX Team/grsecurity) interview](https://slo-tech.com/clanki/10001en)

## 2011
[proactive defense: using read-only memory](http://lwn.net/Articles/415653/)


## 2012

[Why are the grsecurity patches not included in the Vanilla Kernel?]( http://unix.stackexchange.com/questions/59020/why-are-the-grsecurity-patches-not-included-in-the-vanilla-kernel)


## 2013

[How The Linux Foundation and Fedora are Addressing Workstation Security](https://lwn.net/Articles/538600/)


## 2014

[Some Links for Newbies on Grsecurity, and the Big Picture](https://forums.grsecurity.net/viewtopic.php?f=3&t=3906&p=13803&hilit=ANDROID#p13803)

[How GNU/Linux distros deal with offset2lib attack?](http://www.openwall.com/lists/oss-security/2014/12/06/14)


# Recent isues
[locking bug( it may also an issue in upstream)](https://forums.grsecurity.net/viewtopic.php?f=1&t=4143)

[RPI not booting due to randomize layout plugin](https://forums.grsecurity.net/viewtopic.php?f=3&t=3958)
