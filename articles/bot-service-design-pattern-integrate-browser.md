---
title: 将机器人与 Web 浏览器集成 | Microsoft Docs
description: 了解如何通过设计使用户在机器人与 Web 浏览器之间来回进行流畅的转换。
author: matvelloso
ms.author: mateusv
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 7f3a6ace5e3ef8122cca32baf8ec4c9a1f250dfa
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/18/2018
ms.locfileid: "39297857"
---
# <a name="integrate-your-bot-with-a-web-browser"></a>将机器人与 Web 浏览器集成

在某些场景中，仅仅只有机器人是无法满足需求的。 机器人可能需要将用户转到 Web 浏览器以完成一项任务，然后在任务完成后继续与用户聊天。 

## <a name="authentication-and-authorization"></a>身份验证和授权
如果机器人希望能在 Office 365 中读取用户的日历，甚至代表该用户创建约会，则用户必须首先向 Microsoft Azure Active Directory 进行身份验证，并授权机器人访问用户的日历数据。 机器人将用户重定向到 Web 浏览器以完成身份验证和授权任务，然后继续与用户聊天。 

## <a name="security-and-compliance"></a>安全和符合性
安全性和符合性要求通常会限制机器人可与用户交换的信息类型。 在某些情况下，用户可能需要在当前聊天外发送/接收数据。 例如，如果用户想使用第三方支付提供程序进行支付，则不应在聊天的上下文中指定信用卡号。 机器人会将用户定向到 Web 浏览器来完成支付过程，然后继续与用户聊天。

本文探讨了改善用户在机器人与 Web 浏览器之间来回转换效果的过程。 

> [!NOTE]
> 在聊天与 Web 浏览器之间来回转换的效果并不尽如人意，因为在应用程序之间切换很容易让用户迷惑。 为了改善体验，许多通道提供了内置的 HTML 窗口，机器人可使用它们来呈现应用程序，否则这些应用程序会出现在 Web 浏览器中。 利用该技术，用户可以保持聊天，同时仍然可以访问外部资源。 这种方法在概念上类似于在嵌入式 Web 视图中使用 OAuth 管理授权流的移动应用程序。

## <a name="bot-to-web-browser-and-back-again"></a>在机器人与 Web 浏览器之间来回转换

此图显示了机器人和 Web 浏览器之间集成的高级流。 

![机器人与 Web 交互](~/media/bot-service-design-pattern-integrate-browser/bot-to-web1.png)

请注意该流的每个步骤：

1. <a id="generate-hyperlink"></a>机器人生成并显示用于将用户重定向到网站的超链接。 
   超链接通常通过目标 URL 上的查询字符串参数包括指定了关于当前聊天上下文信息的数据，例如聊天 ID、通道 ID和通道中的用户 ID。 

2. 用户单击该超链接，重定向到 Web 浏览器中的目标 URL。 

3. 机器人进入等待状态，等待网站发出通信指示网站流已完成。  
   > [!TIP]
   > 设计此流，使机器人在用户永不完成网站流时，不会永久保持“等待”状态。 换而言之，如果用户放弃 Web 浏览器并再次开始与机器人通信，则机器人应该确认收到，而不是[忽略](~/bot-service-design-navigation.md#the-mysterious-bot)该输入。

4. 用户通过 Web 浏览器完成必要任务。 
   这可能是 OAuth 流或当前场景所需的事件序列。 

5. <a id="generate-magic-number"></a>用户完成网站流时，网站会生成一个“[幻数](#verify-identity)”，指示用户复制该值并将其粘贴回与机器人的聊天中。 

6. <a id="signal-to-bot"></a>网站[向机器人发出信号](#website-signal-to-bot)，指示用户已完成网站流。 
   它将“幻数”传达给机器人并提供任何其他相关数据。
   例如，在 OAuth 流的情况下，网站会向机器人提供访问令牌。

7. 用户返回到机器人并将“幻数”粘贴到聊天中。 
   机器人验证用户提供的“幻数”是否与预期值匹配，从而验证当前用户是否为先前单击超链接以启动网站流的同一用户。 

### <a id="verify-identity"></a>使用“幻数”验证用户身份

通过在机器人转到网站流期间生成“幻数”（上述[步骤 5](#generate-magic-number)），机器人能够随后验证启动网站流的用户是否确实是预期用户。 例如，如果机器人正在与多个用户进行群聊，他们中的任何一个都可单击超链接来启动网站流。 如果没有“幻数”验证过程，机器人就无法知道哪个用户完成了流。 一个用户可以在另一用户的聊天中对访问令牌进行身份验证和注入。 

> [!WARNING] 
> 这种风险不仅仅发生在群聊中。 如果没有“幻数”验证过程，任何人只要获得用于启动网站流的超链接，都可以冒用用户身份。 

幻数应该是使用强加密库生成的随机数字。 有关 C# 中生成过程的示例，请参阅 <a href="https://www.nuget.org/packages/BotAuth" target="_blank">BotAuth</a> 库中的<a href="https://github.com/MicrosoftDX/botauth/tree/master/CSharp" target="_blank">相关代码</a>。 BotAuth 支持 Microsoft Bot Framework 中生成的机器人实现机器人转到网站流，以便对网站中的用户进行身份验证，然后使用从身份验证过程生成的访问令牌。 由于 BotAuth 未对通道的功能做出任何假设，因此这些流应在大多数通道上运行良好。 

> [!NOTE]
> 由于通道生成了自己的嵌入式 Web 视图，应舍弃对“幻数”验证过程的需求。

### <a id="website-signal-to-bot"></a>网站如何向机器人“发出信号”？

当机器人[生成超链接](#generate-hyperlink)（用户单击该超链接可启动网站流）时，该超链接会通过目标 URL 中的查询字符串参数包括有关当前聊天上下文的信息，例如聊天 ID、通道 ID 和通道中的用户 ID。 随后，网站可使用此信息读取和写入该用户或使用 Bot Builder SDK 或 REST API 的聊天的状态变量。 请参阅上述[步骤 6](#signal-to-bot)，了解网站如何向机器人“发出信号”，指示网站流已完成。

## <a name="sample-code"></a>代码示例

如本文所述，<a href="https://github.com/MicrosoftDX/botauth" target="_blank">BotAuth</a> 库支持将 OAuth 流绑定到使用 Microsoft Bot Framework 中的 .NET 和 Node 生成的机器人。

## <a name="additional-resources"></a>其他资源

- [对话](~/dotnet/bot-builder-dotnet-dialogs.md)
- [使用对话管理聊天流 (.NET)](~/dotnet/bot-builder-dotnet-manage-conversation-flow.md)
- [使用对话管理聊天流 (Node.js)](~/nodejs/bot-builder-nodejs-manage-conversation-flow.md)
