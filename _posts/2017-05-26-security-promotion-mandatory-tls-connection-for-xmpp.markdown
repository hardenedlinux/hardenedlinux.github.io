---
layout: post
title:  "Security Promotion - Mandatory TLS Connection for XMPP"
date:   2017-05-26 14:56:25
summary: Although, according to RFC7590 "Use of Transport Layer Security (TLS) in the Extensible Messaging and Presence Protocol (XMPP)", TLS was recommended for XMPP connection. But it is not mandatory. Despite the consensus to switch XMPP on mandatory encryption reached by XMPP communities on 2014, there are still some XMPP service providers support non-encrypted connection as a fallback along with TLS.

categories: security-promotion
---

### Security Promotion: Mandatory TLS Connection for XMPP

Although, according to [RFC7590](https://tools.ietf.org/html/rfc7590)
"Use of Transport Layer Security (TLS) in the Extensible Messaging and
Presence Protocol (XMPP)", TLS was recommended for XMPP connection. But
it is not mandatory. Despite the consensus to [switch XMPP on mandatory encryption](https://lwn.net/Articles/599647/)
reached by XMPP communities on 2014, there are still some XMPP service
providers support non-encrypted connection as a fallback along with TLS.

This will probably lead to some security risks. For example, in some
cases, certain client will try to connect server automatically without
any encryption when they failed to enable TLS. But user is usually not
noticed, or even have no idea about what is TLS. And next, all the
messages will deliverd by cleartext through the network.

xmpp.jp once deal with the connection like that. We tried to contact 
their administrator on the early of this year, to require switch to 
the mandatory TLS. We got the reply message on 16 Mar 2017. They
promised to change settings at next maintenance.

On 29th Apr, we found there is a service outage due to maintenance. 
But it still allow non-TLS login and communication after service coming
back. We contact them again with email and got feedback message
immediately, in which they explained that the switch will happened in
one week as the plan.

On 08 May 2017, we experienced another round of service down. Then we
found xmpp.jp had already switched to mandatory TLS connection, which
was confirmed by testing with Psi+. At the same time, the administrator
sent us a message - "done.", and we replied to show our thanks.

By that time, the task has been finished. It is really a smooth and 
pleasure communitcation, although the duration of whole process is a
little bit long.

We will engage in more promotion actions in future to keep improving
the security of free software related services by finding potential
weakness, and then trying to get connection with service providers.

## 中文版
## 安全促进行动：XMPP 服务的强制 TLS 连接

虽然依据  [RFC7590](https://tools.ietf.org/html/rfc7590) “传输层安全协议
（TLS）在可扩展消息及表示协议（XMPP）中的使用“，TLS 已成为 XMPP 的推荐
连接方式，但其并非是强制的。尽管 XMPP 社区已于2014年达成了[切换至强制加密](https://lwn.net/Articles/599647/)
的共识，但仍有一些 XMPP 的服务提供商在支持 TLS 的同时支持非加密连接，
以作为备用连接方式。

这种行为有可能导致一些安全风险。例如，特殊情况下，某些特定的客户端
会在无法启用 TLS 的情况下，自动尝试使用非加密方式连接服务器。而用户通常
并不会的到任何提示，甚至不知道 TLS 为何物。接着，所有的信息将通过明文
在网络中传递。

xmpp.jp 曾使用此方式处理通信连接。我们在年初曾尝试联系其管理员，请求
将服务切换到强制 TLS 模式。我们在2017年3月16日收到回复，其承诺将于
下次例行维护时修改设置。

2017年4月29日，我们注意到一次由于例行维护导致的服务下线。但是服务恢复后，
服务器仍然允许非 TLS 登录和通讯。我们再次通过邮件方式联系其管理员，并
迅速得到反馈，称按原先计划将于一周之内进行切换。

2017年5月8日，我们经历了另一轮的服务下线。之后便发现 xmpp.jp 已切换到
了强制 TLS 连接模式。通过使用 Psi+ 客户端进行测试证实了这点。与此同时，
收到管理员的来信 - "done."，我们回复表示了感谢。

至此为止，任务完成。这是一次愉快且顺利的沟通，虽然整个过程所经历的
时间略长。

我们将于未来更多的投入到与自由软件相关服务的安全推广行动之中。寻找
潜在的安全缺陷，之后尝试联其系服务提供商解决问题。

