---
layout: post
author: Shawn Chang
title:  "不可避免的内存安全（Memory Safety）之路"
summary: Santizer is the most effective way to enhance the memory safety. Fuzzer helps as well! Fil-C... 
categories: system-security
---

## 背景
Memory safety 近年来成为热门话题。但在讨论“memory safety”时，我们需要先明确究竟在探讨什么、追求什么目标。你是在关注通过编译器完成静态分析（如 [Clang Static Analyzer](https://clang-analyzer.llvm.org/)、rustc 等）来在编译阶段捕获潜在问题，还是更信任编译器让代码顺利编译，通过运行时机制（比如 Go 或 Java 中的垃圾回收）来解决所有问题？或者，你仅仅关注于安全加固的最终目标——即防止系统遭受攻击？内存安全问题的复杂性正反映了安全领域的本质：安全是一门交叉学科，融合了计算机科学和复杂性理论，这使得要完全掌控其复杂性变得异常困难。因此，企图通过单一或者几种 “memory-safe language” 重写现有软件，从而彻底杜绝所有安全隐患，并非现实可行的方案。

一门编程语言在设计时可能就倾向于提供内存安全机制，例如自动垃圾回收、数组边界检查等，这些机制在规范层面上勾画了一个理想状态。但在现实中，不同的实现者会出于需求和性能指标的考虑采取不同的策略。例如，虽然 Lisp 通常配备垃圾回收机制、支持灵活的数据操作和动态类型系统，但这并不意味着所有 Lisp 解释器都能完全消除内存安全问题。如果由于特定需求或追求性能而对部分安全检查作出妥协，那么内存越界或非法指针访问等安全隐患依然有可能出现。

同样，C/C++ 被长期视为“不安全”的语言，因为它允许程序员直接操作内存和执行指针运算。然而，通过严谨的工程化手段（如静态分析工具、严格的代码审查、运行时检测机制等），使得 C/C++ 在特定环境下无限接近无 Bug 状态也是可能的。本文将以 HardenedLinux 过去数十年中在对抗系统复杂性、提升内存安全方面的一些做法为背景进行探讨。总体来看，内存安全不仅关乎编译器或运行时单一环节的责任，而是需要在语言设计、工具支持、工程实践等多方面协同努力，以实现最终“系统不被攻陷”的安全目标。本文不涉足强制访问控制，沙箱，Linux内核加固等议题。

## 为生产环境选择GNU/Linux发行版

Hardened Linux 的早期成员都拥有商业 Linux 发行版的背景，因此我们深知构建一个 GNU/Linux 发行版并不困难。然而，要打造一个长期维护且稳定的发行版，就需要更高的标准。首先，关键的软件和库必须由专业的维护者负责管理。在处理 bug 修复和安全漏洞时，维护者不会轻易地将问题抛给上游版本，而是根据具体情况进行权衡，大部分时间里选择 backports 方案来保留原有特性和稳定性。

正是基于这些考量，Hardened Linux 的最佳实践主要依托于 Debian 发行版，因为在当时，Debian 是唯一一个完全由社区驱动且拥有较高维护水平的发行版社区。一旦确定了基础发行版，其首要挑战便在于如何应对“memory unsafe”语言（如 C/C++）开发的应用程序和依赖库中存在的质量与安全隐患问题。在 sanitizers 和 fuzzers 广泛普及之前，大规模的 bug hunting 通常需要投入巨大的人工成本。然而，现如今，仅依靠启用 sanitizers 并执行常规回归测试就能够在 QA 流程中有效发现大部分问题；进一步来说，还可以引入口径广泛的 fuzzing 测试，以更全面地捕获潜在漏洞。即使对于GNU/Linux发行版级别的QA同样可采用这种方法，只有少量无法被sanitized的组件比如编译器和C runtime（glibc/musl/etc)。

