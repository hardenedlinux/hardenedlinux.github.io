---
layout: post
title:  "PaX/Grsecurity配置选项"
date:   August 17, 2015 12:46 PM
summary: The introduction of  per Grsecurity's option.Translated from "Grsecurity/Appendix/Grsecurity and PaX Configuration Options".
categories: system-security

---

By: CipherGateway

Wiki网址：https://en.wikibooks.org/wiki/Grsecurity/Appendix/Grsecurity_and_PaX_Configuration_Options


# Grsecurity配置选项


[TOC]


## Configuration Method（配置选项）

**Automatic(自动配置)** 【使用此选项，你只需回答一些你希望如何配置内核的问题，Grsecurity和Pax将自动以此标准并按照最通用化完成配置。如果你还想定制其他标准，custom configuration自定义选项的选择同样会生效

**Custom（自定义配置）**【你将完全手动定制Grsecurity和Pax】

#### Usage Type（内核用途）

**Server** 【如果你打算将此作为一个服务器的内核】

**Desktop** 【如果你打算将此作为一个桌面操作系统的内核】

#### Virtyalization Type（虚拟化技术类型）

**None** 【内核将运行在纯物理机上】

**Guest** 【内核将被运行在虚拟机上】

**Host** 【内核将被运行在宿主机上】

**-->(X) EPT/RVI Processor Support** 【Choose this option if your CPU supports the EPT or RVI features of 2nd-gen hardware virtualization.This allows for additional kernel hardening protections to operate without additional performance impact.
To see if your Intel processor supports EPT, see: http://ark.intel.com/Products/VirtualizationTechnology (Most Core i3/5/7 support EPT)
To see if your AMD processor supports RVI, see: http://support.amd.com/us/kbarticles/Pages/GPU120AMDRVICPUsHyperVWin8.aspx】

**-->( ) First-gen/No Hardware Virtualization** 【Choose this option if you use an Atom/Pentium/Core 2 processor that either doesn't support hardware virtualization or doesn't support the EPT/RVI extensions.】

#### Virtualization Software

**Xen** 【（GRKERNSEC_CONFIG_VIRT_XEN）Choose this option if this kernel is running as a Xen guest or host.】

**VMWare** 【Choose this option if this kernel is running as a VMWare guest or host.】

**KVM** 【Choose this option if this kernel is running as a KVM guest or host.】

**VirtualBox** 【Choose this option if this kernel is running as a VirtualBox guest or host.】

#### Required Priorities (Performance)（优先级需求）

**Performance（性能优先）** 【选取这个选项将导致以下特性无法使用：UDEREF on a 64bit kernel, kernel stack clearing,clearing of structures intended for userland, and freed memory sanitizing】

**Security（安全优先）** 【以上特性可用，但在最糟糕的情况下这些特性将导致20%的性能损耗】

#### Default Special Groups（特殊组）

**(1001) GID exempted(被免除) from /proc restrictions**
【通过设定这个GID来决定那个组中的用户能够免除/proc目录下Grsecurity规则的限制，允许特殊组的用户查看网络数据、和其他用户的运行信息。这个GID也可以在开机时在内核命令行中通过“grsec_proc_gid=”设置】

**(1005) GID for TPE-untrusted users(TPE:Tursted Path Execution）** 【该组的用户将会受到TPE的限制。如果sysctl可用，sysctl中一个名为“tpe_gid”选项将会产生】

**(1006) GID for users with kernel-enforced**

## Customize Configuration（自定义配置）

### PaX

#### Enable various PaX features

【Pax为内核提供了防止入侵机制，从而降低了可利用的内存漏洞造成的风险】

#### PaX Control

**Support soft mode** 【（PAX_SOFTMODE）允许通过soft mode(友好的方式??)运行Pax，Pax的特性将不会被强制设定为默认选项，只会在可运行时显示标记。你必须同时启用PT_PAX_FLAGS或XATTR_PAX_FLAGS支持，因为只有它们能在soft mode下标记可运行程序。	soft mode可以使用“pax_softmode=1”参数通过内核命令行在开机时激活。	此外，在运行时你可以通过/proc/sys/kernel/pax的项目控制各种Pax特性。】

**Use legacy ELF header marking** 【（PAX_EI_PAX）启用这个选项将允许你通过“chpax”工具（可在http://pax.grsecurity.net 获得）控制每一个可执行程序的Pax特性。控制信息可以在otherwise reserved 的部分ELF头部（其他情况保存的部分ELF头部？？）读取。这个标记有大量的弊端（没有对soft-mode的支持、工具链不知道ELF头部的不规范用法），因此这个特性相比于PT_PAX_FLAGS and XATTR_PAX_FLAGS已经过时.		要注意的是如果你启用PT_PAX_FLAGS or XATTR_PAX_FLAG标记支持，它们会覆盖传统的EI_PAX标记。		If you enable none of the marking options then all applications will run with PaX enabled on them by default.（仅是没有PT_PAX_FLAGS or XATTR_PAX_FLAG标记支持吗？？）】

**Use ELF program header marking** 【（PAX_PT_PAX_FLAGS）启用这个选项将允许你通过“paxctl”工具（可在http://pax.grsecurity.net 获得）控制每一个可执行程序的Pax特性。控制信息可以从一个Pax特定的ELF程序头部读取。这个标记有同时支持soft-mode和完全整合toolchain。		要注意如果你同时启用了传统的EI_PAX标记支持，EI_PAX标记将被PT_PAX_FLAGS覆盖。		如果你同时启动了PT_PAX_FLAGS and XATTR_PAX_FLAGS支持，你应该确保被标记的二进制文件的标志是相同的。		If you enable none of the marking options then all applications will run with PaX enabled on them by default.】

**Use filesystem extended attributes marking（使用文件系统的扩展属性）** 【（PAX_XATTR_PAX_FLAGS）启用这个选项将允许你通过“setfattr”工具控制每一个可执行程序的Pax特性。控制信息可以从文件的扩展属性user.pax.flags读取。这个标记的好处是：支持可以自检的binary-only applications（如skype）并且不能兼容chpax/paxctl的改变。	该特性主要的缺点是因为一些文件系统（如isofs, udf, vfat)不支持扩展属性，导致通过这类文件系统拷贝文件时将失去扩展属性和Pax标记。		要注意如果你同时启用了传统的EI_PAX标记支持，EI_PAX标记将被XATTR_PAX_FLAGS覆盖。	如果你同时启动了PT_PAX_FLAGS and XATTR_PAX_FLAGS支持，你应该确保被标记的二进制文件的标志是相同的。	If you enable none of the marking options then all applications will run with PaX enabled on them by default.】

#### MAC system integration（集合）（MAC，Mandatory Access Control 强制访问控制）

【MAC系统具有控制每个可执行程序Pax标志的选项。选择你特定的系统支持的方法。		NOTE: this option is for developers/integrators only.】

**none** 【（PAX_NO_ACL_FLAGS）如果MAC系统与Pax没有联系】

**direct** 【（PAX_HAVE_ACL_FLAGS）如果MAC系统自行定义pax_set_initial_flags参数】

**hook** 【（PAX_HOOK_ACL_FLAGS）如果MAC系统使用pax_set_initial_flags_func的回调】

#### Non-executable pages（不可执行页：只允许读和写，而代码执行是被禁止的）

