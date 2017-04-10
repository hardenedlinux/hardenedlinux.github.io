---
layout: post
title:  "The Unseen Benefits of a Security Mindset(自Grsecurity的blog)"
date:   2017-04-10 00:08:45
summary: Shawn：这篇spender于2011年写的文章在一定程度上帮助我们更好的了解历史，不懂历史是很难洞悉未来的，不是吗？
categories: system-security
---

Shawn：这是一个越来越少人静下心读Phrack的年代，这是一个越来越多人参与每年的各种安全“销售”大会的各种PR show的年代。在一个triggering阶段的PoC在twitter上可以有数十条转发，而一个防护性质的最佳实践少有人有兴趣的年代。从行业的角度，在这个人人喊着APT和威胁情报大数据的年代，人们逐渐搞忘了真正的攻击者的面目和手法，甚至对于威胁的评估也靠纯粹的想象力，“just speculating like everyone else, I hear it's how u do infosec these days.”…“speculation is helpful, it makes us feel smart”是不可避免的现象，另一方面，数据中心，移动端（Android）以及嵌入式系统（IoT？）的基础设施的高速发展已经暴露了无数的风险，从过去12年的基础架构层面的攻防对抗来看，虽然Attacking the Core的那个Core早已从内核转移到了[Hypervisor](https://github.com/hardenedlinux/grsecurity-101-tutorials/blob/master/virt_security.md)之后又转移到了[EFI/SMM](https://github.com/hardenedlinux/firmware-anatomy/blob/master/hack_ME/firmware_security.md)最后[Intel ME](https://github.com/hardenedlinux/firmware-anatomy/blob/master/hack_ME/me_info.md)成为了新的Core。但在某种程度上讲，内核依然是一把基路伯之剑，它的一举一动(比如早该实现的[最佳实践](https://lkml.org/lkml/2017/4/5/800))依然会影响到更底层恶魔的行为（抱歉我们暂且不在这里谈这些底层恶魔持久化的问题），而这也是HardenedLinux把PaX/Grsecurity放在了极度重要的位置的原因，我们在过去的4年中无数次收益于PaX/Grsecurity的各种features，至此我们专门为[x86打造了基础架构可信链条的加固方案](https://hardenedlinux.github.io/system-security/2017/03/17/debian_hardened_boot.html)，甚至在[Android内核的原型加固测试](https://github.com/hardenedlinux/armv7-nexus7-grsec)中少量的PaX/Grsec特性轻松的击败了大规模使用的漏洞利用程序以及root相关工具，有朋友甚至开玩笑称PaX for Android为“专坑root大师”。我们很清晰（希望如此）认知攻击者的强大，更好的了解对手才能更好的做好防护，而这篇spender于2011年写的文章在一定程度上帮助我们更好的了解历史，不懂历史是很难洞悉未来的，不是吗？


## 原文：[The Unseen Benefits of a Security Mindset](https://forums.grsecurity.net/viewtopic.php?f=7&t=2574&sid=e87a5e81d136f6db4c7352eae0747be9)， 2011年3月12日

作者：spender

译者：citypw

