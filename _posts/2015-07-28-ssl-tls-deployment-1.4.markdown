---
layout: post
title:  "SSL/TLS部署最佳实践v1.4"
date:   2015-07-28 17:59:07
categories: jekyll update
---

译者：Shawn the R0ck( 1.3), Tom Li( 1.4)
Reviewers: Lenx Wei

原文：[SSL/TLS Deployment Best Practices](https://www.ssllabs.com/downloads/SSL_TLS_Deployment_Best_Practices.pdf)

作者：Ivan Ristić

version 1.4 (8 December 2014)

Copyright © 2012-2014 Qualys SSL Labs


## 摘要：

SSL/TLS是一个看似简单的技术。非常容易部署和让她跑起来，但是...她真的跑
起来了吗？第一部分是真的 —— SSL确实容易部署 —— 然而正确部属她并不容易。
为了确保TLS提供安全性，系统管理员和开发者必须投入额外的精力，去配置服务器和
编写应用程序。

2009年，我们在[SSL Labs](https://www.ssllabs.com/)开始了相关工作，因为我
们想明白TLS到底是在怎么样被使用，我们也打算弥补TLS缺乏易用的工具和文档
的局面。我们进行了对全局TLS使用情况的完整调查，以及实现了在线检测工具，但文
档缺乏的问题依然存在。这份文档是解决这个问题过程中的一步。

我们的目标是让已经不堪负重的系统管理员和程序员尽可能花费少量时间就能完
成安全站点或Web应用的部署，正是因为我们的目的如此，所以这份文档可能不够完备，遗漏了
一些高级主题。因此，我们只提供简单实用容易理解的建议。
对于那些想了解更多信息的读者，可以看看 Section 6。


## 1. 私钥和证书

TLS提供的安全质量完全依赖于私钥和证书。私钥是安全的基础，而证书则用于向访问者表明
服务器的身份。


### 1.1 使用2048位的私钥

在你的所有服务器上使用2048位的RSA，或者等价强度的256位ECDSA私钥。密钥的强度能
保证在相当长时间内的安全，如果你已经使用1024位的RSA，尽快替换它们。如果你
的安全需求必须使用大于2048位的密钥，请考虑ECDSA，因为性能不错。不过ECDSA的缺点
是小部分客户端不支持，因此你有可能需要同时部署RSA和ECDSA以确保互操作性。

> Lenx注：RSA 1024的强度相当于分组加密的80-96bit，已经被视为不安全。[T1]

### 1.2 保护私钥

私钥是重要的资产，尽可能限制能接触到私钥的人。推荐策略包括：

* 在一台可信的计算机(Shawn注:加固过的物理机器)上生成私钥和CSR(
  Certificate Signing Requests)。有一些CA会为你生成密钥和CSR，但这样做
  明显不妥。

* 受密码保护的密钥可以防止从备份系统中泄漏。然而私钥密码在生产系统中使用的
帮助是有限的，因为这并不能阻止一个聪明的攻击者从进程内存中截获私钥。
一些硬件设备可以在服务器被攻陷的情况下确保私钥安全，但这些昂贵的设备
只在对安全有严格要求的机构中使用。

* 在发现系统被攻陷后，吊销老的证书，生成新的密钥和证书。

* 每年更新证书，同时更新私钥。


### 1.3 确保充分的域名覆盖

确保你的证书覆盖到目标站点的所有活跃域名。比如你的主站是www.example.com，但
你可能还有个www.example.net。你的目标就是避免无效证书警告，因为那会让你
的用户产生疑惑从而影响对你的信任。

即使你的服务器只有一个主机名配置，也要记得你不能控制用户是通过什么路径
访问你的站点的，可能是其他的链接过来的。大部分情况下，你应该保证证书能
在没有www前缀的情况下工作(比如，example.com和www.example.com)。这里经验法则
就是：一个安全的WEB服务器应该有一个对所有DNS名称解析都合法的证书配置。

通配符证书(Wildcard certificates)有它的适用场景。但如果这样的配置意味着
暴露私钥给不必要的人群（特别是在跨越部门边界的情形下），则应该避免使用。
换句话说，越少的人能访问私钥越好。此外，要意识到共享证书可能会导致安全漏洞
从一个站点扩散到所有使用相同证书的站点。


### 1.4 从靠谱的CA那里获得证书

选择一个对待安全业务认真可靠的CA( Certificate Authority)。在选择CA过程
中考虑以下因素：

* 对待安全的态度

  大多的CA都会有常规的安全审计（否则根本没有资格当CA），但是其中一些会更重视
  安全。搞清楚哪些更重视安全不是一件容易的事情，但一个可行的做法
  是看看他们在安全方面的历史状况，他们如何响应攻击事件以及如何从错误中学习。

* 足够大的市场占有率

  满足此因素的CA不太可能轻易撤销所有证书，而这种事情过去曾发生在小的CA身上。

* 业务重心

  如果一家机构的核心业务是CA，那么一旦出现严重问题，他们将会受到严重影响。
  因此这些CA不太可能因为追逐利润而忽视证书部门的重要性。

* 提供哪些服务

  在最底线的情况，你选择的CA至少应该提供CRL( Certificate List)和OCSP(
  Online Certificate Status Protocol)这两种召回机制，并且提供一个高性能的OCSP服务。
  CA至少提供域名验证和扩展证书验证功能，最理想的情况可以让你自己选择公
  钥算法(今天大多站点都使用RSA，但在未来ECDSA的性能优势可能会变得重要。)

  > Shawn注：这里作者可能指的是ECDH/ECDHE_ECDSA，即ECDH密钥交换+ECDSA签名的证书或者ECDH算出TLS的临时session key+ECDSA签名

* 证书管理选项

  如果你的运维环境很复杂，需要一大堆的证书，那么选择一个能提供良好管理工具的
  CA。

* 技术支持

  选择一个技术支持优秀的CA提供商。

### 1.5 选择强算法签名证书

证书签名的安全依赖于签名私钥的强度，以及所使用的哈希函数强度。
今天大多数证书使用并不足够安全的SHA1哈希函数。业界正在逐渐
淘汰SHA1，而最后期限则是2016年末，这之后SHA1证书就不可接受了。

然而，Google Chrome在大限到来之前就开始对SHA1证书发出警告，如果你的证书
在2015年左右就要到期，你应该立刻替换这些证书。作为替代，你可以直奔SHA2
算法家族。不过在你动手之前，你需要先看看你的用户是否支持SHA2。一些旧客户
端，例如 Windows XP SP2 的 IE 6 就不支持（但依然在一些国家和机构重度使用）。


## 2. 配置

使用正确的TLS服务器配置，才能够确保将你的信任凭证正确的展现给站点的访问者，
确保只有安全的加密原语被使用，而且确保规避所有已知的安全风险。

### 2.1 部署有效的证书链

一个无效证书链会导致服务器证书失效和客户端浏览器报警告，这个问题有时候
不是那么容易被检测到，因为有些浏览器可以自己重构一个完整的信任链而有些
则不行。

在绝大多数部署场景中，仅服务器自身一个证书是不够的。一般需要多个证书建立一个信
任链。一个常见的问题是正确的配置了服务器证书但却忘了包含其他所需要的
证书。此外，虽然这些其他的证书通常有很长的有效期，但它们也会过期。而且一旦它们过
期就会使整个信任链作废。你的CA应该向你提供所有额外需要的证书。

一个无效证书链会导致服务器证书失效，并且导致客户端浏览器报警。而实际上，这个问题有时候
难以诊断，因为有些浏览器可以自己重构一个完整的信任链而其他浏览器则不行。

### 2.2 使用安全的协议

在SSL/TLS家族中有5种协议：SSLv2, SSL v3, TLS v1.0, TLS v1.1, TLS v1.2。

> Shawn注: TLS v1.3还在draft阶段

* SSL v2不安全，坚决不能用。

> Shawn注: OpenSSL和GnuTLS当前的版本(2014.12.2)不支持SSL v2

* SSL v3用于HTTP已经被确认为不安全，用于其他协议时安全强度也不足。
  它已经过时，不应该再被使用。

> Tom Li注: POODLE漏洞的出现彻底的废掉了SSLv3，受其影响，
  大量程序和库彻底取消了对SSLv3的支持。其实之前很多地方支持SSLv3
  的原因是兼容性问题。

* TLS v1.0在很大程度上是安全的。当用于非HTTP协议时，我们还不知道存在任何
  已知的重大安全漏洞。当用于HTTP协议时，我们能够通过精心的服务器配置，来保证
  它几乎是安全的。

* TLS v1.1和TLS v1.2没有已知的安全漏洞曝光。

> Shawn注: 由于Edward Snowden曝光的内容有关于NSA“今天记录，明天解密"的故事，
  所以大量的自由软件社区和暗网使者们在过去1年中(2013.7--2014)转向了TLS v1.2的PFS，2015年4月，[PCI-DSS v3.1] (https://www.pcisecuritystandards.org/documents/PCI_DSS_v3-1.pdf)规定所有SSL的版本以及早期TLS版本将于2016年6月30日后不再支持)

> Lenx注：某一些TLS 1.x实现由于没有正确实现对PADDING的校验，同样存在POODLE脆弱性问题。
  这些有问题的TLS实现包括F5，A10，Checkpoint, Cisco等厂家的设备。
  同样，Lucky 13攻击一样对老版本的OpenSSL, GnuTLS，F5等大量库/设备实现有效。
  请确认打上补丁。[T2][T3]


TLS v1.2应该成为你的主要协议。这个版本有巨大的优势是因为它有之前版本
没有的特性。如果你的服务器平台（或中间设备）不支持TLS v1.2，做个升
级计划吧。如果你的服务提供商不支持TLS v1.2，要求他们升级。

对于那些老的客户端，你还是需要继续支持TLS v1.0和TLS v1.1。使用临时的解
决方案（接下来会介绍），这些协议对于大多WEB站点依然被认为是足够安全的。


### 2.3 使用安全的加密套件(Cipher Suite)

要安全的通信，首先得保证你是和你想通信的另一方直接通信（而不是
冒充者或者存在能够监听的中间人），并且安全的交换数据。
在SSL/TLS里，加密套件是定义你如何安全通信的。
它们由一堆多样化的组件组成，以确保安全。如果其中一个组件被发现是不安全的，
你应该切换到其他的组件上。

你的目标应该是仅使用128位或者更强的加密、认证套件，其他都应该被排除掉：

* Anonymous Diffie-Hellman (ADH) 套件不提供认证功能
* NULL cipher suites不提供加密
* 出口密钥交换套件 (Export key exchange suites) 使用容易被破解的认证
* 使用强度不够的加密算法(比如40或者56位的加密强度)也容易被破解
* RC4比之前想象的要弱，你应该在检查好兼容问题后，尽快去除掉
> Tom Li 注：2015年三月曝光的手段将RC4攻击实用化，RC4坚决不要再用 [T4]
* 3DES仅提供大约112位的安全系数，这也低于推荐的最低128位，不过依然足够强。
但实践中更大的问题是，她比其他替代算法要慢很多。所以，出于性能我们不推荐她，
但她依然可以放在加密套件的最后面，用来兼容非常陈旧的客户端
> Tom Li 注：RC4安全漏洞曝光后，这是老旧客户端唯一能用的算法了


### 2.4 控制加密套件选型

在SSL v3和后来的版本里，客户端提交一个她支持的加密套件的列表，服务
器从列表中选择一个去跟客户端做协商，以构建一个安全的通信信道。
然而不是所有的服务器都能很好处理这个过程，一些服务器仅仅会简单的从列表中选择第一个。
让服务器选择正确的加密套件对于安全而言是极端重要的（详见 Section 2.7）。


### 2.5 支持正向安全（Forward Secrecy）

正向安全是一个协议特性，它使得安全会话不依赖于服务器的私钥。
当使用不支持正向安全的加密套件时，如果攻击者记录了通信内容，那么她可
以在未来获得私钥后，再解密先前的一切通信。你需要优先支持ECDHE套装，
来让浏览器选择支持正向安全。
为了支持更广泛的客户端，可将DHE套件作为ECDHE的协商回退（fallback)方案。

> Shawn注: NSA就在干这件事情，所以看出PFS有多重要了吧

### 2.6 关闭客户端发起的重协商

在SSL/TLS里，重协商允许一方停止交换数据而去重新协商一个安全会话。有一些
场景需要服务器发起重协商的请求，但客户端并没有发起重协商请求的必要。此
外，曾经出现过客户端发起重协商请求的拒绝服务攻击。

> Shawn注解: 每个重协商请求服务器的计算量是客户端的15倍

### 2.7 降低已知漏洞风险

没有什么是绝对安全的，很多防护方案都会随着时间推移成为安全问题。最佳实
践是随时关注信息安全的世界在发生些什么，然后采取必要的措施。最简单的是你
应该尽快的打每一个补丁。

下面的一些问题应该引起你的注意：

* 关闭不安全的重协商

  重协商特性在2009年时被发现是不安全的，协议需要更新。今天大部分厂商已
  经修复，至少提供了一个临时方案。不安全的重协商很危险，因为她很容易被
  利用，用来进行跨站请求伪造（CSFR）攻击，并在某些情况下引发跨站脚本（XSS）
  攻击。

* 关闭TLS压缩
  
  2012年，CRIME攻击[6]向我们展示了TLS压缩所导致的信息泄漏可以被攻击者用
  于还原部分的敏感数据(比如session cookies)。只有几款客户端支持TLS
  压缩（而现在就更少了），所以即使关掉TLS压缩，也完全不会遇到服务器性能
  问题。针对TLS压缩的攻击风险有限。

* 降低HTTP压缩的信息泄漏风险
  
  2个CRIME的变种攻击在2013年被曝光，不像CRIME针对TLS压缩，TIME和BREACH
  漏洞是针对压缩过的HTTP响应。HTTP压缩对于很多公司都很重要，这个
  问题不容易解决。风险减缓方案可能需要修改业务代码。

  对于TIME和BREACH攻击，只要攻击者有足够攻击你的理由，那影响等同于CSRF。

* 关闭RC4

  RC4 cihpersuites已经被认为是不安全而且应该关闭。目前，对于攻击者最好
  的情况需要百万次的请求，和大量的带宽。因此危害是比较低的，不过我们期待
  未来有改进的攻击手法。在去除RC4之前，检查这是否会影响现有的用户；换句话
  说，你应该查查有没有仅支持RC4的客户端。

* 注意BEAST攻击
  
  2011年曝光的BEAST攻击是2004年的一个针对TLS 1.0或者更早版本但当时被认
  为很难被利用的一个漏洞。一次成功的BEAST攻击的影响约等于会话劫持。在一段时间内，
  尽管问题出在客户端，在服务端避免BEAST攻击是合适的。但不幸的是，
  服务器需要使用RC4来避免问题，而这已经不再推荐了。因为这个原因，再加上
  目前BEAST攻击已经在大量客户端中被解决了，我们不再推荐在服务端避免攻击。
  在有大量旧客户端受BEAST攻击影响的情况下，使用RC4和TLS 1.0也许更安全。
  如何取舍需要在完全了解环境，建立威胁模型后小心决定。

* 关闭SSLv3

  SSLv3受到2014年10月曝光的POODLE攻击威胁。此攻击很容易被利用来攻击HTTP客户端运行
  JavaScript恶意程序。客户端也很容易被攻击者忽悠，从一个更安全的协议（如 TLSv1.2）
  降级到不安全的SSLv3。因此最好的解决方案是在服务器完全禁用SSLv3，绝大多数站点都
  可以安全的实施。

> Lenx注: 由于国内仍然存在大量IE 6客户端，不支持TLS 1.x。目前如果必须
  要支持SSLv3，那么只能选择RC4，并注意开启TLS_FALLBACK_SCSV防止降级攻击。
  此外注意库的及时升级，相关漏洞是一茬接着一茬的。


## 3. 性能

这份文档中安全是主要关注点，但我们也必须注意到性能的问题。一个安全服
务不能满足性能需求无疑会被遗弃掉。然而，因为TLS配置通常不会带来很大的性
能开销，我们把讨论限定在会导致严重性能下降的常见配置问题上。


### 3.1 不要使用强度过高的私钥

在建立一条安全连接的密钥协商的过程当中最大的开销是由私钥大小决定的，使
用密钥过短会不安全，使用密钥过长会的导致在一些场景无法忍受的性能下降。
对于大多的WEB站点，使用超过2048位的RSA/DHE密钥，或者超过256位的ECDSA/ECDHE密钥是浪
费CPU和影响用户体验的。

> Shawn注：256-bit的[ECC密钥强度](https://tools.ietf.org/html/rfc4492)足够胜任很长一段时间

### 3.2 确保正确使用Session重用

Session重用是一种性能优化技术，让耗时的密码计算操作的结果在一段时间里可重复使
用。当Session重用机制失效时可能会导致严重性能下降。

### 3.3 使用持久性链接(HTTP)

今天绝大多数SSL开销并非来自CPU密集型的密码计算操作，而是网络延迟。一个TLS握
手是建立在TCP握手结束后，她需要交换更多的数据包。为了让网络延迟最小化，
你应该启用HTTP持久化( keep-alives)，从而让你的用户能在一个TCP链接上发多次
HTTP请求。

### 3.4 为公共资源开启缓存(HTTP)

当使用TLS通信时，浏览器会假设所有的流量都是敏感信息。浏览器会把一些特定的
资源缓存到内存里，但是一旦你关闭了浏览器，这些内容就丢失了。为了提升性
能，为一些资源开启长期缓存，通过加入"Cache-Control: public"返回header给
浏览器标记为公共资源（比如图片）。

### 3.5 使用 OCSP Stapling

OCSP Staling是改版的OSCP协议，使得传递证书吊销信息成为TLS握手的一部分，直接
从服务器传递到浏览器。因此，浏览器不再需要额外联系OCSP服务器来验证服务器，
从而大幅降低连接耗时。

## 4. 应用设计（HTTP）

HTTP协议和WEB相关平台在SSL诞生后仍然在不断的进化。进化的结果就是有一些
今天包含的特性已经对加密不利。在这个Section里，我们会罗列出这些特性，也
包括如何安全的使用它们。

### 4.1 100%的加密你的网站

事实上”加密是一个备选“的思想大概是今天最严重的安全问题之一。我们来看看
以下问题：

* 网站应该用TLS但没用
* 网站有TLS但不是强制性的使用
* 网站混合了TLS和非TLS的内容，有时候甚至在相同的网页上
* 网站编程错误导致TLS被攻陷

虽然如果你知道你自己在做什么的话，这些问题大部分是可以避免的。然而一般而言，
唯一有效的方式是强制对所有的内容通信进行加密 —— 没有豁免。

### 4.2 避免混合内容

混合内容的页面是已经使用TLS，但有些资源（比如JavaScript文件，图片，
CSS）是通过非TLS的方式传输的。这些页面不安全，主动的中间人攻击者可以劫持这
些不受保护的JavaScript的资源，从而……例如劫持整个用户会话。就算你遵循了前面的
建议加密了自己网站上所有的内容，但也不排除来自第三方网站的资源是没有加密的。

### 4.3 理解信任第三方

网站通常会通过来自其他服务器的JavaScript代码来使用第三方的服务，Google Analytics是一个
应用广泛的例子。内含的第三方代码创建了一个隐式的信任链，让第三方可以完
全控制你的网站。第三方本身可能并没有恶意，但他们很容易成为攻击者的目标。
原因很简单，如果一个大型第三方提供商被攻陷，那攻击者就可以利用这一路径
导致所有使用它的人全都自动被攻陷。

如果你采纳了Section 4.2的建议，至少你的第三方链接在加密后可以防止中间人
攻击。此外，你应该进一步去了解你的站点使用了哪些服务，去除、替换或者承担其中的
风险并继续使用。


### 4.4 安全Cookies
.................................

为了安全，网站需要TLS。然而网站使用的cookies也要标记为安全。如果不能保护cookies，就
让中间人攻击者使用诡计获取信息成为可能，即使网站本身是100%加密的。


### 4.5 部署HSTS
.................................
HTTP严格传输安全（HSTS）是TLS协议的保护伞：它被设计成即使存在配置和实现错误的情况下，依然能保证安全。
设置一个简单的响应header就能在支持HSTS的浏览器（目前是 Chrome、FireFox、Safari 和
Opear，IE 很快就会支持）上激活保护。

HSTS的目的是很简单的：在激活之后，它就会禁止与网站进行任何不安全通信，自动把明文链接转换
成安全链接。一个额外的特性让用户不能无视证书警告（证书警告是
中间人攻击的标志，而研究表明大多数用户都会无视警告，最好永远不要让用户这么做）

支持HSTS是一项能大幅度提高你网站TLS安全性的措施。新的网站应该在设计的时候就考虑到HSTS，而旧的
站点则应该尽快支持。

### 4.6 关闭敏感内容的缓存

敏感内容应该被确实看作是敏感，只能发送给该知道的一方。尽管代理服务器看不到加密流量，
也不能把它共享给别人，但是随着基于云的应用在增加，你必须得小心区分公开资源和敏感内容。


### 4.7 确保没有其他漏洞

TLS不代表就安全，TLS的设计只是涉及安全的一个方面--通信过程中的保密性和
完整性——但还有其他威胁你必须面对。


## 5. Validation
在配置的时候可以进行调整的参数有一大堆，而你很难完全了解修改什么会有什么影响，
而有些时候改动可能是无意的；软件升级也会悄悄引入变化。因此，我们建议使用一款
SSL/TLS评估工具来检查你的配置是不是真的安全，并定期运行检查保证你一直都安全。
对于公开站点，我们推荐使用[SSLLab网站上的免费在线工具](https://www.ssllabs.com/ssltest/)
它的“握手模拟”功能在实践中非常有用，可以让不同的TLS客户端连接时时候的参数一清二楚。


## 6. 高级议题

下面的这些议题超出了这份文档的范畴，她们需要对SSL/TLS和公钥架构(PKI)有
更深的理解，这些议题依然是受到争议的。

* Extended Validation证书

  EV证书是很靠谱的证书，只有经过充分的线下审核后才给予颁发。证书的目的是明确
  机构和它的对应线上网站的身份联系。EV证书更难伪造，提供了更好的安全性，
  并且在浏览器上呈现给用户时的待遇也更高。

* Public key pinning

  Public key pinning的设计是为网站运维能限制哪些CA才可以为他们的网站签
  发证书。这个特性是Google开发的，目前已经硬编码到了Chrome浏览器里面，
  并且证明对避免攻击和引发大众关注非常有效。在2014年，FireFox也加入了对
  硬编码pinning的支持。一个叫做《HTTP的Pubilc Key Pinning扩展》的标准已经
  发展了很长时间了，很快讲会发布。我们期待这个特性今后至少被主流浏览器支持。

* ECDSA私钥

  事实上所有的网站都依赖于RSA私钥。这个算法是WEB通信安全的基础。因为一
  些原因，我们正在从1024位转向2048位的RSA密钥。而增加密钥长度可能会带来
  性能问题。椭圆曲线密码学(ECC)使用了不同的数学，能在较小的密钥长度下有
  较强的安全性。RSA密钥可以被ECDSA替代，目前只有少数的CA支持ECDSA，但我
  们期待未来会有更多。在迁移到ECDSA的时候，一个主要的问题是并非所有的
  客户端都支持它，如果你考虑使用ECDSA，应该确认它是否会影响用户连接服务器。
  有些平台支持双密钥部署，可以让你同时使用RSA和ECDSA以适配所有的客户端。


改动

这份文档的最初的版本是在2012年2月24日发布的。这个Section跟踪了文档修改
的时间，从1.3开始。

版本 1.3 (2013年9月17日)
此版本的改动有：
• 推荐替换1024位证书
• 推荐不对SSLv3进行支持
• 删去推荐使用RC4来服务端避免BEAST攻击的内容
• 推荐禁用RC4
• 推荐在未来禁用3DES
• 警告关于CRIME攻击的变种（TIME和BREACH攻击）
• 推荐支持正向安全
• 加入对ECDSA证书的讨论

版本 1.4 (2014年12月8日）
此版本的改动有：
• 讨论SHA1过时的问题，推荐迁移到SHA2系列算法
• 推荐禁用SSLv3，提及POODLE攻击
• 扩张Sectios 3.1，涵盖DHE和ECDHE密钥交换强度
• 推荐OCSP Stap

感谢

为了有价值的反馈和起草这份文档，特别感谢Marsh Ray (PhoneFactor), Nasko
Oskov (Google), Adrian F. Dimcev和Ryan Hurst(GlobalSign)。也感谢其他慷
慨的分享关于信息安全和密码学的人。这份文档虽然是我写的，但这些内容则来
自整个安全社区。

关于SSL Labs
.................

关于Qualys
................



[1] SHA1 Deprecation Policy (Windows PKI blog, 12 November 2013)
http://blogs.technet.com/b/pki/archive/2013/11/12/sha1-deprecation-policy.aspx

[2] Gradually Sunsetting SHA-1 (The Chromium Blog, 5 September 2014)
http://blog.chromium.org/2014/09/gradually-sunsetting-sha-1.html

[3] On the Security of RC4 in TLS and WPA (Kenny Paterson et al.; 13 March 2013)
http://www.isg.rhul.ac.uk/tls/

[4] Deploying Forward Secrecy (Qualys Security Labs; 25 June 2013)
https://community.qualys.com/blogs/securitylabs/2013/06/25/ssl-labs-deploying-forward-secrecy

[5] Increasing DHE strength on Apache 2.4.x (Ivan Ristić’s blog; 15 August 2013)
http://blog.ivanristic.com/2013/08/increasing-dhe-strength-on-apache.html

[6] TLS Renegotiation and Denial of Service Attacks (Qualys Security Labs Blog, October 2011)
https://community.qualys.com/blogs/securitylabs/2011/10/31/tls-renegotiation-and-denial-of-service-attacks

[7] SSL and TLS Authentication Gap Vulnerability Discovered (Qualys Security Labs Blog; November 2009)
https://community.qualys.com/blogs/securitylabs/2009/11/05/ssl-and-tls-authentication-gap-vulnerability-discovered

[8] CRIME: Information Leakage Attack against SSL/TLS (Qualys Security Labs Blog; September 2012)
https://community.qualys.com/blogs/securitylabs/2012/09/14/crime-information-leakage-attack-against-ssltls

[9] Defending against the BREACH Attack (Qualys Security Labs; 7 August 2013)
https://community.qualys.com/blogs/securitylabs/2013/08/07/defending-against-the-breach-attack

[10] Internet-Draft: Prohibiting RC4 Cipher Suites (A. Popov, 1 October 2014)
http://datatracker.ietf.org/doc/draft-ietf-tls-prohibiting-rc4/

[11] Mitigating the BEAST attack on TLS (Qualys Security Labs Blog; October 2011)
https://community.qualys.com/blogs/securitylabs/2011/10/17/mitigating-the-beast-attack-on-tls

[12] Is BEAST Still a Threat? (Qualys Security Labs; 10 September 2013)
https://community.qualys.com/blogs/securitylabs/2013/09/10/is-beast-still-a-threat

[13] This POODLE bites: exploiting the SSL 3.0 fallback (Google Online Security Blog, 14 October 2014)
http://googleonlinesecurity.blogspot.co.uk/2014/10/this-poodle-bites-exploiting-ssl-30.html

[14] About EV SSL Certificates (CA/B Forum web site)
https://www.cabforum.org/certificates.html

[T1] HAS THE RSA ALGORITHM BEEN COMPROMISED AS A RESULT OF BERNSTEIN'S PAPER?
http://www.emc.com/emc-plus/rsa-labs/historical/has-the-rsa-algorithm-been-compromised.htm

[T2] Poodle Bites TLS
https://community.qualys.com/blogs/securitylabs/2014/12/08/poodle-bites-tls

[T3] Lucky Thirteen: Breaking the TLS and DTLS Record Protocols
http://www.isg.rhul.ac.uk/tls/Lucky13.html