**Enforce non-executable pages** 【(PAX_NOEXEC)勾选这个选项你将可以选择众多防止注入和外来代码的特性。但将使依赖原始方法（RWX权限系统???）和通过malloc()方法族动态分配内存的程序无法使用。如： XFree86 4.x server, the java runtime and wine.】

**Paging based non-executable pages** 【（PAX_PAGEEXEC）这个特性基于CPU的页特性。在没有硬件不可执行位支持的i386CPU里它是可变的，但通常带有较低的性能损耗。然而在基于Intel's P4核心的CPU性能损耗将非常高，所有此类情况你应该禁止这个选项。对于拥有硬件不可执行位支持的CPU：alpha, avr32, ia64, parisc, sparc, sparc64, x86_64 and i386，这个特性没有性能损耗,在ppc下损耗可以忽略不计。】

**Emulate trampolines (模仿GCC扩展（回调？？？））** 【（PAX_EMUTRAMP）总有一些程序和库因为一些原因或意图去执行在禁止执行内存页中的特殊代码段，大量有名的例子是内核自己生成的信号句柄回调代码（kernel signal handler return code）和GCC trampolines。  如果你开启了CONFIG_PAX_PAGEEXEC这个特性这些进程将不能再运行。  作为补救，你可以启用这个选项并通过chpax、paxctl工具来为那些仍受到non-executable pages的保护的受影响的程序启用trampoline emulation。  在工作站(On parisc)你必须开启这个选项和EMUSIGRT，否则你的系统将无法启动！】【另外的，你可以在这里选否，并使用chpax或paxctl工具为受影响的文件禁用CONFIG_PAX_PAGEEXEC and CONFIG_PAX_SEGMEXEC特性。】【开启这个特性可能会从non-executable pages的保护中打开一个漏洞。因此最好的解决方法是不使用任何请求这个选项的文件。这可以通过不使用libc5（因为它依赖kernel signal handler return code）和不使用或重写通过GCC实现的嵌套函数的程序。  有经验的使用者可以直接修改GCC使嵌套函数不干扰Pax。】

**Restrict mprotect()（限制mprotect()）** 【(PAX_MPROTECT)启用这个特性将阻止进程进行以下操作：1、改变未被初始化为可执行的内存页的可执行状态；2、使只读的可执行页可再次写入；3、在分配的内存中创建可执行页；4、让重定位后只读的（RELRO）数据页可再次写入】【你应该启用这个特性来完善non-executable pages提供的强制保护措施.  Note:你可以使用chpax或paxctl来控制每个可执行程序的这个属性】

**Use legacy/compat protection demoting (read help)（使用传统/可兼容的安全措施（如RWX权限？）降低pax安全作用的影响）** 【（PAX_MPROTECT_COMPAT）现在的PAX_MPROTECT实现机制通过发送适当的错误代码给应用程序来阻止RWX权限分配。对于一些broken userland，Python或其他应用可能会造成问题。The current implementation however allows for applications like clamav to detect if JIT compilation/execution is allowed and to fall back gracefully to an interpreter-based mode if it does not.  While we encourage everyone to use the current implementation as-is and push upstream to fix broken userland (note that the RWX logging option can assist with this), in some environments this may not be possible. Having to disable MPROTECT completely on certain binaries reduces the security benefit of PaX,所以这个选项用来使会出现问题的这类环境恢复到old behavior】

**Allow ELF text relocations (read help) (Executable and Linking Format，ELF，可执行文件)** 【(PAX_ELFRELOCS)Non-executable pages and mprotect()的限制对阻止新的可执行代码进入攻击任务的地址空间很有用。 There remain only two venues for this kind of attack:if the attacker can execute already existing code in the attacked task then he can either have it create and mmap() a file containing his code or have it mmap() an already existing ELF library that does not have position independent code in it and use mprotect() on it to make it writable and copy his code there.  While protecting against the former approach is beyond PaX, the latter can be prevented by having only PIC ELF libraries on one's system (which do not need to relocate their code).*（如果攻击者可以执行在攻击任务中已存在的代码，那他可以创建它并mmap（）到一个包含他的代码的文件 或者 把它mmap（）到一个已存在的没有独立位置的代码的ELF库并且使用mprotect（）使其变得可写入并把他的代码复制进去？）*。  如果你确定这跟你的情况相似，因为所有现代Linux发行版都有这个情况，让这个选项禁用——这里应该选否。】

**Enforce non-executable kernel pages** 【该选项相当于内核态的mprotect和pageexec。启用这个选项将使在系统区中注入和外来代码的执行更加困难。】

**-->Return Address Instrumentation Method**

**---->bts**

**---->or**

#### Address Space Layout Randomization(地址空间随机化)

**Address Space Layout Randomization** 【（PAX_ASLR）接下来的选项会让内核在特定的程序部分采取随机化，因此强迫攻击者在众多的情况中猜测程序的位置。在允许内核检测这类意图并回应它的前提下，任何一个失败的猜测都很可能导致被攻击的程序崩溃。启用这个选项你可以选择随机化下面的区域：1、 top of the task’s kernel stack；2、 top of the task’s userland stack；3、 base address for mmap() requests that do not specify one(this includes all libraries)；4、 base address of the main executable。强烈建议打开这个选项，因为它提供有效的保护且损失性能的可以忽略不计。  Note：可以使用chpax和paxctl控制每个可执行程序的该特性。】

**Randomize kernel stack base** 【（PAX_RANDKSTACK）启用这个选项将随机化每一个任务的每一个系统调用内核栈。强迫攻击者猜测，且能防止他利用有关的可能已经泄露的信息。注意，一旦kernel stack紧缺，可能导致堆栈溢出，所以你要认真测试你的系统。  一旦在内核配置中启用这个选项，基于这个特性的文件将无法禁用这个特性。】

**Randomize user stack and mmap() base** 【（PAX_RANDUSTACK）user stack：内核随机化每一个任务的用户栈。随机化过程分为两步，第二步将采用大量的栈顶移位，从而导致程序需要大量的内存（如果SEGMEXEC未被启用将超过2.5G，如果启用了将为1.25G）。  因为这个原因，在每个基于这个特性的文件第二步可以被chpax和paxctl控制。】【（PAX_RANDMMAP）mmap，内存映射机制。如果选是，mmap()请求将不自己指定地址，内核将为之使用基于随机的地址。结果是所有动态加载的库将在一个随机的地址出现，并因此使攻击者执行library code的意图更困难。 Furthermore, if a program is relinked as a dynamic ELF file, its base address will be randomized as well, completing the full randomization of the address space layout.  攻击这样的程序将变成一个猜测游戏。】

#### Miscellaneous hardening features(杂项优化）

**Sanitize all freed memory （清除已释放内存）** 【一旦内存被释放，内核将清除内存页和slab对象（Linux下的一种内存分配机制）。这可以轮流减少存储数据的生命周期，避免密码等敏感数据在内存太长时间。对运行时间短的进程尤其有好处,long lived processes and the kernel itself benefit from this as long as they ensure timely freeing of memory that may hold sensitive information.作为交易，这可能对性能有所影响，在单CPU系统中内核编译可能会慢3%，部署该选项前最好在工作负荷下测试一下性能损失。要注意这个特性不会保护存储在仍然存活的内存页中的数据】

**Sanitize kernel stack** 【内核将在系统调用返回前清除内核栈,这会减少内核栈leak bug泄露的信息。这同样会造成性能损耗，最好在工作环境下测试一下。完整的特性需要GCC插件的支持，例如GCC4.5或更新的版本。】

**Forcibly initialize local variables copied to userland(对复制到用户态的局部变量进行强制初始化)** 【内核将对一些被复制到用户态的局部变量进行0初始化操作。The tradeoff is less performance impact than PAX_MEMORY_STACKLEAK at a much smaller coverage.  This in turn prevents unintended information leakage from the kernel stack should later code forget to explicitly set all parts of the copied variable.完整的特性需要GCC插件的支持】

**Prevent invalid userland pointer dereference（阻止间接引用不合法的用户态指针）** 【在只允许内核指针的环境下内核将阻止间接引用用户态指针，这是一个有用的运行时debugging feature和阻止渗透内核bugs的一个安全方法。作为交换，一些虚拟化措施将遭受到可观的延迟，因此如果内核将运行在这类环境上你应该禁用这个特性。一个给定虚拟环境是否会受到影响最好通过一些简单的测试再决定。性能损耗将会在开机的时候最为明显。一个重要的准则是虚拟机运行在没有硬件虚拟化支持的CPU上会很可能遭到这类延迟。】

**Prevent various kernel object reference counter overflows(阻止各类内核对象引用计数器溢出)** 【内核会检查并阻止溢出的各类（但不是全部）对象引用计数器。作为交易，被溢出引用计数机制保护的数据结构将不会被释放，从而导致内存泄露。需要注意到的是即使没有这种保护，这种泄露也会发生。但是那样的话，泄露出的大量数据会最终引发数据结构的freeing（*//解释为内存释放还是数据自由？*）, 即使这种数据结构正在被用于别处的时候。这会最终导致这个特性所阻止的渗透的情况的发生。鉴于该特性对性能损耗微乎其微，建议启用这个特性】

**Automatically constify eligible structures （自动常量化符合条件的结构)** 【the compiler will automatically constify a class of types that contain only function pointers.它减少了内核的受攻击面并产生了更好的内存配置。此特性需要GCC4.5及以上。要注意的是，如果一些代码不得不修改已常量化的变量，那源代码将作为补丁才能被使用。 Examples can be found in PaX itself (the no_const attribute) and for some out-of-tree modules at http://www.grsecurity.net/~paxguy1/ .】

