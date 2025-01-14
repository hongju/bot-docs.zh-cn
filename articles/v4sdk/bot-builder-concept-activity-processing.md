---
title: 活动处理 | Microsoft Docs
description: 了解机器人 SDK 中的活动处理。
keywords: 机器人适配器, 自定义中间件, 短路, 回退, 事件处理程序
author: jonathanfingold
ms.author: jonathanfingold
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 0e0fe51e33e99bbb35ef1d731b611ff9ca12a138
ms.sourcegitcommit: a295a90eac461f8b96770dd902ba44919acf33fc
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/26/2019
ms.locfileid: "67404317"
---
# <a name="activity-processing"></a>活动处理

[!INCLUDE[applies-to](../includes/applies-to.md)]

机器人和用户通过活动交互和交换信息。 机器人应用程序接收的每个活动都会传递给机器人适配器，该适配器会将活动信息传递给机器人逻辑，并最终将所有响应发送给用户。 接收活动，然后通过机器人对其进行处理，这一过程称为“轮次”；这表示一个完整的机器人运作周期。 所有执行结束，活动完全处理并且机器人的所有层都已完成后，轮次结束。

活动（特别是在机器人轮次期间[从机器人发送](#generating-responses)的那些活动）是异步处理的。 这是生成机器人的必要部分；如果需要了解这一切是如何工作的，请查看 [.NET 异步](https://docs.microsoft.com/dotnet/csharp/async)或 [JavaScript 异步](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Statements/async_function)，具体取决于语言选择。

## <a name="the-bot-adapter"></a>机器人适配器

机器人适配器封装了身份验证过程，并将活动发送到机器人连接器服务并从机器人连接器服务接收活动。 当机器人收到活动时，适配器包装有关该活动的所有内容，为轮次创建一个[上下文对象](#turn-context)，将其传递给机器人的应用程序逻辑，并将机器人生成的响应发送回用户通道。

## <a name="authentication"></a>Authentication

适配器使用活动中的信息和来自 REST 请求的 `Authentication` 标头验证应用程序接收的每个传入活动。 适配器使用连接器对象和应用程序的凭据来验证用户的出站活动。

机器人连接器服务身份验证使用 JWT（JSON Web 令牌）`Bearer` 令牌以及 Azure 在你创建机器人服务或注册机器人时为你创建的 Microsoft 应用 ID  和 Microsoft 应用密码  。 应用程序在初始化时需要这些凭据，以允许适配器验证流量。

> [!NOTE]
> 如果你正在本地运行或测试机器人（例如，使用 Bot Framework Emulator），则可以在不将适配器配置为验证进出机器人的流量的情况下执行此操作。

## <a name="turn-context"></a>轮次上下文

当适配器收到活动时，它会生成一个  轮次上下文对象，该对象提供有关传入活动、发送方和接收方、通道、会话以及处理活动所需的其他数据的信息。 然后，适配器将此上下文对象传递给机器人。 上下文对象持续存在一个轮次的时间，并提供以下信息：

* 会话 - 标识会话并包含有关机器人和用户参与会话的信息。
* 活动 - 会话中的请求和回复是所有类型的活动。 此上下文提供有关传入活动的信息，包括路由信息，有关通道、会话、发送方和接收方的信息。
* 自定义信息 - 如果你通过实现中间件或在机器人逻辑中扩展机器人，则可以在每个轮次中提供其他信息。

上下文对象还可用于向用户发送响应，以及获取对适配器的引用<!-- to create a new conversation or continue an existing one-->.

> [!NOTE]
> 应用程序和适配器将异步处理请求；但是，业务逻辑不需要受请求-响应驱动。

## <a name="middleware"></a>中间件

可以将中间件添加到适配器。 中间件和机器人逻辑使用上下文对象来检索有关活动的信息并执行相应操作。 中间件和机器人还可以向上下文对象更新或添加信息，例如跟踪轮次、会话或其他范围的状态。 有关中间件的更多深入信息，请参阅[中间件文章](~/v4sdk/bot-builder-concept-middleware.md)。

## <a name="generating-responses"></a>生成响应

上下文对象提供了允许代码响应活动的活动响应方法：

* _send activity_ 和 _send activities_ 方法将一个或多个活动发送到聊天。
* 如果通道支持，则  update activity 方法会更新会话中的活动。
* 如果通道支持，则  delete activity 方法会从会话中删除活动。

每个响应方法都在异步进程中运行。 在调用活动响应方法时，它会在开始调用处理程序之前克隆关联的[事件处理程序](#response-event-handlers)列表，这意味着它将包含截至此时添加的所有处理程序，但不包含在进程启动后添加的任何内容。

这同时也意味着无法保证响应顺序，特别是当一个任务比另一个任务更复杂时。 如果机器人可以生成对传入活动的多个响应，请确保用户以任何顺序接收它们都有效。

> [!IMPORTANT]
> 线程处理主机器人轮次完成后处理上下文对象释放。 如果响应（包括其处理程序）占用了大量时间并尝试对上下文对象执行操作，则可能会出现 `Context was disposed` 错误。  确保 `await` 任何活动调用，以便主线程等待生成的活动，再完成处理并释放轮次上下文。

## <a name="response-event-handlers"></a>响应事件处理程序

除了机器人和中间件逻辑之外，还可以将响应处理程序（有时也称为事件处理程序或活动事件处理程序）添加到上下文对象中。 在执行实际响应之前，当前上下文对象上出现相关[响应](#generating-responses)时，将调用这些处理程序。 当知道要在实际事件之前或之后对其余当前响应的该类型的所有活动执行某些操作时，这些处理程序非常有用。

> [!WARNING]
> 注意，请勿从它的相应响应事件处理程序中调用活动响应方法，例如，从  发送活动处理程序中调用发送活动方法。 执行此操作可以生成一个无限循环。

每个新活动都会获得一个要执行的新线程。 创建处理活动的线程后，该活动的处理程序列表将复制到该新线程。 不会针对该特定活动事件执行在此之后添加的任何处理程序。

在上下文对象中注册的处理程序的处理方式与适配器管理[中间件管道](~/v4sdk/bot-builder-concept-middleware.md#the-bot-middleware-pipeline)的方式非常相似。 也就是说，处理程序按照它们添加的顺序进行调用，并且调用  下一个委托将控制权传递给下一个已注册的事件处理程序。 如果处理程序未调用下一个委托，则不会调用任何后续事件处理程序，事件会[短路](~/v4sdk/bot-builder-concept-middleware.md#short-circuiting)，并且适配器不会将响应发送到通道。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [中间件](~/v4sdk/bot-builder-concept-middleware.md)
