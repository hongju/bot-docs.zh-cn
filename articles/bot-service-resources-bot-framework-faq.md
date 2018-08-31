---
title: 机器人服务常见问题 | Microsoft Docs
description: 有关 Bot Framework 元素以及新功能发布时间的常见问题列表。
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 08/28/2018
ms.openlocfilehash: 63aa65e2591d9f98d763863d8d4d56cd0df185ea
ms.sourcegitcommit: f667ce3f1635ebb2cb19827016210a88c8e45d58
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/28/2018
ms.locfileid: "43142423"
---
# <a name="bot-framework-frequently-asked-questions"></a>Bot Framework 常见问题

本文包含有关 Bot Framework 的一些常见问题解答。

## <a name="background-and-availability"></a>背景和可用性
### <a name="why-did-microsoft-develop-the-bot-framework"></a>Microsoft 开发 Bot Framework 的目的是什么？

虽然我们已经推出会话用户界面 (CUI)，但目前很少有开发人员拥有所需的相关专业知识和工具，从而创建新会话体验，或者使现有的应用程序和服务拥有让用户可以顺畅使用的会话界面。 我们创建了 Bot Framework，让开发人员可以更轻松地为用户构建和连接功能强大的机器人，无需考虑他们的交谈位置（其中包括 Microsoft 的主要通道）。

