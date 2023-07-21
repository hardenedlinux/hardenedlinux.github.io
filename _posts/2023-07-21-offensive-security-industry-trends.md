---
layout: post
author: Maor Shwartz
title:  "数字军火行业的兴衰及调整 - 趋势与更新"
summary: The boom, the bust and the adjust The offensive cybersecurity industry - trends and updates中译本
categories: system-security
---

作者：Maor Shwartz

原文：<https://medium.com/@maor_s/the-boom-the-bust-and-the-adjust-ea443a120c6>

译者：Vault Labs

译者序：上一篇文章简单的[介绍了0-day数字军火行业](https://hardenedlinux.github.io/system-security/2023/07/20/0day-industry.html)，这篇<数字军火行业的兴衰及调整 - 趋势与更新>详细的总结了数字军火行业的历史以及现状，我们认为这篇文章不仅可以让公众有机会了解数字军火行业，更重要的是可以给安全工程师，安全分析师，CISO（首席安全官）和其他机构决策者一个参考，在赛博世界中，武器化漏洞利用是攻击者的破局之道，而对于防御的一方来说只有理解了武器化漏洞利用和数字军火行业才能更好的打造属于自身的赛博堡垒，从入行安全到 HardenedLinux 再到 HardenedVault，我们经历了 0ldsk00l 黑客的 “Hacking for fun and profit" 到不可避免的 ”This is cyber, sir!" 网络战年代，这些历史或多或少都在这篇文章中有所提及，不论什么时代性的背景，我们都应该谦卑，这样至少在面对 The Desert of the Real 时降低看花眼的概率。

## 简介

2019年，我有机会在BlackHat USA¹上发表演讲，试图回答“研究人员如何与数字军火行业互动”的问题。

四年后，数字军火行业经历了重大变革，重塑了这个行业。我认为现在是时候讨论导致我们今天的趋势和事件了。

在本文中，我将分析数字军火行业随着时间的推移而发生的商业案例变化。

如果你对数字军火行业的历史不太感兴趣，想要了解当前的趋势和事件，请跳到“调整（2022+）——清算所的崛起和行业供应链的重塑”一节。

*值得一提的是，本文反映了我在行业中的个人经验。根据与同行业同事和行业人士的交谈，他们有着类似的经历。

**如果你是政府特工、研究人员或政府机构的一部分，请特别注意“行动呼吁”部分。

## 关于我

我已经在数字军火行业工作了七年多了（顺便提一下，数字军火行业已经有大约20年的历史了）。从供应链（漏洞/研究人员）的角度来看，我的职责包括帮助研究人员、公司、研究团队、经纪人和政府在数字军火行业中导航。

Twitter: https://twitter.com/malltos92

## 目录

    * 框架
    * 厂商
    * 从社区向行业的转变
    * 繁荣期（2017-2019年）——“牛市中每个人都是天才”
    * 萧条期（2020-2021年）——“只有退潮时，你才会知道谁在裸泳。”
    * 调整期（2022年及以后）——“清算所”的崛起和行业供应链的重塑
    * 行动呼吁
    * 结论
    * 脚注

## 框架

我们可以将数字军火行业的时间线分为三个主要阶段，这基于该行业经历的事件和趋势：

![](/images/0day-industry/1.png)

在每个阶段，我将分析以下每个实体是如何影响其他实体的：

    * 供应链：研究人员，研究小组和漏洞。
    * 一条龙公司：提供感染（即漏洞）目标并安装代理（即恶意软件）收集选定设备数据的一条龙服务的公司。
    * 经纪人：买家、一条龙公司、政府、清算机构和卖家之间的中介。
    * 清算机构：稍后将在本文中解释这个。
    * 政府：运营攻击性网络能力的任何政府组织。

但在我们深入不同阶段之前，有一个主要的驱动力推动了变化 - 厂商。

## 厂商

多年来，厂商（即 Apple，Google，Microsoft 等）投入了大量资源使他们的产品更安全。他们通过引入新的缓解措施，芯片组，修复漏洞，雇佣一流的安全研究人员，创建漏洞赏金计划等来实现这一目标。

对于不熟悉数字军火行业的人来说，值得一提的是，政府和一条龙公司最喜欢的感染途径是从浏览器开始，使用远程代码执行（RCE）并将其与本地特权升级（LPE）结合，以控制设备本身。这种趋势在2017年之前尤为突出。

导致最喜欢的感染途径是从浏览器的主要原因之一是，获得批准访问目标设备上的数据比非法访问属于第三方但并非直接目标的服务器更容易。

厂商如何投资其安全性的一个很好的例子是 Apple。Apple 实施了一种“纵深防御策略”，他们理解只要有代码，产品/软件中就会存在漏洞。

因此，Apple 决定创建防御措施和缓解措施的层（即“纵深防御”），使得利用阶段更为困难。这迫使攻击者找到多个漏洞并将它们链接在一起，同时 Apple 不断更改代码使漏洞变得无关紧要。

Apple 随着时间的推移引入新的缓解措施，缩小了从浏览器可用的攻击面。他们创建了一个强大的沙箱环境，将功能分配给新的芯片组等。

起初，这些新的缓解措施并未被认为是漏洞研究人员的重大障碍。但是随着时间的推移，随着技术的成熟和安全团队的学习和改进这些缓解措施和技术，漏洞研究方面的事情开始变得更加复杂。

在这里，我们可以看到Apple（iOS）引入新的缓解措施的大致时间线，以及他们何时成为问题（填充形状）研究人员需要找到新的漏洞或技术来克服它们。

![](/images/0day-industry/2.png)

除了实施新的缓解措施之外，将不同的漏洞连接在一起以获取“完整链路”（即允许攻击者以提升的权限感染目标机器的完整组件载体）变得越来越困难。

这些努力的最终结果将我们从需要两个漏洞来控制目标设备转变为许多漏洞和利用技术链接在一起形成一个平面的链路。

![](/images/0day-industry/3.png)

Android已经实施了类似于iOS中存在的安全措施。两者之间存在一些值得一提的差异。主要的区别在于，Android 是一个分散的市场（OEM厂商），因此更难在设备的内核方面进行控制（即上游LPE或芯片特定的LPE）。

因此，谷歌明白他们主要需要关注OEM厂商的关系——让像三星这样的厂商尽快实施和发布安全更新，并保护 Chrome 的安全。

![Footnote3](/images/0day-industry/4.png)

## 从社区到工业化的转变

在这一部分，我想介绍一下从社区到工业化的转变是如何发生的，以及社区面临了什么挑战。

## 监管和媒体（新闻）报道

**监管**：几乎没有关于出口漏洞的法规。一些国家（如以色列）将漏洞纳入“出口知识”的出口管制法律中，但大多数国家只是开始讨论“入侵软件”（即恶意软件），而不是漏洞。

媒体（新闻）报道：几乎没有有关进数字军火的新闻文章，包括什么是它，它能做什么等等。

## 供应链

在我之前发布的文章中，讨论了行业中的事件和趋势，我提到了一般来说，数字军火行业中有四种类型的研究人员：

    * 一级：能够完成漏洞挖掘和编写0-day漏洞利用的研究者
    * 二级：能够挖掘0-day漏洞或编写漏洞利用的研究者
    * 三级：能够对现有漏洞利用进行维护工作的研究人员
    * 四级：赌徒

## 所有研究人员生而平等吗？

供应链在行业开始发展时扮演了重要角色。在开始时，漏洞研究人员被认为是稀缺的商品，任何在这个领域有经验的研究人员都被视为一级研究人员。

行业根本不知道如何评价这些研究人员。原因是行业不了解更好的方法。这是一个新兴行业，只有少数人知道成为漏洞研究人员意味着什么。

尽管在行业的早期阶段（即2008-2009年），有研究人员专注于特定目标，被认为是“浏览器专家”或“内核专家”等等，但是对研究人员的需求很高，而且无论这个人在 Web、操作系统或浏览器哪方面有经验都没有关系。

人们相信，如果你在一个领域有经验，你可以很容易地转向其他目标。事实上，转向是可能的，但是需要相当长的时间（从6个月到几年），才能提高研究能力，在这段时间内，研究人员并不会在漏洞挖掘或漏洞利用方面有生产力。

另一个原因是“街头信用”。这意味着“我在 X 部队服役过”、“我在这里那里发现了漏洞”、“我在这里那里玩 CTF”、“我认识这个人和那个人，我们过去曾黑掉过 XYZ”等等。

## 出售漏洞
研究人员面临的众多问题之一是找到如何以及向谁出售他们发现的漏洞。客户（即政府和一条龙服务的公司）和供应端（即研究人员）都不想因为做这样的生意而出名。常见的问题包括“我怎么知道我能信任你？”或“我怎么知道你是你所说的那个人？”。

迄今为止销售的大多数漏洞和利用程序都是以概念证明（PoC）的形式出售的。研究人员提供代码，允许触发漏洞，并且通常 PoC 攻击程序的可靠性值得怀疑，而且 PoC 只适用于特定的操作系统版本/设备。

购买此类项目的客户需要花费大量时间和资源将PoC开发为生产就绪的利用程序。在某些情况下，被出售的漏洞只提供某些类型的基本元素，客户需要继续进行研究，以达到最终目标（即代码执行）。

拥有生产就绪的利用程序意味着什么？

我将引用 Vigilant Labs 的 Mark Dowd 在 BlueHat 2023活动中的话：

    * 它必须能够正常工作并且工作可靠
    * 应该在不利条件下工作
    * 没有明显的副作用（设备锁定、视觉效果等）¹⁹
    * 执行需要继续，就好像什么都没有发生一样

此外，由于销售方被告知（内部或公开）而被修补掉，能力（即漏洞、技术、基本元素等）通常在开发期间得到修补。

研究人员面临的另一个问题是评估其发现的公平市场价格并结构化付款协议（这些是我在 2019 年 BlackHat USA 演讲中涵盖的内容）。

## 经纪人

经纪人在发展中的行业中扮演了重要角色，作为交易的促成者。经纪人了解客户（或至少在积极寻找客户），并且他们拥有想要出售漏洞的研究人员的网络。

经纪人必须建立基于信任的亲密关系，与客户和研究人员都是如此。这种关系是至关重要的，因为交易的性质如此。

简单来说，在同意条款和条件并签署合同后，研究人员需要将漏洞发送给经纪人，然后经纪人将其发送给客户进行验证。只有在验证过程之后，研究人员才会得到付款。

因此，可以想象，当您把价值数十万美元的漏洞发给经纪人时，您需要信任经纪人。

对于经纪人来说，他们喜欢成为守门人，因为他们可以在不需要自己找到漏洞的情况下在交易中分得一杯羹，并保持客户和研究人员的隔离。客户不知道漏洞的来源，研究人员不知道客户的身份。

## 一条龙公司

最初，只有少数知名公司与市场互动以雇用研究人员，了解存在哪些漏洞。只有一小部分公司从市场购买漏洞。

其中一个原因是公司内部的研究团队很容易找到并维护完整的链路。当内部研究团队在特定目标上遇到困难时，公司将与市场互动，试图填补链中的空缺。

一些公司仅在后期从市场购买漏洞和漏洞利用，并且他们必须追赶市场参与者、流程和价格。

## 政府

在2017年左右，只有少数几个政府作为买家活跃在市场上，通常是以壳公司监测市场趋势、漏洞和漏洞利用。

## 会议

会议是从社区向产业转型的加速器之一。会议是研究人员、经纪人、一条龙公司和政府代表相遇并建立关系的地方。

# 繁荣期（2017-2019）-“牛市中每个人都是天才”

![](/images/0day-industry/5.png)

这一阶段的主要特征是增长。从极度保密的社区到价值数十亿美元的产业。

## 一条龙公司

增长的主要推动力是一条龙公司。一条龙公司在教育政府有关数字军火能力的重要性方面做得非常出色，并为政府提供了掌握这些工具的重要性。此外，一条龙公司通过包括先前无法获得这种技术的国家来扩大了可寻址市场。

随着数字军火概念变得普及，政府对此类产品和服务产生了兴趣，特别是那些无法自主开发这些能力的政府。因此，一条龙公司开始在各地涌现。

“十年前，只有几家公司。现在有20多家公司，在世界各地的贸易展上积极推销他们的产品。”伦敦数据权利法律事务所和咨询机构 AWO 的主管埃里克·金德（Eric Kind）说道。

新公司提供了各种不同的感染目标设备的方法-从服务器、PC、IoT、移动设备、浏览器等等。

随着更多公司进入该行业，资金流入也相应增加（来自投资、贷款等）。对于公司而言，任何一条龙公司的核心都是支持业务的漏洞，因此也就产生了研究人员。

对于研究人员（一种稀缺商品）的竞争加剧，公司试图通过以下方式来雇佣研究人员：加入内部研究团队、从市场购买漏洞和攻击程序，以及以有偿的研发或独家格式与供应链（即研究人员和研究组）合作。

雇佣研究人员加入内部研究团队：公司提供高额的薪酬、奖金以及研究人员想要的任何东西（包括更靠近家的办公室、假期、音乐会等），只为了让他们为自己工作。

公司聘请各种熟练程度的研究人员（Tier 1-4），并认为聘请的研究人员越多，找到黄金的可能性就越大。

供应链：与行业互动，特别是从市场购买漏洞，由于公司、研究人员和经纪人开始公开工作，变得更加容易（不像“从社区向产业转型”阶段）。

一条龙公司为从市场购买漏洞和雇佣研究人员分配了“无限”的预算。繁荣期的另一个特征是，公司相对容易实现服务水平协议（SLA）。

## 服务水平协议（SLA）-附注

SLA规定了服务期限的条款和条件，包括但不限于：服务质量、服务可用性、支持请求的响应时间和其他相关因素。

![](/images/0day-industry/6.png)

在数字军火行业中，SLA 意味着该公司具备感染并在目标设备上安装代理的能力。

如果其中一个能力（即感染目标并安装代理的漏洞链）处于离线状态（即漏洞已被修补，厂商更改了代码导致攻击程序失效，厂商发布了新版本，公司需要调整攻击程序以适应新版本）- SLA 规定公司具有一定的允许时间来恢复漏洞利用的功能（通常为几个月）。

![Footnote13](/images/0day-industry/7.png)

据我所知，由于进口数字军火产品几乎没有任何法规限制，在某些情况下，一条龙公司被用作地缘政治领域的战略工具，因此一条龙公司拥有高利润率。

传统的分包商（即武器制造商）早在2000年代中期就开始提供数字军火能力，成功程度不同，并且只专注于向他们最重要的1-2个客户独家销售此技术。

在某个时间点上，传统的分包商意识到他们也需要进入这个行业（即与一条龙公司竞争）并以商业规模提供类似的能力，主要有两个原因：

    * 数字军火是下一个大趋势 - 来自客户的需求。
    * 分包商没有参与这个不断增长的行业，他们让竞争对手（即一条龙公司）主导市场（这是一些分包商的主要业务，与政府和情报有关）。

![Footnote13](/images/0day-industry/8.png)

随着越来越多的公司进入数字军火行业，高利润率的未开发市场的故事引起了人们的兴趣，吸引了那些试图参与但手头几乎没有资源的人们——“一次性公司（一条链）”。

“一次性公司”是指围绕某一特定的漏洞链，例如围绕 Android 攻击链条的公司。这些公司具有以下几个特点：

    * 非常小的团队，没有内部研究团队或非常小的团队。
    * 创始人可能只是几个研究人员、经纪人或想要进入市场并设法稳定特定漏洞利用链条的人。
    * 提供最小化的代理程序，目标受限，没有长期支持。
    * 与成熟公司相比，“一次性公司”的产品价格便宜。

## 一条龙公司的销售周期

一条龙公司的销售周期很长，从第一次会议到合同签订可能需要几年时间。该过程包括会议、规定批准、演示、概念验证、谈判、部署、问答、合同和审批申请（RFA）等等。

对于一条龙公司来说，销售过程中的第一个真正有价值的步骤是合同执行和审批申请。一旦客户（政府）签署了合同，他们需要通过支付总价款的20-40%的首付款来执行合同。其余的支付在合同寿命内的定义里程碑上支付（包括 RFA）。

审批申请通常涉及客户审查和验证厂商提供的产品或服务，以确保其符合协议中的规定规格和要求。一旦客户满意并批准产品，他们会同意或提供接受，从而允许向一条龙公司支付额外款项。

在大多数情况下，交易的完成得到当地代理人（通常是前将军）的帮助，他们的补偿基于交易的百分比。

不少情况下，合同可能会由一组人（采购）签署，而组织内的另一个组（技术人员）会处理审批申请——导致延迟和长时间的“完成”交易。

![Footnote13](/images/0day-industry/9.png)

在繁荣阶段，我每天都会听到新公司的消息，其中大多数都专注于移动领域。

如前所述，漏洞是任何一条龙公司的核心。每个公司所提供的漏洞利用链条状态都不同，有些公司在 RCE 方面存在短板，有些则是 LPE 等等。

公司所掌握的链条状态的差异导致它们从市场购买缺失的缓解。通过这样做，它们为市场提供了流动性。需求很高，如果你找到了一个漏洞，那么将其出售相对容易。

## 经纪人

交易的促成者（交易）通过让客户和研究人员保持隔离，享受着守门人的乐趣。随着对漏洞的需求激增以及有关每个漏洞可以出售多少的故事传开，吸引了许多人看到这个新兴市场作为经纪人的机会。

经纪人不需要理解漏洞的技术细节，也不需要承担责任，也不需要资金来开展业务。因此，一波新的经纪人开始在市场上运作（类似于一条龙公司的数量）。

新的经纪人来自各种背景：

    * 前研究人员/研究人员
    * 与潜在买家有联系的人
    * 没有数字军火市场或网络背景的人

随着更多的经纪人开始在市场上运作，研究人员的竞争变得激烈，经纪人会尽一切努力确保研究人员与他们合作而不是与竞争对手合作。他们会请他们吃饭、参加派对等等。

经纪人试图从各种能力中“确保”研究人员（即确保研究人员在找到新漏洞时首先联系他们）——因为在繁荣阶段，需求很高（操作系统、虚拟化、电子邮件、托管、移动、物联网、网站等）。

经纪人拥有的好处之一是可以从市场获得漏洞和利用程序。其中一些人开设了一条龙公司——如前所述的“一次性公司”。

## 供应方

研究人员是焦点。研究人员可以选择被一条龙公司雇佣，或独立寻找漏洞并承担风险（和回报）。

研究人员受到经纪人、其他研究人员和一条龙公司的追逐。

个人研究人员可以相对轻松地找到并出售漏洞或利用程序。少数研究人员组成了团队。这意味着相对容易找到和出售高价值的漏洞。

研究人员在各种产品和服务中提供漏洞，例如操作系统、虚拟化、电子邮件、托管、移动、物联网、网站等等——而这些漏洞和利用程序的需求很高。

低级别的研究人员（回顾起来）基于“街头信用”被一条龙公司雇佣，而具有漏洞研究经验但不符合公司重点的研究人员也被雇佣，因为人们相信他们可以迅速适应公司的需求。

## 政府

政府通过壳公司、一条龙公司（因为它们是监管机构）和经纪人与市场互动。一些有很多钱的知名国家也进入了市场。

![Footnote⁶](/images/0day-industry/10.png)

政府在该行业的主要活动集中在三个方面：

    * 购买一条龙解决方案：在与一条龙公司打交道时，政府开始意识到，在表面上，公司的提供非常相似。因此，价格是政府选择一家公司而不选择另一家公司的主要动机。
    * 大力投资培训：试图减少对一条龙公司的依赖，并在内部开发能力。
    * 购买各种漏洞和利用程序：Web、虚拟化、电子邮件、托管主机、移动端等等。

## 萧条期（2020-2021）- “只有退潮时，你才会知道谁在裸泳。”

这个阶段的特点是整个行业受到来自厂商、监管、媒体报道等方面的下行压力。

## 监管

随着政府越来越意识到数字军火的全部潜力，以及潜在的滥用可能性，他们开始加强对出口此类产品和知识（即漏洞和利用程序）的监管。

新的监管规定限制了一条龙公司在未经监管机构批准的情况下在某些地区或国家营销和销售其产品。此外，政府首次制定了漏洞和利用程序出口方面的政策并颁布法律。

政府将经纪人和一条龙公司都视为风险，并因此将其中一些列入了制裁名单。

此外，厂商也不再袖手旁观，对违反其条款和条件的一条龙公司提起诉讼，声称某些一条龙公司必须使用公司（即厂商）的基础设施才能对目标设备执行利用程序。

![Footnote⁷](/images/0day-industry/11.png)

![Footnote⁷](/images/0day-industry/12.png)

![Footnote⁷](/images/0day-industry/13.png)

![Footnote⁷](/images/0day-industry/14.png)

![Footnote⁷](/images/0day-industry/15.png)

![Footnote¹¹](/images/0day-industry/16.png)

![Footnote¹¹](/images/0day-industry/17.png)

## 媒体（新闻）报道

随着全球政府开始滥用数字军火能力，新闻记者们没有保持沉默。结果，记者揭露了一条龙公司、它们的客户、运营等等。

本文后面将会看到这些报道的后果。

![Footnote⁸](/images/0day-industry/18.png)

![Footnote⁹](/images/0day-industry/19.png)

![Footnote¹⁰](/images/0day-industry/20.png)

## 一条龙公司

随着一条龙公司面临着日益严格的监管、新闻记者揭露公司的行为和技术滥用、诉讼、制裁以及厂商加强其产品的安全性，一条龙公司不得不面对更多的挑战：

    * 投标竞争
    * 服务级别协议（SLA）
    * 内部研究团队
    * 漏洞被发现
    * 收入来源和监管
    * COVID-19 的经济影响
    * 漏洞价格上涨

这些挑战导致了财务困境，最终使公司破产或转向非数字军火领域。

## 投标竞争

在繁荣期，我告诉你有很多新公司进入了这个行业，它们很难区别自己和其他公司，因为它们最终提供了相同的终极目标。这些公司用来说服客户与他们合作的主要工具之一就是价格。

一条龙公司并没有通过提高服务价格来确保可行性（稍后我将介绍为什么它们的成本增加了而利润率却下降了），而是保持或甚至降低了价格，以便随着时间的推移留住客户。公司之所以能够承受这种策略，一方面是因为低利率环境，另一方面是因为他们明白，如果他们能够留住客户，随着时间的推移（而竞争对手将会破产或转型），他们可以提高价格。

## 服务级别协议（SLA）

为了履行 SLA，一条龙公司应该有能力在提供的最新组件载体上感染目标，例如在最新的原版 Android 上运行的链条（即 Chrome RCE，Chrome SBX 和 Android LPE）。

这些漏洞链应该：

    * 能够正常工作，并且可靠
    * 在不良条件下仍能正常工作
    * 没有明显的副作用
    * 执行需要继续进行，就像什么都没有发生一样

还记得这个吗？

![](/images/0day-industry/21.png)

在萧条期，厂商实施的技术和缓解措施达到了成熟阶段，导致一条龙公司无法履行 SLA。

![](/images/0day-industry/22.png)

一条龙公司为了维护其 SLA 而苦苦挣扎，导致了三个主要问题：

    * 收入收取：一条龙公司面临的困难在于无法让客户感染最新目标，因此难以从客户那里收取付款。
    * 客户流失给竞争对手：政府不想因为其当前厂商没有可用的攻击链而延误操作，而其竞争对手则具备这种能力。
    * 对内部研究团队施加压力以寻求解决方案：稍后会讨论到这个问题。

## 内部研究团队

迄今为止销售的大多数漏洞和漏洞利用都是以概念验证（PoC）的形式出现的。因此，内部研究团队不得不将大部分时间投入到将 PoC 开发成“可投产”的攻击链中，这使得他们面临着巨大的维护负担（例如适应不同的设备、版本、场景等），而不是专注于漏洞研究。

当无法履行 SLA 时，一条龙公司的管理层对内部团队和负责从市场采购漏洞的人/团队施加了很大的压力。简而言之，如果内部团队无法提供解决方案，那么公司就无法获得报酬。

根据我与在这些公司担任研究员的朋友和同事的交谈，研究员对管理层持有怨恨是很常见的，因为研究员试图警告他们研究需要时间，如果他们专注于维护工作，每当公司需要漏洞时，启动新的研究项目需要时间，或者需要研究员迅速掌握团队已经在运行的研究项目。

这种压力有两个有趣的结果：

**交付成果 VS. 技术技能声誉（也称“街头信用”）**：一条龙公司过去习惯于聘请各种研究员（例如三级至四级研究员），认为有更多的研究员就能取得更大的产出（即发现新漏洞）。

实际上，公司发现只有少数研究团队成员能够发现新漏洞并利用它们。 “街头信用”不再是一个因素，而那些在维护工作中努力或无法找到新漏洞的研究员被解雇。

**研究员离开一条龙公司**：有能力的研究员感到自己肩负了公司的重担，再加上来自管理层的额外压力，决定离开一条龙公司。

在那段时期，整个行业都向他们敞开了大门，竞争对手、经纪人等人通过提供机会，让他们建立自己的公司并利用其宝贵的经验专注于发现漏洞和销售漏洞。

另一个值得一提的有趣因素是，一条龙公司停止聘请新一代研究员，而是将资源用于吸引能够发现漏洞并履行 SLA 的高级研究员。

## 漏洞利用在野外被捕获

在萧条期，有很多漏洞被发现。这些事件有两个主要的后果：

* 一条龙公司常常会（无意中）使用相同的漏洞（无论是因为他们购买了相同的漏洞，还是因为两个不同的团队发现了相同的漏洞）。因此，如果一个公司被发现（在利用某一漏洞），其他公司也会受到影响。
* 厂商会审查攻击面并修补其他一条龙公司正在使用的变体，或者完全无效化攻击面。

## 收入来源和监管

新的监管规定限制一条龙公司在没有监管机构批准的情况下在某些地区或国家销售和推广产品。

根据我的经验，有几种不同类型的一条龙公司：

    * 销售给五眼国家（澳大利亚、加拿大、新西兰、英国和美国）的公司。
    * 销售给五眼国家和申根区（23个欧盟国家）的公司。
    * 销售给“西方国家”的公司。
    * 销售给未列入美国制裁名单的国家的公司。

只有少数几家公司仅销售给五眼国家，大多数一条龙公司属于上述列表中的后两类。

这意味着一条龙公司的核心收入流来自未列入美国制裁名单的国家（即非西方国家）。这些国家没有能力自行开发这种机能。

西方国家拥有内部研究能力，并采购一条龙解决方案。非西方国家和西方国家之间的区别在于，西方国家被认为是“好人”，并且知道有许多公司会热切地寻求他们作为客户。因此，西方国家利用这个优势来协商条款，包括价格，将其降至最低。相反，一条龙公司可以声称与“好人”合作。

重点是，由于监管，公司失去了来自非西方国家的收入来源，而这仍然是它们的主要收入来源。

## COVID-19的经济影响

2020年初，COVID-19 成为了国际关注的焦点。各国政府都动用预算和资源来处理这场迅速蔓延到全球的疫情。

由于前方充满不确定性，政府冻结了采购数字军火能力的计划，在某些情况下还推迟了对一条龙公司的付款。

反过来，一条龙公司也对情况感到不确定，并冻结了预算，在没有必要保持现有客户的 SLA 的情况下不会从市场购买漏洞。

## 漏洞价格上涨

攻击链的真实成本因两个主要原因而大幅增加：

* 创建可用的攻击链所需的漏洞数量增加（例如从RCE和LPE到多个漏洞和技术的组合）。
* 发现漏洞并利用它们变得越来越困难。

![](/images/0day-industry/23.png)

另一个需要考虑的问题是，由于监管和漏洞出口管制法规，一条龙公司不得不在许多国家开设实体以容纳与研究人员的交易，从而给一条龙公司带来了额外的成本。

## 破产和转型

一条龙公司面临着许多挑战，包括成本上升、收入下降、监管增加等等。不幸的是，其中一些公司难以应对这些困难，无法有效地管理财务压力。

![](/images/0day-industry/24.png)

这导致了一些公司破产，以及一些公司从数字军火行业转型：

![Footnote¹⁴](/images/0day-industry/25.png)

![Footnote¹⁵](/images/0day-industry/26.png)

![Footnote¹⁶](/images/0day-industry/27.png)

![Footnote¹⁷](/images/0day-industry/28.png)

一条龙市场首次经历了萎缩。这将产生重大的市场影响，我们将在接下来进行介绍。

## 供应链

供应链不仅面临着普遍的下行压力（例如监管和媒体报道），还面临着独特的挑战。

## 监管

加强了围绕漏洞/利用程序出口的监管，导致一些研究人员转向其他行业，因为他们不想或无法处理出口管制许可证流程。在某些情况下，出口完全成为非法行为。自然而然地，能够在需求目标上发现漏洞的研究人员不多，因此由于监管失去了一些研究人员，对供应方造成了相当大的冲击。

## 进入漏洞研究领域的门槛更高

随着厂商增强其安全措施，例如实施额外的缓解技术和缩小潜在攻击面，渴望从事漏洞研究的研究人员现在遇到了更高的门槛。例如：

    * 设备成本
    * IDA（交互式反汇编器（IDA），一种用于反向工程二进制可执行文件的流行反汇编器和调试器。）
    * 模拟（例如 Corellium）
    * 研究范围有限：没有某些能力（例如漏洞），研究人员无法搜索链中的下一个部分。

## 需求
从市场购买漏洞的主要力量是一条龙公司。现在竞争的公司更少，漏洞的总需求也下降了。

此外，在某些情况下，客户不会购买链条的一部分，除非他们拥有其余部分或者他们有足够的信心可以在短时间内购买/找到它们。现代漏洞利用链条很复杂，有很多可能会出问题的部分。因此，如果他们不确定能够迅速利用这些部分，客户将不会冒险使用这些资金。

同样适用于上市时间，如果有一个研究人员发现了客户正在寻找的漏洞，填补了客户需求，那么发现类似漏洞的研究人员将很难将其新发现销售出去，因为公司不会囤积漏洞。

如果在繁荣阶段，研究人员可以销售各种漏洞和利用程序（物联网、Web 等），那么在公司谨慎使用预算的新环境中，需求量较高的产品将优先考虑主要的攻击组件载体，即移动设备，其余需求则是定制请求。

## 媒体报道

尽管记者曝光了一条龙公司、它们的客户、运营和滥用——数字军火行业的形象类似于恐怖分子。这种负面公关将研究人员从该行业中推开，其中一些人转向其他行业。

我观察到的另一个有趣的趋势是，研究人员试图限制公司使用他们的漏洞和漏洞利用程序。这改变了研究人员与购买其输出的公司之间的权力动态。

## 可交付成果 VS. 技术能力声誉

由于对移动端漏洞的需求很高，而其他方面的需求正在下降，一些研究人员试图转向移动领域，但并没有成功，最终离开了该行业。

此外，随着厂商技术的进步，一些先前擅长发现漏洞的研究人员发现自己无法跟上不断发展的技术进展，因此离开了该行业。

还记得这个吗？

![](/images/0day-industry/23.png)

个人研究人员不再能够像过去一样取得同样的产出水平。这种情况导致他们要么与其他研究人员形成协作团队，要么选择完全退出该行业。

## 经纪人

随着一条龙公司和政府开始公开在该行业中工作，大多数经纪人失去了作为门卫的优势。例如，一条龙公司和政府参加了会议，开始直接与研究人员合作，或者相对容易地与他们联系以探索合作机会。

在我提到的繁荣阶段，该行业吸引了许多与数字军火没有或极少联系的人，他们试图找到一种适应新兴行业的方式——成为经纪人是一个相对容易的方式。

随着时间的推移，某些经纪人屈服于贪婪，并将他们成功获得的漏洞和漏洞利用程序的价格过高地标价。其他一些经纪人通过声称只出售该项目一次来误导研究人员，而实际上，他们在未告知研究人员的情况下多次出售了该项目。因此，经纪人未能适当地补偿研究人员。

此外，客户出于各种原因对漏洞或利用程序提出了异议，使经纪人不确定如何处理，在大多数情况下，经纪人接受了客户拒绝该项目。因此，研究人员的收入受到了损失。此外，经纪人说服研究人员在为其获得客户之前发送该项目，以进行演示或 PoC。

随着越来越多的研究人员直接与客户接触，经纪人发现自己拥有的漏洞和利用程序的数量很少，而这些漏洞和利用程序在广泛的经纪人中流通。这种重复的循环导致客户从多个来源接收相同的规格说明（即提供待售漏洞的信息）。因此，客户变得犹豫不决，不愿购买这些漏洞和利用程序，认为它们的寿命很短，即使实际上没有人获得这些漏洞和利用程序。

![](/images/0day-industry/29.png)

## 政府

与一条龙公司类似，当漏洞在野外被发现时，政府也受到了影响，他们对从多个来源接收的规格说明有着相同的看法，等等。

政府在萧条阶段面临的独特问题是：

    * 意识到你可以尽可能地培训你的人，但这并不会将他们变成能够在核心组件载体中发现漏洞的漏洞研究人员。
    * COVID-19（如“一条龙公司部分”中所述）。
    * 学习如何更好地利用一条龙公司，并要求更好的条款和 SLA。

## 调整（2022+）——“清算所”崛起和行业供应链的重塑

新环境的适应主要起源于供应方，重点是如何与政府和一条龙客户合作。

尽管一条龙公司的数量减少了，但它们仍在市场上互相竞争，难以保持他们的 SLA。

## 供应链

供应链不得不面对多重挑战，正如我在萧条阶段所提到的，主要问题包括：

    * 潜在客户的数量下降（即破产的一条龙公司）。
    * 一些经纪人利用了研究人员。
    * 进入漏洞研究领域的门槛更高。
    * 研究人员转移了他们的关注点，并且由于诸如法规、广泛的媒体报道以及发现漏洞的内在难度等因素而从漏洞发现领域转向了其他方向。

供应方所做的重大技术和安全进步对供应链产生了深刻的影响。发现漏洞变得越来越具有挑战性，促使多个研究人员组成协作团队，以增加他们的成功机会。根据我的经验，之前每三个月发现一个漏洞的研究人员现在需要一个团队和六个月才能取得相同的结果。

这导致了两个主要的事情：

    * 交易的控制：由于在主要组件载体（如浏览器和移动操作系统）中发现漏洞的难度越来越大，加上漏洞的寿命变短（它们在野外被检测到到或厂商代码更改），研究人员寻求更大的对与最终客户的交易的控制 - 迫使经纪人从传统的门卫角色转变为代理人角色。

	* 代理人的角色是代表卖方（即研究人员），谈判交易，最终签署合同将在研究人员和最终客户之间达成。作为对其服务的补偿，代理人获得佣金，可以是固定金额或交易价值的百分比。

    * 付费研究与开发：新成立的研究小组发现自己需要长期自我资助，才能发现可出售的漏洞。在一个已经以高风险和高回报为特征的行业中，研究团队寻求通过追求付费研究和开发项目来减轻他们的风险。在这种安排下，潜在客户将提供基本工资以及成功奖金。

	* 从客户的角度来看，他们在整个项目期间承担相对较小的资本风险。作为回报，如果研究团队成功地识别并利用了漏洞，客户将获得独家访问权。这种互惠互利的安排允许客户最小化财务风险，同时潜在地获得团队发现的成果回报。

根据我的经验，今天只有一小部分研究人员仍然是独立研究人员。大多数研究人员在研究小组或“清算所”的一部分（我将很快介绍）。

这些研究小组的规模如何？

小型研究小组通常规模不超过8名研究人员，年收入为数百万美元。

![Screenshot of an accounts statement for 2022 from one of the research groups (public information)](/images/0day-industry/30.png)

## “清算所”崛起

“清算所”是一种专注于漏洞研究的新型实体。他们的独特特点使他们能够在一条龙公司衰落的同时快速扩张。

那么，“清算所”是什么？

    * “清算所”是在行业内具有强大品牌的公司。
    * 他们拥有自己的内部研究团队，雇用高端或有能力的研究人员，专门从事漏洞研究。
    * 他们通过投资付费研发项目，确保研究人员的工作和漏洞或攻击利用的独家供应链。
    * “清算所”从市场上独家购买漏洞并改进它们以达到“生产就绪”的状态。
    * 他们与公司内部研究人员和专门为该公司工作的研究人员（即独立研究人员或研究小组）共享基础设施、漏洞、攻击利用技术等。
    * “清算所”主要通过成功奖金来补偿研究人员和他们的供应链，这些奖金显著高于市场价格，而基本工资相对较低（与一条龙公司相比）。
    * 与一条龙公司类似，“清算所”无法培训新一代研究人员。
    * 这些实体的主要客户是政府机构（在某些情况下，“清算所”也会向一条龙公司销售）。这些政府客户通常通过交易支付和付费研发项目来补偿“清算所”。经营这样的业务需要大量资本，因此，“清算所”必须与多个政府合作。
    * 与一条龙公司不同，“清算所”不需要维护 SLA，并且仅向其客户提供漏洞。在某些情况下，“清算所”将相同的漏洞销售给多个客户以支付成本。
    * 鉴于“清算所”与政府客户之间的强大关系，“清算所”优先考虑满足这些客户的特定需求，这些需求通常围绕着移动链展开。

“清算所”通过在复杂的合作网络中运作并替代市场上的一些一条龙流动性，打乱了供应链。因此，一条龙公司面临额外的挑战，因为“清算所”主要为政府提供服务，成为这些公司的主要客户。这种动态为一条龙公司的运营增加了复杂性。

从这里：

![](/images/0day-industry/31.png)

到这里：

![](/images/0day-industry/32.png)

“清算所”有一个有趣的方面是它们将能力（如漏洞或攻击链）的优先级高于最大化利润。他们认识到，向经验不足的客户出售这些能力可能会对特定客户以及其他客户产生不利影响。此外，在某些情况下，更难找到替代的能力。

因此，清算所强调负责任的使用（OPSEC）和保护他们的能力，以确保客户的长期生存能力和满意度。

那么，“清算所”的规模是多少？

“清算所”雇用相当多（我会说超过15个）的研究人员和独家供应链。它们的规模通常在数千万美元范围内。

![Screenshot of an income statement for 2022 from one of the clearing houses (public information)](/images/0day-industry/33.png)

## 行动呼吁

通过建立基础元素和审查行业中的事件和趋势，我们已经明显地意识到我们正在走向一条具有挑战性的道路。漏洞变得越来越少，而随着时间的推移，维护漏洞链的能力也越来越具有挑战性。

为了确保政府能够维持其运营并从市场上获得漏洞，政府必须采取积极措施。这涉及到政府介入并增加其专门用于付费研发项目的资金。此外，建立更紧密的关系与研究小组和清算所也非常重要。

政府与清算所之间的合作应该有所不同，不同于当前行业的运作方式：

**付费研发**：目前，政府与清算所之间最常见的关系是交易型的，只有少数付费研发项目。

这意味着清算所承担了更高的风险 - 如果清算所无法找到任何可销售的项目（如漏洞、原语、攻击等），他们仍然需要支付与他们合作的研究人员，并可能破产。

为了继续雇用最优秀的研究人员并增加发现漏洞的可能性，清算所需要提供高额的补偿，包括基本工资和成功奖励。

此外，我之前提到在当前环境下，寻找和利用漏洞需要更长的时间，需要团队的努力。

**转移SLA**：在我在2019年的BlackHat USA¹上发表的演讲中，我提到当客户从市场上购买漏洞或攻击时，该项目有一个保修期。如果漏洞在保修期内得到修补，研究人员将无法获得全部的报酬。

这意味着清算所承担了更高的风险，因为如果漏洞或攻击得到修补，他们可能会失去大部分的收入，并且可能停止未来的研究项目，因为他们没有足够的资金支持新的研究项目。

因此，政府需要支持清算所的持续资金，这不完全依赖于最终结果，并允许清算所扩大（即雇用新的研究人员）并支持长期研究项目。

需要强调的是，我们目前处于一个时间紧迫的阶段。如果政府不增加其研发资金，不承认漏洞价格的上涨，并积极参与市场（即开放式直接沟通），研究人员可能会将其关注点转向更具有经济回报的、不那么具有挑战性的领域。

## 结论

在本文中，我从不同的角度讨论了行业动态及其对其他实体的影响。对我来说，沟通主要要点、解释过去几年中发生的趋势和事件，以及导致我们今天所处的位置的多米诺效应并不容易。

我想借此机会总结本文的主要要点，以及它可能如何影响数字军火行业的未来。

    * 在市场上运营的一条龙公司数量正在减少。
    * 能够发现和利用主要组件载体中漏洞的研究人员数量有限且正在减少。
    * 由于厂商的安全改进，研究人员开始集结以实现与过去相同的产出水平。
    * 在主要组件载体中创建（即发现和利用漏洞并将它们连接在一起）并维护链条非常困难。
    * 称为清算所的新实体填补了破产的一条龙公司留下的空缺，只关注漏洞研究。此外，清算所大量投资创建了一个独家的研究人员网络和付费研发项目。
    * 清算所将能力（如漏洞或攻击链）的优先级高于最大化利润，并倾向于与具有强大 OPSEC 操作的客户打交道。
    * 经纪人不得不从门卫转变为代理，因为研究人员要求对漏洞和与最终客户的交易进行控制。

在我看来，我列出的主要潜在受害者，未来可能是政府。尽管清算所是行业的重要稳定器，但它们无法完全取代需要投资于行业以保证政府未来运营需求所需的资金和资源。

## Footnotes

¹https://www.youtube.com/watch?v=JkQxS1l9IPI&ab_channel=BlackHat

²https://support.apple.com/guide/security/operating-system-integrity-sec8b776536b/web

³https://blog.google/technology/safety-security/new-initiatives-to-reduce-the-risk-of-vulnerabilities-and-protect-researchers/

⁴https://medium.com/@maor_s/update-about-the-0-day-industry-8d8bb49e8dbb

⁵https://www.wassenaar.org/app/uploads/2019/12/Stand-alone-Munitions-List-2019.pdf

⁵https://www.wassenaar.org/app/uploads/2019/consolidated/List-of-Dual-Use-Goods-and-Technologies-and-Munitions-List-Corr.pdf

⁶https://www.reuters.com/investigates/special-report/usa-spying-raven/

⁶https://www.ft.com/content/11cb394d-a13e-4826-b580-823b9367fedb

⁷https://home.treasury.gov/news/press-releases/jy1296

⁷https://www.commerce.gov/news/press-releases/2021/11/commerce-adds-nso-group-and-other-foreign-companies-entity-list

⁷https://www.haaretz.com/israel-news/security-aviation/2023-01-16/ty-article/.premium/greek-authorities-fine-intellexa-chief-over-spyware-scandal/00000185-bab3-deab-ad97-fafbd8ae0000

⁷https://www.dpa.gr/sites/default/files/2023-01/2_2023%20anonym.pdf

⁷https://amp.dw.com/en/german-prosecutors-investigate-spyware-maker-finfisher/a-50293812

⁷https://www.al-monitor.com/originals/2022/02/israel-freezes-spyware-exports

⁷https://www.timesofisrael.com/defense-ministry-said-to-freeze-export-licenses-for-israeli-cyberattack-tech/

⁷https://www.apple.com/newsroom/pdfs/Apple_v_NSO_Complaint_112321.pdf

⁷https://www.theregister.com/2023/03/21/meta_employee_spyware/

⁷https://www.haaretz.com/israel-news/security-aviation/2023-03-08/ty-article/.premium/israel-firm-nfv-systems-illegally-selling-classified-spy-tech/00000186-bceb-d2e9-a7df-bdef014c0000

⁷https://www.whitehouse.gov/briefing-room/presidential-actions/2023/03/27/executive-order-on-prohibition-on-use-by-the-united-states-government-of-commercial-spyware-that-poses-risks-to-national-security/

⁷https://www.dw.com/en/germany-charges-executives-for-selling-spyware-to-turkey/a-65701848

⁷https://www.ft.com/content/11cb394d-a13e-4826-b580-823b9367fedb

⁷https://www.timesofisrael.com/report-israel-nixed-quadreams-spyware-deal-with-morocco-leading-to-firms-closure/

⁷https://www.reuters.com/technology/facebook-can-pursue-malware-lawsuit-against-israels-nso-group-us-appeals-court-2021-11-08/

⁷https://www.wassenaar.org/app/uploads/2019/12/WA-DOC-19-PUB-002-Public-Docs-Vol-II-2019-List-of-DU-Goods-and-Technologies-and-Munitions-List-Dec-19.pdf

⁸https://english.almayadeen.net/news/technology/pegasus-nemesis:-meet-quadream-another-israeli-spyware-compa

⁸https://www.reuters.com/technology/exclusive-iphone-flaw-exploited-by-second-israeli-spy-firm-sources-2022-02-03/

⁸https://www.amnesty.org/en/latest/press-release/2021/07/the-pegasus-project/

⁸https://www.businesstimes.com.sg/startups-tech/technology/spyware-trade-grows-amid-claims-activists-amazon-boss-targeted

⁸https://www.forbes.com/sites/thomasbrewster/2016/09/29/wintego-whatsapp-encryption-surveillance-exploits/?sh=bb4d0581aa95

⁹https://www.latimes.com/business/technology/story/2020-01-27/spyware-booming-business-jeff-bezos

¹⁰https://www.theguardian.com/technology/2023/apr/11/canadian-security-experts-warn-over-spyware-threat-to-rival-pegasus-citizen-lab

¹¹https://news.sina.com.cn/c/2023-04-27/doc-imyruepi4556974.shtml?cre=tianyi&tr=181#/

¹¹https://cn.chinadaily.com.cn/a/202304/27/WS6449c3aaa310537989371d7c.html

¹²https://exportctrl.mod.gov.il/About/Pages/AllMessages.aspx?ItemId=242

¹³https://metacpc.org/wp-content/uploads/2022/12/predator.pdf

¹⁴https://www.vice.com/en/article/n7wbnd/hacking-team-is-dead

¹⁴https://www.bloomberg.com/news/articles/2022-03-28/spyware-vendor-finfisher-claims-insolvency-amid-investigation

¹⁴https://www.ecchr.eu/en/case/surveillance-software-germany-turkey-finfisher/

¹⁴https://www.forbes.com/sites/thomasbrewster/2021/07/29/paragon-is-an-nso-competitor-and-an-american-funded-israeli-surveillance-startup-that-hacks-encrypted-apps-like-whatsapp-and-signal/?sh=222a3c8f153b

¹⁴https://intelligencecommunitynews.com/l3-completes-acquisition-of-azimuth-security-and-linchpin-labs/

¹⁵https://www.moodys.com/research/Moodys-downgrades-NSO-to-B3-with-negative-outlook--PR_446947

¹⁵https://www.aljazeera.com/economy/2021/12/14/nso-group-explores-shut-down-of-its-pegasus-spyware-unit-sale

¹⁵https://www.bloomberg.com/news/articles/2022-11-04/israel-s-nso-takes-drastic-measures-to-survive-spyware-scandal?leadSource=uverify%20wall

¹⁶https://www.washingtonpost.com/national-security/2022/07/10/nso-spyware-l3harris-talks-ended/

¹⁶https://seekingalpha.com/news/3855689-l3harris-reportedly-drops-bid-for-israeli-spyware-following-us-concerns

¹⁶https://www.technologyreview.com/2022/06/27/1054884/the-hacking-industry-faces-the-end-of-an-era/

¹⁷https://www.haaretz.com/israel-news/security-aviation/2023-04-16/ty-article/.premium/offensive-israeli-cyber-firm-quadream-closes-and-fires-all-employees/00000187-8b5c-d484-adef-ebdc048c0000

¹⁷https://www.calcalist.co.il/calcalistech/article/rjdbgg3fn

¹⁷https://www.timesofisrael.com/report-israel-nixed-quadreams-spyware-deal-with-morocco-leading-to-firms-closure/

¹⁸First Updates to the New Theoretical Framework of Technology Start-up Lifecycle Stages by Jakub Ulč, Miroslav Mandel

¹⁹https://www.immunityinc.com/downloads/skylar_cansecwest09.pdf