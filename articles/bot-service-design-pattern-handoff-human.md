---
title: 将会话从机器人转给人工 | Microsoft Docs
description: 了解如何设计这样的情形：用户开始与机器人会话，然后必须传给人工。
author: matvelloso
ms.author: mateusv
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: f786f79011da5e50b37f9797dca694f0e132296c
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/18/2018
ms.locfileid: "39297989"
---
# <a name="transition-conversations-from-bot-to-human"></a>将会话从机器人转给人工

无论机器人拥有多少人工智能，但有时候还是需要将会话传给人工。 机器人应该识别何时需要传递，并为用户提供清晰顺畅的转移。

## <a name="scenarios-that-require-human-involvement"></a>需要人工介入的场景

很多场景可能需要将会话控制权从机器人转给人工。 其中几个场景为“会审”“升级”和“监督”。 

### <a name="triage"></a>会审

典型的技术支持通话从机器人可以轻松回答的一些非常基本的问题开始。 作为第一个响应用户的入站请求的响应方，机器人可以收集用户的姓名、地址、问题描述或任何其他相关信息，然后将会话控制权转给代理。 使用机器人会审传入的请求允许代理将时间花在解决问题而非收集信息上。

### <a name="escalation"></a>升级

在技术支持场景中，除了收集信息之外，机器人还可以回答基本问题并解决简单问题（如重置用户密码）。 但是，如果会话表明用户的问题足够复杂，需要人工介入，则机器人需要将问题升级至人工代理。 若要实现这种类型的场景，机器人必须能够区分它可以独立解决的问题和必须升级至人工的问题。 机器人确定是否需要将会话控制权转给人工的方式有很多种。 

#### <a name="user-driven-menus"></a>用户驱动菜单

对于机器人来说，处理这种困境的最简单的方法也许是向用户提供一个选项菜单。 机器人可以独立处理的任务显示在标有“与代理聊天”的链接上方的菜单中。 这种类型的实现不需要高级机器学习或自然语言理解。 当用户选择“与代理聊天”选项时，机器人只是将会话的控制权转给人工代理。 

#### <a name="scenario-driven"></a>场景驱动

机器人可以基于是否确定其能够处理手头的场景来决定是否转移控制权。 机器人收集有关用户请求的一些信息，然后查询其内部功能列表，以确定它是否能够处理此请求。 如果机器人确定它能够解决请求，则它会这样做，但是如果机器人确定请求超出了可以解决的问题范围，则会将会话控制权转给人工代理。

#### <a name="natural-language"></a>自然语言

自然语言理解和情感分析有助于机器人决定何时将会话控制权转给人工代理。 当尝试确定用户何时感到沮丧或想要与人工代理交谈时，这尤其有用。 
 
机器人通过使用推断情绪的<a href="https://www.microsoft.com/cognitive-services/en-us/text-analytics-api" target="blank">文本分析 API</a> 或使用 <a href="https://www.luis.ai" target="_blank">LUIS API</a> 来分析用户消息的内容。 


> [!TIP]
> 自然语言理解可能并不总是确定机器人应该何时将会话控制权转给人工的最佳方法。 与人类一样，机器人并不总是能够猜测正确，无效的响应会让用户感到沮丧。 如果用户从有效选项菜单中进行选择，机器人将始终适当地响应该输入。 

### <a name="supervision"></a>监督

在某些情况下，人工代理需要监视会话而非管控。

例如，想想技术支持场景，其中机器人正在与用户通信以诊断计算机问题。 机器学习模型帮助机器人确定问题的最可能原因，但是在建议用户采取特定行动方案之前，机器人可以私下确认诊断并通过人工代理进行补救并请求继续授权。 然后代理单击一个按钮，机器人向用户呈现解决方案，问题就解决了。 机器人仍然执行大部分工作，但是代理保留最终决定的控制权。 

## <a name="transitioning-control-of-the-conversation"></a>转移会话控制权 

当机器人决定将会话的控制权转给人工时，它可以通知用户她正在转移并将会话置于“等待”状态，直到它确认代理可用为止。 

当机器人正在等待人工时，它可以使用默认回复（例如“排队等候”）自动应答所有传入的用户消息。 此外，如果用户发送了某些消息（例如“没关系”或“取消”），可以让机器人将会话从“等待”状态中移除。

指定在设计机器人时如何将代理分配给等待的用户。 例如，机器人可以实现一个简单的队列系统：先进先出。更复杂的逻辑会基于地理位置、语言或某些其他因素将用户分配给代理。 机器人还可以向代理呈现某种类型的 UI，它们可以用这些 UI 选择用户。 当代理可用时，她会连接到机器人并加入会话。

> [!IMPORTANT]
> 即使在代理人参与之后，机器人仍然是会话的幕后推动者。 用户和代理从不直接相互通信，他们只是通过机器人路由消息。 

## <a name="routing-messages-between-user-and-agent"></a>在用户和代理之间路由消息

在代理连接到机器人之后，机器人开始在用户和代理之间路由消息。 虽然用户和代理可能看起来是彼此正在直接聊天，但他们实际上是通过机器人交换消息。 机器人从用户接收消息并将这些消息发送给代理，然后从代理接收消息并将这些消息发送给用户。 

> [!NOTE]
> 在更高级的场景中，除了在用户和代理之间路由消息，机器人还可以承担更多责任。 例如，机器人可以决定哪种响应是合适的或者仅要求代理进行确认以便继续。

## <a name="sample-code"></a>代码示例

有关显示如何使用 Bot Builder SDK for Node.js 将会话从机器人转给人工的完整示例，请参阅 GitHub 中的<a href="https://github.com/palindromed/Bot-HandOff" target="_blank">机器人移交示例</a>。

## <a name="additional-resources"></a>其他资源

::: moniker range="azure-bot-service-4.0"

- [对话框](v4sdk/bot-builder-dialog-manage-conversation-flow.md)
- <a href="https://www.microsoft.com/cognitive-services/en-us/text-analytics-api" target="blank">文本分析 API</a>

::: moniker-end

::: moniker range="azure-bot-service-3.0"

- [使用对话框管理会话流 (.NET)](~/dotnet/bot-builder-dotnet-manage-conversation-flow.md)
- [使用对话框管理会话流 (Node.js)](~/nodejs/bot-builder-nodejs-manage-conversation-flow.md)
- <a href="https://www.microsoft.com/cognitive-services/en-us/text-analytics-api" target="blank">文本分析 API</a>


::: moniker-end

