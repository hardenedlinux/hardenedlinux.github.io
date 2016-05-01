---
layout: post
title:  "关于RAP的FAQ"
summary: PaX/Grsecurity正式公布了针对Linux内核4.5里的新特性：RAP。RAP是一种在Linux内核层面上的CFI（控制流完整性）的实现，致力于完全消灭代码重用攻击这种漏洞利用的方式，RAP的发布是系统安全的又一里程碑，这意味着自2003年PaX team谈"未来"至今，只剩下data-only attack并未完全解决
categories: system-security
---

**原文：[关于RAP的FAQ](https://grsecurity.net/rap_faq.php)**

作者：The PaX team and Spender

译者：citypw( Shawn C)


> Shawn: 2016年4月28日下午三点，PaX/Grsecurity正式公布了针对Linux内核4.5里的新特性：RAP。RAP是一种在Linux内核层面上的CFI（控制流完整性）的实现，致力于完全消灭代码重用攻击这种漏洞利用的方式，RAP的发布是系统安全领域的又一里程碑，这意味着自[2003年PaX team谈“未来”](http://hardenedlinux.org/system-security/2015/05/23/archeological_hacking_on_pax.html)至今，只剩下data-only attack并未完全解决，从技术选型，研究，开发和测试发布，PaX team一共历经5年左右的时间，[RAP第一次亮相是去年的H2HC](https://pax.grsecurity.net/docs/PaXTeam-H2HC15-RAP-RIP-ROP.pdf)，这次公开发布的版本虽然只支持x86_64，没有经过连接时优化，编译时的静态分析以及返回地址保护等重要feature，但足以适应于大部分的场景，公开的版本是基于GPLv2自由软件许可证发布的。另外PaX/Grsecurity的下一个稳定版选为4.4，3.14的稳定版会一直维护到2017年年底。



### 什么是代码重用攻击？为什么能对抗防御到现在？

要解释整个故事，我们需要了解一些历史背景。从1990年代晚期开始，memory corruption漏洞的漏洞利用大规模的使用了被称为“shellcode"的技术去实现对应用程序的完整控制。2000年，PaX team发布了[PAGEEXEC](http://hardenedlinux.org/system-security/2015/05/25/pageexec-old.html)和[MPROTECT](http://hardenedlinux.org/system-security/2016/03/14/mprotect.html)，shellcode的使用（或者更精确的说法是，攻击者控制的任意代码）成为了不可能。

数年之后，业界基本赶上（处理器中的NX，Windows的DEP, etc），尽管一些操作系统仍然没有完全提供MPROTECT同等级的防护功能，因此每隔几年就会出现聪明的绕过技术。早在2000年引进PAGEEXEC的时候，另一种漏洞利用的类型变得显而易见 -- 最初被称为“[ret2libc](http://seclists.org/bugtraq/1997/Aug/63)”，然后进化为“[borrowed code chunks](https://dl.packetstormsecurity.net/papers/bypass/no-nx.pdf)”，终于正式成为“ROP” -- 攻击者可以重用现有在应用程序的代码而不是引入新的代码来达到同样的目的。例如，system()函数存在于glibc库，所以相比使用shellcode来执行一个shell，一个漏洞利用也可以从任何位置去重定向程序去执行现有的system()函数去强制执行一个shell。

2001年，PaX team开创了众所周知的地址空间布局随机化(ASLR)作为一种简单而廉价的概率性防御手段对抗此种被更多称为代码重用的攻击。代码重用攻击需要知道一些现有执行代码的地址（比如system()函数的地址），ASLR让攻击者更难精准的找到这些地址。而问题在于ASLR只是一个非常简单的设计和实现，它有一个致命的弱点：信息泄漏。一旦攻击者知道或者推断出代码的位置，在很多场景下漏洞利用可以动态的调整自身让ASLR完全失效。这类泄漏也提高了对于代码可能无法预先知道的内容这类场景的可靠性（比如使用不同的系统上运行不同版本的同一个库）。

自从2003年PaX team公布了[pax-future.txt](https://pax.grsecurity.net/docs/pax-future.txt)后，学术界和工业界急于解决这类问题，一个在pax-future.txt中描述过的但被形式化的版本于2年后的2005年成为了众所周知CFI（控制流完整性）。很多学术界的论文和工业界针对这个问题的一些子集进行了研究而产出的防御方案但都被轻松的击败和绕过了。包括微软和Google在内没有一个方案能同时完成3件事情：任意大小的codebase，速度快以及足够安全去防御整个类型的攻击。这里带我们进入RAP( Reuse Attack Protector)。


### 是什么让RAP那么重要？

同时实现上面3点的难度并没有被业界正确的理解。一开始的CFI实现和那些甚至今天还在使用的（比如微软的CFI或者Google的间接函数调用检查）实现都是"forward-edge" CFI。这意味着他们的实现仅在转跳和调用一个具体函数时才会进行安全检查，但对于函数的返回并没有任何安全检查。虽然像SSP这种保护机制已经存在多年，但他们并不是一个真正能抵御针对函数的返回的防御方案，SSP的设计思想和类似的防御方案（比如微软的/GS）是用一个"canary"(Shawn注：直译为金丝雀，矿工使用金丝雀探测矿井里是否有一氧化碳，在本文翻译中如果上下文是canary技术则不做翻译）值放在临近返回地址的地方。在一些栈缓冲区溢出的事件中，canary是放在溢出的缓冲区和目标返回地址之间，canary可以直接被覆盖。就像金丝雀在矿工受伤害前探测到一氧化碳一样，SSP在函数返回前，canary的值会被检查以保证返回地址没有受到影响。但这些方案在3个主要的问题里困了多年：1) 不总是修改返回地址的场景需要覆盖canary，2) canary的值可以被类似我们讨论过的ASLR地址泄漏的方式被泄漏，3) 并未对这些实现进行适当的性能优化以及安全检查没有对应该有的函数进行保护。

RAP定义了一个威胁模型：假设攻击者已经具有最强大的“漏洞利用元素“：具有针对内存读和写任意次数的能力。其他的代码重用攻击方案没有在这个威胁模型下去设计，所以它们失败的地方对于RAP无效。对于这个现实的威胁模型，像ASLR和/GS类的技术已经无效。

回到CFI的议题，另外一个学术界困扰很久的问题是安全和性能之间的平衡。很多forward-edge CFI实现也被称为粗粒度CFI。即一个特定的调用或者转跳进入一个函数，CFI的实现将允许攻击者调用的数量非常大：包括程序以外的合法的执行调用和在程序和库里的。这些粗粒度实现有两个原因：性能和信息限制。更细粒度的实现通常会为每次调用和转跳带来更多的安全检查。特定的C++程序会需要那些检查，这会损害性能。相应的，特别对于那些不需要源代码的CFI方法会让分类方法难以满足限制调用目标的最小集合而消除误报的需求。

尝试解决这个问题让大多数CFI掉进了另外一个坑：可伸缩性。要对函数进行分类，一旦经过了连接时优化（LTO），这些实现都需要知道内存中的整个程序的信息。对于小的codebase这不是一个问题，但对于像Linux内核或者Chromium浏览器（Google自己的CFI也是一样）就是非常大的约束。

最终，一些现有的CFI实现（主要是那些由Google开发的）只针对C/C++的一些细分领域：一些不保护C函数指针，另外一些只保护virtual calls，没有任何实现有返回地址保护。要对比这些弱实现的性能的话，你需要把它们单独的性能损耗加在一起和RAP进行比较。

有一些关键点是让RAP成为最好的代码重用攻击的防护方案。RAP能抵御之前提到的所有攻击，即使在攻击者有最有价值的memory corruption漏洞的情况下。它在编译器适当的层面上被实现，RAP在编译的早期这样让编译器可以针对RAP所做出的改变进行优化从而提升性能。RAP知道什么时候应该加安全检查，更重要的是RAP知道那些检查应该去掉而不降低防御级别。RAP执行每个地址的检查都快于其他的CFI实现，这意味着不用降低覆盖率去提升性能，它甚至能在比其他CFI实现具有更高安全性的同时性能也更优。RAP尽最大可能的分类函数里的特定调用和转跳，具有修改简单代码的能力从而限制分类组。它以这种方式不需要获得所有关于程序在内存的信息从而可以展到很大的codebase。


### RAP怎么样工作？

RAP是以GCC编译器插件的形式实现的。这意味着你不需要使用特定定制过的编译器，你可以使用你的GNU/Linux发行版或者嵌入式厂商提供的任意版本的GCC。RAP的商业版具有两个组件。第一个组件是确定性防御限制从特定地址去调用的函数和函数返回的地址。第二个组件是概率性防御帮助保证一个函数可以返回的不仅仅是一个由第一组件定义的不同调用的组，事实上只能返回到这个函数被调用的地址。

第一防御从程序里使用类型信息和使用了一个hash函数创建一个hashes的集合，这样hash的数量就接近于针对不同类型的函数的数量。之前提到过RAP也可以使用简单的代码修改去增加颗粒度。比如多个函数存在接受一个字符串参数但不返回值。RAP可以使用C/C++的能力去赋予已知类型不同的名字，比如“sensitive_string"分离这一组函数为两个而保持相同代码的语义。因为基于类型信息的hash，所有RAP需要的信息都可以在一次编译中获得，这比（其他CFI实现）需要所有关于程序的信息更实用。

第二防御更复杂一些，一个函数的入口本质上通过函数“加密“了返回地址以让之前的所有代码不能修改返回地址。用于加密返回地址的key存放在CPU的保留寄存器中，这通常情况下保证了key不会被泄漏。加密的返回地址也会保存在一个寄存器中。当一个函数返回，插件中instrumented code会解密已存在的地址（合法的或者攻击者修改过的），使用保留寄存器里的key进行解密，然后对比地址。如果两个不匹配，执行就终止了。一点注意：虽然用于加密的key不应该存在内存里，通过分离两种类型的信息泄漏是有可能推断出key的。这也是为什么确定性的基于类型的hash的RAP防护为从函数返回保留了空间。好消息是大部分的情况用于加密的key不需要在线程，进程或者内核的同一生命周期中出现。在内核里，每个系统调用使用一个新key。同理，无限循环比如内核调度器也使用的新key。这样可以限制信息泄漏带来的潜在风险。

这里只是从比较抽象的层面来讲解RAP -- 当然，为了高性能和高安全性必须做很多细节的实现：指令编码，使用快速指令序列做检查，复杂优化通过和知道怎么样和在哪里去掉安全检查而不牺牲安全性。


### 关于指令不对齐？

一个RAP特性导致的不明显的结果是来自不对齐指令的威胁自然的消失了，不需要去实现性能压力的防御比如强制16-byte的指令对齐。当你去思考关于攻击的顺序时这个原因就会变得清晰：一个函数指针或者返回地址变成了攻击者修改后的指向一跳存在的指令中间而产生一些有用的无意的指令顺序。RAP确定性的保证了所有地址里，不论是潜在受污染的函数指针或者正在使用的返回地址（被称为简介流控制）只能转跳到合法的地址，因此它阻止了转跳到指令的中间或者其他不合法的地址。一个有趣的事实是类型hash编码也保证了一个函数不能返回到一个函数的开始，一个转跳和调用也不能直接移交控制到另外的调用和转跳。


### RAP如何处理共享库？

RAP的基于类型hash的决定性防御在处理共享库上比其他CFI实现更简单。一些细粒度的CFI必须在运行时（通常是加载库的时间）使用复杂，性能消耗大的算法。另外一些CFI方案在处理共享库上采用弱化函数分类，因此降低了安全防御能力。与此相反，由于所有编译单元上类型hash创建标准，在共享库中调用函数与RAP比主可执行文件本身作出间接调用的函数没有什么不同。

RAP可以逐渐的引入更大的codebase。它是可能只使用函数的类型hash而不需要在调用和转跳的地方插入验证检查的代码。这样的话，在库依赖里的函数指针原型和调用（RAP会在编译时检测））之间的不匹配不会需要在应用程序成功运行前去修复。当然，简介控制流在发生在那些未修复的库不会受RAP的决定性类型hash防护。


### RAP如何处理JIT（即时编译）？

现有的JIT引擎在设计时并没有考虑安全。在运行时最安全的生成代码是通过强制把使用代码从创建代码中分离。这可以通过把JIT引擎分离进入单独的进程，这个工作[SDCG](http://wenke.gtisc.gatech.edu/papers/sdcg.pdf)已经完成。更进一步，JIT引擎需要修改可以让RAP生成可用hash和保证JIT编码不会允许攻击者有控制8个JIT输出连续bytes的余地，这样不会被利用成合法RAP hash。为一个C++ virtual call伪造RAP hash，攻击者需要控制16个连续的byte。被现代JIT实现使用的constant-blinding技术足以胜任。


### 如何处理自由软件许可证？

支持GCC插件（比如RAP）的版本是基于GPLv3发布的。不像GPLv2，GPLv3允许一个版权拥有者（比如自由软件基金会）去创建特殊的许可证例外。在创建GCC插件支持，允许访问GCC内部的头文件和APIs，自由软件基金会想要避免GCC插件成为专利在市场上销售GCC开发者多年的工作成果。关于特定的例外，自由软件基金会的详细可见: http://www.gnu.org/licenses/gcc-exception-3.1.en.html

In the exception, called the "GCC Runtime Library Exception", it defines a term called "eligible compilation". The FSF defines eligible compilation as a binary compiled with a toolchain where each component is licensed with something compatible with GCC's GPLv3 license, where the components include GCC itself as well as any associated GCC plugins. The exception states that a binary may only be linked against the GCC runtime libraries (libgcc, libstdc++) if the binary was produced through the eligible compilation process. As the kernel is not linked with the GCC runtime libraries, this exception does not apply, and so the license of the public RAP demo is under the GPLv2. Since however the GPLv2 is incompatible with GPLv3, then this makes the userland binaries (which do link with the GCC runtime libraries) compiled through a non-eligible compilation process. Distributing these userland binaries would be illegal and would violate the copyright of the FSF (but not that of the PaX Team).

作为RAP插件唯一的版权拥有者，PaX team仅针对RAP完整版以GPLv3许可证给商业用户以允许合法的编译用户空间的库。

RAP商业版已经发布，有兴趣的可以联系 contact@grsecurity.net 询问详情。
