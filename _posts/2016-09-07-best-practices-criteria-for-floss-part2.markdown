---
layout: post
title:  "自由/开源软件(FLOSS)的最佳实践标准（第二部分）"
date: 2016-09-07
summary: 本文提供一系列自由/开源软件 (FLOSS) 项目的最佳实践方法。参照这些最佳实践标准的项目可以进行自认证, 以获得核心基础设施促进会(CII)徽章。
categories:
---

**原文：[Best Practices Criteria for Free/Libre and Open Source Software (FLOSS) (version 0.8.0)](https://github.com/linuxfoundation/cii-best-practices-badge/blob/master/doc/criteria.md)**

作者（组织）：Core Infrastructure Initiative Best Practices Badge

Github 提交者：david-a-wheeler, altonius, kfogel, dankohn, NilsEnevoldsen

译者：wnereiz

许可证：MIT 或 CC-BY-3.0+

# 译者注

> wnereiz: 自由/开源软件(FLOSS)的最佳实践标准的重要意义不仅仅在于获得认证，它具有在自由/开源生态领域更加广泛的指导性作用。由于篇幅较长，所以计划将此文章分阶段翻译并最终汇总。本文为第二部分。


# 自由/开源软件(FLOSS)的最佳实践标准 (版本 0.8.0)

*接上文:[自由/开源软件(FLOSS)的最佳实践标准（第一部分）](http://hardenedlinux.org/2016/08/04/best-practices-criteria-for-floss-part1.html)* 


### 变更控制

*公共版本控制下的源代码仓库*

- <a name="repo_public"></a>项目“必须“提供基于版本控制且公开可读的源代码仓库，而且必须支持 URL 访问。
  此 URL “可以“与项目的 URL 相同。
  项目“可以“在某变更仍未公开发布的特定情况下使用私有（非公开）的分支。（例如，在为了修复一个未公开的安全漏洞的情况下）。
  <sup>[<a href="#repo_public">repo_public</a>]</sup>
- <a name="repo_track"></a>源代码仓库“必须“能够追踪所有发生的变更，谁做了这些变更，何时做的这些变更。
  <sup>[<a href="#repo_track">repo_track</a>]</sup>
- <a name="repo_interim"></a>为了能够进行协作检查，项目的源代码仓库“必须”支持临时版本，用来在各发布版本间进行检查；项目“一定不能”只包含最终版本。
  项目“可以“选择在其公开的源代码仓库中忽略掉特定的临时版本（例如，那些修复了特定的非公开安全漏洞的版本，这些版本也许从来不能够公开发布，或者其中的材料不允许合法的发表，且并非最终版本）
  <sup>[<a href="#repo_interim">repo_interim</a>]</sup>
- <a name="repo_distributed"></a>我们“建议”使用那些被广泛应用的版本控制软件（例如，git）。
  Git 并非是指定要求使用的，作为一个项目可以使用中心化的版本控制软件（如 subversion）。
  <sup>[<a href="#repo_distributed">repo_distributed</a>]</sup>

*版本编号*

- <a name="version_unique"></a>项目的每个版本“必须”拥有唯一的发行版本编号，以便于用户使用。
  <sup>[<a href="#version_unique">version_unique</a>]</sup>
- <a name="version_semver"></a>我们推荐使用
  [语义化版本 (SemVer) 格式](http://semver.org/lang/zh-CN) 进行发行版本的标注。
  <sup>[<a href="#version_semver">version_semver</a>]</sup>
- 提交识别号 (Commit IDs) （或类似的编号）“可以”作为发行版本编号使用。
  这些编号是唯一的，但要注意，这种方式会导致用户使用上的问题，因为他们可能会无法判断是否已经更新到了最新版本。
- <a name="version_tags"></a>“建议“项目使用其版本控制系统的功能标识不同的发行版本。
  例如，我们“建议”那些使用 git 管理的项目利用 git tags 来进行发行版本标识。
  <sup>[<a href="#version_tags">version_tags</a>]</sup>

*发布通知 (变更日志)*

- <a name="release_notes"></a><a name="changelog"></a>一个项目“必须”在每一个版本中提供发布通知 (release notes)，以人类可读的*摘要*形式说明此版本中主要的变更内容。
  发布通知“一定不能”是版本控制日志的输出
  （例如，"git log" 命令的输出结果并不能作为发布通知使用）。
  <sup>[<a href="#release_notes">release_notes</a>]</sup>
- <a name="release_notes_vulns"></a><a name="changelog_vulns"></a>发布通知“必须”在新版本中标识出已修复的所有公开的漏洞。
  <sup>[<a href="#release_notes_vulns">release_notes_vulns</a>]</sup>
- 发布通知“可以”通过多种途径实现。
  许多项目将其置于名为 "NEWS", "CHANGELOG", 或 "ChangeLog" 的文件中, 也可选择加上扩展名，如 ".txt", ".md", 或 ".html"。
  历史上，术语 “变更日志(change log)” 的含义是*每一项*变更的记录，但是为了满足这些标准，我们须要的是人类可读的摘要。
  发布通知“可以”由版本控制系统的机制提供，如 [GitHub 的发布工作流程](https://github.com/blog/1547-release-your-software).
- *理论依据*: 发布通知是非常重要的，因为它可以帮助用户决定是否进行升级，以及升级会带来什么影响。
  （例如，新的版本是否修复了漏洞）。

### 报告

*Bug reporting process*

- <a name="report_process"></a>项目“必须”提供一个允许用户提交故障报告 (bug report) 的程序（例如，使用问题追踪系统或邮件列表）。
  <sup>[<a href="#report_process">report_process</a>]</sup>
- <a name="report_tracker"></a>项目“应当”使用问题追踪系统跟踪单独的问题。
  <sup>[<a href="#report_tracker">report_tracker</a>]</sup>
- <a name="report_responses"></a>项目“必须”能够获知在过去2至12个月（包含12个月）内的主要故障报告;
  回应中不必包含故障修正。
  <sup>[<a href="#report_responses">report_responses</a>]</sup>
- <a name="enhancement_responses"></a>项目应当回应过去2至12个月（包含12个月）内的大多数的增强请求。
  项目“可以”选择不做回应。
  <sup>[<a href="#enhancement_responses">enhancement_responses</a>]</sup>
- <a name="report_archive"></a>项目“必须”具有公开的报告和响应归档，以方便日后进行搜索。
  <sup>[<a href="#report_archive">report_archive</a>]</sup>

*漏洞报告流程*

- <a name="vulnerability_report_process"></a>项目“必须”在其网站上发布漏洞报告流程。
  例如， 在 <https://PROJECTSITE/security> 上清晰地指明邮箱地址，通常其形式为 security@example.org。
  此流程“可以”与故障报告流程相同。
  <sup>[<a href="#vulnerability_report_process">vulnerability_report_process</a>]</sup>
- <a name="vulnerability_report_private"></a>如果支持私密的漏洞报告，项目“必须”包含如何以私密方式发送信息的说明。
  例如，在支持 TLS 的网页上，或者使用 OpenPGP 加密的邮件提交私密漏洞报告。
  如果不支持私密缺陷报告，则此标准自动获得满足。
  <sup>[<a href="#vulnerability_report_private">vulnerability_report_private</a>]</sup>
- <a name="vulnerability_report_response"></a>项目对在过去6个月以内收到的任何漏洞报告的初始响应时间“必须”小于或等于14天。
  <sup>[<a href="#vulnerability_report_response">vulnerability_report_response</a>]</sup>

### 质量

*工作构建系统*

- <a name="build"></a>如果软件需要构建才能使用，则项目“必须”提供工作构建系统 (Working build system)，以便于从源代码自动重构建此软件。
  构建系统决定了重新构建软件时应采取哪些行为（以及这些行为的顺序），然后按步骤执行。
  <sup>[<a href="#build">build</a>]</sup>
- <a name="build_common_tools"></a>推荐使用通用工具进行软件的构建。
  例如， Maven, Ant, cmake, autotools, make, 或 rake。
  <sup>[<a href="#build_common_tools">build_common_tools</a>]</sup>
- <a name="build_floss_tools"></a> 项目“应当”仅支持使用 FLOSS 工具进行构建。
  <sup>[<a href="#build_floss_tools">build_floss_tools</a>]</sup>

*自动测试套件*

- <a name="test"></a>项目“必须”具有至少一套自动化测试套件，并作为 FLOSS 公开发布（此测试套件可以作为单独的 FLOSS 项目进行维护）。
  <sup>[<a href="#test">test</a>]</sup>
- <a name="test_invocation"></a>测试套件“应当”能够作为此语言的标准方式进行调用。
  例如， "make check"、"mvn test" 或 "rake test"。
  <sup>[<a href="#test_invocation">test_invocation</a>]</sup>
- <a name="test_most"></a>“建议”此测试套件能够覆盖大多数（理想情况下所有的）代码分支、输入区域和功能。
  <sup>[<a href="#test_most">test_most</a>]</sup>
- <a name="test_continuous_integration"></a>“建议”项目能够实现持续集成（通过此技术，新的或更改过的代码可以持续不断的整合到中心代码仓库中，并可以针对其结果进行自动测试）。
  <sup>[<a href="#test_continuous_integration">test_continuous_integration</a>]</sup>
- 项目“可以”具有多个自动测试套件（例如，其中一个测试套件运行很快，相对而言，另外一个则覆盖更加全面，但需要特定的设备支持）。
- *理论依据*: 自动测试套件能够帮助我们直接发现各类问题。大型测试套件可以发现更多，但即便是小型测试套件也可以找到问题并提供框架进行构建。


*新功能测试*

- <a name="test_policy"></a>对于主要新功能的添加，项目“必须”制定一般性的规则（正式或非正式的），项目“应当”将功能性测试添加到自动测试套件中。
  <sup>[<a href="#test_policy">test_policy</a>]</sup>
- <a name="tests_are_added"></a>项目中“必须”有迹象表明在最近一次的主要变更中已经添加了此测试。一般要在变更日志中提及其主要功能。（在实际场景中，并不需要很完美，仅仅表示出已添加了测试即可。）
  <sup>[<a href="#tests_are_added">tests_are_added</a>]</sup>
- <a name="tests_documented_added"></a>“建议”将测试添加的规则*以文档形式记录*在说明中作为变更提议。然而在实际情况中，只要是添加了测试，即便是非正式的规则也是可接受的。
  <sup>[<a href="#tests_documented_added">tests_documented_added</a>]</sup>

*警告参数(flags)*

- <a name="warnings"></a>项目中“必须”启用一个或多个编译器警告参数，“安全”语言模式，或使用独立的代码扫描("linter")工具查找代码质量层面的错误或普通的简单错误 —— 若存在至少一个适用于所选语言并可实现此规则的 FLOSS 工具。 
  编译器警告参数的例子为 gcc/clang 中的 "-Wall"。
  “安全”语言模式的例子包括 Javascript 中的 "use strict"，以及 perl5 的 "use warning"。
  独立的代码扫描("linter")工具简单来说是检查代码的工具，此工具可以用于查找出代码质量上的错误或普通的简单错误。
  <sup>[<a href="#warnings">warnings</a>]</sup>
- <a name="warnings_fixed"></a>项目“必须”能够处理这些警告信息。
  项目应当修复这些警告，或在源代码中将其标注为误报。
  理想状况是不存在任何警告信息，但作为一个项目，“可以”接受一定的警告。（一般来说少于10个或每100行少于一个）。
  <sup>[<a href="#warnings_fixed">warnings_fixed</a>]</sup>
- <a name="warnings_strict"></a>“建议”项目使用最严格的警告信息设置，但这在现实场景中并不一定实际。
  <sup>[<a href="#warnings_strict">warnings_strict</a>]</sup>

### 安全

*安全部署知识*

- <a name="know_secure_design"></a>项目“必须”拥有至少一名懂得如何设计安全软件的主要开发者参与。并要求此开发者能够理解以下设计原则，其中包括了 [Saltzer 和 Schroeder](http://web.mit.edu/Saltzer/www/publications/protection/)的8条原则：
    - 机制的经济性（保持设计的精简实用，例如广泛采用简化措施）
    - 失效安全默认值（应当默认禁止访问行为，项目安装完成后在默认情况下应当是安全的）
    - 完全仲裁原则（针对所有可能被限制的访问行为都必须检查其授权，且不可跳过）
    - 开放的设计（安全机制不应当依赖于攻击者对设计的不知情，而是应该基于像密钥和密码这样更易被保护和更改的信息）
    - 特权分离 （理论上，对于重要对象的访问应当依赖超过一个条件，这样即便攻陷一个保护系统也不会得到完全的访问权。 
      例如，多要素认证，诸如需要密码加上硬件令牌(token)，要比单要素认证强度更高）
    - 最小特权 （进程应当以所需要的最小特权运行）
    - 最小公共机制 （应当将超过一个用户且所有用户都依赖的公共机制设计成最小化的，例如，临时文件目录）
    - 心理可接受性（人类用户界面必须设计成易于使用的形式，遵循“最小惊讶”原则进行设计将有助于实现此要求）
    - 有限攻击界面（应当限制攻击界面，这些界面包含不同的点，可被攻击者利用，从而进入或提取数据）
    - 基于白名单的输入验证 （在接受任何输入之前，应当对其检查以确定是否有效；此验证方法应当使用白名单的形式（仅接受已知正确的值），而不是黑名单（试图列出已知错误的值））。
      <sup>[<a href="#know_secure_design">know_secure_design</a>]</sup>
- <a name="know_common_errors"></a>至少有一名主要开发者“必须”了解常见的能够导致此类软件漏洞的错误，而且至少了解一种方法应对或缓解所有这些错误。
  例如（依据软件的类别而不同），包括 SQL 注入，操作系统(OS)注入，经典的缓冲区溢出，跨站脚本攻击，认证缺失，授权缺失。
  常用列表请参见 [CWE/SANS top 25](http://cwe.mitre.org/top25/) 或
  [OWASP Top 10](https://www.owasp.org/index.php/Category:OWASP_Top_Ten_Project)
  <sup>[<a href="#know_common_errors">know_common_errors</a>]</sup>
- 项目的“主要开发者”指的是那些熟悉项目的基础代码，可轻易修改代码，而且被其他项目参与者所熟知的开发人员。
  一名主要开发者通常会在过去的一年中做出了若干项贡献（通过代码，文档，或回答问题的方式）。
  具有以下条件的开发人员一般来说会被视为主要开发者：曾经发起了项目（而且没有离开项目超过三年的时间），有权限在私密漏洞报告频道（如果存在的话）接收信息，能够代表项目接受提交，或进行此项目软件最终版本的发布。
  如果项目仅有一名开发人员，则此人为主要开发者。


*良好的加密实践*

*注意*： 此标准并不总是会被使用，因为一些软件无需直接使用加密功能。
“项目安全机制”定义为已发布的项目软件所提供的安全机制。

- <a name="crypto_published"></a>项目的加密软件在默认情况下“必须”仅使用公开发布的加密协议和算法，并经由专家审查。
  <sup>[<a href="#crypto_published">crypto_published</a>]</sup>
- <a name="crypto_call"></a>如果项目软件是应用程序或库，且其主要目的并非实现某种加密功能；则”不应当“重新实现其独有的加密功能。
  <sup>[<a href="#crypto_call">crypto_call</a>]</sup>
- <a name="crypto_floss"></a>项目中所有依赖于加密的功能“必须”使用自由开源软件 (FLOSS) 实现。参见
  [开源促进会的*软件的开放标准要求*](https://opensource.org/osr).
  <sup>[<a href="#crypto_floss">crypto_floss</a>]</sup>
- <a name="crypto_keylength"></a>项目安全机制所使用的默认密钥长度“必须”至少能够满足 NIST 规定的最低要求，此要求（制定于2012年）的时效至2030年止。
  这些最小的位长度为：对称密钥 112，因子分解模数 2048，离散对数密钥 224，离散对数组 2048，椭圆曲线 224，以及哈希 224（此位长度的要求并不涵盖密码哈希，关于密码哈希算法的更多信息可以在<a href="#crypto_password_storage">crypto_password_storage</a>标准中找到）。
  参见 <http://www.keylength.com> 中多个组织的推荐密钥长度之间比较
  软件“必须”可配置为拒绝使用短小的密钥长度。
  软件在一些配置中“可以”允许使用短密钥长度（理想情况下不应当使用，因为可能导致降级攻击 (downgrade attacks)，但更短的密钥长度某些情况下在互操作性上是有需求的）。
  <sup>[<a href="#crypto_keylength">crypto_keylength</a>]</sup>
- <a name="crypto_working"></a>项目默认的安全机制“一定不能”依赖于不安全的加密算法。（例如，MD4, MD5, single DES, RC4, 或 Dual_EC_DRBG）。
  <sup>[<a href="#crypto_working">crypto_working</a>]</sup>
- <a name="crypto_weaknesses"></a>项目的安全机制“不应当“默认依赖于已知具有严重缺陷的加密算法 （例如，SHA-1）。
  <sup>[<a href="#crypto_weaknesses">crypto_weaknesses</a>]</sup>
- <a name="crypto_pfs"></a>项目“应当”实现基于完全前向保密 (perfect forward secrecy) 技术的密钥协商协议，这样，即便将来其中一个长期密钥泄漏，而派生自此长期密钥的会话密钥也无法被获取。
  <sup>[<a href="#crypto_pfs">crypto_pfs</a>]</sup>
- <a name="crypto_password_storage"></a>如果项目要求保存密码以进行外部用户认证，则“必须”使用密钥强度（迭代）算法（例如， PBKDF2, Bcrypt or Scrypt）将其保存为迭代哈希值 (iterated hases)，并添加基于不同用户的盐 (salt)。 
  <sup>[<a href="#crypto_password_storage">crypto_password_storage</a>]</sup>
- <a name="crypto_random"></a>项目“必须”生成所有的加密密钥，并在生成时使用具有密码学安全性的随机数生成器，且“一定不能”使用不具备密码学安全性的生成器。 
  具备密码学安全性的随机数生成器可以是硬件随机数生成器，或者可以是诸如 Hash_DRBG, HMAC_DRBG, CTR_DRBG, Yarrow, 或 Fortuna 这样的具备密码学安全性的伪随机数生成器 (CSPRNG)。 
  <sup>[<a href="#crypto_random">crypto_random</a>]</sup>

*安全的传递机制*

- <a name="delivery_mitm"></a>项目“必须”对其内容提供传递机制以对抗中间人 (MITM) 攻击。
  使用 https 或 ssh+scp 也是可接受的。
  更为强大的机制是发布带有数字签名的软件包，因为这样可以缓解基于发布系统的攻击，但只有在用户可以确定签名的公钥是正确的*且*用户实际上的确检查了签名的情况下才起作用。
  <sup>[<a href="#delivery_mitm">delivery_mitm</a>]</sup>

*已修复的公开漏洞*

- <a name="vulnerabilities_fixed_60_days"></a>“一定”不允许存在仍未打过补丁的已*公开*超过60天的中或高严重等级漏洞。
  漏洞必须由项目自身打补丁并发布（补丁可以在其他某处开发）。
  一但取得了带有可公开非付费信息的 CVE （例如，在[美国国家漏洞数据库 National Vulnerability Database](https://nvd.nist.gov/)中报道），或当项目获得了通知*且*此信息（可能经由项目）发布给了公众，则此漏洞将（为此目的）被视为已公开。
  如果其 [CVSS 2.0](https://nvd.nist.gov/cvss.cfm) 的基本得分大于或等于4，则漏洞为中到高严重等级。
  <sup>[<a href="#vulnerabilities_fixed_60_days">vulnerabilities_fixed_60_days</a>]</sup>
- <a name="vulnerabilities_critical_fixed"></a>项目“应当”在所有的关键漏洞被报道后迅速将其修复。
  <sup>[<a href="#vulnerabilities_critical_fixed">vulnerabilities_critical_fixed</a>]</sup>
- *注意*：这意味着对用户来说，漏洞将会暴露给全世界所有攻击者最多60天的时间。
  相比 Google 在 [重启责任性披露 Rebooting responsible disclosure](https://security.googleblog.com/2010/07/rebooting-responsible-disclosure-focus.html) 中所推荐的*即便*是非公开报道的情况，从项目被通知开始的 60 天的期限，此标准通常更易满足。
- *理论依据*: 我们有意选择从公开获知的时间开始进行测量，而不是从项目取得报告的时间开始，是因为这更易于那些*外部的*项目进行衡量和验证。

*其他安全问题*

- <a name="no_leaked_credentials"></a>公开软件源“一定不能”泄漏用来限制公开访问的有效私密认证信息（例如，可使用的密码或私钥）。
  项目“可以”放出“样例”认证信息，用来测试或用于不重要的数据库，前提是它们并不是用来限制公开访问的。
  <sup>[<a href="#no_leaked_credentials">no_leaked_credentials</a>]</sup>
