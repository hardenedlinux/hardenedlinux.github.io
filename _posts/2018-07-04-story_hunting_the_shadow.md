---
layout: post
title:  隐蔽战争纪元之猎杀暗影：固件自由战争之阻击"Ring -3"世界的恶魔
summary: 自科技之神创世以来，自由软件/固件/硬件社区有非常多的敌人，而Intel ME则是最隐秘的敌人之一，Intel ME( Management Engine)是x86的平台芯片组(PCH)中的一个独立运行且完整的软硬件系统...
categories: system-security
---


# 隐蔽战争纪元之猎杀暗影：固件自由战争之阻击"Ring -3"世界的恶魔

For we wrestle not against flesh and blood, but against principalities, against powers, against the rulers of the darkness of this world, against spiritual wickedness in high places. --- Ephesians 6:12


## 初章：先知降临

自科技之神创世以来，自由软件/固件/硬件社区有非常多的敌人，而Intel ME则是最隐秘的敌人之一，[Intel ME( Management Engine)](https://github.com/hardenedlinux/firmware-anatomy/blob/master/hack_ME/me_info.md)是x86的平台芯片组(PCH)中的一个独立运行且完整的软硬件系统，自2006年加入x86芯片组（AMT 2.0）以来就备受质疑，2009年的Blackhat会议上ITL( Invisible Things Lab)的Rafal Wojtczuk和Alexander Tereshkin公开[展示了通过向特定DMA内存注入的AMT/ME漏洞](https://www.blackhat.com/presentations/bh-usa-09/TERESHKIN/BHUSA09-Tereshkin-Ring3Rootkit-SLIDES.pdf)，这是第一次揭露Ring -3 rootkit的可能，这是Intel还在大力推TPM/TXT的年代，ITL就已经从攻防的角度带给了安全社区一个在技术评估中非常>重要的信息，在Ring 0以下层级的安全研究方面，ITL是先知一般的存在，多年来他们的持续不断的发现为后来的自由固件社区提供了重要的参考。

注：ITL当年为了更好的让社区理解特权等级权限的高低，把操作系统内核称为Ring 0，Hypervisor是Ring -1（VM层的持久化可以和Ring 0的完全对应），UEFI/SMM是Ring -2（在通常情况下，标准的Ring 0（e.g：页表隔离）和Ring -1（影子页表，EPT，芯片组IOMMU/Vt-d）的防护机制对于Ring -2几乎没用），而Intel ME则被看作Ring -3。

![COREs](/images/rings.png)


## 暗影匕首的诞生

从2009年开始不断的有研究人员从各个技术层面对Intel ME的一些代码模块特别是AMT进行分析,陆续爆出一些漏洞和潜在的安全风险，但并未引起行业的高度重视，在很长时间里只是少数的自由固件社区的黑客一直在关注，但对于详细的技术评估和防御策略还是非常缺乏，直到2014年Patrick Stewin的论文描述了一个名为[DAGGER](https://depositonce.tu-berlin.de/bitstream/11303/4494/1/stewin_patrick.pdf)的rootkit如何作为外设去做键盘记录，为了测试方便，DAGGER是使用了2009年ITL的那个漏洞去实现的，DAGGER也是h4rdenedzer0在[Hardening the COREs](https://github.com/hardenedlinux/hardenedlinux_profiles/blob/master/slide/hardening_the_core.pdf)方案中固件安全的起点。


## 猎杀暗影行动

直到2016年，自由固件社区主流方案是直接删除所有ME代码模块和数据，经过研究并配合自由固件社区的测试后发现，Core 2 时期的芯片组中的ME可被完全清除，但在所有Nehalem/Westmere及更新的芯片组上去掉ME代码模块后均以30分钟机器关机而告终（当然，也有一些极端分子能忍受每30分钟重启一次电脑-_-），这困扰了社区很长的时间，直到2016年9月，Trammell Hudson发现了[删除前4kb的ME region并没有导致关机](https://www.coreboot.org/pipermail/coreboot/2016-September/082016.html)，几天后Tramell又发现了只[保留FTPR分区而其他代码模块全部删除也可以让x230不关机](https://www.coreboot.org/pipermail/coreboot/2016-September/082038.html)，2016年11月，Nicola Corna和Federico Amedeo Izzo写了一个[小工具](https://www.coreboot.org/pipermail/coreboot/attachments/20161104/995e9e5d/attachment-0005.obj)删除了大部分代码模块并且创建了一个带FPTR分区的FPT，这也成为了me_cleaner的前生，h4rdenedzer0几乎参与了每一轮测试，在当月团队成员参加俄罗斯的一些研讨会时发现Intel ME承载的一些应用已经严重威胁到了自由固件社区以及企业现有的生产环境，至此h4rdenedzer0决定一定要尽力让更多的安全人员以及自由软件社区参与到对抗Intel ME，在测试了Nicola和Federico的小工具在多台x86设备上运行无误后联系了作者并且建议把代码上传至github并且要求更多的个人和企业来参与测试，之后这个工具被正式命名为[me_cleaner](https://github.com/corna/me_cleaner)，在此期间h4rdenedzer0对更多的机型进行了测试并且公开了[测试结果](https://github.com/hardenedlinux/hardenedlinux_profiles/tree/master/coreboot)，为了更好的让更多的社区成员参与测试，我们随后公开了[标准化操作文档](https://hardenedlinux.github.io/firmware/2016/11/17/neutralize_ME_firmware_on_sandybridge_and_ivybridge.html)并于当月受到了IT媒体的[关注](https://hackaday.com/2016/11/28/neutralizing-intels-management-engine/)和[报道](https://news.ycombinator.com/item?id=13056997)，一个月后（2016年12月）在汉堡举行的欧洲最大的黑客会议33C3上自由固件社区coreboot组织的workshop让上百人成功的[测试了各自带来的机型](https://github.com/corna/me_cleaner/issues/3)，这一系列的事件是引爆2017年Intel ME事件的导火索。


## 全面战争

由于自由软件社区越来越多的人开始关注Intel ME，这也引起了逆向一流的俄罗斯安全研究人员的注意，在经历长达数月的努力，Dmitry Sklyarov于2017年3月公开了[逆向Intel ME的一些细节](https://www.troopers.de/downloads/troopers17/TR17_ME11_Static.pdf)，这使得我们可以进一步的了解最新版本的Intel ME的构造，在此之前我们研究过2015年的x86处理器Skylake中最大的特性之一是[SGX](https://github.com/hardenedlinux/firmware-anatomy/blob/master/notes/sgx.md)，SGX是基于ME的多个特性实现的，除了ARC*/Sparc性能问题，这可能是MEv11使用了x86 CPU的主要原因，在Dmitry这次的逆向中发现MEv11所运行的OS并非之前ARC*架构的ThreadX，而是基于微内核MINIX定制的系统。当时me_cleaner并没有在Skylake以及更新的处理器上做太多测试，更多的代码模块处理的细节需要me_cleaner做调整，h4rdenedzer0成员捐赠了2600欧元给me_cleaner的维护者用于购买测试设备，在经过了几个月的bug fix和大量测试me_cleaner最终也实现了在MEv11和MEv12( Skylake/Kabylake/Cannonlake)的清除工作，Skylake以上的平台大部分的ME实现仅保留RBE, KERNEL, SYSLIB和BUP这几个模块。

2017年8月，Mark Ermolov和Maxim Goryachy通过逆向[找到了NSA作为防御使用的ME隐藏开关](http://blog.ptsecurity.com/2017/08/disabling-intel-me.html)，指出ME其实是可以被完全关闭的，一个被称为HAP的位（在基于ARC的ME上有等效的AltMeDisable，以及更古老的、用于Core 2芯片组无ME运行的ICH_MeDisable、MCH_MeDisable和MCH_AltMeDisable）被隐藏于描述符的PCHSTRP0字段（老版本中等效位的位置不同，详见 me_cleaner 的源代码）中，这个位并没有官方文档记录，戏剧性的是HAP的全称为High Assurance Platform，是NSA发起的构建下一代安全防御体系的项目，如果NSA的机器都是开启了HAP位（ME在完成必要初始化工作BringUP后即处于关闭状态），而这个世界上绝大部分的x86机器则是默认运行ME的，从这个层面上讲ME作为后门或者“后门帮凶”对于核心基础设施的风险极高。对于高安全性需求的业务，因主CPU运行的操作系统内核不安全从而把核心业务转嫁到外设CPU运行的操作系统上并不能解决本质的问题并且引入了新的风险，而且还带来了不可审计的麻烦，[Linux内核虽然不安全](https://www.solidot.org/story?sid=53333)但依然是开源的，不存在开放审计的问题，但ME上的软硬件对于大部分人都是闭源的，可审计的问题在x86上一直没有得到解决。

2017年11月，Google的工程师在德国召开的欧洲自由固件coreboot会议上宣称对于重要业务的机器要[干掉Intel ME和不开源的UEFI固件实现](https://www.solidot.org/story?sid=54291)，因为Google也对于Intel ME的风险评估结果感到害怕。

2017年12月，俄罗斯的安全研究者在Blackhat EU上展示了[Intel ME在BringUP（BUP）阶段](http://blog.ptsecurity.ru/2018/01/intel-me.html)的“不作为”让某种特定的攻击链条成为可能，这是否是Intel故意而为之目前并没有非常确切的证据，这个漏洞即使HAP开启也无法防御，只能从内核和固件层做防御方案，针对服务器可以配合SA-00075（AMT漏洞）远程打击目标，影响2015年至2017年的所有机器，这个漏洞是自从2009年ITL的固件安全先知们开启一个时代以后的2.0序章，标志着Ring -3层级的攻防已经进入廉价化时代。

2018年6月，俄罗斯的安全研究者也在[对IDLM代码模块分析](https://github.com/ptresearch/IntelME-Crypto/blob/master/Intel%20ME%20Security%20keys%20Genealogy%2C%20Obfuscation%20and%20other%20Magic.pdf)发现了一些奇怪的现象，Intel对于一些ME的代码模块的安全性非常高，但对于IDLM这种高风险的代码模块却“不作为”，俄罗斯的安全研究人员认为不排除是Intel故意而为之，在ME相关的代码模块上都有不少


## 解决方案

对于高安全环境需求的场景中应该避免使用ME系统，包括运行于其上的一切应用。对于金融，电信等重要行业，短期内完全替换x86几乎不可能，但可以评估业务的重要性和优先级来决定是否对ME模块进行清除。

长期来看，一方面是x86平台的微码和ME是固件不可审计的主要障碍，而这个障碍Intel帮忙解决的可能性很小，另一方面，整个芯片行业进入3.0的新时代，随着RISC-V这样的开放指令级的CPU架构的兴起，芯片和固件的可审计性问题会得到逐步的解决，只有当硬件和固件变得可以被开放的审计，这才是硬件自由和固件自由的基础。


## 结语

一直有社区的朋友在询问关于Intel ME的方方面面：风险评估，后门故事，设计和实现，逆向进度，etc。这篇文章仅仅以h4rdenedzer0的角度分享，在整个猎杀暗影行动中帮助自由固件社区的黑客，安全研究者和志愿者有很多，因为一些原因不在这里一一列出。


## ME实现进化历程

2006-- 2014: 

  * 笔记本和台式机使用标准ME实现，运行在Arc*处理器上，OS是ThreadX。
  * 从Nehalem/Westmere系列开始，ME无法被完全去除。
  * 嵌入式设备使用TXE，运行于Sparc处理器，OS是未知的RTOS。
  * 服务器使用SPS，运行在Arc*处理器上，OS是ThreadX。

2015 -- 今天：
  
  * 使用Minix 3定制的OS，运行于x86处理器。
