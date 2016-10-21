---
layout: post
title:  "自由/开源软件(FLOSS)的最佳实践标准（第三部分）"
date: 2016-10-21
summary: 本文提供一系列自由/开源软件 (FLOSS) 项目的最佳实践方法。参照这些最佳实践标准的项目可以进行自认证, 以获得核心基础设施促进会(CII)徽章。
categories:
---

**原文：[Best Practices Criteria for Free/Libre and Open Source Software (FLOSS) (version 0.8.0)](https://github.com/linuxfoundation/cii-best-practices-badge/blob/master/doc/criteria.md)**

作者（组织）：Core Infrastructure Initiative Best Practices Badge

Github 提交者：david-a-wheeler, altonius, kfogel, dankohn, NilsEnevoldsen

译者：wnereiz

许可证：MIT 或 CC-BY-3.0+

# 译者注

> wnereiz: 自由/开源软件(FLOSS)的最佳实践标准的重要意义不仅仅在于获得认证，它具有在自由/开源生态领域更加广泛的指导性作用。由于篇幅较长，所以计划将此文章分阶段翻译并最终汇总。本文为第三部分。


# 自由/开源软件(FLOSS)的最佳实践标准 (版本 0.8.0)

*接上文:[自由/开源软件(FLOSS)的最佳实践标准（第二部分）](http://hardenedlinux.org/2016/09/07/best-practices-criteria-for-floss-part3.html)* 


### 分析

*静态代码分析*

- <a name="static_analysis"></a>对于所有主要产品的待发布版本，如果针对项目所选择的程序设计语言，存在自由/开源(FLOSS)的静态代码分析工具可以实现此规范，则在此版本发布之前都必须使用至少一种这类工具进行分析。
  静态代码分析工具可以检查软件代码（例如源代码、中间代码，或可执行文件），而不必以特定的输入运行此软件。
  就此标准的目的而言，编译器警告以及“安全”语言模式并不是静态代码分析工具（一般来说，它们都会避免进行更深入的分析，因为速度更加重要）。
  这类静态代码分析工具的例子包括
  [cppcheck](http://cppcheck.sourceforge.net/)、
  [clang static analyzer](http://clang-analyzer.llvm.org/)、
  [FindBugs](http://findbugs.sourceforge.net/)
  (包括 [FindSecurityBugs](https://h3xstream.github.io/find-sec-bugs/))、
  [PMD](https://pmd.github.io/)、
  [Brakeman](http://brakemanscanner.org/)、
  [Coverity Quality Analyzer](https://scan.coverity.com/) 
  和 [HP Fortify Static Code Analyzer](http://www8.hp.com/au/en/software-solutions/static-code-analysis-sast/)。
  包含更多此类工具的列表可以在例如以下这样的页面中找到
  [Wikipedia list of tools for static code analysis](https://en.wikipedia.org/wiki/List_of_tools_for_static_code_analysis)、
  [OWASP information on static code analysis](https://www.owasp.org/index.php/Static_Code_Analysis)、
  [NIST list of source code security analyzers](http://samate.nist.gov/index.php/Source_Code_Security_Analyzers.html) 
  以及 [Wheeler's list of static analysis tools](http://www.dwheeler.com/essays/static-analysis-tools.html)。
  [SWAMP](https://continuousassurance.org/) 是一个能够通过多种工具评估软件漏洞的免费平台。
  <sup>[<a href="#static_analysis">static_analysis</a>]</sup>
- <a name="static_analysis_common_vulnerabilities"></a>“建议“使用至少一种静态分析工具应用于静态分析标准，此标准包括了一系列规则或方法，用于在所分析的语言或环境中发掘常见的漏洞。 
  <sup>[<a href="#static_analysis_common_vulnerabilities">static_analysis_common_vulnerabilities</a>]</sup>
- <a name="static_analysis_fixed"></a>在静态代码分析中发现的所有中、高严重级别的可利用漏洞，“必须“在其得到确认之后及时修复。
  如果漏洞的 [CVSS 2.0](https://nvd.nist.gov/cvss.cfm) 达到 4 或更高，则为中到高严重级别。
  <sup>[<a href="#static_analysis_fixed">static_analysis_fixed</a>]</sup>
- <a name="static_analysis_often"></a>“建议“针对每次提交都进行静态源代码分析，或者至少每天一次。
  <sup>[<a href="#static_analysis_often">static_analysis_often</a>]</sup>

*动态分析*

- <a name="dynamic_analysis"></a>对于所有主要产品的待发布版本，“建议“在其发布之前使用至少一种动态分析工具进行分析。
  动态分析工具可以通过执行特定的输入对软件进行检查。
  例如，项目可以使用模糊测试工具(fuzzing tool)
  (如 [American Fuzzy Lop](http://lcamtuf.coredump.cx/afl/))
  或者网站应用程序扫描器
  (如 [OWASP ZAP](https://www.owasp.org/index.php/OWASP_Zed_Attack_Proxy_Project)
  或 [w3af.org](http://w3af.org/))。
  就此标准的目的而言，动态分析工具需要以某种方式具备多样化的输入，从而可以发掘不同种类的问题，*或者*此工具是具有至少80%分支覆盖率的自动测试套件。
  [Wikipedia 上关于动态分析的页面](https://en.wikipedia.org/wiki/Dynamic_program_analysis) 以及
 [OWASP 上关于模糊测试的页面](https://www.owasp.org/index.php/Fuzzing) 给出了一些动态分析工具。
  <sup>[<a href="#dynamic_analysis">dynamic_analysis</a>]</sup>
- <a name="dynamic_analysis_unsafe"></a>如果是使用非保护型内存的语言（例如 C 或 C++）开发的应用层软件，则“建议“使用至少一种动态工具（如模糊测试工具(fuzzer)或网络应用程序扫描器）以某种常规机制探测内存安全性问题，例如缓冲区溢出。
  探测内存安全性问题的机制包括如 Address Sanitizer (ASAN) 和 [valgrind](http://valgrind.org/)。
  也可使用被普遍应用的断言。
  如果并非是应用层软件，或者软件没有使用非保护型内存(memory-unsafe)语言开发，则此标准自动满足。
  <sup>[<a href="#dynamic_analysis_unsafe">dynamic_analysis_unsafe</a>]</sup>
- <a name="dynamic_analysis_enable_assertions"></a>“建议“在动态分析时对包含多个运行时断言的软件进行检查。
  <sup>[<a href="#dynamic_analysis_enable_assertions">dynamic_analysis_enable_assertions</a>]</sup>
- 分析工具“可以“重点集中于发掘安全漏洞，但这不是必需的。
- <a name="dynamic_analysis_fixed"></a>在动态代码分析中发现的所有中、高严重级别的可利用漏洞，“必须“在其得到确认之后及时修复。
  如果漏洞的 [CVSS 2.0](https://nvd.nist.gov/cvss.cfm) 达到 4 或更高，则为中到高严重级别。
  <sup>[<a href="#dynamic_analysis_fixed">dynamic_analysis_fixed</a>]</sup>
- *理论依据*: 静态源代码分析和动态分析的目的是要发现不同种类的缺陷（包括导致安全漏洞的缺陷）。所以将二者结合起来会更有效。

### 未来

以下“未来”的标准指的是我们想要在不久以后添加的标准。

- <a name="installation_common"></a>（未来的标准）项目“应当”提供一种方法，可以使用被普遍应用的常规方式轻松地对软件进行安装和卸载。
  例如，（系统或语言级的）包管理器，"make install/uninstall"（支持 DESTDIR），标准格式的容器，或标准格式的虚拟主机镜像文件。
  安装和卸载流程（例如，打包）“可以”通过第三方实现，但前提是其必须为自由/开源软件(FLOSS)。
  <sup>[<a href="#installation_common">installation_common</a>]</sup>
- <a name="build_reproducible"></a>
  （未来的标准）“建议”项目支持[可重现构建(reproducible build)](https://reproducible-builds.org/)。
  通过可重现构建，多方开发者可以独立地重复进行从源码文件生成所需信息的过程，并得到完全一致的结果。
  [在可重现构建项目中可以找到相关的文档介绍如何去做](https://reproducible-builds.org/docs/)。
  如果不需要构建（例如，脚本语言，其源代码可以直接使用而无须编译），则此标准不适用。
  <sup>[<a href="#build_reproducible">build_reproducible</a>]</sup>
  *理论依据*: 如果项目需要构建，但并没有可工作的构建系统，则那些潜在的协助开发者将无法便捷地贡献代码，而且许多安全分析工具将会失效。如果不需要构建任何东西，则工作构建系统的标准不适用。
- <a name="crypto_used_network"></a>（未来的标准）当可以使用加密的网络通信协议（如 HTTPS/TLS 和 SSH）时，项目“不应当”使用未加密协议（诸如 HTTP 和 telnet），除非用户明确地提出需求或者明确需要做此配置。
  <sup>[<a href="#crypto_used_network">crypto_used_network</a>]</sup>
- <a name="crypto_tls12"></a>（未来的标准）如果项目支持 TLS，则其“应当”至少支持 TLS 1.2 版。注意，TLS 的前身叫 SSL。
  <sup>[<a href="#crypto_tls12">crypto_tls12</a>]</sup>
- <a name="crypto_certificate_verification"></a>（未来的标准）如果项目支持 TLS，则在其使用过程中“必须”默认执行 TLS 证书验证，这也包括了项目的次级资源。
   注意，不正确的 TLS 证书验证方式属于常见的错误。详细信息参见
   ["The Most Dangerous Code in the World: Validating SSL Certificates in Non-Browser Software" by Martin Georgiev et al.](http://crypto.stanford.edu/~dabo/pubs/abstracts/ssl-client-bugs.html) 和
   ["Do you trust this application?" by Michael Catanzaro](https://blogs.gnome.org/mcatanzaro/2016/03/12/do-you-trust-this-application/)。
   <sup>[<a href="#crypto_certificate_verification">crypto_certificate_verification</a>]</sup>
- <a name="crypto_verification_private"></a>（未来的标准）如果项目支持 TLS，则其应当在发送隐私信息（如 secure cookies）的 HTTP 头之前进行证书验证。
   <sup>[<a href="#crypto_verification_private">crypto_verification_private</a>]</sup>
- <a name="hardened_site"></a>（未来的标准）“建议”项目的网站、软件仓库（如果可以通过网页访问）以及下载站点（如果是独立的）能够支持带有非宽容值的密钥加固头。
  注意，已知 GitHub 是满足此条件的。
  网站如 https://securityheaders.io/ 可以帮助进行快速检查。
  这些密钥加固头为：
  Content Security Policy (CSP)、HTTP Strict Transport Security (HSTS)、X-Content-Type-Options (as "nosniff")、X-Frame-Options 以及 X-XSS-Protection。
  <sup>[<a href="#hardened_site">hardened_site</a>]</sup>
- <a name="hardening"></a>
  （未来的标准）
  “建议”项目使用加固机制，这样可以减少由于软件缺陷导致安全漏洞的可能性。加固机制可以包括诸如内容安全规则 (CSP) 这样的 HTTP 头、用来缓解攻击的编译器参数（如 -fstack-protechtor）或移除未定义行为的编译器参数。
  基于我们的目的，最低权限不能作为一种加固机制（最低权限非常重要，但它是独立的方法）。
  <sup>[<a href="#hardening">hardening</a>]</sup>

## 非标准

我们计划*不*依赖于任何特定的产品或服务。
特别是我们计划*不*依赖私有工具或服务，因为许多[自由软件](https://www.gnu.org/philosophy/free-sw.en.html)开发者拒绝这类标准。
所以，我们会有意地*不*依赖于 git 或 GitHub。
我们也不会要求或禁止任何特定的编程语言（然而，我们可能会推荐一些语言）。
这也意味着可以应用新的工具和技能，项目可以快速切换到其中而不用担心违背任何标准。
然而，此标准有时会识别某些行为的通用方式或方法（特别是 FLOSS），因为这样的信息可以帮助人们理解并符合标准。
我们计划为那些在 GitHub 上使用 git 的项目创建一个“易于操作的流程“。
我们欢迎优秀的补丁，能够帮助我们为其他软件仓库平台上的项目提供这种“易于操作的流程”。

我们没有计划对有关项目中活跃用户的讨论做出要求。有些非常成熟的项目很少有变动，所以活跃度会很低。
然而，如果项目存在漏洞报告，我们则*会*要求其作出回应（见上述内容）。

## 唯一地识别某项目

如何能够唯一地识别项目是一大挑战。
我们的 rails 程序针对每个新项都要目给出一个唯一的 id，这样我们可以通过此 id 来识别项目。 
然而，这种方法并不能帮助人们在不知道 id 的情况下搜索某个项目。

基于我们的目的，规定项目的“首页” URL 和/或其软件仓库的 URL 作为其*真实*名称。
多数项目具有人类可读的名称，但使用这些名称是不够的。
同样的可读名称可以用于不同的项目（包括项目分支），而且同一个项目可以有许多不同的名称。
许多情况下，指出项目的其他名称非常有用（例如，Debian 中的源代码包名称，一些编程语言仓库中的包名称，或其在 OpenHub 中的名称）。

在未来我们会尝试更仔细地检查用户是否合法的展现了其项目。
当前，我们主要集中精力于检查项目是否使用了 GitHub 仓库；如果其他情况变得更重要，则会有许多方式去处理。
我们希望在绝大部分情况下用户*无法*编辑 URL。因为如果可编辑，他们就可以进行欺骗，让人以为他们控制着一个实际并非其所控制的项目。
所以说，这些人无法从创建伪造的行条目中得到好处；最关键的部分是项目提及其徽章时所使用的 id，而这是由项目本身来决定的。

因此，徽章信息将包含作为名称的 URL，年限范围，以及级别/名称（如果超过一个的话）。

我们将会实现一些搜索机制，人们可以通过输入通用名称检索某项目。


## 为何制定此标准？

文章 [Open badges for education: what are the implications at the
intersection of open systems and badging?](http://www.researchinlearningtechnology.net/index.php/rlt/article/view/23563) 中给出了使用徽章系统的三个普遍性的理由（用在这里都是有效的）：

1. 徽章可以作为某种行为的动力。我们希望通过最佳实践的鉴别，鼓励那些未实现的项目实现这些最佳实践方法。
2. 徽章可以作为一种教学工具。一些项目可能并没有意识到其他项目所使用的最佳实践方法，或者只是部分应用了这些方法。
   徽章将帮助他们意识到这些最佳实践以及实现它们的方法。
3. 徽章可以作为一种象征物或凭证。
   潜在的用户会趋向于使用那些应用了最佳实践的项目，来获得一贯良好的效果；徽章可以让项目表明其符合最佳实践，而且让用户易于得知有哪些项目这样去做了。

我们选择了使用自认证的方式，因为这样可以使得数量众多的项目（甚至是小项目）参与进来。这么做的风险是有些项目可能错误的声称自己是符合的，但我们认为风险很小，而且用户自己可以对其检查。

## 改进标准

我们希望从所有人那里获得好的建议和反馈；请您参与进来！

我们当前计划（一旦准备完毕就）启动单一的徽章等级。
此后，最终可能存在多个等级（铜，银，金）或其他（有前提条件）的徽章。
我们经常在讨论的一个问题是，是否应该在标准中添加关于持续集成的要求；如果不添加的话，则我们希望将其放在更高等级之中。
更多的信息请参见[other(其他)](./other.md)。

您也可以查阅“[background(背景)](./background.md)”文件以获得关于此标准的更多信息，以及“[implementation(实现)](./implementation.md)”中有关徽章应用程序(BadgeApp)的注解。
