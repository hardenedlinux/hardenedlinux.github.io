---
layout: post
title:  "HardenedLinux: The way to the Ark"
date:   2017-04-29 02:08:45
summary: PaX/Grsecurity no longer provides the public access to test patch in Apr 26 2017. In the FAQ of the announcement, PaX team and Spender listed a couple of reasons why they do this. As some people already know, it's not the whole story.

categories: announcement
---

### HardenedLinux: The way to the Ark

PaX/Grsecurity no longer [provides the public access to test patch](https://grsecurity.net/passing_the_baton.php) in Apr 26 2017. In the [FAQ of the announcement](https://grsecurity.net/passing_the_baton_faq.php), PaX team and Spender listed a couple of reasons why they do this. As some people already know, it's not the whole story. As the result of a discussion inside h4rdenedzer0, we believe that Linux foundation is the culprit behind all this result that the commercial/individual/community users losing access to the test patches. And we support this decision PaX team/Spender has made because:

* [Core Infrastructure Initiative](https://www.coreinfrastructure.org/) has been funded by 19 big corps( $1.9 mil per year) and organized by Linux foundation. [KSPP was funded by CII in the begining](https://lwn.net/Articles/663361/). [KSPP](https://www.coreinfrastructure.org/grants) is trying to [port PaX/Grsecurity features](https://hardenedlinux.github.io/system-security/2016/12/13/kernel_mitigation_checklist.html) or implement similar ones (and also port PaX/Grsecurity's implementation to more archs, e.g: arm64) to vanilla kernel. It was of very good motivation and was a very good starting point. But... till now they've hardly accomplished anything compared to what PaX/Grsecurity did, e.g. they ported some mitigations while introducing more (exploitable?) bugs or incomplete implementations.
To make it worse, Linux foundation has been doing [marketing and PR](https://forums.grsecurity.net/viewtopic.php?f=7&t=4476) to convince the public that they are the “Neo”. Those marketing PRs blatantly steal credits from PaX/Grsecurity. AFAIK, not even one KSPP maintainer has stepped out to reveal the truth to the public, and that's very unfortunately. One of h4rdenedzer0 member tried to [talk with CII/LF but being ignored](https://lwn.net/Articles/703000/).

* The ability to create stuff out of nothing( from 0 to 1) is rare. PaX/Grsecurity is the origin of OS defense mitigation and still is the most effective defense solution. If you are a GNU/Linux x86 user, you have benefited from the contribution of PaX/Grsecurity in one way or another since 2001. Your machine has some PaX/Grsecurity features to some extent. From SEGEXEC/PAGEEXEC to NX/DEP, PaX's ASLR to vanilla/OSX/Windows ASLR, KERNEXEC/UDEREF to SMEP/SMAP( PXN/PAN on armv7/arm64), etc. For many years PaX/Grsecurity has always been leading the industry. More importantly, PaX team/Spender generously shared their work with the FLOSS world in the past 16 years. Security experts have made comments about how powerful PaX/Grsecurity is in the past couple of days( See how ppl reacted on twitter or GNU/Linux distro mailinglist). Sadly, that's exactly how infosec is like these days: only the minority knows the truth. If most people hold the false assumption that KSPP can be the alternative defense solution, business supporters of PaX/Grsecurity will disappear. And that will be the last thing we want to see.

* Closing the public access doesn't make PaX/Grsecurity a non-free/libre software. Those who purchase subscriptions can access the source code. We don't see GPL violated in any way here. After all, it's PaX team/Spender's creation and they can do anything they want. We understand why PaX team/Spender do this. No one feels the pain more than PaX team/Spender do when things like Linux foundation keeping stealing credits from PaX/Grsecurity, and big corps (WinRiver/Intel) making money out of it but never contributing back, etc, happens.

* PaX/Grsecurity has been supporting the FLOSS community for a very long time While  most of us never take security and privacy serious. As a supporter of Free/libre software/firmware/hardware, please ask yourself: Where were you when PaX/Grsecurity needed help? Maybe that will wipe that thought to complain out of your mind. Just as RMS once said, our future depends on our philosophy. We make the world where we live in.

* If you are a security consultant, we wish you learn the truth and advise your customers about security in real sense instead of the cargo-cult drugs, which has gone too much for this small world.

* One more quote from the [interview with Spender](https://www.theregister.co.uk/2017/04/26/grsecurity_linux_kernel_freeloaders/):"There are many commentators and complainers today, especially when it involves free software, and very few people dedicating half of their life to creating useful original work. When those efforts suddenly get co-opted by companies using misleading marketing and essentially corporate-funded plagiarism, it's not conducive to the desire to create and publish new work. So we're refocusing our efforts back to those who respect and value our time.". The FLOSS world has been losing real hackers like [Jonathan Zdziarski](https://www.zdziarski.com/blog/?p=6296), PaX team and Spender. The world is a evil place not because of too many bad people, but because of what we called "good people" who don't do anything about it.


We've been sharing some of our works on security practices ( [STIG-4-Debian](https://github.com/hardenedlinux/STIG-4-Debian), [Debian GNU/Linux profiles](https://github.com/hardenedlinux/Debian-GNU-Linux-Profiles), etc) for servers running in data center. PaX/Grsecurity is the corner stone to most of our solutions. Evidences have revealed that PaX/Grsecurity can defeat multiple public exploits w/o any patch fixes in critical scenarios for a long run. With PaX/Grsecurity, for the 1st time we believe that we can build the defense based on [free/libre & open source software/firmware solution](https://github.com/hardenedlinux/hardenedlinux_profiles/blob/master/slide/hardening_the_core.pdf) to prevent many threats from Ring 3/0/-1/-2/-3. HardenedLinux is going to continue develop solutions of defense based on PaX/Grsecurity. From our point of view, we see no other option. Please remember this date: Apr 26 2017. This is the day we lost our Ark.

Last but not least, we'd sincerely like to thank PaX team, Spender and other contributors of PaX/Grsecurity for the past 16 years. Because of 0ldsk00l hackers like them, this world has become a better place.


## 方舟之役

PaX/Grsecurity的test patch于2017年4月26日[关闭公开下载](https://grsecurity.net/passing_the_baton.php)。在PaX team和Spender在[公告FAQ](https://grsecurity.net/passing_the_baton_faq.php)中罗列的一些为什么这么做的理由。一些人已经知道这并不是故事的全部。h4rdenedzer0经过内部讨论后，我们坚信Linux基金会是导致商业用户，个人用户和社区用户失去访问test patch的权利的罪魁祸首。以下是我们为什么支持PaX team和Spender决定的理由：

* [基础架构联盟](https://www.coreinfrastructure.org/)是由19家大厂商赞助190万美金每年，由Linux基金会管理的组织。[KSPP一开始是由基础架构联盟资助](https://lwn.net/Articles/663361/)，[KSPP](https://www.coreinfrastructure.org/grants)一直在尝试[移植和实现PaX/Grsecurity](https://hardenedlinux.github.io/system-security/2016/12/13/kernel_mitigation_checklist.html)的一些功能到主线内核，一开始的动机和起点都是不错的。但是KSPP所完成的防御性功能完全无法和PaX/Grsecurity相提并论，比如他们移植防御机制的同时也引入了bug（能被利用？）或者由于Linux内核社区政治的原因导致了防御机制不完整实现。更糟糕的是Linux基金会[不断的市场PR](https://forums.grsecurity.net/viewtopic.php?f=7&t=4476)让更多的大众误认为他们是"Neo"。这些市场PR盗窃了PaX/Grsecurity的名声。据我们所知，非常遗憾，目前为止没有一位KSPP的维护者站出来向公众揭露真相，曾经有一名h4rdenedzer0成员[尝试与LF/CII建立对话](https://lwn.net/Articles/703000/)但没有结果。

* 从0到1的创造力是稀有资源，而PaX/Grsecurity作为操作系统防御的起源即使到今天也是最有效的防御方案。如果你是x86的GNU/Linux用户，自从2001年以来你或多或少的收益于PaX/Grsecurity。你的机器有一些PaX/Grsecurity产物，从SEGEXEC/PAGEEXEC到NX/DEP，从PaX ASLR到Linux主线内核包括OSX和Windows的ASLR，从KERNEXEC/UDEREF到SMEP/SMAP( armv7/arm64的PXN/PAN)等等。多年以来PaX/Grsecurity一直领先业界。更重要的是PaX team和Spender慷慨的在过去16年中把他们的工作成功分享给了自由软件世界。一些真正的安全专家过去几天发表的评论都在赞叹PaX/Grsecurity的强大（看看twitter和一些GNU/Linux发行版的邮件列表），不幸的是，这正是信息安全行业的现状：只有少数人知道真相。如果人们幻想KSPP可以成为另外一个可选防御方案，那PaX/Grsecurity的商业支持者可能会停止资金上的支持。这是我们所不愿意看到的。

* 关闭公开下载并不意味着PaX/Grsecurity变成了非自由软件。那些购买订阅的用户依然可以访问源代码。我们没有看到任何有违反GPL的地方。总之，这是PaX team和Spender的作品，他们有权做任何的决定。我们明白为什么他们这么做。当像Linux基金会持续从PaX/Grsecurity盗取名声和大厂商（WinRiver/Intel）从PaX/Grsecurity某些特性上赚钱但从来没有任何贡献这种事情发生时，没人会比PaX team和Spender更痛苦。

* PaX/Grsecurity在支持自由软件社区很长的时间里，我们中的大多数人并没有把安全和隐私当回事。作为一名自由软件/固件/硬件的支持者，请问问自己：当PaX/Grsecurity需要帮忙时你在哪里？或许这样会打消因为没有免费的test patch而抱怨的想法。正如大胡子RMS所说的我们的未来取决于我们当前的哲学观。

* 如果你是一名安全顾问，我们希望你能学习真相然后向你的客户建议真正意义上的安全防护方案而不是自欺欺人的把戏。

* 来自最近Spender访谈的引用：“当涉及自由软件时，太多的评论家和抱怨者，太少的能花费大半生时间专注于原创的工作。当这些原创的工作被大公司抄袭并且用于误导性的市场宣传，只会越来越少的人愿意持续工作和分享。所以我们重新专注于那些尊重我们时间的人。”。自由软件世界持续的失去像[Jonathan Zdziarski](https://www.zdziarski.com/blog/?p=6296), PaX team和Spender这样真正的黑客。这个世界之所以邪恶并不是因为有太多坏人，而是因为太多被我们称为“好人”的群体的不作为。


我们在过去的一段时间里也分享了一些关于运行在数据中心里的服务器的最佳实践（[STIG-4-Debian](https://github.com/hardenedlinux/STIG-4-Debian), [Debian GNU/Linux profiles](https://github.com/hardenedlinux/Debian-GNU-Linux-Profiles), etc）。PaX/Grsecurity是我们方案的房角石。不少证据显示PaX/Grsecurity可以在极端场景下持续常时间不打补丁防御住公开的漏洞利用。有了PaX/Grsecurity，我们第一次坚信基于[自由软件和固件的防御方案](https://github.com/hardenedlinux/hardenedlinux_profiles/blob/master/slide/hardening_the_core.pdf)可以对抗来自Ring 3/0/-1/-2/-3的威胁。HardenedLinux会继续开发基于PaX/Grsecurity的防御方案。从我们的观点来看，真的没有其他可选方案。请记住这个日子：2017年4月26日。这是我们失去方舟的日子。

最后，我们真诚的感谢PaX team，Spender以及其他PaX/Grsecurity的贡献者在过去16年中的贡献。因为有了他们这样的oldsk00l黑客，世界可以变得更安全。
