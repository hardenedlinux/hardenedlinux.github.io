---
layout: post
title: "云基础架构之固件安全合规"
summary: 经过数年的发展，数据中心，移动端（Android）以及嵌入式系统（IoT？）已经高度依赖于自由软件/固件/硬件，过去 12 年的基础架构层面的攻防对抗来中，Attacking the Core 的那个 Core 早已从内核转移到了 Hypervisor 之后又转移到了 EFI/SMM 最后 Intel ME 成为了新的 Core。但在某种程度上讲，内核依然是一把基路伯之剑，它的一举一动依然会影响到更底层恶魔的行为
categories: system-security
---

by Shawn C[ a.k.a "citypw"]

## 云基础架构安全之固件安全合规

固件作为IT核心基础架构的一部分，由于对于大多数用户“不可见”的特性长期在安全方面受到忽视，但[固件层面的攻防](https://github.com/hardenedlinux/firmware-anatomy/blob/master/hack_ME/firmware_security.md)一直都没有停过，Attacking the Core中的"Core"不断的向更底层转移，这个术语早已经不是[2007年语境下所指的操作系统内核](http://phrack.org/archives/issues/64/6.txt)，以x86为例，[RING 0](https://github.com/hardenedlinux/grsecurity-101-tutorials/blob/master/kernel_vuln_exp.md)是操作系统内核，[RING -1](https://github.com/hardenedlinux/grsecurity-101-tutorials/blob/master/virt_security.md)是VMM/Hypervisor，RING -2是UEFI/SMM，[RING -3](https://github.com/hardenedlinux/firmware-anatomy/blob/master/hack_ME/me_info.md)则是Intel ME/AMD PSP，越往下风险越高防御难度越大。随着公有云/私有云的兴起，核心基础架构（固件，操作系统内核，虚拟化，编译器，网络协议栈，密码工程，etc）级别的防护面临更大的挑战，从安全防御（攻击也一样）的视角看，从Research（研究）到Engineering（工程化）到Operation（运维）的链条的构建是现代数据中心的必要条件，继NIST于2011年和2014年NIST（美国国家标准技术研究所）发布了[SP 800-147（BIOS防护指南）](http://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-147.pdf)和[SP 800-147B（服务器BIOS防护指南）](http://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-147B.pdf)后，2017年5月30日发布了[SP 800-193（平台固件抗性指南）](http://csrc.nist.gov/publications/PubsDrafts.html#SP-800-193)的草案，SP 800-193是建立在SP 800-147/147B的基础上，虽然并未就攻防进化中的细节针对性的对固件安全的设计和实现提出强制性要求（NIST的一贯风格），但对于安全工程师，数据中心的管理人员以及硬件采购以及个人用户还是提供了一个参考，SP 800-193对于固件安全合规也有促进作用，本文对于SP 800-193草案所提到的部分内容进行探讨。

SP 800-193对于防护，检测，恢复和抗性提出了明确的要求：

* 防护( Sections 4.1/4.2)，固件代码和核心数据的完整性必须受到保护，比如固件升级流程中的验签名以及完整性，小心处理比RING0更高权限的代码
* 检测( Sections 4.1/4.3)，固件代码和核心数据被篡改的检测机制，比如在初始评估阶段就建立供应链的白名单
* 恢复( Sections 4.1/4.4)，固件代码和核心数据恢复的机制，比如双SPI flash
* 抗性( 符合所有Section 4)

信任根(ROOT OF TRUST, RoT)和信任链条( Chains of trust, CoT)是SP 800-193贯彻全篇会用到的术语，RoT指最初的信任链的基础，比如基于Intel ME的Bootguard或者UEFI的SEC，CoT指RoT之后的信任链条，比如grub的shim或者带签名的内核，Section 4.1.1对于RoT和CoT作出了如下要求：

* 1), 2)的要求Verified boot可以基本满足
* 3)要求完整性校验，可以使用基于TPMv1.2/2.0的measured boot完成
* 4)要求所有的固件信任链条更新，检测以及恢复都必须实现并且存放在非易失存储设备中，推荐平台厂商维护一个从固件启动到OS的防御，HardenedLinux社区的hardenedboot方案可以满足
* 5)要求运行OS的主CPU无法干涉RoT和CoT，这一点可行实现的前提是RING 0到RING -2的链条不被攻陷
* 6)要求软件从主CPU到平台固件的信息传递是不可信任的，在SP 800-147B中规定了如果不使用Service processor更新固件则不应该在实现上提供相关操作的权限，如果做不到则无法满足6)，对于这一点所有基于Intel ME的code module（包括AMT）都是巨大的风险
* 7)要求CoT的链条或许暂时性的扩展到从非易失性存储读取信息，所以在使用前必须由上一级的CoT进行密码学的验证
* 8)要求跨越了设备边界或者为共生设备提供服务的RoT和CoT应该使用安全的通讯机制