**Harden heap object copies between kernel and userland （加固用户与内核之间堆对象的拷贝）** 【内核会限制堆对象的大小，当它们被在内核态和用户态之间的目录拷贝时。这种检查防止在内核向用户拷贝时造成内核堆的信息泄露，并且防止在用户向内核拷贝时造成内核堆溢出。】

**Prevent various integer overflows in function size parameters** 【选择这个选项内核会重新计算被标记为size_overflow特征的具有双精度整数的函数参数的表达式。The recomputed argument is checked against TYPE_MAX and an event is logged on overflow and the triggering process is killed.】

**Generate some entropy during boot and runtime （在开机和运行时生成一些无序状态量（平均信息量））** 【选择这个选项内核会利用某些内核代码从原始以及手动产生的程序状态中提取无序状态量This will help especially embedded（嵌入式） systems where there is little ‘natural’ source of entropy normally. The cost is some slowdown of the boot process and fork（分岔） and irq（中断请求） processing.注意这个提取entropy的方法不是密码级别的安全】


### Memory Protections（内存保护）

**Deny reading/writing to /dev/kmem, /dev/mem, and /dev/port** 【（GRKERNSEC_KMEM）如果你启用这个选项，在运行的内核上/dev/kmem和/dev/mem将无法读写，/dev/port也将不允许被打开。/dev/cpu/*/msr和kexec的支持将被删除。If you have module support disabled, enabling this will close up six ways that are currently used to insert malicious code（恶意代码） into the running kernel.就算这个特性被启用，我们依然高度建议你使用RBAC系统，因为攻击者仍然可能通过鲜为人知的代码修改运行的内核。启用这个选项会阻止cpupower和powertop工具的运行。】

**Disable privileged I/O** 【（kernel.grsecurity.disable_priv_io）启用这个选项，所有ioperm和iopl调用都会返回error。不幸的是一些程序需要这个通道来正确地运行，最著名的是XFree86 and hwclock。hwclock可以通过内核的RTC（real-time clock)支持补救，所以如果这个选项启用，那RTC也将可用，用来保证hwclock正确运行。如果你正使用XFree86或Xorg2012或更早的版本，你可能导致无法使用图形界面，在这种情况下你应该使用RBAC系统。】

**Harden BPF JIT against spray attacks** 【（GRKERNSEC_JIT_HARDEN）？？？】

**Disable unprivileged PERF_EVENTS usage by default** 【sysctl中的/proc/sys/kernel/perf_event_paranoid取值范围将被允许扩大，并且的得到一个新的默认值3。当sysctl被设置成这个值时，对PERF_EVENTS系统调用接口没有权限的使用将被允许。虽然PERF_EVENTS能被合法地用来管理性能和低等级应用分析，但是这建立在强制忽略配置上，是一些漏洞的原因所在，也为边缘途径和信息泄露产生新的机会。这个特性将PERF_EVENTS设置为一个默认的安全状态，并且如果有不具备权限的程序分析需要，允许管理员暂时更改它的值。】

**Insert random gaps between thread stacks（在线程栈中插入随机间隙）** 【（GRKERNSEC_RAND_THREADSTACK）一个随机大小的间隙将会强制插入已分配的线程栈中。Glibc’s NPTL and other threading libraries that pass MAP_STACK to the kernel for thread stack allocation are supported.这个implements目前为间隙提供8位的平均信息。Many distributions do not compile threaded remote services with the -fstack-check argument to GCC, causing the variable-sized stack-based allocator, alloca(), to not probe the stack on allocation. 这将允许一个无界限alloca()来跳过所有保护页面，潜在并有把握地修改其他的线程栈。一个强制随机的间隙可以降低这类攻击的可靠性and increases the chance that such a read/write to another thread’s stack instead lands in an unmapped area,导致崩溃并触发Grsecurity的anti-bruteforcing（反暴力破解）逻辑。】

**Harden ASLR against information leaks and entropy reduction （加强ASLR来抵御信息泄露和平均状态量减少）** 【（GRKERNSEC_PROC_MEMMAP）如果启用这个特性，并且Pax中的依赖随机地址特性在任务中被启用，那/proc//maps和/proc//stat文件将不再给出地址映射信息。另外还将清除这些信息并禁用那些危险源的信息。this option causes reads of sensitive /proc/ entries（条目） where the file descriptor（描述子） was opened in a different task than the one performing the read.这类意图将会被记录。This option also limits argv/env strings for suid/sgid binaries to 512KB to prevent a complete exhaustion of the stack entropy provided by ASLR. Finally, it places an 8MB stack resource limit on suid/sgid binaries to prevent alternative mmap layouts from being abused.】【如果你使用Pax，启用这个选项至关重要，因为它关闭了许多让完整的ASLR局部失效的漏洞】

**Prevent kernel stack overflows** 【（GRKERNSEC_KSTACKOVERFLOW）内核的进程栈将使用vmalloc来进行分配，而不是内核默认的分配器。This introduces guard pages that in combination with the alloca checking of the STACKLEAK feature prevents all forms of kernel process stack overflow abuse.要注意这个特性与内核栈缓存溢出不一样.】

**Deter exploit bruteforcing（阻止暴力破解）** 【（GRKERNSEC_BRUTE， kernel.grsecurity.deter_bruteforce）启动这个选项，尝试暴力破解分支后台进程如apche和sshd以及suid/sgid下的二进制文件的意图都会被阻止。当一个forking deamon的子进程被Pax或因为一个不合法的操作或其他可疑信号杀死、崩溃时，（the parent process will be delayed 30 seconds upon every subsequent fork）父进程将在之后的每个分支延迟30秒，直到管理员能评定这个情况并重启这个后台程序。 在suid/sgid方面，这个意图会被记录，the user has all their existing instances of the suid/sgid binary terminated，并在15分钟内不能执行任何suid/sgid的二进制程序。 建议你在审计部分开启信号记录使得每当进程触发可疑信号时就生成记录。 如果sysctl选项可用，一个名为”deter_bruteforce”的选项会被生成。】

**Harden module auto-loading（模板自动加载的加固）** 【（GRKERNSEC_MODHARDEN）module auto-loading in response to use of some feature implemented by an unloaded module will be restricted to root users。Enabling this option helps defend against attacks by unprivileged users who abuse（滥用） the auto-loading behavior to cause a vulnerable（有漏洞的） module to load that is then exploited. 如果这个选项阻止了非root用户合理的自动加载，管理员可以通过使用警告日志中提到的准确的模块的名字手动执行模块。另外，管理员可以通过修改init脚本添加模块到开机模块加载列表中去。 拥有加密home目录支持的ubuntu服务器可能更需要对init脚本的修改，as the first non-root user logging in will cause the ecb(aes), ecb(aes)-all, cbc(aes), and cbc(aes)-all modules to be loaded.】

**Hide kernel symbols** 【（GRKERNSEC_HIDESYM）启用这个选项，users with CAP_SYS_MODULE通过系统调用从已加载的模块中获得信息和显示所有内核标志的操作将被限制。由于软件的兼容性的原因，root用户对/proc/kallsyms目录将会被限制。RBAC系统将会隐藏entry（条目？入口？），即使是针对root用户。 该选项同样通过/proc一些条目防止内核地址的泄露。 要注意，当且仅当遇到下列情况时，该选项才会起作用：1、内核正使用的Grsecurity没有被一些发行版预先编译过了；2、你还应该同时启用了GRKERNSEC_DMESG特性；3、你正使用RBAC系统并且隐藏其他文件，例如kernel image and System.map。除此之外，启用这个选项将获得在编译时改变/boot, /lib/modules, and the kernel source这些目录的许可，以防止非root用户查看。  如果以上的情况都符合，这个选项将有助于对局部内核可利用溢出和任意读写漏洞（local kernel exploitation of overflows and arbitrary read/write vulnerabilities）提供有用的保护。 强烈建议你再此之外启用GRKERNSEC_PERF_HARDEN特性】

**Randomize layout of sensitive kernel structures （敏感内核结构的随机配置）** 【（GRKERNSEC_RANDSTRUCT）许多敏感内核结构（如task, fs, cred, etc)和all structures composed entirely of function pointers (aka “ops” structs)的配置将在编译的时候被随机化。This can introduce the requirement of an additional infoleak vulnerability for exploits targeting these structure types.		使用这个选项会产生一些性能损耗，轻微地增加内存使用，并且阻止取证工具（forensic tools）如Volatility的使用（除非内核资源树在内核安装完后没有清除。		编译的种子放在tools/gcc/randomize_layout_seed.h中。这在一个允许的make clean命令后会保留下来，用来使用先用的种子来编译external modules，并且可以通过make mrproper或make distclean被删除。】

**Use cacheline-aware structure randomization ** 【（GRKERNSEC_RANDSTRUCT_PERFORMANCE）If you say Y here, the RANDSTRUCT randomization will make a best effort at restricting randomization to cacheline-sized groups of elements. It will further not randomize bitfields in structures. This reduces the performance hit of RANDSTRUCT at the cost of weakened randomization.】

**Active kernel exploit response** 【（GRKERNSEC_KERN_LOCKOUT）启用这个选项，当一个Pax警告触发，原因是可疑的内核活动(from KERNEXEC/UDEREF/USERCOPY)或因为bad memory accesses导致一个OOPS（意外？？）发生，我们将使用一下两个措施中的一个，而不是直接终止攻击进程(and potentially allowing a subsequent（后来的） exploit from the same user)：1如果用户是root，we will panic the system；2、如果用户不是root，我们会记录这个行为，终止该用户的所有进程，然后在系统重启前阻止他们创建任何新进程。This deters repeated kernel exploitation/bruteforcing attempts and is useful for later forensics.】

### Role Based Access Control Options

**Disable RBAC system** 【（GRKERNSEC_NO_RBAC）如果这里选是，/dev/grsec设备将从内核中删除，来阻止RBAC系统生效。		应该仅在你不打算使用RBAC时才选Yes，以防止当可加载的模块支持并且/dev/[k]mem已被锁定时，攻击者通过错误使用的RBAC系统使用root权限来隐藏文件或进程。】

**Hide kernel processes** 【（GRKERNSEC_ACL_HIDEKERN）如果这里选是，除了有”view hidden processes”标志的subject，所有内核线程将对所有进程隐藏。】

**() Maximum tries before password lockout （最大密码尝试次数）** 【（GRKERNSEC_ACL_MAXTRIES）这个选项强制设定用户在Grsecurity RBAC系统中认证自己的最大次数，超过特定次数将被拒绝再次认证。数字越小，密码越难被暴力破解。】

**() Time to wait after max password tries, in seconds** 【（GRKERNSEC_ACL_TIMEOUT）这个选项指定用户在RABC系统认证失败达最大次数后必须等待的时间。数字越大，密码越难被暴力破解。】

### Filesystem Protections

**Proc restrictions** 【（GRKERNSEC_PROC）如果选是，/proc文件系统的访问权限将会改变，以加强系统安全和隐私。你必须在用户限制和用户及用户组限制中选择一个。根据你的选择，你可以把用户限制在只能查看自己运行的进程，或choose a group that can view all processes and files normally restricted to root if you choose the “restrict to user only” option. 		注意，如果你以非root身份运行identd或者ntpd（网络时间校正协议(network time protocol daemon)），你必须在你指定的组里运行。】

**Restrict /proc to user only** 【（GRKERNSEC_PROC_USER）如果选是，非root用户将只能查看他们自己的进程，并且会被禁止查看网络相关的信息、内核标志和模块信息。】

**Allow special group** 【（GRKERNSEC_PROC_USERGROUP）如果选是，你可以选择一个组，这个组将可以查看所有进程的信息以及网络相关的信息。如果你启用了GRKERNSEC_HIDESYM选项，内核的标志信息可能依旧对这个组隐藏。 如果你想以非root身份运行identd，这将很有帮助。 你选择的组也可以在开机时通过”grsec_proc_gid=”在内核命令行选择。】

**() GID for special group** 【（GRKERNSEC_PROC_GID）设置这个GID决定那个组将可以避免Grsecurity对/proc的限制，允许该特殊组的用户查看网络数据和其他用户进程的存在信息。该GID也可以在开机时通过”grsec_proc_gid=”参数在内核命令行设定。】

**Additional restrictions** 【（GRKERNSEC_PROC_ADD）如果选是，/proc将会添加附加的限制来阻止用户查看device和slabinfo信息（因为这些信息对exploits（渗透？）很有用）】

**Linking restrictions** 【（GRKERNSEC_LINK、kernel.grsecurity.linking_restrictions）如果选是，race exploits将被阻止，因为在可写+t（world-write +t)的目录下（如/tmp)用户将不能再跟踪其他用户的符号链接，除非符号链接的所有者也是目录的所有者。 用户同样无法硬链接到不属于他们的文件上。如果sysctl被启用，一个名为”linking_restrictions”的sysctl选项被创建。】

**Kernel-enforced SymlinksIfOwnerMatch*** 【？？？】

**（）GID for users with kernel-enforced SymlinksIfOwnerMatch** 【】

**FIFO restrictions** 【（GRKERNSEC_FIFO、kernel.grsecurity.fifo_restrictions） users will not be able to write to FIFOs they don’t own in world-writable +t directories (e.g. /tmp), unless the owner of the FIFO is the same owner of the directory it’s held in.】

**Sysfs/debugfs restriction** 【（GRKERNSEC_SYSFS_RESTRICT）如果选是，在这个限制下正常挂载的sysfs以及所有在该目录下的文件系统都只能被root访问。		这些文件系统通常会提供硬件信息和调试信息的访问入口，然而这些信息却不适合被没有权限的用户看见。因为sysfs和debugfs也是一个潜在漏洞的最大的信息源ranging from infoleaks to local compromise.There has been very little oversight with an eye toward security involved in adding new exporters of information to these filesystems,所以不建议使用这些文件系统。 	因为兼容性的原因，一些目录对非root用户的访问提供了白名单：/sys/fs/selinux、/sys/fs/fuse、/sys/devices/system/cpu】

**Runtime read-only mount protection** 【（GRKERNSEC_ROFS、kernel.grsecurity.romount_protect）如果选是，一个名为“romount_protect”的sysctl选项被产生。在运行时将这个选项设为1，文件系统将会获得如下保护：1、不允许新的可写入的加载的文件系统；2、现有的只读加载的文件系统不能被重新加载为可读可写；3、所有设备区块的写入选项都会被拒绝。 This option acts independently of grsec_lock: once it is set to 1, it cannot be turned off.		因此，如果该选项在只读系统的初始化脚本中被启用，请留意resulting behavior。		Also be aware that as with other root-focused features, GRKERNSEC_KMEM and GRKERNSEC_IO should be enabled and module loading disabled via config or at runtime.这个特性主要面向安全嵌入式系统（secure embedded systems）。】

**Eliminate stat/notify-based device sidechannels(（排除基于 进程状态/通知 的设备旁道攻击）** 【（GRKERNSEC_DEVICE_SIDECHANNEL）If you say Y here, timing analyses on block or character devices like /dev/ptmx using stat or inotify/dnotify/fanotify will be thwarted for unprivileged users. If a process without CAP_MKNOD stats such a device, the last access and last modify times will match the device’s create time. No access or modify events will be triggered through inotify/dnotify/fanotify for such devices。This feature will prevent attacks that may at a minimum allow an attacker to determine the administrator’s password length.】

**Chroot jail restrictions** 【（GRKERNSEC_CHROOT）如果选是，你可以选择多个选项使突破chroot的限制更困难。如果在这些选项下没有不兼容的软件，建议你把每个都打开。】

**-->Deny mounts** 【（GRKERNSEC_CHROOT_MOUNT、kernel.grsecurity.chroot_deny_mount）如果选是，在chroot下的进程将不能再挂载或重新挂载文件系统】

**-->Deny double-chroots** 【（GRKERNSEC_CHROOT_DOUBLE、kernel.grsecurity.chroot_deny_chroot）如果选是，在chroot下的进程不能再一次使用chroot。因为这个操作是打破chroot限制的普遍方法，不应该被允许。】

**-->Deny pivot_root in chroot** 【（GRKERNSEC_CHROOT_PIVOT、kernel.grsecurity.chroot_deny_pivot）如果选是，在chroot下的进程将不能使用pivot_root()方法，它就像会改变root文件系统的chroot in一样。这个方法可能在已经chroot的进程中被错误地使用来试图打破chroot限制，因此不该被允许。】

**-->Enforce chdir(“/“) on all chroots** 【（GRKERNSEC_CHROOT_CHDIR、kernel.grsecurity.chroot_enforce_chdir）如果选是，所有刚刚chroot的应用的当前工作目录都会被设定到chroot的根目录下。 因为目前所知这个特性不妨碍任何软件，建议你在这里选是。】

**-->Deny (f)chmod +s** 【（GRKERNSEC_CHROOT_CHMOD、kernel.grsecurity.chroot_deny_chmod）如果选是，在chroot下的进程将不能对文件使用chmod和fchmod来使它们具有SUID或SGID位。这特性保护chroot不被其他已公布的方法打破。】

**Deny fchdir and fhandle out of chroot** 【（GRKERNSEC_CHROOT_FCHDIR、kernel.grsecurity.chroot_deny_fchdir）如果选是，一个非常有名的打破chroot限制的方法将会被阻止，这个方法是fchdir’ing to a file descriptor of the chrooting process that points to a directory outside the filesystem.】

**Deny mknod** 【（GRKERNSEC_CHROOT_MKNOD、kernel.grsecurity.chroot_deny_mknod）如果选是，在chroot下的进程将不能使用mknod命令。   在chroot使用mknod的问题是：这将允许攻击者生成一个device entry，就像在物理层次上获得你系统的root权限，which could range from anything from the console device to a device for your harddrive(which they could then use to wipe the drive or steal data).  建议选是，除非你运行的软件不兼容。	 如果sysctl可用，一个名为"chroot_deny_mknod"的选项将被创建。】

**Deny shmat() out of chroot** 【（GRKERNSEC_CHROOT_SHMAT、kernel.grsecurity.chroot_deny_shmat）如果选是，在chroot下的进程将不能连接到在chroot jail外创建的共享内存段。  建议在这里选是。 如果sysctl可用，一个名为"chroot_deny_shmat"的选项将被创建。】

**Deny access to abstract AF_UNIX sockets out of chroot** 【（GRKERNSEC_CHROOT_UNIX、kernel.grsecurity.chroot_deny_unix）如果选是，在chroot下的进程将不能连接到限制在chroot外部的abstract（表示不属于一个文件系统） Unix domain sockets.  建议选是。  如果sysctl可用，一个名为"chroot_deny_unix"的选项将被创建】

**Protect outside processes** 【（GRKERNSEC_CHROOT_FINDTASK、kernel.grsecurity.chroot_findtask）如果选是，在chroot下的进程将不能杀死chroot外的进程，不能使用fcntl, ptrace, capget, getpgid, setpgid, getsid向chroot外的进程发送信号，不能查看chroot外的任何进程。	如果sysctl可用，一个名为"chroot_findtask"的选项将被创建】

**Restrict priority changes** 【（GRKERNSEC_CHROOT_NICE、kernel.grsecurity.chroot_restrict_nice）如果选是，chroot下的进程将不能提升chroot下进程的优先级，或改变chroot外面的进程的优先级。	这个选项比从进程的能力设定中简单地删除CAP_SYS_NICE提供了更高的安全型。	如果sysctl可用，一个名为"chroot_restrict_nice"的选项将被创建】

**Deny sysctl writes** 【（GRKERNSEC_CHROOT_SYSCTL、kernel.grsecurity.chroot_deny_sysctl）如果选是，在chroot下的攻击者将不能对sysctl的条目进行写入操作，不管是通过sysctl(2)还是通过/proc的接口。		强烈建议你在这个地方选是。如果sysctl可用，一个名为"chroot_sysctl"的选项将被创建】

**Capability restrictions** 【（GRKERNSEC_CHROOT_CAPS、kernel.grsecurity.chroot_caps）如果选是，在chroot jail下的所有进程的性能都会被降低到关闭以下功能：模块插入、raw I/O、系统和网络服务管理、重启系统、修改immutable文件、修改其他用户的IPC和改变系统时间。留下这个选项是因为它可能破坏某些应用。如果你的chroot下的应用执行这些任务时出现了问题，那请禁用这个选项。如果sysctl可用，一个名为"chroot_caps"的选项将被创建】

**Exempt initrd tasks from restrictions** 【（GRKERNSEC_CHROOT_INITRD）如果选是，先前为了初始化而启动的tasks将可以免除Grsecurity的chroot限制。This option is mainly meant to resolve Plymouth's performing privileged operations unnecessarily in a chroot.】

### Kernel Auditing

**Single group for auditing** 【（GRKERNSEC_AUDIT_GROUP、kernel.grsecurity.audit_gid、kernel.grsecurity.audit_group）如果选是,the exec and chdir logging features将只在你选定的组里面运作。	如果你只是想看特定的用户而不是系统大量的logs，这个选项建议启用。】

**GID for auditing** 【GRKERNSEC_AUDIT_GID】

**Exec logging** 【（GRKERNSEC_EXECLOG、kernel.grsecurity.exec_logging）如果选是，所有execve()调用都会被记录。对shell-servers的好处是保持对用户的跟踪。		如果sysctl可用，一个名为"exec_logging"的选项将被创建。	警告：这个选项启用时会产生大量的日志，特别是在活动的系统。】

**Resource logging** 【（GRKERNSEC_RESLOG、kernel.grsecurity.resource_logging）如果选是，所有超越resource限制的意图都将随着resource的名字、请求资源的大小和目前的限制被记录。高度建议你在这里选Y。如果sysctl可用，一个名为"resource_logging"的选项将被创建。如果RBAC系统也可用，这个sysctl选项的值将被忽略。】

**Log execs within chroot** 【（GRKERNSEC_CHROOT_EXECLOG、kernel.grsecurity.chroot_execlog）如果选是，在chroot中的所有执行都将被记录在syslog上。如果特定的程序已安装在系统上，这会产生大量的日志。  因此留下了这个选项。】

**Ptrace logging** 【（GRKERNSEC_AUDIT_PTRACE、kernel.grsecurity.audit_ptrace）如果选是，所有通过ptrace连接进程的意图都将被记录。】

**Chdir logging** 【（GRKERNSEC_AUDIT_CHDIR、kernel.grsecurity.audit_chdir）如果选是，所有chdir()调用都会被记录。】

**(Un)Mount logging** 【（GRKERNSEC_AUDIT_MOUNT、kernel.grsecurity.audit_mount）如果选是，所有挂载和卸载都会被记录。】

**Signal logging** 【（GRKERNSEC_SIGNAL、kernel.grsecurity.signal_logging）如果选是，特定的重要信息（如SIGSEGV）将被记录，当进程发生一个错误时它会作为一个结果通知你，这在一定程度上可能意味着a possible exploit attempt。】

**Fork failure logging** 【（GRKERNSEC_FORKFAIL、kernel.grsecurity.forkfail_logging）所有失败的fork()尝试都会被记录。这可能意味着fork炸弹，或有人正试图使他们的程序越权。】

**Time change logging** 【（GRKERNSEC_TIME、kernel.grsecurity.timechange_logging）所有系统时钟的改变都将被记录】

**/proc/<pid>/ipaddr support** 【（GRKERNSEC_PROC_IPADDR）如果选是，一个包含用户IP地址的新条目将会被加在每个/proc/<pid>目录下。The IP is carried across local TCP and AF_UNIX stream sockets.This information can be useful for IDS/IPSes to perform remote response to a local attack.这个条目只能被进程的所有者看见（root用户也能看见，如果他拥有CAP_DAC_OVERRIDE。这可以通过RBAC系统删除。）因此不会产生相关的隐私】

