---
layout: post
title:  "PaX的技术考古之旅"
date:   2015-05-23 19:20:45
summary: PaX是针对linux kernel的一个加固版本的补丁，它让linux内核的内存页受限于最小权限原则，是这个星球上有史以来最极端和最优秀的防御系统级别0day的方案，第1版的设计和实现诞生于2000年，那可是一个没有ASLR/RELRO/NX/CANARY/FORITY/PIE都没有的年代，这些今天意义上的现代mitigation技术不管是linux/windows/macosx都多少抄袭和模仿了PaX的设计和实现，但有很多朋友会问：既然这东东这么厉害，为什么不在linux mainline里？PaX没有进入Linux内核upstream的原因不止一个，甚至有时候都不是纯粹技术本身的问题
categories: system-security
---

h4rdenedzer0会尝试翻译PaX/Grsecurity的一些文档，请关注。

## 0. 什么是Grsecurity/PaX?

[PaX](http://pax.grsecurity.net/)是针对linux kernel的一个加固版本的补丁，它让linux内核的内存页受限于
最小权限原则，是这个星球上有史以来最极端和最优秀的防御系统级别0day的方
案，第1版的设计和实现诞生于2000年，那可是一个没有
ASLR/RELRO/NX/CANARY/FORITY/PIE都没有的年代，这些今天意义上的现代
mitigation技术不管是linux/windows/macosx都多少抄袭和模仿了PaX的设计和实
现，但有很多朋友会问：既然这东东这么厉害，为什么不在linux mainline里？
PaX没有进入Linux内核upstream的原因不止一个，甚至有时候都不是纯粹技术本身的问题：

1) [PaX Team并不在意PaX是否进入Linux主干代码](http://unix.stackexchange.com/questions/59020/why-are-the-grsecurity-patches-not-included-in-the-vanilla-kernel)，但多年来有很多关心Linux内核安全的人不断的尝试把PaX的代码分拆成小的patch提交给Linux内核社区。

2) Linux内核社区认为PaX的代码难以维护，而Linux内核社区更喜欢花时间在性能和新功能上，而非安全。

3) Linux内核社区和Linux基金会受到由各个大厂商的影响，大厂商对于安全的要求取决于他们的客户的需求，如果很多真相不曝光大厂商是不会在意安全性的

4) 商业公司在意他们的主要利润来源，比如Five-eyes国家（美国，英国，加拿大，澳大利亚和新西兰）的政府项目都会统一采购SELinux的项目，所以大厂商都会花费精力去满足这一需求，这也是SELinux虽然备受争议但是一直有厂商和NSA持续投入的原因。

SELinux也是一个著名的开源MAC（强制访问控制）实现，是由NSA(美国国家安全局）于1990年代末发起的项目，于2000年以GPL自由软件许可证开放源代码，2003年[合并到Linux内核中](http://www.internetnews.com/ent-news/article.php/3317331)，过去10年中关于[是否NSA在其中放后门的争论](https://www.schneier.com/blog/archives/2008/04/nsas_linux.html)没有停过，一些人认为应该信任SELinux，因为它是以GPL自由软件许可证公开的源代码，也有人认为它是NSA参与过的项目，所以不应该信任。2013年Snowden曝光棱镜后更多的人极度的不信任NSA，认为[NSA有对Android代码植入后门的前科](http://www.zerohedge.com/news/2013-07-09/nsa-has-inserted-its-code-android-os-bugging-three-quarters-all-smartphones)，所以应该[怀疑所有NSA积极参与的项目包括SELinux](http://www.eteknix.com/nsa-has-code-running-in-the-linux-kernel-and-android/)。目前MAC的开源实现里，SELinux主要由RedHat/CentOS/Fedora社区维护，Apparmor主要由OpenSuSE/Ubuntu社区维护，关于SELinux是否应该使用是一个长久争论的话题，个人认为这取决于你的威胁建模，如果你是Five-Eyes阵营你当然应该使用SELinux，如果你是其他阵营比如德国或者中国，或许你应该考虑其他选择。

针对Linux内核的MAC实现都是基于LSM( Linux Security Module)去实现的，LSM利用了一堆CAPABILITY的
机制提供了一些限制用户态程序访问控制的接口，SELinux和Apparmor都是基于LSM开发的，注意LSM并不是一个传统意义上的linux kernel module，至少在2个地方不同于普通module:

1) 必须在bootloader启动内核时启动，不能在内核加载完后启动。

2) 不能同时启动2个LSM的实现。