此外，我们花费了多年时间构建基于状态的 Linux kernel fuzzer，并于 2020 年在 Google Syzkaller 上游贡献了 coverage filter 特性。这或许是来自亚洲开源社区为数不多的具有重大影响力的功能之一。模糊测试结合 sanitizer 的技术路线，是一种典型的增强内存安全的方案。以 Linux 内核为例，不同的用户场景对内核子系统的依赖各不相同：存储服务器更加依赖于文件系统，而网络服务器则更侧重于网络协议栈。后来被称为[VaultFuzzer](https://hardenedvault.net/zh-cn/blog/2022-08-07-state-based-fuzzer-update/)的基于状态模型的模糊测试工具——支持对特定目标系统进行压力测试。举例来说，在配备 32 核 CPU 和 64GB 内存的条件下，仅需 20 小时左右，就可以使主要网络协议实现部分的代码覆盖率达到约 72%。当然，有人可能会质疑，剩余 28% 的代码区域依然存在内存安全风险。对此，我们的思路是采用整体安全架构设计，例如内核层面的 runtime mitigation 措施，来进一步弥补这一缺口。马上将会讨论这一问题。

## 从Exploitability到memory safety方案

一个漏洞的全生命周期通常包括以下阶段：

* 找到 bug，并评估其是否可用于攻击
* 若被评估为 exploitable，则确认其为漏洞，并编写 PoC（Proof of Concept）
* 对 PoC 进行适配，使其成为一个稳定的 exploit
* 数字军火商将其整合到 weaponized framework 中

在 QA 工程师寻找 bug 的过程中，其使用的工具和流程（例如 fuzzer + sanitizer）与安全研究人员颇为相似。不过，从安全研究人员的视角来看，一个漏洞利用过程一般可分为以下三个阶段：

![](/images/exp-stage.png)

* 前漏洞利用阶段（Bug 触发）
* 漏洞利用阶段
* 后漏洞利用阶段（如植入 rootkit）

其中，Bug 触发过程被视作前漏洞利用阶段。从自由开源软件项目的角度，这一阶段的问题都应该被 QA 流程处理。那么，除了常用的 sanitizer 与 fuzzer，还有哪些方法能在这一阶段对漏洞进行检测呢？这正是我们为之振奋的[开源项目 Fil-C](https://github.com/pizlonator/llvm-project-deluge) 的价值所在。Fil-C 是由 Epic Games 开发的一套针对 C/C++ 的 memory safe 方案，其方法较为激进。Fil-C 定制了 Clang/LLVM 编译器，并对编译器相关库进行改进，以便更好地实现各类转换 (passes) 和 C 运行时（例如 musl）的支持；同时，部分库和应用也需要进行少量代码适配。从这一[系列测试程序](https://github.com/hardenedlinux/memory-safety-coverage-test)：

|Bug type     | Sanitizers | Fil-C             | Clang bounds-safety |
|:-----------:|:-----------------:|:-----------------:|:-------------------:|
|0-out-of-bounds-access.c| YES (ASAN)    | NO                | NO                  |
|2-out-of-bounds-access.c| YES (ASAN)   | YES               | NO                  |
|3-out-of-bounds-in-bounds.c| YES (ASAN) | YES               | NO                  |
|1-overflowing-out-of-bounds.c | NO (ASAN) | YES             | N/A                 |
|4-bad-syscall.c | NO (ASAN)             | YES               | N/A                 |
|5-type-confusion.c | NO (ASAN)         | YES               | N/A                 |
|6-use-after-free.c | YES (ASAN)        | YES               | N/A                 |
|7-pointer-races.c | YES (ASAN/TSAN)    | Partially               | N/A                 |
|8-data-races.c | YES (TSAN)        | NO               | N/A                 |

通过对比 Fil-C 与常规 sanitizer 的检测效果可以看出：虽然 Fil-C 不能解决所有问题，但从 exploitability 的角度来看，其成功实现了 memory safe C/C++，使得常见的漏洞失去了威胁。更有趣的是，Fil-C不仅可以帮助你找出问题，你甚至可以用其编译一些重要的组件作为独立的运行时（或许下一代Epic game的游戏中会有体现？），当然目前Fil-C的性能和工程化还无法达到通用的地步，但这是一个非常值得关注的方向。

## Exploit vs. Mitigation
既然已经讨论了漏洞挖掘（bug hunting）的过程，那么额外探讨下 mitigation 也是非常必要的。如果依赖于 sanitizer+fuzzer 的模式并不能找出所有 bug，而目标程序由于性能考量又无法采用像 Fil-C 这样的 memory safe C/C++ 方案，那么可以考虑由编译器和 C runtime library 提供的一系列 mitigation 技术作为补充。

这些 mitigation 技术既包含硬件层面的实现（如 NX、CET、BTI 等），也涵盖大量的软件实现。不要小看这类软件层面的保护措施：在关键时刻，它们往往能发挥重要作用，虽然2007年的Attacking the CORE已经转向内核，但无数的案例表明即使用户空间程序也是需要它们的，近期一篇关于 [ret2 的 write-up](https://github.com/hardenedlinux/memory-safety-coverage-test) 就介绍了如何在 Pwn2Own Ireland 2024 大赛上，针对 Synology DiskStation DS1823xs+ 的 RCE 远程利用进行漏洞利用构造。如果 Synology 的安全团队仅通过简单开启 FULL RELRO 和 CFI，那么防御体系将更加完善，漏洞利用的难度也会大大增加，故事的结局可能完全不同。我们常用的mitigation列表：

|Bug type     | Sanitizers |
|:-----------:|:-----------------:|
|Stack canary | -fstack-protector-*|
|Shadow stack | -mshstk (GCC), -fsanitize=safe-stack (CLANG)|
|Fortified source| -D_FORTIFY_SOURCE=3 |
|Full ASLR with PIE| -pie  |
| Control flow integrity| x86: -fcf-protection=, arm64: -mbranch-protection=, SW|
| Relocation only| -Wl,-z,now |
|Bounds check | -XClang -fexperimental-bounds-safety (CLANG) |

## Call for action
在过去几年中，与开源开发者甚至安全研究人员交流时，我曾多次听到这样的质疑：Google试图通过漏洞挖掘（bug hunting）来提前发现漏洞，但依然有不少问题漏网而出；既然如此，我们是不是应该用内存安全语言重写软件和库？这种观点乍看起来似乎有一定道理，但其实它与其他事实相悖。几个月前，我与一位南欧的安全工程师交流时，他向我抱怨某个与硬件安全模块（HSM）相关的开源密码学组件竟然存在“段错误”漏洞。出于疑惑，我问他难道在调试或测试版本中没有启用sanitizer吗？遗憾的是，我们确认了这个库确实没有集成sanitizer。请不要轻易猜测：这个库不是OpenSSL，而OpenSSL早已集成了sanitizer，并且确保所有回归测试都通过。回到“如果连Google都没解决这类问题，那我们就必须重写”的论调，就显得有问题。如果从0ldsk00l hacker和cypherpunks的语境来看，个体都应该拥有self-soverignty，难道仅仅因为Google没有解决所有问题，我们就可以忽视这些问题？绝对不能，你需要亲自修复问题，这不仅能为你自己带来好处，也会惠及整个社区。

因此，我在此呼吁所有自由开源软件的开发者：请在调试和测试版本中至少加入sanitizer编译选项。如果大家都能这样做，这将成为让世界变得更安全的最低成本方案。

## 重写软件
使用memory safe language重写软件和库是成本高昂的事情，如果你经过深思熟虑后决定要这么干，请考虑用Lisp/Scheme来重写。

## 引用
CC Bounds Checking Example
<https://williambader.com/bounds/example.html>

The Fil-C Manifesto: Garbage In, Memory Safety Out!
<https://github.com/pizlonator/llvm-project-deluge/blob/deluge/Manifesto.md>
<https://github.com/pizlonator/llvm-project-deluge/blob/deluge/invisicaps_by_example.md>

Exploiting the Synology DiskStation with Null-byte Writes: Achieving remote code execution as root on the Synology DS1823xs+ NAS
<https://blog.ret2.io/2025/04/23/pwn2own-soho-2024-diskstation/>

memory-safety-coverage-test
<https://github.com/hardenedlinux/memory-safety-coverage-test>

Technical analysis of syzkaller based fuzzers: It's not about VaultFuzzer!
<https://hardenedvault.net/blog/2022-08-07-state-based-fuzzer-update/>