由于PaX team仍然是通过“真正的工作”比如让[clang可以编译x86](http://comments.gmane.org/gmane.comp.compilers.clang.devel/13365)和[amd64的Linux内核](http://lwn.net/Articles/409614/)而推迟写他们的第一篇blog，我决定再次写一篇文章。今天要讨论的议题会比较短，我认为下面的数据会更有意思而且能佐证我的观点：使用PaX/Grsecurity的益处不能被简单的归结于某一个单独的特性。有另一个重要的因素；虽然无形中渗透到了我们所产出的所有代码中：一个安全的态度。成为一名“bug fetishist”（Shawn：这里暂时翻译成“恋bug癖”）并不是一个必要条件（和倾向于通过类似产出获得名声的），而是真正在乎用户和软件的安全。

一个安全态度的标准包括：
* 找到新的可以影响现有防护或者安全边界的feature
* 找到新的可以极大增加攻击平面的feature
* 抽样检查那些没有安全准则编码的新代码
* 为防护开发回归测试
* 不要假设，定期审计
* 提前预估用户错误并且防御
* 专注于最近的顶级安全研究

剩下（为了更好的理解，我们称他们为“Linux内核开发者”）的标准包括：
* 只为了讲故事而去听一个演讲而没有任何有效的进展
* 不跟进一个漏洞是否被修复
* 花了数月的时间去修复一个已知的漏洞(发行版可以再多花数月的时间)
* 从来不检查有ASLR的场景下CONFIG_GRKERNSEC_PROC_MEMMAP是否存在
 

阻断影响upstream内核的漏洞利用通常是一个得不到任何来自Linux内核社区感谢的工作。我们不会每一次都公开相关漏洞信息，虽然我[时不时的会放纵一下](http://lwn.net/Articles/285513/)。缺乏正确的安全态度导致了upstream在抄袭grsec时导致了最终的不完整feature或者不正确的实现。代码在一开始有人跟进，但没有哪怕一个人能持续多年的去维护和改进直到正确而完整的实现。一个例子是函数指针的常量化。当upstream在2007年引入了一些结构体后，只有grsec做好了相关的维护（我甚至很nice的在KERNEXEC中把security_ops和selinux_enable设置成RO）。Upstream从来没有维护过常量化的工作。有时我的公开抱怨会让一些人去跟进但最终都无果而终。经常有些人会从grsec代码中提交一系列常量化的patch，但只有一部分被upstream接受（没有理由剩下的部分不被接受）。Upstream没有人跟进连续性或者质量，他们只在乎名声和安全的面子工程。同样的，正确的安全态度容不下NIH综合症 -- 不仅仅是因为它忽略掉了其他贡献者，而是这是一种对用户不负责的行为。

为了更具体的展现之前未被看见的安全态度，我提供了以下一个PaX/Grsecurity中未公开的漏洞防护列表去对比他们（有些是9年后）重新发现的可利用bug。这不是一个完整的列表;)，但包含了一些最近公开的漏洞。

* 关掉slub caches的merging：
PaX于2009年4月加入
于2009年被复制为[KERNHEAP Phrack article](http://phrack.org/issues/66/15.html)的一部分
于[2011年3月](http://www.spinics.net/lists/linux-mm/msg15421.html)upstream给了heap加固（没有结果）的建议

* 对非特权用户限制/proc/slabinfo：
在于noir讨论后于2002年作为CONFIG_GRKERNSEC_PROC的一部分加入grsec
于2011年3月[建议upstream](https://lkml.org/lkml/2011/3/3/306)


* eip/esp在/proc/pid/stat中被适当的保护：
于2002年早些时候实现第一版CONFIG_GRKERNSEC_PROC_MEMMAP加入grsec
来自LWN的[Jake Edge于2009年4月修复](http://lkml.org/lkml/2009/4/30/265?h1=4d3afeb44daf01af97a0b9c725dcaf1453b7886c&h2=c1efa214c08fa9819a71f7e09815c7910f9a879b)，因为没有人愿意写这个patch即使相关[演讲已经公开](https://code.google.com/p/fuzzyaslr/)

* start_code/end_code在/proc/pid/stat中被适当的保护：
于2002年早些时候实现第一版CONFIG_GRKERNSEC_PROC_MEMMAP加入grsec
2009年4月没有被修复，尽管相同类型的infoleak在[同一个文件](http://lkml.org/lkml/2009/4/30/265?h1=4d3afeb44daf01af97a0b9c725dcaf1453b7886c&h2=c1efa214c08fa9819a71f7e09815c7910f9a879b)中被修复（wchan部分被误导，基于错误的描述，Tavis/Julien的工作根本[没有使用wchan](http://lwn.net/Articles/330866/)）
在Kees Cook(被提醒)查阅了grsec中的CONFIG_GRKERNSEC_PROC_MEMMAP后，最终于[2011年3月（CVE-2011-0726）被修复](https://lkml.org/lkml/2011/3/11/380)。

* /proc/pid/auxv适当的保护对抗ASLR信息泄漏：
于2007年8月被增加到grsec
漏洞于2011年1月被[重新发现](http://seclists.org/fulldisclosure/2011/Jan/421)
于2011年2月通告LKML（[CVE-2011-1020](https://lkml.org/lkml/2011/2/7/368)，目前没有被修复）


* /proc/pid/syscall适当的保护对抗信息泄漏：
这个feature是linux内核2.6.28引入的，grsec于2008年进行了加固
一个[不错的直觉](http://lkml.org/lkml/2008/7/17/529)（可惜没有结果）
揭示了”[我没对于/proc/pid/syscall进行太多考虑](http://lkml.org/lkml/2008/7/21/60)“
于2011年2月提交给LKML（没有修复，[CVE-2011-1020](https://lkml.org/lkml/2011/2/7/368))

/proc/pid/stack防护对抗信息泄漏：
这个feature是linux内核2.6.28引入的，grsec于2008年进行了加固
于2011年2月提交给LKML（没有修复，[CVE-2011-1020](https://lkml.org/lkml/2011/2/7/368)）

内核poison值指向用户空间：
2008年2月加入PaX
其中一部分被修复于[2008年7月](http://kerneltrap.org/mailarchive/linux-kernel/2008/7/14/2456844)

ACPI解释器加固对抗物理内存读写：
于2007年10月加入PaX
于[2011年2月](http://git.kernel.org/?p=linux/kernel/git/torvalds/linux-2.6.git;a=commitdiff;h=884b821fa27a5e3714d4871976d3e7c3abfa0d1b)已经修复（要么是悄悄修复或者他们根本没意识到已经修复）

模块加载加固（恰当的RX/RW分离)：
于2005年11月加入PaX（当PaX针对KERNEXEC加入模块支持）
于2011年的2.6.38作为[调试选项](http://lwn.net/Articles/422487/)加入upstream

更新:
04/26/2012：
get_robust_list()防护对抗信息泄漏
在我报告这个漏洞给Kees Cook后，get_robust_list()的ASLR信息泄漏最终于2012年4月被upstream修复。这个信息泄漏的修复于2009年在grsec中修复。 

11/04/2015：
wchan防护内核信息泄漏
最终于2015年11月[修复](http://git.kernel.org/cgit/linux/kernel/git/torvalds/linux.git/commit/?id=b2f73922d119686323f14fbbe46587f863852328)。这个patch应该是在2004年9月10日之后的grsec就已经修复。

享受余下的周末时光,
-Brad
