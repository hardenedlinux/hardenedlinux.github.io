---
layout: post
author: David A. Wheeler
title:  "Fully Countering Trusting Trust through Diverse Double-Compiling (DDC) - Countering Trojan Horse attacks on Compilers"
summary: David A. Wheeler 的 DDC 论文的中译本
categories: system-security
---

# David A. Wheeler's Page on Fully Countering Trusting Trust through Diverse Double-Compiling (DDC) - Countering Trojan Horse attacks on Compilers

作者：David A. Wheeler

译者：Yingnan Ju, Yue Chen

翻译版Reviewer：Shawn the R0ck

## 开篇

这里是关于我对反击“Trusting Trust”攻击相关工作的信息。“Trusting Trust”攻击是一种恶毒的攻击方法，到现在为止都被假设为无法进行有效的防御。自从 Ken Thompson 公开的对其进行阐述后，我一直对此问题非常忧心。一种已知但却无法有效防御的攻击存在，我们还应该继续使用计算机吗？欣慰的是，我认为有一种被我命名为“Diverse Double-Compiling (DDC)”的有效反制方法。

## 2009 博士论文

![](/images/tt_image1.jpg)

[***Fully Countering Trusting Trust through Diverse Double-Compiling***](https://www.dwheeler.com/trusting-trust/dissertation/html/wheeler-trusting-trust-ddc.html)
([*PDF version*](https://www.dwheeler.com/trusting-trust/dissertation/wheeler-trusting-trust-ddc.pdf), [*HTML version*](https://www.dwheeler.com/trusting-trust/dissertation/html/wheeler-trusting-trust-ddc.html), [*OpenDocument text version*](https://www.dwheeler.com/trusting-trust/dissertation/wheeler-trusting-trust-ddc.odt)) 是我2009年关于如何通过\"Diverse Double-Compiling\"(DDC)对抗\"Trusting trust\"攻击的博士论文。这篇论文被我的博士委员会接受日期为2009年10月26日。

[*这个视频是我的正式的公开答辩*](https://www.dwheeler.com/trusting-trust/dissertation/wheeler-trusting-trust-video.html)(webm或者mp4)，这个演讲是2009年11月23日下午1点到3点进行的([*podcase/RSS*](https://www.dwheeler.com/trusting-trust/countering-trusting-trust.rss))。演讲的材料同样以[*PDF*](https://www.dwheeler.com/trusting-trust/dissertation/fully-countering-trusting-trust-ddc-presentation.pdf)和[*OpenDocument (ODP)*](https://www.dwheeler.com/trusting-trust/dissertation/fully-countering-trusting-trust-ddc-presentation.odp)格式提供下载。公开答辩的地点是[*George Mason University*](https://www.gmu.edu), Fairfax, Virginia,[*Innovation Hall*](https://itu.gmu.edu/innovationhall/aboutih.html), room 105
[*location on campus*](http://itu.gmu.edu/innovationhall/images/academiciv.gif),[*Google map*](http://maps.google.com/maps?f=q&source=s_q&hl=en&geocode=&q=38.828240,+-77.307578&sll=38.828258,-77.307615&sspn=0.011417,0.01929&ie=UTF8&ll=38.827539,-77.307336&spn=0.011417,0.01929&z=16)。

*这里是论文的摘要：*

*Ken Thompson的图灵奖演讲“[Reflections on Trusting Trust](http://delivery.acm.org/10.1145/360000/358210/reflections.pdf?ip=170.178.162.104&id=358210&acc=OPEN&key=4D4702B0C3E38B35.4D4702B0C3E38B35.4D4702B0C3E38B35.6D218144511F3437&CFID=727534328&CFTOKEN=23851549&__acm__=1446695913_4dd9ed7d68cfada7dbae845681d8baf3)”是基于对空军使用的Multics系统的评估，这个演讲展示了编译器可以被利用向重要的软件甚至编译器自己植入恶意木马。如果“trusting trust”攻击无法被检测，那代码审计也不能找到恶意代码。之前已知的反制方法都是很低效的。如果这类攻击不能被反制，攻击者可以悄悄的搞定一堆计算机系统，获得金融机构，基础设施，军队或者商业系统的全部权限。这篇论文的题目是 trusting trust 攻击可以被检测并且使“"Diverse Double-Compiling (DDC)”技术是有效的反制手段，如展示，
(1)形式化证明DDC可以判断源代码和生成的可执行代码的对应，
(2)演示了DDC使用4个编译器（一个轻量级C编译器，一个轻量级Lisp编译器，一个轻量级的恶意版Lisp编译器和工业级C编译器GCC），
(3)描述了应用DDC在真实世界的场景。DDC会要求源代码被两次编译：编译器本身的源代码先被一个可信的编译器编译，之后用第一个编译结果去编译需要测试的编译器的源代码。如果DDC结果是与原生的编译器可执行程序bit-for-bit一致，那说明这个需要测试的编译器的可执行程序完全对应相关的公认代码。*

论文有一部分章节解释了DDC是如何根据[*我以前2005年的ACSAC论文*](https://www.dwheeler.com/trusting-trust/#acsac)进行扩展的。这篇论文概括了之前的ACSAC论文（现在编译器不需要再自我编译了），包括其形式化的证明部分，论文还包含了用GCC编译器（以展示其扩展性）和恶意编译器的演示部分。

如果你读过论文，那么你也应该看看[*论文勘误表*](https://www.dwheeler.com/trusting-trust/dissertation-errata.html)（勘误表并不影响论文中任何的论证基础）。

我要感谢论文委员会的成员，他们对我帮助颇多。我要特别感谢Ravi Sandhu博士；我想要做的博士论文是完全不走寻常路的，但是他十分开明，让我放手去做。在这个过程中他还提了许多很棒的建议。Daniel A. Menascé博士让我演示使用恶意编译器的方法（我实现了）。Jeff Offutt博士向我提问了DDC和N-版本程序设计的关系（所以我加入了关于它和N-版本编程的区别的材料）。Paul Ammann博士对N-版本编程材料提了一些有趣的意见；我才知道原来他个人当时曾参与了这一里程碑式的研究！Yutao Zhong博士问过我关于T-图的问题（所以我加进了为什么*没有*使用T-图的材料）。委员会的每个人都提出了很棒的问题，尤其是在公开答辩前的小范围演示中的提问；感谢你们！

## 2005年ACSAC论文

这是我2005年的论文，由ACSAC正式审核并出版：[*Countering Trusting Trust through Diverse Double-Compiling*](http://www.acsa-admin.org/2005/abstracts/47.html)
(DDC), David A. Wheeler, *Proceedings of the Twenty-First [*Annual Computer Security ApplicationsConference*](http://www.acsa-admin.org/)* (ACSAC)*, December 5-9, 2005, Tucson, Arizona, pp. 28-40, Los Alamitos: IEEE Computer Society. ISBN 0-7695-2461-3, ISSN 1063-9527, IEEE Computer Society Order Number P2461。如果你无法从ACSAC获取论文，那么可以从本地下载：[*Countering Trusting Trust through Diverse Double-Compiling (DDC)*](https://www.dwheeler.com/trusting-trust/wheelerd-trust.pdf)。你也可以下载PDF版[*"Countering Trusting Trust through Diverse Double-Compiling (DDC)"*](https://www.dwheeler.com/trusting-trust/acsac-countering-trusting-trust-20050922-alt.pdf) 和ODT格式的[*"Countering Trusting Trust through Diverse Double-Compiling (DDC)"*](https://www.dwheeler.com/trusting-trust/acsac-countering-trusting-trust-20050922.odt)。（[*我可以在此发布*](https://www.dwheeler.com/trusting-trust/acsac2005-can-post.txt)）

很荣幸ACSAC2005年会议接受了我的论文。他们每年都收到很多优秀的论文提交，但是在2005年他们毙掉了他们收到的77%的论文提交。我之所以向ACSAC提交我的论文的一个原因，是我相信在网络上的发布对于论文的广泛传播是非常关键的；ACSAC一直致力于网络发布，并已成为[*开放式访问*](http://www.earlham.edu/%7Epeters/writing/jbiol.htm)的会议。

ACSAC论文和新版论文之间的符号对应表:

+---------------------+------------------+-----------------+
|                     | **ACSAC (2005)** | **论文 (2009)** |
+---------------------+------------------+-----------------+
| Trusted compiler    | T                | c~T~            |
|                     |                  |                 |
| 可信编译器          |                  |                 |
+---------------------+------------------+-----------------+
| Compiler under test | A                | c~A~            |
|                     |                  |                 |
| 被测编译器          |                  |                 |
+---------------------+------------------+-----------------+
| Parent compiler     | \-               | c~P~            |
|                     |                  |                 |
| 原生编译器          |                  |                 |
+---------------------+------------------+-----------------+

我有基于ACSAC版论文的一个展示。我最初是在ACSAC会议上做的原始版本的展示；后来每得到一点不同的反馈，我就根据反馈做出相应的更新。

演示可下载的版本包括：

-   *PDF格式*
-   [*ODT格式*](https://www.dwheeler.com/trusting-trust/counter-trusting-trust-presentation-20060228.odp)
    \--
    这是演示交流的国际标准格式。如果你打不开这个格式，最好的办法是下载[*LibreOffice*](http://www.libreoffice.org/)或[*OpenOffice.org*](http://www.openoffice.org)，这些都是免费的开放的文档处理软件，支持[*ODT格式*](http://opendocumentfellowship.org/Main/HomePage)。当然也可以使用其他支持ODT格式的软件。

注：ACSAC2005年论文"Countering Trusting Trust through Diverse
Double-Compiling" 有一个错字。在第4节的最后一段，就在图前面，写道："if
c(sA,c(sA,T)), A, and c(sA,T) are identical,
\..."。"c(sA,T)"应该是"c(sA,A)"；你也可以根据图中的内容来确定这个错误，图中清楚的显示"c(sA,A)"而不是"c(sA,T)"。感谢Ulf
Dittmer向我指出这一点。

## 关于作品引用（拜托，我叫 David A. Wheeler）

如果你要引用我的作品，请在写我的名字的时候，一定保留我的中间名，*至少*保留一个缩写A，当然全拼"David A. Wheeler"更好。请不要在任何书面的作品中（包括互联网上的电子作品），在引用我的名字时拼写为"David Wheeler"或"D. Wheeler"。有太多叫David Wheelers的人，所以这样的拼写根本分不出到底是谁。如果强制使用缩写，请至少使用"D. A. Wheeler"。但是当然我还是最希望你们按照我说的方法引用。通常，人们都是按照被引用者的习惯来引用，而不是把它改成其他人的名字来引用。谢谢。

## 复现试验所需的详细数据

我相信科学实验一定是可以复现的。不过很可惜，许多号称计算科学的玩意儿并不是那么回事，因为其结果根本不可能复现。这是公开的秘密，[*Victoria C. Stodden的论文*](http://academiccommons.columbia.edu/catalog/ac:140124)[*Reproducible Research: Addressing the Need for Data and Code Sharing in Computational Science" (Computing in Science & Engineering, 2010)*](http://academiccommons.columbia.edu/catalog/ac:140124)就曾经讨论过这个问题。不过，我*一定会*提供必要的信息来复现我的结果。对于ACSAC论文来说，请参考我的[*Tiny C Compiler (tcc)*](https://www.dwheeler.com/trusting-trust/tcc.html)页面，这样就能复现ACSAC实验和其他和tcc相关的实验。在我的PhD论文中，请参考[*关于详细数据的单独页面*](https://www.dwheeler.com/trusting-trust/dissertation)。它们都提供了足够的信息来复现实验，并有助于您对实验进行进一步研究。

## 消除错误概念

有些错误概念根深蒂固，所以我（也）要在此做出澄清。

**DDC方法不假定在输入相同的情况下，两个完全不同的编译器会生成相同的二进制输出。**

我在ACSAC论文中说过这个问题，PhD论文中也讲过，但是仍然有人不理解，所以我再讲一次。

ACSAC论文和PhD论文都**不**认为不同的编译器会生成相同的结果。实际上，二者都明确指出不同的编译器通常编译出不同的结果。而事实上论文提到了一点，被信任的编译器生成与被试的编译器不同架构的代码其实是一种进步。很明显，如果两个编译器为不同的CPU生成代码，那么两个二进制输出通常*不会*总是一样。

这种方法*一定*要求可信编译器使能够编译被测编译器的原生编译器的源代码（这种语言）。你总不能使用Java编译器来直接编译C的源代码吧。

至学究：是的，有时可以写出在完全不同CPU的架构上的机器码。甚至也可以先设计代码来决定他在什么架构上运行，然后再跳转到该架构的"正确"代码上。这样可以利用不同机器码的精确值，而且这肯定是聪明的骇客手段。但如果你想这样做，包含有多段（每个架构一段）的[*胖二进制*](http://en.wikipedia.org/wiki/Fat_binary)是更好的办法------这是一种更利索的办法。不管怎么样，这不是重点，重点是不要求被测编译器和可信编译器生成相同的输出代码。

**非确定性硬件对DDC来说不是问题**

当原生编译器编译被测编译器的时候，DDC确实要求原生编译器是确定性的。这和认为两个不同编译器一定生成相同结果*不是*一回事。确定性编译器是指当两次输入相同（选项标志等都相同）时，输出相同。如果（随机生成数）种子确定，你甚至可以使用随机数生成器（例如gcc可以使用命令行选项来设定种子）。例如，在Unix/Linux系统中，可以使用以下命令行来确定这一点：

 \$ mycompiler input.c \# 编译，结果保存在\"a.out\"中\
\$ mv a.out a.out.saved \# 保存第一次的结果\
\$ mycompiler input.c \# 再来一次\
\$ cmp a.out a.out.saved \# 如果结果相同，则是确定性的\

这是一个相对简单的约束，也是一个大多数编译器作者所希望的编译器属性（因为非确定性编译器很难调试）。编译器通常是确定性的，除了嵌入时间戳时的例外------因此我在论文中讨论了如何处理嵌入时间戳的问题。有时候可能需要使用标志位（例如，在GCC C++编译器中设置随机数生成器种子）。

原生编译器可能在内部使用某些非确定性的结构体（例如非确定性调度线程），但是如果需要这样做的时候，必须使用某种机制来确保每次相同的输入都一定导致相同的输出。如今底层CPU拥有各种非确定性属性（例如，多内核线程，或非时变性）；"现代CPU本身是随机的，上层复杂的通用OS大幅地放大了这种内在的随机性"\[[*"Analysis of inherent randomness of the Linux kernel"*](http://lwn.net/images/conf/rtlws11/random-hardware.pdf) 作者Nicholas Mc Guire，Peter Okech，和Georg Schiesser\]。但如果CPU非确定性这么强，那么写入数据时，就不太可能可靠地根据某一特定顺序写入，因此也就不能运行编译器或其他程序了。所以原生编译器仅仅需要一种方式，来确保在数据写入时非确定性效应不影响输出结果。例如，原生编译器可以使用锁，来保证不同的线程调度不会导致不同的结果。在实践中，开发者一般也都倾向于这么做。

可信编译器（在ACSAC论文中用"compiler T"表示；在PhD论文中用"compiler cT"表示）不需要是确定性的。

如果需要更多细节，请参考论文中sP\_portable\_and\_deterministic假设的部分。

## DDC使用可信编译器从根本上提高了可信度

过去的一些方法使用了第二编译器，但是他们基本上只是选择一个不得不完全信任的编译器。但事实上，万一信任的是包含恶意代码的编译器时，结果只会变得更糟。

与此相反的是，DDC一开始就使用了额外的编译器来做*检验*。这从根本上改变了这一情况，因为现在攻击者必须同时破坏原始编译器和DDC中使用的*所有*编译器。破坏多个编译器比仅仅破坏一个要难得多，尤其是在使用DDC的情况下。安全人员不但可以选择DDC使用哪些编译器，也可以选择在遭受攻击后DDC使用哪些编译器。

## 为什么不在所有地方都使用可信编译器

使用不同的可信编译器极大地增加了编译器可执行文件和对应的源代码的置信度。当DDC使用第二编译器的时候，攻击者必须攻击多个可执行文件和可执行文件进程才能实现无法被检测到的"trusting trust"攻击。如果只使用可信编译器，那就又回到最开始的状况了，在我看来，没有一个可行的验证过程来验证对单一编译器可执行文件的完全信任。

此外，正如[*章节4.6*](https://www.dwheeler.com/trusting-trust/dissertation/html/wheeler-trusting-trust-ddc.html#4.6.Why%20not%20always%20use%20the%20trusted%20compiler)所述，有很多原因导致可信编译器可能不适合通用用途，这些原因包括运行缓慢，生成效率低下的代码，生成非所需CPU架构的代码，成本高，或产生人们不希望产生的的软件许可限制等等。它对于通用用途来说，可信编译器可能缺少很多必要的有用的功能。而在DDC中，可信编译器只需要能编译原生编译器即可，没有必要提供其他功能。

最后，请注意"可信"编译器可能是恶意软件，但仍可在DDC中正常运行。我们需要正常看待此事，可信编译器中的任何触发器或负载在被应用到被测编译器时，并不会影响DDC过程。这非常非常容易证明。

## "fully"的含义

当我说到"fully"的时候，我的意思是可以检测到"trusting trust攻击并有效反击"（正如我在论文中所述）。了解一些背景知识会有助于理解我口中的"fully"。

首先，如果抱怨人与人的信任，那这是浪费时间。在现代社会中人们*必须*互相信任。没人完全自给自足，所以我们必须互相信任。但是如果你不能独立独立验证你所信任的人，那就会出现严重的系统问题。你要努力"*信任，但是（也要）验证*"。

我相信，trusting trust攻击造成的根本问题是，对于你所依赖的东西（执行文件）所对应于人类可识别的形式（源代码）的独立验证是不切实际的。这是因为程序处理的程序可以把通过人类评审的代码在最终实际的使用中替换为其他代码。因此Ken Thompson的论文不叫"Reflections on trust"，他叫"Reflections on trusting trust"。同样，我相信问题不在于信任，而在于缺乏有效的独立验证的过程。

通过DDC，我们现在对于*独立验证*源代码和对应的可执行文件有了可行的过程。DDC*完全*解决了对于程序处理的程序（例如编译器）*缺乏有效的*的独立验证过程的问题。

我相信，我们有必要理解，任何结果都是有局限性的。[*章节8.14*](https://www.dwheeler.com/trusting-trust/dissertation/html/wheeler-trusting-trust-ddc.html#8.14.How%20can%20an%20attacker%20counter%20DDC)详细解释了，攻击者如何破坏DDC的方法。因为已经通过形式化的数学验证来证明，*唯一*突破DDC的办法是伪造证明假设。防御者做出这样的伪造都是*十分*困难的。举例来说，防御者，不是攻击者，需要选择作为可信编译器的编译器；防御者甚至可以自己编写编译器。虽然不那么明智的防御者可以使用差不多的组件，但是章节6描述了如何获取不同组件的方法。一旦防御者了解了他们应该获取不同组件，那防御者就能了解提供不同组建的多种方法。

我的目标是创建独立验证的过程。DDC提供了独立的并且可行的验证过程。我在四个不同的编译器可执行文件中应用了DDC过程，其中包括广泛使用的gcc。因此，DDC充分满足了独立的并且可行的验证过程的需求。

那么，为什么我在论文标题要使用*fully*这个词呢？好吧，我需要某种方法来区份ACSAC论文和PhD论文的标题。我意识到我旧的ACSAC论文有一个重要的限制：它仅适用于自我编译的编译器。许多编译器不是自我编译的，因此旧的ACSAC论文对现在所使用的许多编译器并不适用。相反，2009年的论文可以适用于所有编译器，不管他是不是自我编译的。因此，改论文"fully"提供了验证编译器可执行文件的过程，而不在乎编译器是否是自我编译的。我需要指出的是，论文标题已经定下来了，我想改也改不了了。

**应用到硬件上怎么样？**

在ACSAC会议上和论文中，我提到过把DDC方法应用到硬件上。很显然，如果软件完好，而硬件遭到攻击被破坏，那么最终你还是难以避免损失。ACSAC演讲和论文都更详细地提到了这一点。DDC也可以被应用到硬件上。正如我所说的，有两方面的问题：法律和技术。

法律问题是，越来越多的芯片设计方和制造商不能合法地获取芯片上*应有的*信息。例如，芯片上使用的各种"IP核"的开发者禁止芯片设计方和制造商获取和使用这些信息。

关键的技术问题是硬件上有意义的"相等"测试。我推测各种技术，例如扫描电子显微镜，可以用来帮助实现相等性测试。其他硬件验证机制（例如，参考[*Semiconductor IP Validation Gets Faster*](http://www.semiwiki.com/forum/content/3221-semiconductor-ip-validation-gets-faster.html)）也可以起到一定的作用。但是它从根本上很难实现硬件相等性测试（与软件相比）。我在论文中引用了好几篇关于这个的文章。你可以从那之后发表的论文中看到更多的质疑，例如[*"Stealthy Dopant-Level Hardware Trojans" 作者 Georg T. Becker, Francesco Regazzoni, Christof Paar, 和 Wayne P. Burleson*](http://people.umass.edu/gbecker/BeckerChes13.pdf) ([*Bruce Schneier*](https://www.schneier.com/blog/archives/2013/09/surreptitiously.html)简要讨论了这一点)，还有[*"Integrated Circuit Security Threats and Hardware Assurance Countermeasures" Karen Mercedes Goertzel (CrossTalk, November/December 2013)*](http://www.crosstalkonline.org/storage/issue-archives/2013/201311/201311-Goertzel.pdf) \[[*alternate URL*](http://www.academia.edu/5182829/Integrated_Circuit_Security_Threats_and_Hardware_Assurance_Countermeasures)\]。如何对抗被攻击破坏的硬件绝对是未来很有潜力的研究领域。

## 软件专利和API版权

只有在你可以有多种方法来实现某种计算机语言（编译器）时，这里描述的方法才适用。这样做没有任何技术问题，但是有些组织试图使人们难以合法地使用多种方法来实现（某种计算机语言）。任何这些对于开发和分发某种计算机语言多种实现方法的限制，其实都是对这种语言用户的极为危险的隐患。对于那些使用这种语言编写的程序的用户来说，这同样是隐患。

人们一般认为API和语言是在版权保护范围之内的。但其实著作权只保护具体的软件实现和文档，而API和语言本身只是基础思想，并非某种固定的表达。人们早就这样理解，但是2012年的许多（美国和欧洲的）裁决让这一看法更加明确......当然事情并不那么简单，威胁无处不在。2012年甲骨文和谷歌之间的"[*Order RE Copyrightability of Certain Replicated Elements of the Java Application programming Interface*](http://www.groklaw.net/pdf3/OraGoogle-1202.pdf)"案件表明，"只要实现软件的具体代码不相同，那么任何人都可以在版权法的保护下自由地编写他（她）们自己的代码，以实现JAVA
API中的任何方法的功能或规范。方法名或声明是否相同并不重要，因为在Java的规则中，必须用相同的方法名来声明实现某种功能的方法，甚至实现方法不同的时候亦如此。当只有一种方法来表达某种想法或实现某种功能的时候，所有人都有权这么做，没有人能够垄断这种功能的实现。而且，虽然Android的方法和类的名称和Java中对应的方法和类的名称可能是不同的，并且他们都能正常实现，但是版权保护从未把他们的法律触手伸向名称或短语这个级别......这一命令结构是根据版权法的章节102(b)的系统或操作方法，因此并不受版权保护。"（[*博客网站Groklaw在网站上也这么写道。*](http://www.groklaw.net/article.php?story=20120531173633275)）同样，欧盟司法法院也在[*SAS研究院和World Programming Ltd.,的案例C-406/10中发现"计算机程序的功能和编程语言不受版权保护。"*](http://curia.europa.eu/jcms/upload/docs/application/pdf/2012-05/cp120053en.pdf)（[*这里有C-406/10案例的真实判决*](http://curia.europa.eu/juris/documents.jsf?num=C-406/10)）。美国法律下的版权，明确不包括任何的"想法、流程、过程、系统、操作方法、概念、原理或发现"；这么做（注意，这个清单中远不止想法一个点）的历史和原因在[*Pamela Samuelson的"Why Copyright Law Excludes Systems and Processes from the Scope of Its Protection"*](http://people.ischool.berkeley.edu/%7Epam/papers/102_b_%20dr4.pdf)中可以找到。但是在2014年5月9日，美国联邦巡回法院部分驳回了地方法院的裁决，给出了在版权问题中有利于甲骨文的判决，并将是否合理使用（软件）的问题发回地方法院重审。我希望这只是狭义的解释，如果这是广义解释的话，那么版权可能会扼杀将来所有的竞争。软件必须相互兼容相互通信，这是一个很实际的问题；如果某个公司可以禁止软件的这种兼容的实现方式，那么这个公司就足以扼杀所有良性竞争，以及像DDC这类的验证手段。可以参考[*Computer Scientists Ask Supreme Court to Rule APIs Can't Be Copyrighted*](https://www.eff.org/press/releases/computer-scientists-ask-supreme-court-rule-apis-cant-be-copyrighted)获取更多关于API和版权的信息。

不幸的是，正如本文中讨论的，来自专利方面的风险依然如此之大。更多信息，请参考[*我写的关于软件专利的那些内容*](https://www.dwheeler.com/essays/software-patents.html)。

## 关于论文的其他想法

当我说“trusted compiler（可信编译器）”的时候我用了“trusted”这个词。我要指出，“trusted”和“trustworthy”这两个词的区别还是很大的。“trustworthy”表明有证据证明他是值得信任的；而“trusted”表明个人的信任（希望是因为他们确定他是“trustworthy”的）。如果你使用DDC，你需要使用可信编译器------因为你得相信它的编译输出，顾名思义来看它是“trusted”（可信的）。但是你*应该*选择值得信赖的编译器作为可信编译器。

好消息是，你并不用去找一个完美无瑕永远不会犯错的编译器，这样的编译器简直凤毛麟角。相反，你只需要使用符合论文中描述的条件的编译器即可，这容易找得多。

我在的小论文[*"How to prove stuff automatically"*](https://www.dwheeler.com/formal_methods/how-to-prove-stuff.html)里，总结了一些关于使用工具来做证明的方法的经验教训。

我的论文发表后，在[*Mike Stute的关于\"What is a coder\'s worst nightmare?\"的回答*](https://www.quora.com/What-is-a-coders-worst-nightmare/answer/Mick-Stute?srid=tQ46&share=1)中，我见到了另一种被破坏的编译器。他尝试修改一个程序，但是并没能如愿。努力了15天以后，“我突然意识到我已经修改到了编译器内部......每一次编译原始的代码都会把一段潜藏的代码加到源代码中......几天后......我们重新从源代码编译编译器。问题解决了......除了没能......这个曾经的研究生成功地让编译器中了毒，每次编译器重新编译的时候都把病毒带到下一代中去......我们还发现如果编译/sbin/login目录，则会在其中加入后门，使任何人都可以使用某个特定密码作为root用户登录。这样就可以通过调制解调器和Tymnet远程登录。最终这引起了计算中心的关注。天才！不过动因十分可怕。”

## 论功行赏

正如我在论文中明确指出的，关于DDC这一手段的最原始想法并不是我想到的。最初的想法是由天才的Henry Spencer提出的。但是他并没有去实现这一想法。事实上随着时间的推移他可能都忘记了这一设想。我看到了他关于这一想法表述的那几句话，并随后极大地进行了扩展，包括更详细的分析说明，证明和演示。例如，他最初的想法假定自我编译，而我的PhD论文中去除了这一限制。非常感谢他和他最初的想法，也感谢他一直以来提出的宝贵建议。

我还要在功劳簿上为那些把这一问题第一时间通知全世界的人们记上一笔：Paul Karger，Roger Schell和Ken Thompson。Paul Karger和Roger Schell 关于 Multics 的开创性分析使人们第一次认识到这个问题。解决问题的关键步骤是要先确定有这么个问题。我和Paul Karger深入讨论了几次，他对这一项目非常热情，提供了很多有用的建议。不幸的是，[*Paul Karger去世于2010年*](http://www.ieee-security.org/Cipher/Newsbriefs/2010/karger.html)，这是整个世界的一大损失；不过万幸的是，我在他在去世前把这一成果告诉了他，他对此十分欣慰。我也和 Roger Schell 聊过这一问题。我也要感谢 Ken Thompson（这是他和团队的共同成就），他演示了这一攻击过程，使得更多人意识到了这个问题。

## 关注者都有谁

第一份将我的2005年ACSAC论文作为必读材料的课程是[*CSC 593: Secure Software Engineering Seminar*](http://www.nku.edu/%7Ewaldenj1/classes/2006/spring/csc593/schedule.html)，由北肯塔基大学的James博士在2006年的春季班中开设。同样作为必读材料的还有Ken Thompson的1984年经典论文[*Reflections on Trusting Trust*](http://www.acm.org/classics/sep95/)。乔治·梅森大学（GMU）也有类似主题的课程："Advanced Topics in Computer Security: Cyber-Identity, Authority and Trust" (IT962)，由Ravi Sandhu开设。我有幸获得在2006年春季课程参观一天的机会，并做了演讲。[*多特蒙德工业技术大学的Lehrstuhl Informatik VI课程（Dr. Ulrich Flegel 和Dr. Michael Meier开设)(WS 2007/2008)*](http://ls6-www.informatik.uni-dortmund.de/uploads/tx_ls6ext/resi05_01.pdf)也引用了我的论文。[*Linux Luddites podcast \#21 (August 2,2014)
的从1:41开始*](http://linuxluddites.com/mp3/podcast-21/)也特别提到了我的论文。

ACSAC论文被多处引用，包括美国海军研究生院（NPS）[*Steven Anthony Schearer的论文"Increasing Open Source Software Integration on the Department of Defense Unclassified Desktop"*](http://edocs.nps.edu/npspubs/scholarly/theses/2008/Jun/08Jun_Schearer.pdf)，里斯本大学信息系的[*Obelheiro等人(Sep 2006)*](http://docs.di.fc.ul.pt/jspui/handle/10455/2992)的论文[*"How Practical Are Intrusion-Tolerant Distributed Systems?"*](http://docs.di.fc.ul.pt/jspui/handle/10455/2992)，以及[*邦德大学信息技术学院Alexander Zangerl的PhD论文"Tamper-resistant Peer-to-Peer Storage for File Integrity Checking"*](http://epublications.bond.edu.au/context/theses/article/1047/index/1/type/native/viewcontent/)。

ACSAC论文也在很多地方被提到或讨论到，包括[*Bugtraq*](http://seclists.org/lists/bugtraq/2005/Dec/0156.html),
[*comp.risks (the Risks digest)*](http://catless.ncl.ac.uk/Risks/24.13.html#subj12),
[*Bruce Schneier's weblog (the source for Crypto-Gram)*](http://www.schneier.com/blog/archives/2006/01/countering_trus.html),
[*Lambda the ultimate*](http://lambda-the-ultimate.org/node/view/1184),
[*SC-L (the Secure Coding mailing list)*](http://securecoding.org/pipermail/sc-l/2005/000029.html),
[*LinuxSecurity.com*](http://www.linuxsecurity.com/content/view/120991/65/),
[*Chi Publishing's Information Security Bulletin*](http://www.chi-publishing.com/index.php?newsID=574),
[*Wikipedia's "Backdoor" article*](http://en.wikipedia.org/wiki/Backdoor),
[*Open Web Application Security Project (OWASP) (mailing list)*](http://sourceforge.net/mailarchive/forum.php?thread_id=9240254&forum_id=45046)，
等等。

[*Bruce Schneier在其论文尤其浓墨重彩长篇大论地写了关于我论文的评论*](http://www.schneier.com/blog/archives/2006/01/countering_trus.html)，他的网站和Lamba-the-Ultimate都有很多博客地址。文章[*Open Source is Securable*](http://www.leuf.net/ww/wikidn?OpenSourceIsSecurable)讨论了论文和衍生物------尤其是，他很有可能根据源代码分析做出有力的结论。

[*BartK's "Defeating the Trust Attack"*](http://imgur.com/a/BWbnU#0)总结了我的PhD论文；并据此写了[*a spirited reddit discussion in September 2013*](http://www.reddit.com/r/programming/comments/1m4mwn/a_simple_way_of_defeating_the_compiler_backdoor/)。

## 我的论文与众不同？

那当然。尤其是我的论文把之前极少搭界的不同的技术领域整合到了一起。实际的演示包括了分析C编译器生成的机器代码（不只是汇编代码！），以及Lisp生成的S-表达式。为了*证明*这确实有效，我使用了一阶谓词逻辑（一种数学逻辑表达）和多种不同的工具来辅助自动化这一过程。我的数学模型需要考虑不同文本编码系统之类的东西，因为我需要这些模型精确建模，这样才能对抗来自与复杂的真实世界的各种攻击。有些论文深入探讨了机器码的技术细节，而另一些人专注于的数学证明抽象性；还有极少数人二者兼而探讨之。坦率地说，我认为这种不走寻常路的组合会带来更有趣的结果；所以我也推荐你们这样做。

很多人坚定地*认为*我在实现不可能的梦想，所以我尽可能地证明他的正确性。我不仅仅提供了数学证明；我还提供了形式化证明，每一步都明确有明确说明（数学书的大多数证明都写着"证明过程略"，而我不会这样）。我的证明是Hilbert（3列）风格，对每一步都有说明。我直接使用了校准工具的输出；我本可以修改一下结果，这样会更清楚；但是直接使用这些输出使我避免了在转换时可能犯错，更重要的是，我可以使用单独的工具（ivy）来对证明做充分的检查。许多人没有这方面的数学背景，所以我给出了每一步骤的参考，并详细解释了每一个数学命题的细节。

## 相关材料

在相关领域也有很多人在做相关工作，特别是实现可重复（确定性）的编译（这可以精确重建可执行文件的源代码）或证明程序按照他们希望的方式在运行。我在2015年在旧金山的RSA会议上的[*\"Countering Development Environment Attacks\"*](http://www.rsaconference.com/events/us15/agenda/sessions/1613/countering-development-environment-attacks)（Dan Reddy和我的演讲）中提到了一些这样的问题和反制措施。这是其中的一些重点。

## 工具链攻击：真实且存在的**

保护开发环境很*重要*，其中还包括开发的工具链。Ken Thompson在上世纪80年代演示过对工具链的攻击，而且是完全成熟的\"trusting trust\"攻击。我的论文也讨论了对于Delphi编译器的攻击。据透露在2015年[*由于XCodeGhost的攻击，超过4,000个苹果iOS应用遭到破坏并流入苹果的应用市场*](https://www.fireeye.com/blog/executive-perspective/2015/09/protecting_our_custo.html)。攻击欺骗开发者下载苹果的XCode开发环境的一个包含恶意代码的版本。许多著名应用都受到感染，包括愤怒的小鸟2和微信。[*CNBC报道*](http://www.cnbc.com/2015/09/20/apples-ios-app-store-suffers-first-major-attack.html)说苹果正在"清理iOS应用商店，移除那些在第一轮大范围攻击中被发现的受感染的iPhone和iPad应用"。据报道一些网络安全公司最先发现了一个被称为XcodeGhost的恶意程序被嵌入了成百上千的合法应用中，随后苹果公司也说他们采取了行动......苹果表示，黑客们通过欺骗开发者下载一个假冒的的包含恶意代码的iOS和Mac应用开发工具，即人们通常所说的Xcode，从而把恶意代码嵌入到这些应用中去。[*路透社也做了类似的报道。*](http://www.reuters.com/article/2015/09/20/us-apple-china-malware-idUSKCN0RK0ZB20150920)

## 可重复的（确定性的）编译版本

创建可重复的编译版本（也叫确定性版本）是监测许多种对于开发的攻击行之有效的办法，也是应用DDC的先决条件。[*reproducible-builds.org*](https://reproducible-builds.org)网站有很多关于这个话题的信息。这是一些可能很有帮助的重点。

Tor项目对可重复性（确定性）编译版本非常重视：

-   [*Tor项目的Mike Perry在2013年6月18日解释道*](https://mailman.stanford.edu/pipermail/liberationtech/2013-June/009257.html)，"我没有折腾六个星期来给Tor浏览器构建确定性版本，只为了证明我的诚实守信。考虑到现在在计算机安全和网络战争的形势，我这么做只是因为我不相信基于单一来源信任的软件开发模型的安全性能足以对抗高明的攻击者......我不相信基于软件的GPG密钥是黑客们无法获取到的，也不相信一台即使用于离线编译的机器就能原理恶意攻击......因此我们需要确定性编译版本：每个人都可以使用我们的匿名网络来下载源代码，对照公开签名的、公开审核的和镜像git仓库来对代码做验证，并重新精确编译我们的版本......否则，我真的不认为从现在开始5-10年后我们还有真正安全的电脑能用来工作:/。"如果编译器本身被植入了恶意代码，那么确定性编译版本也还是不够的，但是谢天谢地，DDC确保了对于编译器可执行文件的多方验证（检查代码仍然必不可少，但这个工作相对来说要容易许多）。
-   [*Deterministic Builds Part One: Cyberwar and Global Compromise*](https://blog.torproject.org/blog/deterministic-builds-part-one-cyberwar-and-global-compromise)和[*Deterministic Builds Part Two: Technical Details*](https://blog.torproject.org/blog/deterministic-builds-part-two-technical-details)有许多关于Tor和确定性编译版本的材料。

[*Jos van den Oever的"Is that really the source code for this software?"(2013-06-19)*](http://blogs.kde.org/2013/06/19/really-source-code-software)
中描述了试图从源代码创建可执行文件的问题。有时候这是可行的，例如对于Debian来说，"从Debian源代码构建的二进制包和发布的二进制包并不完全一致，但是差异仅限于可执行文件的时间戳和构建标识。"但正如我几年前发现的那样，这点有时也是很难实现的，因为通常需要的编译信息比所能得到的要多。你不仅需要源代码和编译脚本；你还需要知道所有相关编译软件的确切版本号，相关配置以及其他东西。但是这些是*不可能*都得到的，所以需要重复这个步骤，不断地重复，以确保得到所有的信息。如果记录了这些信息，那么你会遇到另一个问题，"我怎么知道我的编译工具是不是干净的？"所以，DDC派上用场了......因为DDC就是来回答这个问题的。

[*Debian ReproducibleBuilds project*](https://wiki.debian.org/ReproducibleBuilds)的目标是可以以精确到字节的精度重现Debian每个包的每一个编译版本。他们已经取得了很大的进展，这使我十分欣慰。他们的[*Overview of known issues related to reproducible builds*](https://reproducible.debian.net/index_issues.html)概述了会导致问题的常见原因；其中包括各种原因导致的嵌入式生成时间戳（最大的问题）和随机/顺序排序。此外，[*sources.debian.net*](http://sources.debian.net/)网站提供了途径来访问Debian的源代码。[*How Debian Is Trying to Shut Down the CIA and Make Software Trustworthy Again*](http://motherboard.vice.com/read/how-debian-is-trying-to-shut-down-the-cia-and-make-software-trustworthy-again)也讨论了这个话题。

[*Reproducible Builds for Fedora*](https://securityblog.redhat.com/2013/09/18/reproducible-builds-for-fedora/)是一个类似的项目，可以精确复现Fedora的每个包。

[*F-Droid*](https://f-droid.org/)和[*The Guardian Project*](https://guardianproject.info/)致力于复现Android的每个编译版本。更多信息可以参考[*LWN.net*](https://lwn.net/Articles/633106/)，[*Guardian的首个可重复编译的版本的信息 (这是一个开发者工具)*](https://guardianproject.info/2014/06/09/our-first-deterministic-build-lil-debi-0-4-7/)，和[*他们在应用的Checkey方面取得的成功*](https://guardianproject.info/2015/02/11/complete-reproducible-app-distribution-achieved/)。

[*How I compiled TrueCrypt 7.1a for Win32 and matched the official binaries*](https://madiba.encs.concordia.ca/%7Ex_decarn/truecrypt-binaries-analysis/)描述了TrueCrypt的一个确定性版本的实现。这是一个加密软件，可以实现文件级、分区级或基于磁盘的虚拟磁盘级别的即时加密，但是它的作者是匿名的，这使得一些人担心可执行文件有后门。注：虽然源代码是可见的，但是没有使用标准OSS许可证，这种限制很可能表明它并不符合OSS规范；[*it许多主流Linux发行商，包括Debian、Ubuntu、Fedora、openSUSE和Gentoo并不把它当作自由软件看待*](http://lists.freedesktop.org/archives/distributions/2008-October/000276.html)。最近，TrueCrypt的作者停止了开发，又由于它缺少真正的OSS许可，所以没人能继续提供支持。

[*Gitian*](https://gitian.org/)是一种"安全的以源代码控制为导向的软件分发方法，（所以）你可以下载由多个编译源验证的可信的二进制执行文件。"

[*Vagrant*](https://www.vagrantup.com/)的目的是"创建和配置轻便的、可重复性强的和绿色便携的开发环境。"[*Seth Vargo*](http://opensource.com/business/15/9/ato-interview-seth-vargo)简要地讨论了这一问题。

[*Buildroot*](http://buildroot.net)是一个通过交叉编译创建嵌入式Linux系统的简单机制。

[*Byzantine Askemos Language Layer (BALL)*](http://ball.askemos.org/)是[*Askemos Distributed Virtual Machine*](http://askemos.org/)的实现。它创建了"应用程序的自主虚拟执行环境"，和传统的云环境不同的是，它的目标是容错性和防篡改。它在几个不同的机器、运行库、编译器和操作系统等环境上执行代码，并比较加密签名。因此，这可以实现对多种底层组件攻击的防护。

Christophe Rhodes在[*Still working on reproducible builds*](http://christophe.rhodes.io/notes/blog/posts/2014/still_working_on_reproducible_builds/)和[*Reproducible builds - a month ahead of schedule*](http://christophe.rhodes.io/notes/blog/posts/2014/reproducible_builds_-_a_month_ahead_of_schedule/)中讨论过在不同系统上构建Steel Bank Common Lisp
(SBCL)的编译版本时的问题。虽然他的文章都是SBCL的具体问题，但是从中也能反映更多的一般性问题。他指出，SBCL从CMCL分离的原因之一是"使编译的结果与编译器无关。"这一目标不仅仅是为了防止攻击，也是为了消除难以发现的bug："......我们怎么知道没有潜伏在编译器里的微小偏差，平时不会影响运行，但是会在将来导致难以发现的问题呢？（事实上这样的问题有很多，不合时宜地入雨后春笋般冒出来）。我一直在处理这些问题，SBCL!编译器是Common Lisp代码编写的，我试图让一般Lisp代码足够便携，这样在不同的环境下都可以执行，并且都可以生成精确到位的相同的输出。这样的话，也只有这样，我们才有信心，不依赖特定的具体实施细节的某些不可预知的方式......"。下面是他（可能还有其他SBCL开发者）发现并修复的一些问题，可以作为一些例子，告诉我们究竟要找怎样的问题：

1.  Common Lisp规范允许实现计算(write-to-string \'(quote foo) :pretty nil) as either \"(QUOTE FOO)\" or \"\'FOO\"。为了创建可复现的编译版本，他们需要用点小手段（），这样才不会让差异产生影响。
2.  反引用的时候会产生一个相关问题：Common Lisp规范允许实现决定值是否合并，但是这可能导致不同的结果；解决办法是修改代码，以确保结果都是相同的。
3.  不同的和集合相关的函数（例如，集合求差，并集，交集等等）并不会按顺序返回集合，这也可能导致编译版本产生差异。解决方案是对集合操作的结果进行排序，以保证集合按照特定已知顺序排列。
4.  调用maphash会直接影响Lisp镜像。通常，当遍历其内容时，hash表并没有任何特定的排序，所以在遍历时需要强制指定一个顺序。
5.  通过实现定义的常量，尤其是最大的正长整数和最小的负长整数，但同样是阵列维度限制和每秒内部时间单位。
6.  一些函数不同，例如随机函数、哈希函数，这导致了访问模式的不同。
7.  Lisp的排序不是稳定排序，所以应该使用稳定排序，以确保其确定性。
8.  阵列的初始值是不确定的，所以不要依赖这些值！

要注意的关键是创建可以由其他编译器创建可重现的（确定性）版本的编译器这件事，在现实世界中需要不少工作......但他确实是可行的。

有多种工具可以帮助建立可重现的编译版本。例如，如果嵌入了编译路径，可以强制固定路径值，这样编译版本就具有重现性。有的工具无需root权限就可以实现这个，包括我的工具[*user-union*](https://www.dwheeler.com/user-union/)和[*auto-destdir*](https://www.dwheeler.com/auto-destdir/)，还有像[*proot*](http://proot.me/)这样的工具。

## 形式化方法/证明

Xavier Leroy（OCaml的主要开发者）正在使用Coq来开发一个认证的编译器[*compcert*](http://pauillac.inria.fr/%7Exleroy/research.html#compcert)，它保证C源程序的语义支持PowerPC的汇编语言。[*这一编译器后端的"规范"（可惜不是Coq证明）是GPL软件。*](http://pauillac.inria.fr/%7Exleroy/compcert-backend/)

你可能还对MITRE Vlisp项目的结果感兴趣。Vlisp的Readme文件中写道："认证程序语言实现项目（"The Verified Programming Language Implementation project）开发了模式程序设计语言的形式化验证实现，缩写简称Vlisp......Vlisp的手册里也有项目的概述。[*Vlisp的其他资料的PDF也可以在此链接找到*](http://library.readscheme.org/)。你可能还对另一篇论文感兴趣：[*Jonathan A. Rees. "A Security Kernel Based on the Lambda-Calculus". PhD. Thesis. February 1995*](http://www.swiss.ai.mit.edu/ftpdir/users/jar/archive/whole.ps)。

## 杂项

攻击者可以利用编译器的bug，他们故意编写代码来触发编译器bug以破坏程序。这是另一种编写恶意程序的方法（在我论文中我讨论的一个话题）。这种攻击和"trusting trust"攻击DDC计数器不同，但二者是确实相关的。论文[*\"Deniable Backdoors Using Compiler Bugs\" by Scott Bauer, Pascal Cuoq, and John Regehr, *Pastor Manul Laphroaig's Export--Controlled Church Newsletter, June 20, 2015*](https://www.alchemistowl.org/pocorgtfo/pocorgtfo08.pdf)描述了编译器感染（通常通过fuzzing测试发现）是如何被利用的。在他们的案例中，他们描述了如何通过看似正常的代码利用sudo命令。[*John Regehr在他的博客 \"Defending Against Compiler-Based Backdoors\" (2015-06-21) 就指出了这一点*](http://blog.regehr.org/archives/1241)，注明"这类后门的优点是巧妙、不宜追踪和可针对特定目标，他还提出了一些很好的观点。尤其是，他指出编译器开发者需要尽快修复已知的关于编译错误的bug，并且要使用fuzz工具进行模糊测试；编译器对安全性要求极高的，但人们往往不注意这一点。开源软件包的维护者们需要对一些比较"标新立异"的补丁提交提高警惕，并且需要考虑重写这些补丁。这些攻击比传统的"trusting trust"攻击要脆弱一些，但是仍然可以在任何程序里出现，所以他们潜在的危险性十分高，并且目前来看难以检测。短期来说，最好专注于在广泛使用的编译器中检测和消灭这些缺陷。对于消灭编译器缺陷，没人会提出抱怨，我们现在有许多技术可以应用，如果编译器缺陷变得 geng 难以触发，那么对于这些后门（的攻击）的尝试会变得更加明显。但是短期策略还远远不够；我希望人们也会想出一些长期策略。

[*"Some Remarks about Random Testing" by B A Wichmann*](http://www.npl.co.uk/upload/pdf/random_testing.pdf),[*National Physical Laboratory*](http://www.npl.co.uk/), Teddington, Middlesex, TW11 0LW, UK, May 1998, 讨论了为编译器做随机测试。

[*Kegel对于gcc/glibc的跨工具链编译和测试*](http://kegel.com/crosstool/)有许多好消息。[*GCC explorer*](http://xania.org/201205/gcc-explorer)交互地展示了GCC的编译输出（在不同的输入下）。

[*The RepRap Project*](http://en.wikipedia.org/wiki/Reprap)正在开发廉价的3D打印机设计，这有希望（最终）可以实现打印其自身。这非常有意思，将来可能（和我们的讨论）非常相关。

[*Open proofs web site*](http://www.openproofs.org)鼓励"开源证明"的开发，这样所有的实现，证明和所需要的工具就都是开源软件了。

[*Mark Mitchell'的"Using C++ in GCC is OK" (Sun, 30 May 2010 17:26:16 -0700)*](http://thread.gmane.org/gmane.comp.gcc.devel/114407) 官方报告了"GCC指导委员会和FSF已经批准了C++在GCC本身的使用"。当然，不是因为我们可以使用C++，我们就去用它。我们的目标是为用户开发更好的编译器，而不是为了C++本身的福祉。[*Mark Mitchell 后来解释了他期望GCC会小心使用C++。*](http://gcc.gnu.org/ml/gcc/2010-05/msg00757.html)对于DDC来说，这意味着将DDC应用到GCC代码会需要一个C++编译器（至少需要一个支持GCC所使用那一部分的编译器），而不是C编译器。我使用Intel的icc，其中个C++编译器，所以这不会对我的例子产生影响......, 他也肯定不会改变方法的有效性。

这篇文章和停机问题（halting problem）有许多关联。[*Proof That Computers Can\'t Do Everything (The Halting Problem)*](http://youtu.be/92WHN-pAFCs) 是一个很有启发性的视频，它展示了对于停机问题（halting problem）的传统的证明，但是角度很巧妙。[*Beyond Computation: The P vs NP Problem - Michael Sipser*](http://youtu.be/msp2y_Y5MLE)同样很吸引人。

像make这样的编译工具对于许多大型系统来说非常重要。[*Improving make*](https://www.dwheeler.com/essays/make.html)描述了我对于改进POSIX标准（对于make和make的实现）所作出的努力，尤其是对于这篇文章的洞察力的支持[*Peter Miller的文章：Recursive Make Considered Harmful*](http://citeseerx.ist.psu.edu/viewdoc/summary?doi=10.1.1.20.2572).

Juniper后门很有意思------他看起来是一个被自身后门了的加密的后门。[*Matthew Green*](http://blog.cryptographyengineering.com/2015/12/on-juniper-backdoor.html)和[*rpw*](https://rpw.sh/blog/2015/12/21/the-backdoored-backdoor/)提出了很多有趣的评述。

## 规范/标准

Open Group的[ ](https://www2.opengroup.org/ogsys/catalog/c147)[*Open Trusted Technology Provider Standard (O-TTPS), Version 1.1: Mitigating Maliciously Tainted and Counterfeit
Products*](https://www2.opengroup.org/ogsys/catalog/c147)很有意思。根据网站的描述，"O-TTPS这个开源标准包含一套组织准则、要求和对集成商、供应商和零部件提供商的建议，以加强全球供应链的安全性和商用现货供应（COTS）信息和通信技术（ICT）的整合。如果人们可以遵守这一标准，将有助于在COTS ICT产品生命周期的以下几个阶段对抗恶意软件：设计、采购、编译、完善、分发、维护和终止。Open Group Trusted Technology Forum (OTTF)是一个全球性的倡议，它邀请了工业界、政府和其他感兴趣的与会者共同努力来扩展这一文档和其他OTTF交付。

[*TUF (The Update Framework)*](https://github.com/theupdateframework/tuf)帮助开发人员保护他们新的或已有的软件更新系统。在系统级包管理员、[*编程语言特定包管理器/仓库*](http://www.modulecounts.com/)和应用程序指定更新系统之间，有许多软件更新人员，需要确保他们的安全性。

## 使用OpenOffice.org/LibreOffice和OpenDocument的提示

我用[*OpenOffice.org*](http://www.openoffice.org)写了这篇文章，它完全可以胜任。OpenOffice.org在撰写大型文档方面非常好用。[*Document Foundation's LibreOffice Productivity Suite*](http://www.documentfoundation.org/download/)由OpenOffice.org（我用的就是这个）派生而来，也支持OpenDocument，所我我对OpenOffice.org的赞誉也适用于LibreOffice。（截至2011年初，似乎[*LibreOffice正在取代OpenOffice.org*](http://www.technewsworld.com/story/72329.html?wlc=1303794319&wlc=1303827127)，LibreOffice有一个更为活跃的社区。）

我开发了[*OpenDocument template for George Mason University (GMU)*](https://www.dwheeler.com/misc/gmu-sample-format.odt)，她可以帮我自动调整所有格式。这是我可以很轻松地把注意力集中在文字本身而不是格式上。

使用OpenOffice.org或其他文字处理软件编写大型文档*最重要*的规则是尽可能自动处理所有东西，尤其是要使用样式。**永远不要**手动设置大段文字的字体大小，字体类型等等（只有一个例外，可以用斜体或粗体设置强调的文字）。相反，所有的格式信息都应被放在段落样式里，然后确保每一段都应用这些样式。使用"Text body"（而不是"Default"）来设定正文字体，各种"Heading1"，"Heading2"等等样式来设定各种标题。类似的，使用插入\>交叉引用（Insert \> Cross-Reference）来引用其他文章的段落，这样的话，程序就可以正确地为他们重新编号。

OpenOffice.org可以让用户控制一行内的单词断开的方式；关于这个的更详细功能可以参考[*"Easy way to insert nonbreaking hyphen, etc. in OpenOffice.org Writer" (作者Solveig Haugland)*](http://openoffice.blogs.com/openoffice/2008/10/easy-way-to-insert-nonbreaking-hyphen-etc-in-openofficeorg-writer.html)。基本上，可以在工具\>选项\>语言设置\>语言（Tools \> Options \> Language Settings \> Languages）中选择"开启复杂文本结构"（"Enabled for Complex Text Layout"）选项来获得更多关于断字的选项。"宽度不够则不断开"（"no width no break"）的字符，也叫"胶水"字符，把它两边的字符"粘"在一起，以防止在此换行断开。类似的，"宽度不够可选断开"（"no width optional break"）字符则会在原来一般不会断开的地方通知OpenOffice.org可以插入换行。你也可以插入非换行空格，非换行连字符和可选连字符。

大多数情况下，段落样式应该让段落用正确的方式跨页断开（例如，段落样式应该有合理的默认"单行"或"孤段"的设置，标题段落样式应该有"和下一段一致"的设置）。但是在某些情况下，段落不会合适地跨页断开，因为程序并没有很好地"理解"文本。例如，如果文本和下一段对应，可能需要右键单击那一段，并且设置"和下一段一致"。在某些特殊情况下可能也需要设置不同的"单行"或"孤段"的设置。

OpenOffice.org支持公式，这是我常用的功能。例如，他的"栈"和"矩阵"选项对于多行公式来说有时非常好用。对于单行公式，我推荐将公式边框设置为0。你可以在编辑公式的时候选择公式\>间距，边框（Format\>Spacing, category Borders），然后将所有的边框宽度设置为0（我建议将这个设置为默认）。否则，当尝试组合公式的时候，公式中会有额外的空格，使它们看起来很怪异。

在最终版本中，我使用工具\>更新所有（Tools \> Update All），这样可以更新所有的目录和交叉引用等等，移动到文档的开始部分，并且保存，然后点击文件\>导出为PDF（File \> Export as PDF）。

## 杂项

在无数乏味的编译工作之后，[*Xkcd关于编译的漫画*](http://xkcd.com/303/)终于使我展颜舒眉。[*halting problem解决的图示*](http://xkcd.com/1266/)也和之相关:-)。

Dilbert也提到了很长的编译时间：[*Dilbert 2013-06-22*](http://dilbert.com/2013-06-22/)[*Dilbert 2005-09-23*](http://dilbert.com/strips/comic/2005-09-23/)[*Dilbert 1998-06-21*](http://dilbert.com/strips/comic/1998-06-21/)。[*Dilbert曾经注意到"也许编译器本身有个bug"。*](http://books.google.com/books?id=7jF1vg_A8OIC&pg=PA143#v=onepage&q&f=false)[*Dilbert还说明了为什么软件单源策略不好。*](http://dilbert.com/strips/comic/2000-03-19/)

我给出了一个简单可读的Lisp s表达式的例子；[*可读的Lisp s表达式项目对于curly-infix表达式有规范和实现。*](http://readable.sourceforge.net)

[*neoteric表达式和sweet表达式*](http://readable.sourceforge.net)可以让Lisp符号**更易**读懂。

[*Mortality.pvs是一个简短的演示，演示了如何用PVS表达"所有男人都是凡人"的例子。*](https://www.dwheeler.com/trusting-trust/mortality.pvs)

[*点击链接进入在SGI IRIX上安装gcc的说明。*](http://www.ve3syb.ca/software/irix/irix-gcc.html)

[*ERESI*](http://www.eresi-project.org/)（ERESI逆向工程软件接口）是一个"基于可执行和链接格式（ELF）的针对操作系统的统一的多体系结构二进制分析框架"。[*developerworks关于ELF有一篇很好的文章*](http://www-128.ibm.com/developerworks/power/library/pa-spec12/)。Brian Raiter写了[*Elfkickers*](http://www.muppetlabs.com/%7Ebreadbox/software/elfkickers.html)，他还写了[*A Whirlwind Tutorial on Creating Really Teensy ELF Executables for Linux*](http://www.muppetlabs.com/%7Ebreadbox/software/tiny/teensy.html)和[*Albert Einstein's Theory of Relativity: In Words of Four Letters or Less*](http://www.muppetlabs.com/%7Ebreadbox/txt/al.html)。这篇[*早期的文章分析了ELF的优点。*](http://www.linuxjournal.com/article/1059)

我试图确保这篇文章能够尽量保存得久一些。这是我的论文[*"Fully Countering Trusting Trust through Diverse Double-Compiling*](http://digilib.gmu.edu:8080/dspace/handle/1920/5667)的GMU页面，还有[*"Fully Countering Trusting Trust through Diverse Double-Compiling"的arXiv.org*](http://arxiv.org/abs/1004.5534)副本和[*我的博士论文"Fully Countering Trusting Trust through Diverse Double-Compiling"的UMI ProQuest副本*](http://disexpress.umi.com/dxweb#download?&type=pdf&dpubno=3393623)（通过[*ProQuest*](http://disexpress.umi.com/dxweb)搜索）。[*Archive.org*](https://web.archive.org/web/20130413043959/http://www.dwheeler.com/trusting-trust/dissertation/wheeler-trusting-trust-ddc.pdf)也有副本。这些都是额外的副本，内容相同。我提交的pdf属性如下：

  ------------------ -------------------------------------------------------------------------------------------------------------------------------------
  标题               Fully Countering Trusting Trust through Diverse Double-Compiling
  作者               David A. Wheeler
  日期               2009 秋季学期 (实际上是 2009-11-30)
  文件名             wheeler-trusting-trust-ddc.pdf
  长度               1,971,698 字节
  页数               199
  **MD5 hash**       5320ff082ec060e7f58409b3877cb687
  **SHA-1 hash**     20c8b702dd4b7c6586f2 59eb98f577dbadd359dd
  **SHA-256 hash**   024bccc5254eaffe9466f12afe39f72b 154f63a6919f4e1add5d0513092b2052
  **SHA-512 hash**   0004998431af5da486a87794969a5314 07cb607ffc411c966a23343a58636c20 72ceb85835ffe6eef727696ffc41b1dd d6d9e0fd090cbc85a33041c25acd2e55
  ------------------ -------------------------------------------------------------------------------------------------------------------------------------

## 微污染

另外：在ACSAC 2005上，Aleks
Kissinger（来自塔尔萨大学）也介绍了他和我在微污染方面所做的工作。由于这似乎已经从网络上消失了，我想我应该在这里简要描述一下。

Aleks的演讲题目是"Fine-Grained Taint Analysis using Regular Expressions"，这是[*正在进行的工作*](http://www.acsa-admin.org/2005/wip.html)的一部分。基本上我们注意到，不是为一整个值（如字符串）分配"污点"，您可以对子组件（如每个字符）分配污点。然后，您可以位识别输入路径和可能进来的东西分配规则（通常为零个或多个污染字符），同样还有输出路径规则。我们聚焦在为那些合法的东西定义正则表达式，不过任何其他表达式模式，例如BNFs也可以。我们注意到，你可以静态或动态地进行检查。对于静态情况，当您后向检查时，如果检查"失败"，您甚至可以轻易地导出导致安全故障的输入模式（从这些信息，应该很容易找到修复方案）。

Aleks最近通过将正则表达式转换为DFA而取得了一些进展。还有另一个关于使用Java进行污点分析的ACSAC演示，但这是在许多语言中使用的传统的"整个变量"方法，但是许多漏洞是通过这种方法实现的。我们希望这种微污染方法能够在软件交付给最终用户之前，改进软件中检测安全漏洞的工具。

弗吉尼亚大学（UVA）在进行一些我们知道的相关工作，不过我们只是通过我们的网络（通过Usenix）半路发现的。
关于UVA工作的更多信息在[*Anh Nguyen-Tuong，Salvatore Guarnieri，Doug Greene，Jeff Shirley和David Evans的"Automatically Hardening Web Applications Using Precise Tainting"中*](http://dependability.cs.virginia.edu/publications/2005/sec2005.pdf)。

他们专注于PHP，并且只关注动态情况; 我们对两者都感兴趣，但对静态情况（某些漏洞永远不会发生，因此不需要任何运行时开销来处理它们）尤其感兴趣。

其他相关工作包括[*BRICS Java字符串分析器*](http://www.brics.dk/JSA/)（GPL;使用BSD许可的dk.brics.automaton）。[*Hampi*](http://people.csail.mit.edu/akiezun/hampi/)可能能够静态地实现这一点，这很厉害。

在数据流，静态类型和安全性方面的工作有着悠久的历史（如[*Dennis Volpano*](http://www.cs.nps.navy.mil/people/faculty/volpano/papers/)等人的工作）。他们做的很好，但和我们的关注点有些偏差。这些工作倾向于将变量视为一个整体，而我们正在跟踪*更小的*数据单元。我们还跟踪包含具有不同安全级别的数据的序列（如数组），大多数这样的工作把数组作为单一的单位处理（这是一种简化的，从根本上与我们的方法不同）。

在这里可以看到[*我的教育经历*](https://www.dwheeler.com/trusting-trust/education-timeline.html)，[*我的关于安全软件的书籍*](https://www.dwheeler.com/secure-programs)，[*FlawFinder*](https://www.dwheeler.com/flawfinder)，和[*我的主页*](https://www.dwheeler.com)。
