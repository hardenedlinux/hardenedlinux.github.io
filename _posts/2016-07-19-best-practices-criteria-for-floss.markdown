---
layout: post
title:  "自由/开源软件(FLOSS)的最佳实践标准"
summary:
categories:
---

**原文：[Best Practices Criteria for Free/Libre and Open Source Software (FLOSS) (version 0.8.0)](https://github.com/linuxfoundation/cii-best-practices-badge/blob/master/doc/criteria.md)**

作者（组织）：Core Infrastructure Initiative Best Practices Badge

Github 提交者：david-a-wheeler, altonius, kfogel, dankohn, NilsEnevoldsen

译者：wnereiz

许可证：MIT 或 CC-BY-3.0+

# 自由/开源软件(FLOSS)的最佳实践标准 (版本 0.8.0)


## 简介

本文提供一系列自由/开源软件 (FLOSS) 项目的最佳实践方法。
参照这些最佳实践标准的项目可以进行自认证, 以获得核心基础设施促进会(CII)徽章。
要做到这点无需任何费用，您的项目可以使用 Web 应用（BadgeApp)
来证明其如何符合这些实践标准以及其详细情况。

任何实践标准都无法保证软件从不出现缺陷或漏洞；甚至形式化方法在规范或假设错误的情况下都会失败。
也没有任何实践可以保证一个项目可以支撑健康和运作良好的社区。
然而，遵循最佳实践可以帮助改进项目的成果。
例如，一些实践标准要求在发布之前进行多人审查，这可以帮助找到其他情况下难以发现的漏洞，同时可以帮助建立信任，以及满足不同组织的开发者之间的重复合作的要求。

这些最佳实践标准可以用来：

1. 鼓励项目遵循最佳实践。
2. 帮助新的项目发现这些最佳实践是什么，还有
3. 帮助用户了解哪些项目遵循了最佳实践（这样用户可以选择此类项目）。

