---
layout:         post
title:          Cleanse ME firmware on SandyBridge and IvyBridge platforms
date            2016-11-17
auther:         persmule
mail:           persmule@tya.email, persmule@gmail.com
summary:        Record how I "cleanse" the ME firmware on my Thinkpad X220, in order to present its way.
categories:     Firmware
---

# Cleanse ME firmware on SandyBridge and IvyBridge platforms

## 00 ME: Management Engine

First introduced in Intel's 965 Express Chipset Family, the Intel Management Engine (ME) is a separate computing environment physically located in the (G)MCH chip (for Core 2 family CPUs which is separate from the northbridge), or PCH chip replacing ICH(for Core i3/i5/i7 which is integrated with northbridge).

The ME consists of an individual processor core, code and data caches, a timer, and a secure internal bus to which additional devices are connected, including a cryptography engine, internal ROM and RAM, memory controllers, and a direct memory access (DMA) engine to access the host operating system's memory as well as to reserve a region of protected external memory to supplement the ME's limited internal RAM. The ME also has network access with its own MAC address through the Intel Gigabit Ethernet Controller integrated in the southbridge (ICH or PCH).

The Intel Management Engine with its proprietary firmware has complete access to and control over the PC: it can power on or shut down the PC, read all open files, examine all running applications, track all keys pressed and mouse movements, and even capture or display images on the screen. And it has a network interface that is demonstrably insecure, which can allow an attacker on the network to inject rootkits that completely compromise the PC and can report to the attacker all activities performed on the PC. It is a threat to freedom, security, and privacy that can't be ignored.

## 01 Early efforts to remove ME

The ME's boot program, stored on the internal ROM, loads a firmware "manifest" from the PC's SPI flash chip. This manifest is signed with a strong cryptographic key, which differs between versions of the ME firmware. If the manifest isn't signed by a specific Intel key, the boot ROM won't load and execute the firmware and the ME processor core will be halted. 

The ME working with Core 2 processors (Q43, Q45, GM45 and the like) can be disabled by setting a couple of values in the SPI flash memory. The ME firmware can then be removed entirely from the flash memory space. [libreboot](https://libreboot.org/) [does this](https://libreboot.org/docs/hcl/gm45_remove_me.html) on the Intel 4 Series systems that it supports, such as the Libreboot X200 and Libreboot T400. Later ME found on all systems with an Intel Core i3/i5/i7 CPU and a PCH, include "ME Ignition" firmware that performs some hardware initialization and power management. If the ME's boot ROM does not find in the SPI flash memory an ME firmware manifest with a valid Intel signature, the whole PC will shut down after 30 minutes.

(The above two paragraphs are excerpted from [this article](https://libreboot.org/faq/#intelme), with some minor modifications)

## 02 Minimize ME's power on platforms with PCH

As mentioned above, completely removing the ME is hardly possible on platforms with PCH, so (at least) my goal on such platforms should be:

Leave minimalist ME function to keep the whole system stable (thus prevent the 30-minutes-shutdown [Defective by Design](https://defectivebydesign.org/)), and then remove all remaining function unrelated to this, especially those threatening our freedom, security, and privacy.

ME's sectional and modular design makes it possible. Different ME modules are stored in different partitions in the SPI flash, and their signature are verified separately, so it is possible to complete remove one modules with its signature without interfering another.

On sep. 2016, [Trammell Hudson](mailto:hudson@trmm.net) detected that [erasing the first 4kiB page of the ME region in the SPI flash did not shutdown his x230 30 minutes later](https://www.coreboot.org/pipermail/coreboot/2016-September/082016.html).

Finally on Nov. 2016, [Nicola Corna](mailto:nicola@corna.info) and [Federico Amedeo Izzo](federico.izzo42@gmail.com) found that [Sandy Bridge accepts an Intel ME firmware with just the FTPR partition, both with and without a valid FPT (the partition table of the Intel ME image)](https://www.coreboot.org/pipermail/coreboot/2016-November/082331.html), and they wrote [a python script that removes all the non-fundamental partitions and creates a new FPT with a single FPTR partition entry](http://www.coreboot.org/pipermail/coreboot/attachments/20161104/995e9e5d/attachment-0005.obj), and few days later they published the script [on github.com](https://github.com/corna/me_cleaner).

With that script and coreboot's utilities, I successfully cleansed the ME firmware on my x220, with vendor bios untouched.

## 03 Great effort needs great tools.