4.1.2关于信任根更新(RTU)和信任链更新(CTU)要求固件的更新必须验证签名，算法实现参考FIPS 186-4，而4.1.3的信任根检测(RTD)和信任链检测(CTD)要求对于每个环节都做检测，对于单个链条被攻陷的情况密码工程会有部分遏制作用，但对于整个固件被攻陷的场景处于同一层级的固件可能无法获得正确的信息，而这也会影响取证，切换OS并且执行取证软件也可能无法检测真实信息（同时也需要考虑某些无法dump的内存），可能性的解决方案：

* 事后处理，通过外部编程器获取固件文件并且逆向分析，这种方式成本高
* 事前处理，可以使用外设RoT比如基于Intel ME的Bootguard，如果选择Bootguard则需要在供应链采购时和OEM生产厂商协商写融丝公钥的事宜，这会加大密钥管理的风险和成本，对于数据中心并不推荐


4.1.4关于信任根恢复(RTRec)和信任链恢复(CTRec)可能性的实现方式：

* UEFI中实现自动化回滚机制
* 双SPI flash手工模式恢复，单SPI会出现一个容量的问题，SPI flash最大容量为16MB，而ME region就会占用8MB，除非干掉ME并且使用reloc


4.2除了满足BIOS的防护( e.g: NIST SP 800-147, NIST SP 800-147B)外，也需要处理包括Option ROM，管理控制器，服务处理器，硬盘和flash的固件驱动，网络控制器和图形处理单元的防护问题，4.2.1.1规定固件的升级必须进行签名验证，签名算法满足FIPS 186-4的要求，安全强度至少112位以满足SP 800-57的密钥管理合规的要求。4.2.1.2要求针对固件文件本身的完整性保护，硬件或者软件实现都可以。4.2.1.3规定防止攻击者通过绕过固件升级的认证过程进行恶意篡改固件，TCG spec里规定的CRTM( Core Root of TRust for Measurement)必须存在于不可涂改的ROM，即使攻击者重写了SPI flash但运行于内存中的CRTM依然可以通过TPM计算measurement，但UEFI/BIOS层面的漏洞利用会导致认证被绕过（更早期的BIOS实现映射1MB以下的内存甚至可以被Linux内核漏洞影响），Intel TXT曾经尝试简化(排除BIOS/Boot/OS loader)和实现独立于BIOS提供的信任链条，但TXT的问题在于Intel过渡依赖于SMM用于加载hypervisor和OS，而问题是UEFI/BIOS一旦被攻陷就可以任意加载SMM，导致这个问题又绕回了Bootguard。4.2.2谈到不可升级的内存比如ROM需要评估不可修复bug的风险，如果使用也必须保证现场不可升级内存的写保护无法修改。4.2.3中定义了关键性平台固件是指带有以下功用的：

* 执行防护，检测，恢复以及更新
* 维护关键性数据
* 为关键性数据实现不可被绕过的接口的代码

正是有了4.2.3的需求所以防护才从OS内核进入了更底层的领域，x86的SMM和ARM的Secure-el1/el3是基于主CPU的实现，也可以是外设运行Intel ME或者AMD PSP的代码模块，而在关键性固件和非关键性固件的代码运行时会通过特权隔离，但设计和实现上通常都会犯错，严格意义上讲Intel ME中任何一个code module的设计和实现的问题（比如错误的解锁区域让主CPU可以任意读写）都严重违反了4.2.3，因为在这个层级的任何错误都可以导致关键性代码和数据受到最高程度的安全威胁。4.2.4谈到关键性数据必须只能通过设备本身或者设备固件提供的接口去修改，比如读写SMRAM的数据通过I/O ports是一种常见形式，修改之前必须验证，比如格式检查，越界检查等，所有操作必须由管理员权限，这也强调了操作系统内核依然是重要堡垒，一旦基路伯之剑( RING0)被攻陷就为攻击者提供了进入地狱之门( RING -1/-2/-3)的钥匙。

另外，所有Deployment/Operational Phase（部署/运维阶段）的风险情况都是取决于Provisioning Phase（初始评估阶段）是否建立了可靠的评估机制，HardenedLinux社区建议为数据中心供应链建立OEM的固件以及基于自由开源实现的固件白名单，而OEM是否可信是另外一个问题，对于安全等级较高的机器所使用的固件如果是来自OEM厂商，应该要求厂商提供源代码，没有源代码的情况可以考虑进行二进制审计是否存在后门和漏洞，最可靠的方案是基于自由固件（比如[coreboot](https://www.coreboot.org/)/[libreboot](https://libreboot.org/)）的实现，从可审计的角度（注：不是安全防护），即使使用部分binary blobs的自由固件也比OEM固件更可靠，这些原则不仅仅适用于x86，也同样适用于arm64和[RISC-V](https://github.com/riscv/)。