### <a name="what-is-the-v4-sdk"></a>什么是 v4 SDK？
Bot Builder v4 SDK 基于先前的机器人生成器 SDK 的反馈和学习而创建。 它引入了正确的抽象层次，同时实现了机器人构建基块的丰富组件化。 可以从一个简单的机器人开始，使用模块化和可扩展的框架来增强机器人的复杂性。 可以在 GitHub 上找到针对 SDK 的[常见问题解答](https://github.com/Microsoft/botbuilder-dotnet/wiki/FAQ)。


## <a name="channels"></a>声道
### <a name="when-will-you-add-more-conversation-experiences-to-the-bot-framework"></a>何时可以向 Bot Framework 添加更多会话体验？

我们计划不断改进 Bot Framework，其中包括其他通道，但目前无法提供具体安排。  
如果你希望向该框架添加特定通道，请[告诉我们][Support]。

### <a name="i-have-a-communication-channel-id-like-to-be-configurable-with-bot-framework-can-i-work-with-microsoft-to-do-that"></a>我希望可以使用 Bot Framework 配置我的某个信道。 我可以与 Microsoft 协作实现吗？

我们没有为开发人员提供向 Bot Framework 添加新通道的一般机制，但你可以通过 [Direct Line API][DirectLineAPI] 将机器人连接到应用。 如果你是信道开发人员，并希望与我们协作在 Bot Framework 中启用通道，[请与我们联系][Support]。

### <a name="if-i-want-to-create-a-bot-for-skype-what-tools-and-services-should-i-use"></a>如果我想创建用于 Skype 的机器人，我应该使用哪些工具和服务？

Bot Framework 旨在为 Skype 和许多其他通道构建、连接和部署高质量、响应迅速、高性能的可扩展机器人。 可以使用 SDK 创建文本/短信、图像、按钮和支持卡的机器人（构成先进跨会话体验的大多数机器人交互）以及特定于 Skype 的机器人交互，例如丰富的音频和视频体验。

如果你已经有一个很棒的机器人，并且想要与 Skype 受众沟通，你的机器人可以通过 Bot Builder for REST API 轻松连接到 Skype（或任何支持的通道），前提是它具有可通过 Internet 访问的 REST 终结点。

## <a name="security-and-privacy"></a>安全性和隐私
### <a name="do-the-bots-registered-with-the-bot-framework-collect-personal-information-if-yes-how-can-i-be-sure-the-data-is-safe-and-secure-what-about-privacy"></a>注册到 Bot Framework 的机器人是否会收集个人信息？ 如果是，我如何确保数据的安全性？ 如何保护隐私？

每个机器人都是自己的服务，这些服务的开发人员必须根据其开发人员行为准则提供服务条款和隐私声明。  可以从机器人目录中的机器人卡中访问此信息。

为了提供 I/O 服务，Bot Framework 将你的消息和消息内容（包括你的 ID）从你使用的聊天服务传输到机器人。

### <a name="can-i-host-my-bot-on-my-own-servers"></a>是否可在自己的服务器上托管机器人？
是的。 可将机器人托管在 Internet 上的任何位置， 包括自己的服务器、Azure 或其他任何数据中心。 唯一的要求是机器人必须公开一个可公开访问的 HTTPS 终结点。

### <a name="how-do-you-ban-or-remove-bots-from-the-service"></a>如何从服务中禁用或删除机器人？

用户可以通过目录中的机器人联系卡报告存在不当行为的机器人。 开发人员必须遵守 Microsoft 服务条款才能参与该服务。

### <a name="which-specific-urls-do-i-need-to-whitelist-in-my-corporate-firewall-to-access-bot-framework-services"></a>需要在企业防火墙中将哪些特定的 URL 列入白名单才能访问 Bot Framework 服务？
如果出站防火墙阻止了机器人的流量发往 Internet，则你需要在防火墙中将以下 URL 列入白名单：
- login.botframework.com（机器人身份验证）
- login.microsoftonline.com（机器人身份验证）
- westus.api.cognitive.microsoft.com（用于 Luis.ai NLP 集成）
- state.botframework.com（机器人原型状态存储）
- cortanabfchanneleastus.azurewebsites.net（Cortana 通道）
- cortanabfchannelwestus.azurewebsites.net（Cortana 通道）
- *.botframework.com（通道）

### <a name="can-i-block-all-traffic-to-my-bot-except-traffic-from-the-bot-connector-service"></a>是否可以阻止发往机器人的所有流量，但来自机器人连接器服务的流量除外？
不是。 这种 IP 地址或 DNS 白名单不切实际。 Bot Framework 连接器服务托管在全球的 Azure 数据中心，Azure IP 列表会不断变化。 将特定的 IP 地址列入白名单可能只管用一天，当 Azure IP 地址发生变化时，这种做法就不起作用。
 
### <a name="what-keeps-my-bot-secure-from-clients-impersonating-the-bot-framework-connector-service"></a>当客户端模拟 Bot Framework 连接器服务时，机器人受到哪种保护？
1. 向机器人发出的每个请求所随附的安全令牌中包含一个编码的 ServiceUrl，这意味着，即使攻击者获取了该令牌的访问权限，也无法将聊天重定向到新的 ServiceUrl。 SDK 的所有实现都会强制实施此机制，身份验证[参考](https://docs.microsoft.com/en-us/azure/bot-service/rest-api/bot-framework-rest-connector-authentication?view=azure-bot-service-3.0#bot-to-connector)材料中对此做了阐述。

2. 如果传入的令牌缺失或格式不当，Bot Framework SDK 不会在响应中生成令牌。 当机器人配置不当时，此机制可以限制损害程度。
3. 在机器人内部，可以手动检查令牌中提供的 ServiceUrl。 当服务拓扑发生更改时，此机制会使机器人变得更脆弱，因此，这种机制是可行的，但不建议。


请注意，机器人与 Internet 之间存在出站连接。 Bot Framework 连接器服务不是使用 IP 地址或 DNS 名称的列表来与机器人通信。 不支持入站 IP 地址白名单。

## <a name="rate-limiting"></a>速率限制
### <a name="what-is-rate-limiting"></a>什么是速率限制？
Bot Framework 服务必须保护其自身及其客户免受滥用呼叫模式攻击（例如，拒绝服务攻击），这样一来，没有一个机器人可以对其他机器人的性能产生负面影响。 为了实现这种保护，我们为所有终结点添加了速率限制（也称为限制）。 通过强制执行速率限制，我们可以限制机器人进行特定呼叫的频率。 例如：启用速率限制后，如果机器人想要发布大量活动，则必须在一段时间内将它们分开。 请注意，速率限制的目的不是限制机器人的总量。 它旨在防止滥用不遵循人类会话模式的会话架构。

### <a name="how-will-i-know-if-im-impacted"></a>我如何知道我是否受到影响？
即使在数量较大时，你也不太可能遇到速率限制。 最大速率限制只会由于（从机器人或客户端）大量发送活动、极限负载测试或 bug 而发生。 当请求受到限制时，将返回 HTTP 429（请求过多）响应以及 Retry-After 标头，指示在重试请求成功之前等待的时间（以秒为单位）。 可以通过 Azure Application Insights 为你的机器人启用分析，从而收集此信息。 或者，可以在机器人中添加代码来记录消息。 

### <a name="how-does-rate-limiting-occur"></a>速率限制如何发生？
在以下情况下可能出现速率限制：
-   机器人发送消息的频率过高
-   机器人客户端发送消息的频率过高
-   Direct Line 客户端请求新的 Web 套接字的频率过高

### <a name="what-are-the-rate-limits"></a>什么是速率极限？
我们不断调整速率极限，使其尽可能宽松，同时为我们的服务和用户提供保护。 由于阈值有时会发生变化，因此，我们目前不会公布这些数字。 如果你受到速率限制的影响，请通过下面的邮箱随时与我们联系：[bf-reports@microsoft.com](mailto://bf-reports@microsoft.com)。

## <a name="related-services"></a>相关服务
### <a name="how-does-the-bot-framework-relate-to-cognitive-services"></a>Bot Framework 与认知服务具有怎样的关系？

Bot Framework 和[认知服务](http://www.microsoft.com/cognitive)都是 [Microsoft Build 2016](http://build.microsoft.com) 大会上介绍的新功能，也将集成到 GA 的 Cortana Intelligence Suite 中。 这两个服务都是通过多年研究和在热门的 Microsoft 产品中使用的基础上构建的。 这些功能与“Cortana Intelligence”相结合，使每个组织都能利用数据、云和智能的优势构建自己的智能系统，从而开启新机遇、提升业务速度，引领他们为客户提供服务的行业。

### <a name="what-is-cortana-intelligence"></a>什么是 Cortana Intelligence？

Cortana Intelligence 是一个完全托管的大数据、高级分析和智能套件，可将数据转换为智能操作。  
它是一个综合套件，汇集了基于 Microsoft 多年研究和创新积累的技术（涵盖高级分析、机器学习、大数据存储和云中处理），具备以下功能：

* 用户可以以可扩展且安全的方式收集、管理和存储可以经济高效地无缝增长的所有数据。
* 提供由云提供支持的简单可操作分析，使用户能够针对最严苛的问题进行预测、指示并自动化决策制定。
* 通过认知服务和代理实现智能系统：用户能够以更多的上下文和更自然的方式看、听、解释和理解周围的世界。

我们希望能够通过 Cortana Intelligence 帮助我们的企业客户开启新机遇、提升业务速度，成为行业领导者。

### <a name="what-is-the-direct-line-channel"></a>什么是 Direct Line 通道？

Direct Line 是一个 REST API，可用于将机器人添加到服务、移动应用或网页。

可以采用任何语言编写 Direct Line API 的客户端。 直接按照 [Direct Line 协议][DirectLineAPI]编码，在 Direct Line 配置页生成一个机密，就能从代码所在的任何位置与机器人交谈。

Direct Line 适用于：

* iOS、Android、Windows Phone 及更多系统中的移动应用
* Windows、OSX 及更多系统中的桌面应用程序
* 你需要在[可嵌入网上聊天通道][WebChat]产品/服务的基础上自定义更多内容的网页
* 服务到服务应用程序

[DirectLineAPI]: http://docs.botframework.com/en-us/restapi/directline/
[Support]: bot-service-resources-links-help.md
[WebChat]: bot-service-channel-connect-webchat.md