**Denied RWX mmap/mprotect logging** 【（GRKERNSEC_RWXMAP_LOG、kernel.grsecurity.rwxmap_logging）如果选是，当被PAX_MPROTECT阻止时，calls to mmap() and mprotect() with explicit usage of PROT_WRITE and PROT_EXEC together将会被记录。 这个feature也会记录其他的当PAX_MPROTECT开启时可能有问题的方案，比如textrels and PT_GNU_STACK】

### Executable Protections

**Dmesg(8) restriction** 【（GRKERNSEC_DMESG、kernel.grsecurity.dmesg）如果选是，非root用户将无法使用dmesg命令查看内核环日志缓冲。内核日志缓冲经常包含有内核地址和其他对攻击者有用的标识信息，in fingerprinting（数字指纹） a system for a targeted exploit。】

**Deter ptrace-based process snoopin** 【（GRKERNSEC_HARDEN_PTRACE、kernel.grsecurity.harden_ptrace）如果选是，通过ptrace实现的tty嗅探程序和其他恶意监督程序将被阻止。如果你一直使用着RBAC系统，那所有用户的这个选项已经启用了好几年了，with the ability to make fine-grained（细粒度？） exceptions. 这个选项只会影响到非root用户ptrace到其他不是ptrace的子孙进程的能力。 This means that strace ./binary and gdb ./binary will still work, but attaching to arbitrary processes will not.】