我们当前集中精力识别那些现已良好运行的典型项目所遵从的最佳实践标准。
我们也在关注其他的实践标准，今后会创建[更高级的徽章](https://github.com/linuxfoundation/cii-best-practices-badge/blob/master/doc/other.md)。
最佳实践标准，以及为其定义的更详细的标准，受到各种信息来源的启发。
更多信息请参见单独的"[背景](https://github.com/linuxfoundation/cii-best-practices-badge/blob/master/doc/background.md)"页面。

我们期待可以更新这些实践以及其详细的标准，甚至是在徽章发布之后。
这样，标准（以及徽章）可以有一年的标识时间，并会在一年或两年之后失效。
我们期待更新信息会非常容易，这样相对短的徽章生命周期就不会成为屏障。
我们计划添加此信息并维护其徽章。

*非常*欢迎通过[在 GitHub 网站上发布问题(issues)或提交申请(pull requests)](https://github.com/linuxfoundation/cii-best-practices-badge)来向我们反馈。
这里还有一个[讨论一般性话题的邮件列表](https://lists.coreinfrastructure.org/mailman/listinfo/cii-badges)

以下是当前的标准，以及如何获取更多的信息。
在此文档中的关键词“必须”，“一定不能”，”应当“，“不应当”，和“可以”的解释依据
[RFC 2119](https://tools.ietf.org/html/rfc2119).
增加了附加术语“建议的”，定义如下：

- 术语“必须的”是绝对要求做的，而“一定不能”是表示绝对禁止。
- 术语“应当“指一个标准应被实现，但是如果存在有效的原因，则在特定的情景下可以不这样去做。在选择不同的方式之前，必须考虑，理解，并谨慎权衡所有相牵连的方方面面。
- “建议的“用来作为”应当“的替代术语，此术语用于必须考虑此标准，但不这样做的有效原因比“应当”更为普遍的时候。
- 作为一个标准而言，通常来说都会被置于“应当”或“建议"的状态，因为也许此标准实现起来很困难，或者花费的成本很高。
- 术语“可以”提供了完成某工作的一种方式，例如，澄清所描述的实现方法是可接受的。
- 要获得徽章，须要满足所有“必须”和“一定不能”的标准。
满足所有"应当“的标准，或者如果无法实现此标准，必须将其理论依据记录在文档中。
而所有“建议的”标准必须被考虑（评估为符合或不符合）。
在某些情况下，也许需要添加 URL 作为此标准的解释说明的一部分。
- 文字“（未来的标准）”标记了那些当前并不需要的标准，但这些标准可能在将来会是必须的。

我们假设您已经熟悉了软件开发方式并正在运行着一个 FLOSS 项目；
如果没有，参见介绍材料如
[*Producing Open Source Software* by Karl Fogel](http://producingoss.com/).


## 当前标准：FLOSS 的最佳实践

以下为当前的标准。请注意：

* 方括号内的文字是标准的短命名。
* 在少数情况下理论依据也被包含在内。
* 我们期待另外有少数其他字段来说明项目名称，项目描述，项目 URL，软件仓库 URL （也许与项目 URL 相同），和许可证。
* 在一些情况下，可以并允许使用 N/A（“不适用”）。

如果此项目遵循标准约定，并架设在支持可被接受的 API 的网站上（如 GitHub），我们则会尝试自动测试并填写相关信息。

### 基础部分

*项目网站*

- <a name="homepage_url"></a>项目“必须“存在一个具有稳定 URL 的公开网站。（徽章程序强制需求此 URL 来创建徽章条目） <sup>[<a href="#homepage_url">homepage_url</a>]</sup>

*基本项目网站内容*

- <a name="description_good"></a>项目网站“必须“简洁地描述软件是用来做什么的（软件解决了什么问题？）。
  “必须“使用潜在用户所能理解的语言（例如，最少限度使用专业术语）。
  <sup>[<a href="#description_good">description_good</a>]</sup>
- <a name="interact"></a>项目网站“必须“提供以下内容：
  - 如何获取软件，
  - 如何进行反馈（如 bug 报告或改进措施），
  - 如何向软件做贡献。
  <sup>[<a href="#interact">interact</a>]</sup>
- <a name="contribution"></a>如何进行贡献的相关信息“必须“给出针对贡献流程的解释（例如，是否使用提交申请(pull requests)？）
  除非另外注明，我们假设<a href="https://guides.github.com/activities/contributing-to-open-source/">在 GitHub 上的项目使用问题(issues) 和 提交申请(pull request)</a>
  <sup>[<a href="#contribution">contribution</a>]</sup>
- <a name="contribution_requirements"></a>如何贡献的相关信息“应当“包含可接受的贡献的具体要求（例如，关于所要求的一切代码规范的参考信息）。
  <sup>[<a href="#contribution_requirements">contribution_requirements</a>]</sup>

*FLOSS 许可证*

- <a name="floss_license"></a> 软件“必须”使用 FLOSS 许可证。
  FLOSS 的软件发布方式要满足：
  [开放源代码定义](https://opensource.org/osd-annotated) 或
  [自由软件定义](http://www.gnu.org/philosophy/free-sw.en.html).
  这些许可证包括例如
  [CC0](http://creativecommons.org/publicdomain/zero/1.0/)、
  [MIT](https://opensource.org/licenses/MIT)、
  [BSD 2-clause](https://opensource.org/licenses/BSD-2-Clause)、
  [BSD 3-clause revised](https://opensource.org/licenses/BSD-3-Clause)、
  [Apache 2.0](https://opensource.org/licenses/Apache-2.0)、
  [Lesser GNU General Public License (LGPL)](https://opensource.org/licenses/lgpl-license) (所有版本)、
  以及 [GNU General Public License (GPL)](https://opensource.org/licenses/gpl-license) (所有版本)。
  基于我们的目的，这些许可证“必须”是被：
    - [开放源代码促进会(OSI)所批准的许可证](https://opensource.org/licenses)，或
    - [自由软件基金会(FSF)所批准的自由许可证](https://www.gnu.org/licenses/license-list.html)，或
    - [可被 Debian main 接受的自由许可证](https://www.debian.org/legal/licenses/)，或
    - [Fedora 所定义的“good”的许可证](https://fedoraproject.org/wiki/Licensing:Main?rd=Licensing).
  <sup>[<a href="#floss_license">floss_license</a>]</sup>
- <a name="floss_license_osi"></a> “建议”使用所有被[开放源代码促进会 (OSI) 所批准的](https://opensource.org/licenses)许可证。
  OSI 使用严格的证书批准流程来决定哪些证书是 OSS 的。
  <sup>[<a href="#floss_license_osi">floss_license_osi</a>]</sup>
- <a name="license_location"></a>项目“必须”将许可证发布在标准的位置（例如，最上层的名为 LICENSE 或 COPYING 的文件）。
  证书文件名“可以“加上扩展名，诸如“.txt“或“.md“
  <sup>[<a href="#license_location">license_location</a>]</sup>
- 软件也“可以”以其他方式使用许可证（例如，“GPLv2 或私有许可“也是可接受的）。
- *理论依据*: 这些标准是为 FLOSS 项目而设计的，所以我们要确保仅在 FLOSS 项目上使用。
  一些项目被错误的认为是 FLOSS ，然而它们并不是（例如，项目没有任何许可证，这种情况下将默认遵循本国的法律系统，或者这些项目也许会使用非 FLOSS 许可证）。
  非普遍性许可证可能会导致 FLOSS 项目产生长远的问题，而且会在使用工具处理方面更加困难。
  我们期待发布[更高级的徽章](./other.md)，这样可以带来更高的标准
  (例如，*"必须"* 使用OSI 所批准的证书发布软件）。

*文档*

- <a name="documentation_basics"></a>项目“必须”以某种媒介（如文本或视频）为软件提供基本的文档。文档中应包括：
  - 如何安装，
  - 如何启动，
  - 如何使用（可以带有实例教程），以及
  - 如何安全地使用软件（例如，要做什么，不要做什么），前提是这对软件来说是个适当的话题。 

  安全相关的文档不必太长。
  <sup>[<a href="#documentation_basics">documentation_basics</a>]</sup>
- <a name="documentation_interface"></a>项目“必须”包含描述其界面的参考文档。
  <sup>[<a href="#documentation_interface">documentation_interface</a>]</sup>
- 项目“可以”使用超文本链接指向非项目的材料作为其文档。


*其他*

- <a name="sites_https"></a>项目网站（网站，软件仓库，以及下载链接）“必须”支持使用 TLS 的 HTTPS 连接方式。你可以从 [Let's Encrypt](https://letsencrypt.org/) 免费获得证书。
  <sup>[<a href="#sites_https">sites_https</a>]</sup>
- <a name="discussion"></a>项目“必须”具有一个或多个讨论的机制（包括计划变更和问题），此机制要求：
    - 可搜索，
    - 允许消息和主题通过 URL 访问，
    - 允许新人参与某些讨论，以及
    - 不须要在客户端安装私有软件。

  可接受的机制包括例如
  GitHub 问题(issue) 和 提交申请(pull request)的讨论、Bugzilla、Mantis，以及 Trac。
  异步讨论机制（如 IRC）是可接受的，前提是要满足如下标准；确保具有可通过 URL 访问的存档机制。
  我们不鼓励使用私有的 Javascript，但却是允许的。
  <sup>[<a href="#discussion">discussion</a>]</sup>
- <a name="english"></a>项目“应当”包括英文文档，并且可以接受英文的 缺陷(bug)报告和评论。
  英语是当前计算机技术领域的<a
  href="https://en.wikipedia.org/wiki/Lingua_franca">通用语</a>；支持英语可以在全世界范围内增加不同的潜在开发者和评论者。
  项目是能够满足此条件的 —— 即便其核心开发者所使用的主要语言不是英语。
  <sup>[<a href="#english">english</a>]</sup>

### Change control

*Public version-controlled source repository*

- <a name="repo_public"></a>The project MUST have a version-controlled
  source repository that is publicly readable and has a URL.
  The URL MAY be the same as the project URL.
  The project MAY use private (non-public) branches in specific cases while the
  change is not publicly released
  (e.g., for fixing a vulnerability before it is revealed to the public).
   <sup>[<a href="#repo_public">repo_public</a>]</sup>
- <a name="repo_track"></a>The source repository MUST track what changes
  were made, who made the changes, and when the changes were made.
  <sup>[<a href="#repo_track">repo_track</a>]</sup>
- <a name="repo_interim"></a>To enable collaborative review,
  the project's source repository MUST include interim
  versions for review between releases;
  it MUST NOT include only final releases.
  Projects MAY choose to omit specific interim versions
  from their public source repositories
  (e.g., ones that fix specific non-public security vulnerabilities,
  may never be publicly released, or include material that cannot
  be legally posted and are not in the final release).
  <sup>[<a href="#repo_interim">repo_interim</a>]</sup>
- <a name="repo_distributed"></a>It is SUGGESTED that common distributed
  version control software be used (e.g., git).
  Git is not specifically required and projects can use centralized version
  control software (such as subversion).
  <sup>[<a href="#repo_distributed">repo_distributed</a>]</sup>

*Version numbering*

- <a name="version_unique"></a>The project MUST have a unique version number
  for each release intended to be used by users.
  <sup>[<a href="#version_unique">version_unique</a>]</sup>
- <a name="version_semver"></a>It is SUGGESTED that the
  [Semantic Versioning (SemVer) format](http://semver.org) be used for releases.
  <sup>[<a href="#version_semver">version_semver</a>]</sup>
- Commit IDs (or similar) MAY be used as version numbers.
  They are unique, but note that these can cause problems for users as they may
  not be able to determine  whether or not they're up-to-date.
- <a name="version_tags"></a>It is SUGGESTED that projects identify each
  release within their version control system.
  For example, it is SUGGESTED that those using git identify each release
  using git tags.
  <sup>[<a href="#version_tags">version_tags</a>]</sup>

*Release notes (ChangeLog)*

- <a name="release_notes"></a><a name="changelog"></a>The project
  MUST provide, in each release, release notes that are
  a human-readable *summary* of major changes in that release.
  The release notes MUST NOT be the output of a version control log
  (e.g., the "git log" command results are not release notes).
  <sup>[<a href="#release_notes">release_notes</a>]</sup>
- <a name="release_notes_vulns"></a><a name="changelog_vulns"></a>The
  release notes MUST identify every publicly known vulnerability
  that is fixed in each new release.
  <sup>[<a href="#release_notes_vulns">release_notes_vulns</a>]</sup>
- The release notes MAY be implemented in a variety of ways.
  Many projects provide them in a file named "NEWS", "CHANGELOG",
  or "ChangeLog", optionally with extensions such as ".txt", ".md", or ".html".
  Historically the term "change log" meant a log of *every* change,
  but to meet these criteria what is needed is a human-readable summary.
  The release notes MAY instead be provided by
  version control system mechanisms such as the
  [GitHub Releases workflow](https://github.com/blog/1547-release-your-software).
- *Rationale*: Release notes are important because they help users
  decide whether or not they will want to update, and what the impact would be
  (e.g., if the new release fixes vulnerabilities).

### Reporting

*Bug reporting process*

- <a name="report_process"></a>The project MUST provide a process for users
  to submit bug reports (e.g., using an issue tracker or a mailing list).
  <sup>[<a href="#report_process">report_process</a>]</sup>
- <a name="report_tracker"></a>The project SHOULD use an issue
  tracker for tracking individual issues.
  <sup>[<a href="#report_tracker">report_tracker</a>]</sup>
- <a name="report_responses"></a>The project MUST acknowledge a majority of
  bug reports submitted in the last 2-12 months (inclusive);
  the response need not include a fix.
  <sup>[<a href="#report_responses">report_responses</a>]</sup>
- <a name="enhancement_responses"></a>The project SHOULD respond to most
  enhancement requests in the last 2-12 months (inclusive).
  The project MAY choose not to respond.
  <sup>[<a href="#enhancement_responses">enhancement_responses</a>]</sup>
- <a name="report_archive"></a>The project MUST have a publicly available
  archive for reports and responses for later searching.
  <sup>[<a href="#report_archive">report_archive</a>]</sup>

*Vulnerability reporting process*

- <a name="vulnerability_report_process"></a>The project MUST publish the
  process for reporting vulnerabilities on the project site.
  E.g., a clearly designated mailing address on <https://PROJECTSITE/security>,
  often in the form security@example.org.
  This MAY be the same as its bug reporting process.
  <sup>[<a href="#vulnerability_report_process">vulnerability_report_process</a>]</sup>
- <a name="vulnerability_report_private"></a>If private vulnerability reports
  are supported, the project MUST include how to send the information in a
  way that is kept private.
  E.g., a private defect report submitted on the web using TLS or an email
  encrypted using OpenPGP.
  If private vulnerability reports are not supported this criterion
  is automatically met.
  <sup>[<a href="#vulnerability_report_private">vulnerability_report_private</a>]</sup>
- <a name="vulnerability_report_response"></a>The project's
  initial response time for any vulnerability report received
  in the last 6 months MUST be less than or equal to 14 days.
  <sup>[<a href="#vulnerability_report_response">vulnerability_report_response</a>]</sup>

### Quality

*Working build system*

- <a name="build"></a>If the software requires building for use,
  the project MUST provide a working build system that can automatically
  rebuild the software from source code.
  A build system determines what actions need to occur to rebuild the software
  (and in what order), and then performs those steps.
  <sup>[<a href="#build">build</a>]</sup>
- <a name="build_common_tools"></a>It is SUGGESTED that common
  tools be used for building the software.
  For example, Maven, Ant, cmake, the autotools, make, or rake.
  <sup>[<a href="#build_common_tools">build_common_tools</a>]</sup>
- <a name="build_floss_tools"></a> The project SHOULD be buildable
  using only FLOSS tools.
  <sup>[<a href="#build_floss_tools">build_floss_tools</a>]</sup>

*Automated test suite*

- <a name="test"></a>The project MUST have at least one automated test suite
  that is publicly released as FLOSS
  (this test suite may be maintained as a separate FLOSS project)."
  <sup>[<a href="#test">test</a>]</sup>
- <a name="test_invocation"></a>A test suite SHOULD be invocable in
  a standard way for that language.
  For example,  "make check", "mvn test", or "rake test".
  <sup>[<a href="#test_invocation">test_invocation</a>]</sup>
- <a name="test_most"></a>It is SUGGESTED that the test suite cover most
  (or ideally all) the code branches, input fields, and functionality.
  <sup>[<a href="#test_most">test_most</a>]</sup>
- <a name="test_continuous_integration"></a>It is SUGGESTED that the project
  implement continuous integration
  (where new or changed code is frequently integrated into a central code
  repository and automated tests are run on the result).
  <sup>[<a href="#test_continuous_integration">test_continuous_integration</a>]</sup>
- The project MAY have multiple automated test suites
  (e.g., one that runs quickly, vs. another that is more thorough but
  requires special equipment).
- *Rationale*: Automated test suites immediately help detect a
  variety of problems.  A large test suite can find more problems,
  but even a small test suite can detect problems and
  provide a framework to build on.


*New functionality testing*

- <a name="test_policy"></a>The project MUST have a general policy
  (formal or not) that as major new functionality is added,
  tests of that functionality SHOULD be added to an automated test suite.
  <sup>[<a href="#test_policy">test_policy</a>]</sup>
- <a name="tests_are_added"></a>The project MUST have evidence that such
  tests are being added in the most recent major changes to the project.
  Major functionality would typically be mentioned in the ChangeLog.
  (Perfection is not required, merely evidence that tests are
  typically being added in practice.)
  <sup>[<a href="#tests_are_added">tests_are_added</a>]</sup>
- <a name="tests_documented_added"></a>It is SUGGESTED that this policy on
  adding tests be *documented* in the instructions for change proposals.
  However, even an informal rule is acceptable as long as the tests
  are being added in practice.
  <sup>[<a href="#tests_documented_added">tests_documented_added</a>]</sup>

*Warning flags*

- <a name="warnings"></a>The project MUST enable one or more compiler
  warning flags, a "safe" language mode, or use a separate "linter" tool to
  look for code quality errors or common simple mistakes,
  if there is at least one FLOSS tool that can implement this criterion
  in the selected language.
  Examples of compiler warning flags include gcc/clang "-Wall".
  Examples of a "safe" language mode include Javascript "use strict"
  and perl5's "use warnings".
  A separate "linter" tool is simply a tool that examines the source
  code to look for code quality errors or common simple mistakes.
  <sup>[<a href="#warnings">warnings</a>]</sup>
- <a name="warnings_fixed"></a>The project MUST address warnings.
  The project should fix warnings or mark them in the source
  code as false positives.
  Ideally there would be no warnings, but a project MAY accept some warnings
  (typically less than 1 warning per 100 lines or less than 10 warnings).
  <sup>[<a href="#warnings_fixed">warnings_fixed</a>]</sup>
- <a name="warnings_strict"></a>It is SUGGESTED that projects be
  maximally strict with warnings, but this is not always practical.
  <sup>[<a href="#warnings_strict">warnings_strict</a>]</sup>

### Security

*Secure development knowledge*

- <a name="know_secure_design"></a>The project MUST have at least one
  primary developer who knows how to design secure software.
  This requires understanding the following design principles,
  including the 8 principles from
  [Saltzer and Schroeder](http://web.mit.edu/Saltzer/www/publications/protection/):
    - economy of mechanism (keep the design as simple and small as practical,
      e.g., by adopting sweeping simplifications)
    - fail-safe defaults (access decisions should deny by default, and
      projects' installation should be secure by default)
    - complete mediation (every access that might be limited must be
      checked for authority and be non-bypassable)
    - open design (security mechanisms should not depend on attacker
      ignorance of its design, but instead on more easily protected and
      changed information like keys and passwords)
    - separation of privilege (ideally, access to important objects
      should depend on more than one condition, so that defeating
      one protection system won't enable complete access.
      E.G., multi-factor authentication, such as requiring both a password
      and a hardware token, is stronger than single-factor authentication)
    - least privilege (processes should operate with the
      least privilege necessary)
    - least common mechanism (the design should minimize the mechanisms
      common to more than one user and depended on by all users,
      e.g., directories for temporary files)
    - psychological acceptability
      (the human interface must be designed for ease of use,
      designing for "least astonishment" can help)
    - limited attack surface (the attack surface, the set of the different
      points where an attacker can try to enter or extract data,
      should be limited)
    - input validation with whitelists
      (inputs should typically be checked to determine if they are valid
      before they are accepted; this validation should use whitelists
      (which only accept known-good values),
      not blacklists (which attempt to list known-bad values)).
      <sup>[<a href="#know_secure_design">know_secure_design</a>]</sup>
- <a name="know_common_errors"></a>At least one of the primary developers
  MUST know of common kinds of errors that lead to vulnerabilities in this kind
  of software, as well as at least one method
  to counter or mitigate each of them.
  Examples (depending on the type of software) include SQL injection,
  OS injection, classic buffer overflow, cross-site scripting,
  missing authentication, and missing authorization.
  See the [CWE/SANS top 25](http://cwe.mitre.org/top25/) or
  [OWASP Top 10](https://www.owasp.org/index.php/Category:OWASP_Top_Ten_Project)
  for commonly used lists.
  <sup>[<a href="#know_common_errors">know_common_errors</a>]</sup>
- A "primary developer" in a project is anyone who is familiar with
  the project's code base, is comfortable making changes to it, and is
  acknowledged as such by most other participants in the project.
  A primary developer would typically make a number of contributions
  over the past year (via code, documentation, or answering questions).
  Developers would typically be considered primary developers if they
  initiated the project (and have not left the project more than three
  years ago), have the option of receiving information on a private
  vulnerability reporting channel (if there is one), can accept commits
  on behalf of the project, or perform final releases of the project software.
  If there is only one developer, that individual is the primary developer.


*Good cryptographic practices*

*Note*: These criteria do not always apply because some software has no
need to directly use cryptographic capabilities.
A "project security mechanism" is a security mechanism provided
by the delivered project's software.

- <a name="crypto_published"></a>The project's cryptographic software MUST
  use by default
  only cryptographic protocols and algorithms that are publicly published
  and reviewed by experts.
  <sup>[<a href="#crypto_published">crypto_published</a>]</sup>
- <a name="crypto_call"></a>If the project software is an application
  or library, and its primary purpose is not to implement cryptography,
  then it SHOULD only call on software specifically designed to implement
  cryptographic functions;
  it SHOULD NOT re-implement its own.
  <sup>[<a href="#crypto_call">crypto_call</a>]</sup>
- <a name="crypto_floss"></a>All project functionality that depends
  on cryptography MUST be implementable using FLOSS.  See the
  [*Open Standards Requirement for Software* by the Open Source Initiative](https://opensource.org/osr).
  <sup>[<a href="#crypto_floss">crypto_floss</a>]</sup>
- <a name="crypto_keylength"></a>The project security mechanisms
  MUST use default keylengths that at least
  meet the NIST minimum requirements
  through the year 2030 (as stated in 2012).
  These minimum bitlengths are: symmetric key 112, factoring modulus 2048,
  discrete logarithm key 224, discrete logarithmic group 2048,
  elliptic curve 224, and hash 224 (password hashing is not covered by this
  bitlength, more information on password hashing can be found in the
  <a href="#crypto_password_storage">crypto_password_storage</a> criterion).
  See <http://www.keylength.com> for a comparison of keylength
  recommendations from various organizations.
  The software MUST be configurable so that it will reject smaller keylengths.
  The software MAY allow smaller keylengths in some configurations
  (ideally it would not, since this allows downgrade attacks,
  but shorter keylengths are sometimes necessary for interoperability).
  <sup>[<a href="#crypto_keylength">crypto_keylength</a>]</sup>
- <a name="crypto_working"></a>The default project security mechanisms MUST NOT
  depend on cryptographic algorithms that are broken
  (e.g., MD4, MD5, single DES, RC4, or Dual_EC_DRBG).
  <sup>[<a href="#crypto_working">crypto_working</a>]</sup>
- <a name="crypto_weaknesses"></a>The project security mechanisms
  SHOULD NOT by default depend on cryptographic algorithms with known
  serious weaknesses (e.g., SHA-1).
  <sup>[<a href="#crypto_weaknesses">crypto_weaknesses</a>]</sup>
- <a name="crypto_pfs"></a>The project SHOULD implement perfect forward
  secrecy for key agreement protocols so a session key derived from a set
  of long-term keys cannot be compromised if one of the long-term keys is
  compromised in the future.
  <sup>[<a href="#crypto_pfs">crypto_pfs</a>]</sup>
- <a name="crypto_password_storage"></a>If passwords are stored for
  authentication of external users, the project MUST store them as
  iterated hashes with a per-user salt by using a key stretching
  (iterated) algorithm (e.g., PBKDF2, Bcrypt or Scrypt).
  <sup>[<a href="#crypto_password_storage">crypto_password_storage</a>]</sup>
- <a name="crypto_random"></a>The project MUST generate all
  cryptographic keys and nonces
  using a cryptographically secure random number generator,
  and MUST NOT do so using generators that are not cryptographically secure.
  A cryptographically secure random number generator may be a
  hardware random number generator, or it may be
  a cryptographically secure pseudo-random number generator (CSPRNG) using
  an algorithm such as Hash_DRBG, HMAC_DRBG, CTR_DRBG, Yarrow, or Fortuna.
  <sup>[<a href="#crypto_random">crypto_random</a>]</sup>

*Secured delivery mechanism*

- <a name="delivery_mitm"></a>The project MUST provide its materials
  using a delivery mechanism that counters man-in-the-middle (MITM) attacks.
  Using https or ssh+scp is acceptable.
  An even stronger mechanism is releasing the software with digitally signed
  packages, since that mitigates attacks on the distribution system,
  but this only works if the users can be confident that the public keys
  for signatures are correct *and* if the users will
   actually check the signature.
  <sup>[<a href="#delivery_mitm">delivery_mitm</a>]</sup>

*Publicly known vulnerabilities fixed*

- <a name="vulnerabilities_fixed_60_days"></a>There MUST be no unpatched
  vulnerabilities of medium or high severity that have been *publicly* known
  for more than 60 days.
  The vulnerability must be patched and released by the project itself
  (patches may be developed elsewhere).
  A vulnerability becomes publicly known (for this purpose) once it has a
  CVE with publicly released non-paywalled information
  (reported, for example, in the
  [National Vulnerability Database](https://nvd.nist.gov/))
  or when the project has been informed *and* the information has been
  released to the public (possibly by the project).
  A vulnerability is medium to high severity if its
  [CVSS 2.0](https://nvd.nist.gov/cvss.cfm) base score is 4 or higher.
  <sup>[<a href="#vulnerabilities_fixed_60_days">vulnerabilities_fixed_60_days</a>]</sup>
- <a name="vulnerabilities_critical_fixed"></a>Projects SHOULD fix all
  critical vulnerabilities rapidly after they are reported.
  <sup>[<a href="#vulnerabilities_critical_fixed">vulnerabilities_critical_fixed</a>]</sup>
- *Note*: this means that users might be left vulnerable to all
  attackers worldwide for up to 60 days.
  This criterion is often much easier to meet than what Google recommends in
  [Rebooting responsible disclosure](https://security.googleblog.com/2010/07/rebooting-responsible-disclosure-focus.html),
  because Google recommends that the 60-day period start when the
  project is notified *even* if the report is not public.
- *Rationale*: We intentionally chose to start measurement from the time of
  public knowledge, and not from the time reported to the project,
  because this is much easier to measure and verify by those
  *outside* the project.

*Other security issues*

- <a name="no_leaked_credentials"></a>The public repositories
  MUST NOT leak a valid private credential
  (e.g., a working password or private key) that is intended to limit
  public access.
  A project MAY leak "sample" credentials for testing and
  unimportant databases, as long as they are not intended to limit
  public access.
  <sup>[<a href="#no_leaked_credentials">no_leaked_credentials</a>]</sup>

### Analysis

*Static code analysis*

- <a name="static_analysis"></a>At least one static code analysis tool
  MUST be applied to any proposed major production release of the software
  before its release, if there is at least one FLOSS tool that implements this
  criterion in the selected language.
  A static code analysis tool examines the software code
  (as source code, intermediate code, or executable)
  without executing it with specific inputs.
  For purposes of this criterion, compiler warnings and "safe"
  language modes do not count as static code analysis tools
  (these typically avoid deep analysis because speed is vital).
  Examples of such static code analysis tools include
  [cppcheck](http://cppcheck.sourceforge.net/),
  [clang static analyzer](http://clang-analyzer.llvm.org/),
  [FindBugs](http://findbugs.sourceforge.net/)
  (including [FindSecurityBugs](https://h3xstream.github.io/find-sec-bugs/)),
  [PMD](https://pmd.github.io/),
  [Brakeman](http://brakemanscanner.org/),
  [Coverity Quality Analyzer](https://scan.coverity.com/),
  and [HP Fortify Static Code Analyzer](http://www8.hp.com/au/en/software-solutions/static-code-analysis-sast/).
  Larger lists of tools can be found in places such as the
  [Wikipedia list of tools for static code analysis](https://en.wikipedia.org/wiki/List_of_tools_for_static_code_analysis),
  [OWASP information on static code analysis](https://www.owasp.org/index.php/Static_Code_Analysis),
  [NIST list of source code security analyzers](http://samate.nist.gov/index.php/Source_Code_Security_Analyzers.html),
  and [Wheeler's list of static analysis tools](http://www.dwheeler.com/essays/static-analysis-tools.html).
  The [SWAMP](https://continuousassurance.org/) is a no-cost platform
  for assessing vulnerabilities in software using a variety of tools.
  <sup>[<a href="#static_analysis">static_analysis</a>]</sup>
- <a name="static_analysis_common_vulnerabilities"></a>It is SUGGESTED
  that at least one of the static analysis tools
  used for the static_analysis criterion
  include rules or approaches to look for
  common vulnerabilities in the analyzed language or environment.
  <sup>[<a href="#static_analysis_common_vulnerabilities">static_analysis_common_vulnerabilities</a>]</sup>
- <a name="static_analysis_fixed"></a>All medium
  and high severity exploitable vulnerabilities discovered
  with static code analysis MUST be fixed
  in a timely way after they are confirmed.
  A vulnerability is medium to high severity if its
  [CVSS 2.0](https://nvd.nist.gov/cvss.cfm) is 4 or higher.
  <sup>[<a href="#static_analysis_fixed">static_analysis_fixed</a>]</sup>
- <a name="static_analysis_often"></a>It is SUGGESTED that
  static source code analysis occur on every commit or at least daily.
  <sup>[<a href="#static_analysis_often">static_analysis_often</a>]</sup>

*Dynamic analysis*

- <a name="dynamic_analysis"></a>It is SUGGESTED that at least one
  dynamic analysis tool be applied to any proposed major production
  release of the software before its release.
  A dynamic analysis tool examines the software by executing
  it with specific inputs.
  For example, the project MAY use a fuzzing tool
  (e.g., [American Fuzzy Lop](http://lcamtuf.coredump.cx/afl/))
  or a web application scanner
  (e.g., [OWASP ZAP](https://www.owasp.org/index.php/OWASP_Zed_Attack_Proxy_Project)
  or [w3af.org](http://w3af.org/)).
  For purposes of this criterion the dynamic analysis tool needs to vary
  the inputs in some way to look for various kinds of problems *or*
  be an automated test suite with at least 80% branch coverage.
  The [Wikipedia page on dynamic analysis](https://en.wikipedia.org/wiki/Dynamic_program_analysis)
  and the [OWASP page on fuzzing](https://www.owasp.org/index.php/Fuzzing)
  identify some dynamic analysis tools.
  <sup>[<a href="#dynamic_analysis">dynamic_analysis</a>]</sup>
- <a name="dynamic_analysis_unsafe"></a>It is SUGGESTED that if the
  software is application-level software
  written using a memory-unsafe language (e.g., C or C++) then at
  least one dynamic tool (e.g., a fuzzer or web application scanner)
  be routinely used with a mechanism to detect memory safety problems
  such as buffer overwrites.
  Examples of mechanisms to detect memory safety problems
  include Address Sanitizer (ASAN) and
  [valgrind](http://valgrind.org/).
  Widespread assertions would also work.
  If the software is not application-level,
  or is not in a memory-unsafe language, then this criterion is
  automatically met.
  <sup>[<a href="#dynamic_analysis_unsafe">dynamic_analysis_unsafe</a>]</sup>
- <a name="dynamic_analysis_enable_assertions"></a>It is SUGGESTED that
  the software include many run-time assertions that are
  checked during dynamic analysis.
  <sup>[<a href="#dynamic_analysis_enable_assertions">dynamic_analysis_enable_assertions</a>]</sup>
- The analysis tool(s) MAY be focused on looking for security
  vulnerabilities, but this is not required.
- <a name="dynamic_analysis_fixed"></a>All medium and high
  severity exploitable vulnerabilities discovered with dynamic
  code analysis MUST be fixed in a timely way after they are confirmed.
  A vulnerability is medium to high severity if its
  [CVSS 2.0](https://nvd.nist.gov/cvss.cfm)
  base score is 4 or higher.
  <sup>[<a href="#dynamic_analysis_fixed">dynamic_analysis_fixed</a>]</sup>
- *Rationale*: Static source code analysis and dynamic
  analysis tend to find different kinds of defects
  (including defects that lead to vulnerabilities),
  so combining them is more likely to be effective.

### Future

These 'future' criteria are criteria we intend to add in the near future.

- <a name="installation_common"></a>(Future criterion) The project SHOULD
  provide a way to easily install and uninstall the software using a
  commonly-used convention.
  Examples include using a package manager (at
  the system or language level), "make install/uninstall" (supporting
  DESTDIR), a container in a standard format,
  or a virtual machine image in a standard format.
  The installation and uninstallation process (e.g., its packaging)
  MAY be implemented by a third party as long as it is FLOSS.
  <sup>[<a href="#installation_common">installation_common</a>]</sup>
- <a name="build_reproducible"></a>(Future criterion)
  It is SUGGESTED that
  the project have a [reproducible build](https://reproducible-builds.org/).
  With reproducible builds, multiple parties can independently redo the
  process of generating information from source files and get exactly
  the same result.
  The [reproducible builds project has documentation on how to do this](https://reproducible-builds.org/docs/).
  This criterion does not apply if no building occurs
  (e.g., scripting languages where the source code
  is used directly instead of being compiled).
  <sup>[<a href="#build_reproducible">build_reproducible</a>]</sup>
  *Rationale*: If a project needs to be built but there is no working
  build system, then potential co-developers will not be able to easily
  contribute and many security analysis tools will be ineffective.
  Criteria for a working build system are not applicable if there is
  no need to build anything for use.
- <a name="crypto_used_network"></a>(Future criterion)
   The project SHOULD NOT use
   unencrypted network communication protocols (such as HTTP and telnet)
   if there an encrypted equivalent (e.g., HTTPS/TLS and SSH),
   unless the user specifically requests or configures it.
  <sup>[<a href="#crypto_used_network">crypto_used_network</a>]</sup>
- <a name="crypto_tls12"></a>(Future criterion) The project SHOULD,
   if it supports TLS, support at least TLS version 1.2.
   Note that the predecessor of TLS was called SSL.
  <sup>[<a href="#crypto_tls12">crypto_tls12</a>]</sup>
- <a name="crypto_certificate_verification"></a>(Future criterion)
   The project MUST,
   if it supports TLS, perform TLS certificate verification by default
   when using TLS, including on subresources.
   Note that incorrect TLS certificate verification is a common mistake.
   For more information, see
   ["The Most Dangerous Code in the World: Validating SSL Certificates in Non-Browser Software" by Martin Georgiev et al.](http://crypto.stanford.edu/~dabo/pubs/abstracts/ssl-client-bugs.html) and
   ["Do you trust this application?" by Michael Catanzaro](https://blogs.gnome.org/mcatanzaro/2016/03/12/do-you-trust-this-application/).
   <sup>[<a href="#crypto_certificate_verification">crypto_certificate_verification</a>]</sup>
- <a name="crypto_verification_private"></a>(Future criterion)
   The project SHOULD,
   if it supports TLS, perform certificate verification
   before sending HTTP headers with private information
   (such as secure cookies).
   <sup>[<a href="#crypto_verification_private">crypto_verification_private</a>]</sup>
- <a name="hardened_site"></a>(Future criterion)
  It is SUGGESTED that the project website, repository (if accessible
  via the web), and download site (if separate) include key hardening headers
  with nonpermissive values.
  Note that GitHub is known to meet this.
  Sites such as https://securityheaders.io/ can quickly check this.
  The key hardening headers are:
  Content Security Policy (CSP), HTTP Strict Transport Security
  (HSTS), X-Content-Type-Options (as "nosniff"), X-Frame-Options,
  and X-XSS-Protection.
  <sup>[<a href="#hardened_site">hardened_site</a>]</sup>
- <a name="hardening"></a>
  (Future criterion)
  It is SUGGESTED that hardening mechanisms be used so software defects
  are less likely to result in security vulnerabilities.
  Hardening mechanisms may include
  HTTP headers like Content Security Policy (CSP),
  compiler flags to mitigate attacks
  (such as -fstack-protector), or compiler flags to
  eliminate undefined behavior,
  For our purposes least privilege is not considered a hardening
  mechanism (least privilege is important, but separate).
  <sup>[<a href="#hardening">hardening</a>]</sup>

## Non-criteria

We plan to *not* require any specific products or services.
In particular, we plan to *not* require
proprietary tools or services,
since many [free software](https://www.gnu.org/philosophy/free-sw.en.html)
developers would reject such criteria.
Therefore, we will intentionally *not* require git or GitHub.
We will also not require or forbid any particular programming language
(though for some programming languages we may be able to make
some recommendations).
This also means that as new tools and capabilities become available,
projects can quickly switch to them without failing to meet any criteria.
However, the criteria will sometimes identify
common methods or ways of doing something
(especially if they are FLOSS) since that information
can help people understand and meet the criteria.
We do plan to create an "easy on-ramp" for projects using git on GitHub,
since that is a common case.
We would welcome good patches that help provide an "easy on-ramp" for
projects on other repository platforms.

We do not plan to require active user discussion within a project.
Some highly mature projects rarely change and thus may have little activity.
We *do*, however, require that the project be responsive
if vulnerabilities are reported to the project (see above).

## Uniquely identifying a project

One challenge is uniquely identifying a project.
Our rails application gives a unique id to each new project, so
we can certainly use that id to identify projects.
However, that doesn't help people who searching for the project
and do not already know that id.

The *real* name of a project, for our purposes, is the
project "front page" URL and/or the URL for its repository.
Most projects have a human-readable name, but these names are not enough.
The same human-readable name can be used for many different projects
(including project forks), and the same project may go by many different names.
In many cases it will be useful to point to other names for the project
(e.g., the source package name in Debian, the package name in some
language-specific repository, or its name in OpenHub).

In the future we may try to check more carefully that a user can
legitimately represent a project.
For the moment, we primarily focus on checking if GitHub repositories
are involved; there are ways to do this for other situations if that
becomes important.
We expect that users will *not* be able to edit the URL in most cases,
since if they could, they might fool people into thinking they controlled
a project that they did not.
That said, creating a bogus row entry does not really help someone very
much; what matters is the id used by the project when it refers to its
badge, and the project determines that.

Thus, a badge would have its URL as its name, year range, and level/name
(once there is more than one).

We will probably implement some search mechanisms so that people can
enter common names and find projects.


## Why have criteria?

The paper [Open badges for education: what are the implications at the
intersection of open systems and badging?](http://www.researchinlearningtechnology.net/index.php/rlt/article/view/23563)
identifies three general reasons for badging systems (all are valid for this):

1. Badges as a motivator of behavior.  We hope that by identifying
   best practices, we'll encourage projects to implement those
   best practices if they do not do them already.
2. Badges as a pedagogical tool.  Some projects may not be aware
   of some of the best practices applied by others,
   or how they can be practically applied.
   The badge will help them become aware of them and ways to implement them.
3. Badges as a signifier or credential.
   Potential users want to use projects that are applying best
   practices to consistently produce good results; badges make it easy
   for projects to signify that they are following best practices,
   and make it easy for users to see which projects are doing so.

We have chosen to use self-certification, because this makes it
possible for a large number of projects (even small ones) to
participate.  There's a risk that projects may make false claims,
but we think the risk is small, and users can check the claims for themselves.


## Improving the criteria

We are hoping to get good suggestions and feedback from the public;
please contribute!

We currently plan to launch with a single badge level (once it is ready).
There may eventually be multiple levels (bronze, silver, gold) or
other badges (with a prerequisite) later.
One area we have often discussed is whether or not to require
continuous integration in this set of criteria;
if it is not, it is expected to be required at higher levels.
See [other](./other.md) for more information.

You may also want to see the "[background](./background.md)" file
for more information about these criteria,
and the "[implementation](./implementation.md)" notes
about the BadgeApp application.
