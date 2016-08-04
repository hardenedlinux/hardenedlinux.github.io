---
layout: post
title:  "自由/开源软件(FLOSS)的最佳实践标准（第一部分）"
date: 2016-08-04
summary: 本文提供一系列自由/开源软件 (FLOSS) 项目的最佳实践方法。参照这些最佳实践标准的项目可以进行自认证, 以获得核心基础设施促进会(CII)徽章。
categories:
---

**原文：[Best Practices Criteria for Free/Libre and Open Source Software (FLOSS) (version 0.8.0)](https://github.com/linuxfoundation/cii-best-practices-badge/blob/master/doc/criteria.md)**

作者（组织）：Core Infrastructure Initiative Best Practices Badge

Github 提交者：david-a-wheeler, altonius, kfogel, dankohn, NilsEnevoldsen

译者：wnereiz

许可证：MIT 或 CC-BY-3.0+

> wnereiz: 自由/开源软件(FLOSS)的最佳实践标准的重要意义不仅仅在于获得认证，它具有在自由/开源生态领域更加广泛的指导性作用。由于篇幅较长，所以计划将此文章分阶段翻译并最终汇总。本文为第一部分。

# 译者注

# 自由/开源软件(FLOSS)的最佳实践标准 (版本 0.8.0)


## 简介

本文提供一系列自由/开源软件 (FLOSS) 项目的最佳实践方法。
参照这些最佳实践标准的项目可以进行自认证, 以获得核心基础设施促进会(CII)徽章。
要做到这点无需任何费用，您的项目可以使用 Web 应用（BadgeApp) 来证明是如何符合这些实践标准的以及其详细状况。

任何实践标准都无法保证软件从不出现缺陷或漏洞；甚至形式化方法在规范或假设错误的情况下都会失败。
也没有任何实践可以保证一个项目可以支撑健康和运作良好的社区。
然而，遵循最佳实践可以帮助改进项目的成果。
例如，一些实践标准要求在发布之前进行多人审查，这有助于找到其他情况下难以发现的漏洞，同时可以帮助项目参与者建立信任，以及满足不同组织的开发者之间的重复合作的要求。

这些最佳实践标准可以用来：

1. 鼓励项目遵循最佳实践。
2. 帮助新的项目找到那些它们要遵循的最佳实践，还有
3. 帮助用户了解哪些项目遵循了最佳实践（这样用户可以更倾向于选择此类项目）。

我们当前集中精力识别那些现已良好运行的典型项目所遵从的最佳实践标准。
我们也在关注其他的实践标准，今后会创建[更高级的徽章](https://github.com/linuxfoundation/cii-best-practices-badge/blob/master/doc/other.md)。
最佳实践标准，以及为其定义的更详细的标准，受到各种信息来源的启发。
更多信息请参见单独的"[背景](https://github.com/linuxfoundation/cii-best-practices-badge/blob/master/doc/background.md)"页面。

我们期待可以更新这些实践内容以及其详细的标准，乃至在徽章发布之后。
这样，标准（以及徽章）可以有一年的标识时间，并会在一年或两年之后失效。
我们期待能够很容易更新这些信息，这样建立相对短的徽章生命周期就不会成为障碍。
我们计划添加新的标准，但是会将其标注为“未来的”，这样这些项目可以添加此信息并维护其徽章。

*非常*欢迎通过[在 GitHub 网站上发布问题(issues)或提交申请(pull requests)](https://github.com/linuxfoundation/cii-best-practices-badge)来向我们反馈。
这里还有一个[讨论一般性话题的邮件列表](https://lists.coreinfrastructure.org/mailman/listinfo/cii-badges)

以下是当前的标准，以及如何获取更多的信息。
在此文档中的关键词“必须”，“一定不能”，”应当“，“不应当”，和“可以”的解释依据
[RFC 2119](https://tools.ietf.org/html/rfc2119).
本文增加了附加术语“建议的”，定义如下：

- 术语“必须”是绝对要求做的，而“一定不能”是表示绝对禁止。
- 术语“应当“指一个标准应该被实现，但是如果存在有效的原因，则在特定的情景下可以不这样去做。在选择不同的方式之前，必须考虑，理解，并谨慎权衡所有相牵连的方方面面。
- “建议的“用来作为”应当“的替代术语，此术语用于必须考虑此标准，但不这样做的有效原因比“应当”更为普遍的时候。
- 作为一个标准而言，通常来说都会被置于“应当”或“建议的"的状态，因为也许此标准实现起来很困难，或者花费的成本很高。
- 术语“可以”提供了完成某工作的一种方式，例如，澄清所描述的实现方法是可接受的。
- 要获得徽章，须要满足所有“必须”和“一定不能”的标准。
满足所有"应当“的标准，或者如果无法实现此标准，必须将其理论依据记录在文档中。
而所有“建议的”标准则必须考虑（评估为符合或不符合）。
在某些情况下，也许需要添加 URL 作为此标准的解释说明的一部分。
- 文字“（未来的标准）”标记了那些当前并不需要的标准，但在将来可能会是必须的。

我们假设您已经熟悉了软件开发方式并正在运行着一个 FLOSS 项目；
如果没有，参见介绍材料如
[*Producing Open Source Software* by Karl Fogel](http://producingoss.com/).


## 当前标准：FLOSS 的最佳实践

以下为当前的标准。请注意：

* 方括号内的文字是标准的短命名。
* 在少数情况下也包含了理论依据。
* 我们期待另外包含少数其他字段来说明项目的名称，项目描述，项目 URL，软件仓库 URL （也许与项目 URL 相同），和许可证。
* 在一些情况下，可以并允许使用 N/A（“不适用”）。

如果此项目遵循标准约定，并架设在具有可被支持的 API 的网站上（如 GitHub），我们则会尝试自动测试并填写相关信息。

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
  除非另外注明，我们假设<a href="https://guides.github.com/activities/contributing-to-open-source/">在 GitHub 上的项目使用问题(issues)和提交申请(pull request)</a>
  <sup>[<a href="#contribution">contribution</a>]</sup>
- <a name="contribution_requirements"></a>如何贡献的相关信息“应当“包含可接受贡献的具体要求（例如，关于所要求的一切代码规范的参考信息）。
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
- 软件也“可以”以其他方式使用许可证（例如，“GPLv2 或私有许可“也是可以的）。
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
- <a name="discussion"></a>项目“必须”具有一个或多个讨论的机制（包括计划中的更改和问题），此机制要求：
    - 可搜索，
    - 允许消息和主题通过 URL 访问，
    - 允许新人参与某些讨论，以及
    - 不须要在客户端安装私有软件。

  可接受的讨论机制包括例如
  GitHub 问题(issue)和提交申请(pull request)的讨论、Bugzilla、Mantis，以及 Trac。
  可以使用异步讨论机制（如 IRC），前提是要满足如下标准；确保具有可通过 URL 访问的存档机制。
  我们不鼓励使用私有的 Javascript，但却是允许的。
  <sup>[<a href="#discussion">discussion</a>]</sup>
- <a name="english"></a>项目“应当”包括英文文档，并且可以接受英文的 缺陷(bug)报告和评论。
  英语是当前计算机技术领域的<a
  href="https://en.wikipedia.org/wiki/Lingua_franca">通用语</a>；支持英语可以在全世界范围内增加不同的潜在开发者和评论者。
  作为一个项目是能够满足此条件的 —— 即便其核心开发者所使用的主要语言不是英语。
  <sup>[<a href="#english">english</a>]</sup>