**Require read access to ptrace sensitive binaries** 【（GRKERNSEC_PTRACE_READEXEC、kernel.grsecurity.ptrace_readexec）如果选是，没有权限的用户将不能ptrace不可读的二进制程序。这个选项对在移除了有suid的二进制程序的可读权限后的环境很有用——可防止内容泄露。  这个选项增加了这种文件使用的稳定性，因为当没有执行权限且ptracing时，二进制程序能被正常地读取。】

**Enforce consistent multithreaded privileges** 【（GRKERNSEC_SETXID、kernel.grsecurity.consistent_setxid）If you say Y here, a change from a root uid to a non-root uid in a multithreaded application will cause the resulting uids, gids, supplementary(增补的） groups, and capabilities in that thread to be propagated（传送，扩散） to the other threads of the process.  在大多数情况下这是不必要的，因为glibc在应用程序的维护上会仿真这个行为。  其他的libc不会做同样的事情——允许进程的其他线程以root权限继续运行。】

**Disallow access to overly-permissive（过分宽松） IPC objects** 【（GRKERNSEC_HARDEN_IPC、kernel.grsecurity.harden_ipc)如果选是，过分宽松的IPC对象通道（共享内存、队列信息和信号量）将被禁止，进程将在正常权限下加入以下准则检查：1、If the IPC object is world-accessible and the euid doesn't match that of the creator or current uid for the IPC object*（//IPC对象是否可访问以及euid是否与创建者或现在IPC对象的uid不匹配）* 2、If the IPC object is group-accessible and the egid doesn't match that of the creator or current gid for the IPC object。  [It's a common error to grant too much permission to these objects, with impact ranging from denial of service and information leaking to privilege escalation.]*（授予这些对象太多权限是一个常见的错误，权限的提高可能导致拒绝服务和信息泄露）*。  拥有CAP_IPC_OWNER属性的进程仍然可以access these IPC objects.】

**Trusted Path Execution (TPE)** 【（GRKERNSEC_TPE、kernel.grsecurity.tpe、kernel.grsecurity.tpe_gid）如果选是，你将可以选择一个GID加入增补的组中，里面的成员都被标记为"untrusted."这些用户将不能执行在只有root能写入的但不属于root用户的目录中的所有文件。】

**Partially restrict（局部限制） all non-root users** 【（GRKERNSEC_TPE_ALL、kernel.grsecurity.tpe_restrict_all）如果选是，所有非root用户都将在一个削弱的TPE限制下。 This is separate from, and in addition to, the main TPE options that you have selected elsewhere.*（//与你在别处选择的TPE选项有区别，并且效果叠加？）*  因此，如果一个"trusted" GID被选择，这个限制同样会应用在这个GID上。 Under this restriction, all non-root users will only be allowed to execute files in directories they own that are not group or world-writable, or in directories owned by root and writable only by root.*（在这个限制下，所有非root用户只被允许执行属于他们自己 并且*不是*一个组或可写入的目录、或属于root且只能被root写入的目录下的文件。）*】

**Invert（反转？） GID option** 【（GRKERNSEC_TPE_INVERT、kernel.grsecurity.tpe_invert）如果选是，the group you specify in the TPE configuration will decide what group TPE restrictions will be *disabled* for.*（你在TPE配置中指定的组将判断那些TPE组限制将会被禁用）*  如果你想TPE限制应用在大多数用户上，这个选项很有用。  Unlike other sysctl options, this entry will default to on for backward-compatibility.】

**-->GID for TPE-untrusted users** 【（GRKERNSEC_TPE_UNTRUSTED_GID）Setting this GID determines what group TPE restrictions will be *enabled* for.】

**-->GID for TPE-trusted users** 【（GRKERNSEC_TPE_TRUSTED_GID）Setting this GID determines what group TPE restrictions will be *disabled* for.】


### Network Protections

**Larger entropy pools** 【（GRKERNSEC_RANDNET）如果选是，用来存放Linux和Grsecurity的各种features的entropy pool的大小会翻倍。由于有些Grsecurity的features使用附加的随机性，所有建议你在这里选是——相当于修改/proc/sys/kernel/random/poolsize】

**TCP/UDP blackhole and LAST_ACK DoS(Denial of Service, 拒绝服务攻击)prevention** 【（GRKERNSEC_BLACKHOLE、kernel.grsecurity.ip_blackhole、kernel.grsecurity.lastack_retries）如果选是，当外来数据包发送到没有程序监听的端口上，TCP重置和ICMP目标不可达数据包都不会被发送来相应该外来数据包。这个features支持IPV4和IPV6并且可以绕过回环网卡避免转发到黑洞。启用这个选项是主机对DoS攻击变得更有适应力（更主动），还能减少对扫瞄者的可视度。  这里的blackhole features实现相当于FreeBSD的blackhole feature，它对所有数据包不只是SYNs阻止RST（reset）响应。  在大多数程序执行下这不会造成问题，但程序（如haproxy）可能不会通过在远程端干净利落地终止他们来关闭特定的连接，从而让远程主机停留在LAST_ACK状态。 Because of this side-effect and to prevent intentional LAST_ACK DoSes,所有这个特性也增加automatic mitigation来应对这类攻击。  这个mitigation大幅度地减少一个socket花费在LAST_ACK state的时间。 如果你正使用haproxy并且不是所有的正连接的服务器都启用了这个选项，那在haproxy主机上禁用这个选项。  如果sysctl可用，两个名为"ip_blackhole" and "lastack_retries"的选项将被产生。当ip_blackhole进行标准的 0/非0、开/关 切换时，lastack_retries使用相同的值（"tcp_retries1" and "tcp_retries2"）。 默认的值 4 阻止一个socket保持LAST_ACK状态45秒。】

**Disable TCP Simultaneous（同时的） Connect** 【（GRKERNSEC_NO_SIMULT_CONNECT）如果选是，该特性将会移除Linux的strict implementation of TCP的缺点——允许两个没有均进入监听状态的client相互连接。这个缺点可以让一个攻击者很容易地阻止一个client连接上一个已知的一提供正确连接端口的服务器。  因为这个缺点可能被用来阻止杀毒软件或者IPS from fetching updates，或者阻止SSL网关获取CRL，因此这个弱点应该通过启用这个选项除去。  虽然Linux是为数不多的支持同时连接的操作系统之一，当这在现实中没有合理的用途，并且缺少防火墙的支持。】

**Socket restrictions** 【（GRKERNSEC_SOCKET）如果选是，你可以选择以下几个选项。如果你在你的系统上分配一个GID并且把它作为额外的用户组——你希望限制其中的用户对socket的访问，这个patch将表现为基于你的选项表现为三种状态。】

**Deny any sockets to group** 【（GRKERNSEC_SOCKET_ALL、kernel.grsecurity.socket_all、kernel.grsecurity.socket_all_gid）如果选是，你可以选择一个GID，这个组里的用户都将不能通过该机器连接到其他主机上或在该机器上运行服务端应用。】

**GID to deny all sockets for** 【（GRKERNSEC_SOCKET_ALL_GID）在这里你可以选择一个禁止socket访问功能的GID，要记得在这个组添加你想要禁止socket访问功能的用户。】

**Deny client sockets to group** 【（GRKERNSEC_SOCKET_CLIENT、kernel.grsecurity.socket_client、kernel.grsecurity.socket_client_gid）如果选是，你可以选择一个GID，这个组里的用户都将不能通过该机器连接到其他主机上，但可以运行服务。 如果这个选项启用，当你的机器在shell中启动ftp transfers时，所有在这个组里的用户将必须使用passive mode（被动模式？）】

**GID to deny client sockets for** 【（GRKERNSEC_SOCKET_CLIENT_GID）在这里你可以选择一个禁止socket访问功能的GID，要记得在这个组添加你想要禁止socket访问功能的用户。】

**Deny server sockets to group** 【（GRKERNSEC_SOCKET_SERVER、kernel.grsecurity.socket_server、kernel.grsecurity.socket_server_gid）如果选是，你可以选择一个GID，这个组里的用户都将不能在该机器上运行服务端应用。】

**GID to deny server sockets for** 【（GRKERNSEC_SOCKET_SERVER_GID）在这里你可以选择一个禁止socket访问功能的GID，要记得在这个组添加你想要禁止socket访问功能的用户。】

### Physical Protections

**Deny new USB connections after toggle** 【（GRKERNSEC_DENYUSB、kernel.grsecurity.deny_new_usb）如果选是，一个新的名为"deny_new_usb"的sysctl选项将被生成。 将这个选项设为1将阻止任何被OS识别的新的USB设备。所有USB设备的插入记录都会被记录。  这个选项被用来在各种USB设备中阻止被设计为渗透漏洞的USB。  为了最大的效益，在相关的启动脚本执行后这个sysctl应该被设置。  如果在发行版中的每一个用户都可以选择是否切换这个sysctl，那启用这个选项是安全的】

**Reject all USB devices not connected at boot** 【（GRKERNSEC_DENYUSB_FORCE）如果选是，一个将不生成sysctl项目的GRKERNSEC_DENYUSB的变型会被启用（意味着编译后无法再动态改变）。这个选项应该只在你确定要拒绝所有运行时的USB连接、并且不再改变初始化脚本时才启用。 这个选项不应该被distros启用。 It forces the core USB code to be built into the kernel image so that all devices connected at boot time can be recognized and new USB device connections can be prevented prior to init running.*（//强制USB的代码被build在kernel image里，以使所有在开机时就已连接的设备能被识别，并且新的USB设备将会被阻止。）*】

### Sysctl Support

**Sysctl support** 【（GRKERNSEC_SYSCTL）如果选是，你可以改变Grsecurity的开机选项，而不用重新编译内核。  你可以echo 变量的值到/proc/sys/kernel/grsecurity目录来开启（1）或禁用（0）各种features。"grsec_lock"条目被设置成非0值之前，所有sysctl条目都是可变的。  如果你没有在开机时对"Turn on features by default"选项选是，所有内核配置中可用的feature都将不可用。  所有选项应被设置为startup，并且当所有选项设置好后grsec_lock应该被设置成非零值。】

**Extra sysctl support for distro（发行版） makers (READ HELP)** 【（GRKERNSEC_SYSCTL_DISTRO）如果选是，将会生成附加的sysctl选项用来控制针对root下运行进程的features。因此，当grsec_lock在开机后被启用了，使用这个选项很重要。    Only distros with prebuilt kernel packages with this option enabled that can ensure grsec_lock is enabled after boot should use this option.  *Failure to set grsec_lock after boot makes all grsec features this option covers useless* 。目前这个选项生成如下sysctl项目："Disable Privileged I/O": "disable_priv_io"】

**Turn on features by default** 【（GRKERNSEC_SYSCTL_ON）如果选是，所有内核配置选项中选用的features都会在开机时被启用。 建议你选是，除非因为某些原因你想默认禁用所有sysctl可调的features。  就像其他地方提到的，在你完成对sysctl项目的修改之后启用grsec_lock项目非常重要。】

### Logging Options

**Seconds in between log messages (minimum)** 【（GRKERNSEC_FLOODTIME）这个选项允许你设定Grsecurity生成日志的时间。默认选项应该适合大多数人，如果你要更改它，选择一个足够小的数来允许生成能提供信息的日志，但应足够大以防止泛洪。  设定这个值并且将GRKERNSEC_FLOODBURST设为0，能防止Grsecurity日志的任何限制发生的概率】

**Number of messages in a burst (maximum)** 【（GRKERNSEC_FLOODBURST）This option allows you to choose the maximum number of messages allowed within the flood time interval you chose in a separate option.  默认选项应该适合大多数人, 然而，如果你发现你的许多日志被解释为flooding，你可能需要提高这个值。Setting both this value and GRKERNSEC_FLOODTIME to 0 will disable any rate limiting on grsecurity log messages.】

## 知识索引

1、TPE:Tursted Path Execution， 是Linux内核模块。
【Trusted Path Execution (TPE) is a feature that basically denies users the ability to execute programs that are not owned by the root user, or that they can write to. This prevents all kinds of exploits（漏洞利用程序） that would have otherwise rooted your system.】

2、MPROTECT：
【The goal of MPROTECT is to help prevent the introduction of new executable code into the task's address space. This is accomplished by restricting the mmap() and mprotect() interfaces.】
   The restrictions prevent
   - creating executable anonymous mappings
   - creating executable/writable file mappings
   - making an executable/read-only file mapping writable except for performing
     relocations on an ET_DYN ELF file (non-PIC shared library)
   - making a non-executable mapping executable

3、PTE( page table entries， 页表项)

4、NX（No Excute， 不可执行）

5、ASLR（Address space layout randomization） 【是一种针对缓冲区溢出的安全保护技术，通过对堆、栈、共享库映射等线性区布局的随机化，通过增加攻击者预测目的地址的难度，防止攻击者直接定位攻击代码位置，达到阻止溢出攻击的目的。】

6、kexec 【kexec 是 Linux 内核的一个补丁，让您可以从当前正在运行的内核直接引导到一个新内核。在上面描述的引导序列中，kexec 跳过了整个引导装载程序阶段（第一部分）并直接跳转到我们希望引导到的内核。不再有硬件的重启，不再有固件操作，不再涉及引导装载程序。完全避开了引导序列中最弱的一环 -- 固件。这一功能部件带来的最大益处在于，系统现在可以极其快速地重新启动。】

7、Linux下设置端口权限的系统调用:
ioperm 【为调用进程设置I/O端口访问权能。ioperm的使用需要具有超级用户的权限，只有低端的[0-0x3ff] I/O端口可被设置，要想指定更多端口的权能，可使用iopl函数。这一调用只可用于i386平台。】

iopl 【该调用用于修改当前进程的操作端口的权限。可以用于所有65536个端口的权限。因此，ioperm相当于该调用的子集。和ioperm一样，这一调用仅适用于i386平台。】

8、hwclock 【用来显示与设定硬件时钟。在Linux中有硬件时钟与系统时钟等两种时钟。硬件时钟是指主机板上的时钟设备，也就是通常可在BIOS画面设定的时钟。系统时钟则是指kernel中的时钟。当Linux启动时，系统时钟会去读取硬件时钟的设定，之后系统时钟即独立运作。所有Linux相关指令与函数都是读取系统时钟的设定。】

9、shmat(把共享内存区对象映射到调用进程的地址空间)：连接共享内存标识符为shmid的共享内存，连接成功后把共享内存区对象映射到调用进程的地址空间，随后可像本地空间一样访问

10、dmesg 【dmesg用来显示开机信息，kernel会将开机信息存储在ring buffer中。若是开机时来不及查看信息，可利用dmesg来查看。开机信息亦保存在/var/log目录中，名称为dmesg的文件里。】

11、ptrace 【ptrace提供了一种使父进程得以监视和控制其它进程的方式，它还能够改变子进程中的寄存器和内核映像，因而可以实现断点调试和系统调用的跟踪。】

12、ACK 【ACK (Acknowledgement），即确认字符，在数据通信中，接收站发给发送站的一种传输类控制字符。表示发来的数据已确认接收无误。在TCP/IP协议中，如果接收方成功的接收到数据，那么会回复一个ACK数据。通常ACK信号有自己固定的格式,长度大小,由接收方回复给发送方。其格式取决于采取的网络协议。当发送方接收到ACK信号时，就可以发送下一个数据。如果发送方没有收到信号，那么发送方可能会重发当前的数据包，也可能停止传送数据。具体情况取决于所采用的网络协议。】

13、IPC 【Inter-Process Communication，进程间通信】
