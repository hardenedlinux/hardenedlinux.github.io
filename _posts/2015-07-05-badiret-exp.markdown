---
layout: post
title:  "后续故事：数字军火级别的\"BadIRET\"漏洞利用(CVE-2014-9322)"
date:   2015-07-05 15:46:39
categories: jekyll update
---


作者：pi3, July 4 2015

原文：[Follow-up on Exploiting “BadIRET” vulnerability (CVE-2014-9322)](http://blog.pi3.com.pl/?p=509)

译者：Shawn the R0ck, July 5 2015


Shawn：nergal在2015年2月公开的针对BadIRET漏洞的分析文章仿佛就在昨天，但Linux内核社区对安全的态度的确比10年还糟糕，Linux内核社区一如既往坚持"Security through obscurity"这种简单到根本不用考虑斯拉夫兵工厂的威胁建模，不辛的是，BadIRET又是一例Linux内核社区认为几乎不可利用但实际是可利用的漏洞，更糟糕的是一些商业GNU/Linux厂商因为各种原因（在后棱镜时代，不得不考虑非商业因素的可能）也承认是否有公开的漏洞利用会成为他们对于风险评估的重要指标，这种说法好像是当0-day exploit从来都不存在一样，这的确是一个黑暗的时代，就算没有Mr.Snowden，这的确也称的上是一个黑暗的信息时代，anyway，这次pi3公开了BadIRET的漏洞利用代码，这对于安全研究人员来说是一个好事，也希望整个事件能帮助个人以及企业GNU/Linux用户对于安全有正确的认识。

一个漏洞的产生到漏洞利用至少会经历好几个阶段：Bug --> exploitable bug(vulnerability) --> poc --> exploit --> reliable/weaponized exploit。虽然skiddie都喜欢把fuzzing出来的bug讲成blah-blah-blah的故事或者作为PR，但让数字军火商或者斯拉夫兵工厂关心的漏洞属于能到最后两个阶段的vulnerability。

---------------------------------------------------
by pi3

Hi,

分析[CVE-2014-9322](http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-9322)的旅程并非一帆风顺，但它值得我们花一些时间去分析所有的信息。我会尽力...

## 1) Introduction – non-technical (almost)

一切都开始于[CVE-2014-9090](http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-9090)。这个漏洞是由Andy Lutomirski发现的，引用MITRE的描述：
<pre>
"Linux内核3.17.4在arch/x86/kernel/traps.c里的do_double_fault函数没有正确的处理SS(Stack Segment)寄存器相关的错误处理，导致本地用户可以发启DoS攻击让内核panic(...)"
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
