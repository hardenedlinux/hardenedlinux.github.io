---
layout: post
title: "云基础架构之CHIPSEC固件安全基线"
summary: 不论是数据中心里的服务器还是工作站，厂商的闭源实现还是自由固件社区的开源实现都必须面临这个层面的安全风险，目前我们的测试数据显示服务器方面自从Haswell开始都比较注重固件的安全基线设置，而工作站和台式机则直到Skylake依然有不少机器会遗漏安全防护
categories: system-security
---

by Shawn C[ a.k.a "citypw"]

## 云基础架构之CHIPSEC固件安全基线

不论是数据中心里的服务器还是工作站，厂商的闭源实现还是自由固件社区的开源实现都必须面临这个层面的安全风险，目前我们的测试数据显示服务器方面自从Haswell开始都比较注重固件的安全基线设置，而工作站和台式机则直到Skylake依然有不少机器会遗漏安全防护，McAfee高级威胁研究团队成员在今年BlackHat公布的数据也显示了不少Gigabyte和MSI主板也存在安全隐患。过去数十年的固件层面的攻防进化中产生的一些攻击平面和方法已经多到足以建立基础的安全基线检查，McAfee高级威胁团队此前开源了固件安全检测工具[CHIPSEC](https://github.com/chipsec/chipsec)框架对硬件平台实现进行了抽象可以更容易的编写固件安全检测的代码，CHIPSEC的开源是对于防御者的福音从而结束了硬编码的年代;-) 据McAfee高级威胁研究团队公开的[固件安全培训](https://github.com/advanced-threat-research/firmware-security-training)，大概有以下安全问题需要检测：

| 安全风险           | CHIPSEC模块    | 引用          |
|:---------------:|:-----------------:|:-------------------:|
| SMRAM Locking   | common.smm        | [CanSecWest 2006](http://www.ssi.gouv.fr/archive/fr/sciences/fichiers/lti/cansecwest2006-duflot.pdf)|
| BIOS Keyboard Buffer Sanitization | common.bios_kbrd_buffer | [DEFCON 16](http://www.slideshare.net/endrazine/defcon-16-bypassing-preboot-authentication-passwords-by-instrumenting-the-bios-keyboard-buffer-practical-low-level-attacks-against-x86-preboot-authentication-software) |
| SMRR Configuration | common.smrr | [ITL 2009](http://www.invisiblethingslab.com/resources/misc09/smm_cache_fun.pdf), [CanSecWest 2009](http://cansecwest.com/csw09/csw09-duflot.pdf) |
| BIOS Protection | common.bios_wp | [BlackHat USA 2009](http://www.blackhat.com/presentations/bh-usa-09/WOJTCZUK/BHUSA09-Wojtczuk-AtkIntelBios-SLIDES.pdf), [CanSecWest 2013](https://cansecwest.com/slides/2013/Evil%20Maid%20Just%20Got%20Angrier.pdf), [Black Hat](http://c7zero.info/stuff/Windows8SecureBoot_Bulygin-Furtak-Bazhniuk_BHUSA2013.pdf) [2013](https://www.blackhat.com/us-13/briefings.html), [NoSuchCon 2013](http://www.nosuchcon.org/talks/D2_01_Butterworth_BIOS_Chronomancy.pdf) |
| SPI Controller Locking | common.spi_lock | [Flashrom](http://www.flashrom.org/), [Copernicus](http://www.mitre.org/capabilities/cybersecurity/overview/cybersecurity-blog/copernicus-question-your-assumptions-about) |
| BIOS Interface Locking | common.bios_ts | [PoC 2007](http://powerofcommunity.net/poc2007/sunbing.pdf) |
| Secure Boot variables with keys and configuration are protected | common.secureboot.variables | [UEFI 2.4 Spec](http://uefi.org/) , All Your Boot Are Belong To Us ([here](https://cansecwest.com/slides/2014/AllYourBoot_csw14-intel-final.pdf) & [here](https://cansecwest.com/slides/2014/AllYourBoot_csw14-mitre-final.pdf)) |
| Memory remapping attack | remap | [Preventing and Detecting Xen Hypervisor Subversions](http://www.invisiblethingslab.com/resources/bh08/part2-full.pdf) |
| DMA attack against SMRAM | smm_dma | [Programmed I/O accesses: a threat to VMM?](http://www.ssi.gouv.fr/archive/fr/sciences/fichiers/lti/pacsec2007-duflot-papier.pdf), [System Management Mode Design and Security Issues](http://www.ssi.gouv.fr/uploads/IMG/pdf/IT_Defense_2010_final.pdf) |
| SMI suppression attack | common.bios_smi | [Setup for Failure: Defeating Secure Boot](https://www.hackinparis.com/sites/hackinparis.com/files/JohnButterworth.pdf) |
| Access permissions to SPI flash descriptor | common.spi_desc | [Flashrom](http://www.flashrom.org/) |
| Access permissions to UEFI variables defined in UEFI Spec | common.uefi.access_uefispec | [UEFI 2.4 Spec](http://uefi.org/) |
| Module to detect PE/TE Header Confusion Vulnerability | tools.secureboot.te | [All Your Boot Are Belong To Us](https://cansecwest.com/slides/2014/AllYourBoot_csw14-intel-final.pdf) |
| Module to detect SMI input pointer validation vulnerabilities | tool.smm.smm_ptr | [CanSecWest 2015](https://cansecwest.com/slides/2015/A%20New%20Class%20of%20Vulnin%20SMI%20-%20Andrew%20Furtak.pdf) |
| SPI Flash Descriptor Security Override Pin-Strap | common.spi_fdopss | FLOCKDN |
| IA32 Feature Control Lock | common.ia32cfg | IA32_Feature_Control MSR lock bit |
| Protected RTC memory locations | common.rtclock | ?? |
| S3 Resume Boot-Script Protections | common.uefi.s3bootscript | ?? |
| Host Bridge Memory Map Locks | memconfig | PCI cfg |

### 在Debian 9上安装CHIPSEC需要的包:
<pre>
apt-get install build-essential python-dev python gcc linux-headers-$(uname -r) nasm python-pip git
</pre>

如果你使用的是[PaX/Grsecurity 4.9.x](https://github.com/minipli/linux-unofficial_grsec):
<pre>
apt-get install build-essential python-dev python gcc nasm python-pip git
</pre>

### 安装[CHIPSEC](https://github.com/chipsec/)
<pre>
cd chipsec/
pip install setuptools
python setup.py install
</pre>


### 固件白名单

由于固件安全领域的特殊性，可以为承载重要业务的机器在进入生产环境前建立白名单，比如:
<pre>
chipsec_util spi dump firmware.bin
chipsec_main -m tools.uefi.whitelist -a generate,harbianN_list.json,firmware.bin
</pre>

审计时可以检查白名单:
<pre>
chipsec_main -i -n -m tools.uefi.whitelist -a check,efi_lenovo.json,/fw-content/9sjt91a.img
</pre>


### coreboot加固

[coreboot](https://www.coreboot.org/)是一个自由开源实现的固件，可以支持多种平台，我们使用过的平台主要是[x86](https://github.com/hardenedlinux/hardenedlinux_profiles/tree/master/coreboot)和[RISC-V](https://github.com/hardenedlinux/embedded-iot_profile/tree/master/docs/riscv)，由于coreboot社区的哲学是用户在享受自由（“超级开发者”模式？）的同时也必须注意所带来的安全风险，所以coreboot并不会默认在启动时开启包括写保护在内的安全特性：

![](https://pbs.twimg.com/media/DGEaPmEUQAAuP8u.jpg:large)

从安全评估的角度，在评估coreboot时并不应该使用和传统UEFI实现一样的方法，因为一些差异显而易见，比如coreboot并没有EFI runtime的特性以及涉及SMM的代码量比UEFI实现小很多，但即使如此，开源只是代表具有可审计性，开源并不直接影响安全防护能力，[coreboot也是一样](https://recon.cx/2017/montreal/resources/slides/RECON-MTL-2017-DiggingIntoTheCoreOfBoot.pdf)，对于固件基线的防护特性在coreboot上有一些是可以在运行时开启，而另一些则需要修改代码。对于前者，借助于CHIPSEC框架可以轻松的完成设置：
<pre>
# Hardening tweaks via CHIPSEC framework:
#	Enabling some security features at runtime in case of which vendor provided implementation improperly.
# WARNING: Please note that this script might put your prodcution at risk

from chipsec.chipset import *


def D_LCK_set():
	# check if BIOS_CNTL register is available
	if not cs.is_register_defined( 'PCI0.0.0_SMRAMC'  ) :
		raise Exception( "Couldn't find SMRAMC")

	regval = cs.read_register( 'PCI0.0.0_SMRAMC')
	d_lock = cs.get_register_field( 'PCI0.0.0_SMRAMC', regval, 'D_LCK')

	if d_lock == 0:
		cs.write_register( 'PCI0.0.0_SMRAMC', 0x1a)
		regval = cs.read_register( 'PCI0.0.0_SMRAMC')
		d_lock = cs.get_register_field( 'PCI0.0.0_SMRAMC', regval, 'D_LCK')
        
		if d_lock == 1:
			print "Enabled D_LCK successfully: SMRAMC: %x; D_LCK: %x" % (regval, d_lock)
	else:
		print "D_LCK is set already!"

def BIOS_WP_set():
	regval = cs.read_register( 'BC')
	ble = cs.get_control( 'BiosLockEnable')
        bioswe = cs.get_control( 'BiosWriteEnable')

	if ble != 1 or bioswe != 0:
		bioswe = cs.set_control( 'BiosWriteEnable', 0)
		ble = cs.set_control( 'BiosLockEnable', 1)
		print "BLE & BIOSWE are looking good!"
	else:
		print "BLE is set already!"

def BIOS_TS_set():
        bild = cs.get_control( 'BiosInterfaceLockDown')

        if bild != 1:
                cs.set_control( 'BiosInterfaceLockDown', 1)
                print "BiosInterfaceLockDown (BILD) is enabled!"
        else:
                print "BILD is set already!"

def TSEG_LOCK_set():
        tseg_base_lock = cs.get_control( 'TSEGBaseLock')
        tseg_limit_lock = cs.get_control( 'TSEGLimitLock')

        if tseg_base_lock !=1 or tseg_limit_lock !=1:
                cs.set_control( 'TSEGBaseLock', 1)
                cs.set_control( 'TSEGLimitLock', 1)
                print "TSEGBase & TSEGLimit are locked!"
        else:
                print"TSEGBase & TSEGLimit are set already!"

def SPI_LOCK_set():
        flockdn = cs.get_control( 'FlashLockDown')

        if flockdn != 1:
                cs.set_control( 'FlashLockDown', 1)
                print "FLOCKDN is locked!"
        else:
                print "FLOCKDN is set already!"

def BIOS_SMI_set():
        tco_en = cs.get_control( 'TCOSMIEnable')
        gbl_smi_en = cs.get_control( 'GlobalSMIEnable')
        tco_lock = cs.get_control( 'TCOSMILock')
        smi_lock = cs.get_control( 'SMILock')

        if tco_en != 1 or gbl_smi_en != 1:
                return -1
        elif tco_lock != 1 or smi_lock != 1:
                cs.set_control( 'TCOSMILock', 1)
                cs.set_control( 'SMILock', 1)
                print "TCO/SMI are locked!"
        else:
                print "TCO/SMI are set already!"

if __name__ == '__main__':
    	# hardening init...
    	cs = chipsec.chipset.cs()
    	cs.init(None, True)

    	# common.smm
    	D_LCK_set()

	# common.bios_wp
	BIOS_WP_set()

        # common.bios_ts
        BIOS_TS_set()

        # smm_dma
        TSEG_LOCK_set()

        # common.spi_lock
        SPI_LOCK_set()

        # common.bios_smi
BIOS_SMI_set()
</pre>