但PaX Team是一群old school security hackers，他们认为LSM一方面打破了
"security as a whole"的哲学，另外一方面对于[内核漏洞没有防御能力](https://grsecurity.net/compare.php)，虽然在早年Linux内核社区以及大厂商不管是刻意还是无意的想要掩盖这一点，但[时间](https://grsecurity.net/spender_summit.pdf)证明PaX Team是正确的。其
实当人们谈到Gnu/Linux安全性比windows/OSX更好时，其实未必，至少linux内核社区并没有把安全性放在首位，Linus Torvalds从来都不是太care安全问题，不是吗？

PaX从一开始就主要关注如何防御和检测memory corruption，PaX由PaX team维护，Grsecurity主要包括了RBAC（基于角色的访问控制）和一系列对PaX的改进，Grsecurity主要由Spender维护，最近几年这2组Patch都合并到了一起发布，所以我们都称这组补丁为Grsecurity/PaX或者PaX/Grsecurity。


## 0.1 PaX的诞生

这个section描述的是这篇“过时”的[论文](https://pax.grsecurity.net/docs/pageexec.old.txt)，这是PaX的Genesis，1999年7月的
[plex86社区](http://www.plex86.org/)(old school虚拟化社区之一)打算验证一个概念，当时Pentium(包
括P6family)处理器新增加了一个功能，就是CPU把TLB区分为DTLB(数据TLB)和
ITLB(指令TLB)，TLB主要是PTE( page table entries)的缓存，因此存放着
user/kernel spaces的访问权限信息，在正常的情况下，ITLB和DTLB entries从
相同的PTE里读出相同的状态，但如果状态有所改变的话也就意味着可以把数据读
写和代码执行分开，如果这个POC能成功也就意味着可以对抗缓冲区溢出的最佳方
案，这个成为了今天的NX=>要么可读写要么可执行。
 
在PaX的初始设计文档中经过了对PTE中的2个flags的分析：
 
Present位，如果设置1，指向的page(或者page table)是存在于内存里的；如果
设置为0，page没有在内存里和保存的入口位(bits)可能会被OS用作其他用途。如
果page table或者page directory的入口需要执行地址转换(线性地址到物理地
址)时Present位被清0，会产生一个异常:page fault异常。
 
U/S位，权限管理，U->user space, S->kernel space
 
关于ITLB和DTLB的状态之间的转换这篇paper里已经有非常详细的描述，这里就不
多阐述了，linux内核的实现问题，虚拟内存管理的主要结构是vm_area_struct，
主要是描述连续的线性地址的一些属性包括
READ/WRITE/EXECUTE/SHARED/PRIVATE等，里面有2个结构体成员需要关注：
vm_flags，vm_page_prot。PaX在出现page fault的时候多增加了一些动作包括模
拟page table entry里的可访问U标志位和在模拟PTE中检查访问权限。
 
PaX的第一版的副作用也不小，
1，用户态可执行的栈是不可能的
2，性能损耗在5%--8%
 
old school社区plex86在1999年的一个概念验证建立了后来NX(目前是硬件支持)
的基础，个人觉得最有意思的地方是防御缓冲区溢出利用最早的策略是基于对于
TLB的研究导致的，这听起来怎么那么像emerging property, Out of C0ntrol?
KK? Ring the bell?


## 1. 2003年PaX谈"未来"
 
PaX在2003年的时候开始思考如何在[未来[(2003以后)在根本上根除漏洞利用](https://pax.grsecurity.net/docs/pax-future.txt)，PaX
对于W-xor-X的实现非常奏效，具体在[PAGEEXEC原始设计文档](https://pax.grsecurity.net/docs/pageexec.old.txt)里已经有所描述。
 
从defensive的平面来看，当时GNU/Linux平台主要依赖PaX的patch来进行加固(包
括ASLR)，ASLR和NX进入linux内核mainstream是后来的事情，OpenBSD和Windows
XP SP2和OSX 10.5也加入了NX，但都是抄袭PaX的设计(或许也包括实现)，
 
从offensive的平面来看，2003年的背景是stack-based overflow和string
format vuln已经泛滥，但ROP还没有大规模的流行，但old school社区对于ROP的
研究已经有相当的研究，包括Solar Designer在1997年发到[bugtraq里的讨论](http://seclists.org/bugtraq/1997/Aug/63)，之后更精彩的paper是在2001年[Phrack Issue 58的那篇论文"The advanced return-into-lib(c) exploits: PaX case study"](http://phrack.org/archives/issues/58/4.txt)。
 
注意：2003年时SK的那篇Borrowed code chunks还没有发布。
 
2003年，PaX team认为会导致漏洞利用的bug给予了攻击者(区分攻击者和黑客是
不同的term)在3个不同层面上访问被攻击的进程：

(1) 执行任意代码
(2) 执行现有代码但打破了原有的执行顺序
(3) 原有的执行顺序执行现有代码，但加载任意数据
 
NOEXEC( Non-executable pages)和MPROTECT(mmap/mprotect)能防御(1)，但有一
种情况是例外：如果攻击者能创建和写入一个文件然后mmap()到被攻击的进程空
间里，这样可以执行任意的代码。
 
ASLR在一定程度上降低了(1),(2),(3)的风险，但如果内核有信息泄露的bug例外。
PaX team在当时就认为把内核当成可信计算( Trusted Computing)的基础是一件
可笑的事情，因为内核跟用户空间一样容易遭受各种攻击。所以他们认为"未来"
需要做一些事情(注：这些事情今天都已经搞定）：
 
(a) 尝试处理(1)不能处理的那个例外情况
(b) 实现所有可能在内核态自己的防御机制
(c) 为(2)实现确定性( deterministic)防护，可能也为(3)实现类似的机制
(d) 为(2)实现概率行( probalilistic)防护以实现阻止信息泄露
 
---------------------------------------------------
处理(a)更好的解决方案是使用访问控制和可信路径执行来限制，Grsecurity今
天就是这么做的
---------------------------------------------------
 
之后这篇文档里详细的罗列了针对(a)(b)(c)(d)需要去实现的加固方案。在[这里已经能看出一些后来出现的mitigation技术](https://raw.githubusercontent.com/citypw/security-regression-testing-for-suse/master/other/vulns_hardening_assessment.log)：Stack Canary, RELRO,
pointer constant/encryption?


## 1.1 2003年PaX里vma mirroring的设计
 
在2003年的晚些时候PaX实现了[虚拟内存空间的镜像( vma mirroring)](https://pax.grsecurity.net/docs/vmmirror.txt)，vma
mirroring的目的是为了在一组物理页上做特殊文件隐射时有2个不同的线性地址，
这2个地址即使在swap-out/swap-in或者COW后依然是不会改变的。这样做的目的
为了满足几种场景：
 
1，把可执行的区域在使用SEGMEXEC隐射进入代码段。在32-bit的linux内核里的
4GB地址空间是其中3GB给用户空间，1GB给内核空间，而vma mirroring把用户空
间的3GB划分成了2个1.5GB分别是给代码段和数据段，在可执行区域里包含的数据
的部分(常量字符串，函数指针表等)都会mirroring到数据段里。
 
2，实现可执行区域的地址随机化( RANDEXEC)。
 
3，这个引出了第3种情况，就是SEGMEXEC和RANDEXEC同时激活，个人觉得这个的
效果应该和PIE＋ASLR的效果类似，不同的不是整个elf binary的代码段随机化，
而是在mirroring时对代码段和数据段进行随机化。
 
之后这篇文章开始聊到实现的问题，对于一个普通的用户态binary在执行后，内
核得做一系列的工作，fs/binfmt_elf.c里的load_elf_binary()负责进程地址空
间的一些基本的映射包括stack,动态连接器和binary本身。而文件的映射是通过
elf_map()调用do_mmap()完成的。用户态binary的第1条指令从ld.so或者binary
自己fetch到后会raise一个page fault，linux内核内存管理是按需分配内存的，
所以在binary刚执行时是没有建立有效的物理映射的。x86架构的page fault
handler在arch/i386/mm/fault.c文件里的do_page_fault()去找到vma结构体，
VMA里包含了物理页的数据(ELF文件里的代码段, etc)。
 
当时的PaX的做法大致是这样的，vma mirror是根据已经内存映射mmap()后的地址，
用户态通过mmap()是无法直接去做vma mirror请求的，所有的mmap()请求多会经
过include/linux/mm.h的do_map()，PaX扩展( SEGMEXEC)也是在这个地方处理，
原始内核通过调用do_mmap_pgoff()来调用do_mmap()，PaX在这里为了确保
SEGMEXEC能知道来自用户态和内核态的原生文件映射请求所以略过
do_mmap_pgoff()而直接调用do_mmap()，而vma mirror请求使用一些特殊参数传
递给do_mmap_pgoff():
 
    'file'    必须是NULL，因为mirror会引用相同文件的vma作为镜像
    'addr'    正常使用
    'len'    必须是0
    ‘prot'    正常使用
    'flags'    正常使用，除了一种情况：指定MAP_MIRROR和只能指定private映射
    'pgoff'    指定vma的线性起始地址作为镜像
 
文章给出了一个例子：
 
\# cp /bin/cat /tmp/
\# /tmp/cat /proc/self/maps
 
激活PaX的2个功能: SEGMEXEC, MPROTECT
 
    [1] 08048000-0804a000 R-Xp 00000000 00:0b 1109       /tmp/cat
    [2] 0804a000-0804b000 RW-p 00002000 00:0b 1109       /tmp/cat
    [3] 0804b000-0804d000 RW-p 00000000 00:00 0
    [4] 20000000-20015000 R-Xp 00000000 03:07 110818     /lib/ld-2.2.5.so
    [5] 20015000-20016000 RW-p 00014000 03:07 110818     /lib/ld-2.2.5.so
    [6] 2001e000-20143000 R-Xp 00000000 03:07 106687     /lib/libc-2.2.5.so
    [7] 20143000-20149000 RW-p 00125000 03:07 106687     /lib/libc-2.2.5.so
    [8] 20149000-2014d000 RW-p 00000000 00:00 0
    [9] 5fffe000-60000000 RW-p fffff000 00:00 0
   [10] 68048000-6804a000 R-Xp 00000000 00:0b 1109       /tmp/cat
   [11] 80000000-80015000 R-Xp 00000000 03:07 110818     /lib/ld-2.2.5.so
   [12] 8001e000-80143000 R-Xp 00000000 03:07 106687     /lib/libc-2.2.5.so
 
这个binary是一个动态连接的可执行程序，所以在执行时会映射其他的库文件。
 
[1] 这个binary文件/tmp/cat的第1个PT-LOAD段映射为有读和执行的权限，包含
了可执行的代码和只读的初始化后的数据。因为是可执行的所以被[10]镜像。
 
[2] 第2个PT_LOAD段，映射为读写权限，包含了可写的数据(所有初始化和没有初
始化的)
 
[3] brk()管理的堆，在运行时会根据malloc()/free()来调整大小
 
[4][5] 动态连接器
 
[6][7] C库 ，[4][6]被映射到了[11][12]，因为他们是可执行的。
 
[8] 一个针对C库的初始化数据的匿名映射
 
[9] 一个匿名映射包含了stack。我们能观察到这个地址在用户空间的数据部分的
结束地址，开启SEGMEXEC后是TASK_SIZE/2。
 
[10][11][12] 分别映射可执行镜像[1][4][6]。

 
激活PaX的3个功能的情况: SEGMEXEC，RANDEXEC，MPROTECT
 
    [1] 08048000-0804a000 R-Xp 00000000 00:0b 1109       /tmp/cat
    [2] 0804a000-0804b000 RW-p 00002000 00:0b 1109       /tmp/cat
        0804b000-0804d000 RW-p 00000000 00:00 0
    [3] 20000000-20002000 ++-p 00000000 00:00 0
    [4] 20002000-20003000 RW-p 00002000 00:0b 1109       /tmp/cat
        20003000-20018000 R-Xp 00000000 03:07 110818     /lib/ld-2.2.5.so
        20018000-20019000 RW-p 00014000 03:07 110818     /lib/ld-2.2.5.so
        20021000-20146000 R-Xp 00000000 03:07 106687     /lib/libc-2.2.5.so
        20146000-2014c000 RW-p 00125000 03:07 106687     /lib/libc-2.2.5.so
        2014c000-20150000 RW-p 00000000 00:00 0
    [5] 5fffe000-60000000 RW-p 00000000 00:00 0
    [6] 80000000-80002000 R-Xp 00000000 00:0b 1109       /tmp/cat
        80003000-80018000 R-Xp 00000000 03:07 110818     /lib/ld-2.2.5.so
        80021000-80146000 R-Xp 00000000 03:07 106687     /lib/libc-2.2.5.so
 
RANDEXEC有一些改变，[3]成了第1个可执行的PT_LOAD段的匿名映射，[4]成为第
2个PT_LOAD段的的mirror，[2]和[4]有相同的页偏移值，文档说[1]被[6]给
mirror后是超出了TASK/SIZE/2的范围，但个人觉得这个地方是代码段的区域所以
必然是在1.5G以上(如果数据段在0-1.5G的话)，还有就是在RANDUSTACK开启后由
于stack的第1部分不能关闭随机化，所以多比第1个例子多占了1个page，这个怎
么得出的呢？靠我真不知道，可能是fffff000 xor ffffffff = fff来的？
 
  
激活PaX的3个功能的情况: PAGEEXEC, RANDEXEC, MPROTECT
 
    [1] 08048000-0804a000 R--p 00000000 00:0b 1109       /tmp/cat
    [2] 0804a000-0804b000 RW-p 00002000 00:0b 1109       /tmp/cat
        0804b000-0804d000 RW-p 00000000 00:00 0
    [3] 40000000-40002000 R-Xp 00000000 00:0b 1109       /tmp/cat
    [4] 40002000-40003000 RW-p 00002000 00:0b 1109       /tmp/cat
        40003000-40018000 R-Xp 00000000 03:07 110818     /lib/ld-2.2.5.so
        40018000-40019000 RW-p 00014000 03:07 110818     /lib/ld-2.2.5.so
        40021000-40146000 R-Xp 00000000 03:07 106687     /lib/libc-2.2.5.so
        40146000-4014c000 RW-p 00125000 03:07 106687     /lib/libc-2.2.5.so
        4014c000-40150000 RW-p 00000000 00:00 0
        bfffe000-c0000000 RW-p fffff000 00:00 0
 
最后的这种情况是vma mirroring所产生最简单的内存layout，只有binary本生被
镜像了，[1]被[3]，[2]被[4]镜像了。注意[1]没有R-X权限了，在PAGEEXEC下只
有R--。
 
虽然现在的PaX实现肯定不是这个设计的版本，但读读原始的paper会有一些意想
不到的收获，也算技术进化考古的过程了;-)


## 2. 0ld sch00l黑客的出埃及记

Grsecurity/PaX目前[应用广泛](https://grsecurity.net/the_case_for_grsecurity.pdf)，特别是具有高安全性的环境，Gnu/Linux发行版里
Gentoo提供PaX作为加固选项，最近半年Debian社区发起的对抗大规模监控的加固
项目[Mempo](http://mempo.org/)在内核中也使用了Grsecurity/PaX。

这篇文章仅仅是在学习PaX的3篇paper里的记录，PaX的思路的确非常的震撼，那
都是10多年前的设计和实现，在这个一天云计算一天雾计算的年代，虽然关注本
质的黑客越来越少，但地下精神并未死去，PaX Team就是一个活生生的例证，相
反，不少old school黑客都坚信其实old school的数量并没有减少，至少我个人
相信这是真的...Phrack没死，Grsecurity/PaX没死，DNFWAH也没死，希望更多的
黑客分享自己的hacking之旅。


Phrack is not dead, Grsecurity/PaX is not dead, DNFWAH is not dead,
The Underground spirit is not dead.....If they were, that'd be on us!

=--------------------------------------------------------------------=

To one of the most respected old school communities:
Grsecurity/PaX. We/I salute you!!!

