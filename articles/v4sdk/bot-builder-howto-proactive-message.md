---
title: 向用户发送主动通知 | Microsoft Docs
description: 了解如何发送通知消息
keywords: 主动消息, 通知消息, 机器人通知
author: jonathanfingold
ms.author: jonathanfingold
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 4/15/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: e40a5c0e5cfa020f62c122e3aea134e0dd131372
ms.sourcegitcommit: f84b56beecd41debe6baf056e98332f20b646bda
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/03/2019
ms.locfileid: "65033426"
---
# <a name="send-proactive-notifications-to-users"></a>向用户发送主动通知

[!INCLUDE[applies-to](../includes/applies-to.md)]

通常，机器人向用户发送的每条消息与用户先前的输入直接相关。
在某些情况下，机器人可能需要向用户发送与当前聊天主题或用户发送的最后一条消息不直接相关的消息。 这些类型的消息称为主动消息。

主动消息在各种场景中都可以发挥作用。 例如，如果用户之前已经请求机器人监控产品的价格，则机器人可以在产品价格下降了 20% 时提醒用户。 或者，如果机器人需要一些时间来编译对用户问题的响应，则可以通知用户延迟并允许会话在此期间继续。 当机器人编译完对问题的响应时，将与用户共享该信息。

在机器人中实现主动消息时，不要在短时间内发送多条主动消息。 某些通道强制限制机器人向用户发送消息的频率，并且如果违反这些限制，将禁用机器人。

临时主动消息是最简单的主动消息类型。 只要触发了消息，机器人就会简单地将消息插入到聊天中，而不考虑用户当前是否正在致力于与机器人进行的聊天的单独主题，并且不会尝试以任何方式更改聊天。

若要更顺畅地处理通知，请考虑将通知集成到聊天流中的其他方法，例如在聊天状态中设置标志或将通知添加到队列。

## <a name="prerequisites"></a>先决条件

- 了解[机器人基础知识](bot-builder-basics.md)。
- 以 **[CSharp](https://aka.ms/proactive-sample-cs) 或 [JavaScript](https://aka.ms/proactive-sample-js)** 编写的主动消息示例的副本。 此示例用于解释本文中所述的主动消息。

## <a name="about-the-proactive-sample"></a>关于主动示例

此示例有一个机器人，以及另一个用来将主动消息发送到机器人的控制器，如下图所示。

![主动机器人](media/proactive-sample-bot.png)

## <a name="retrieve-and-store-conversation-reference"></a>检索和存储聊天引用

当模拟器连接到机器人时，机器人会收到两个聊天更新活动。 在机器人的聊天更新活动处理程序中，会检索聊天引用并将其存储在字典中，如下所示。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Bots\ProactiveBot.cs**

[!code-csharp[OnConversationUpdateActivityAsync](~/../botbuilder-samples/samples/csharp_dotnetcore/16.proactive-messages/Bots/ProactiveBot.cs?range=26-37&highlight=3-4,9)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**bots/proactiveBot.js**

[!code-javascript[onConversationUpdateActivity](~/../botbuilder-samples/samples/javascript_nodejs/16.proactive-messages/bots/proactiveBot.js?range=13-17&highlight=2)]

[!code-javascript[onConversationUpdateActivity](~/../botbuilder-samples/samples/javascript_nodejs/16.proactive-messages/bots/proactiveBot.js?range=41-44&highlight=2-3)]

---

注意：在现实场景中，你会将聊天引用持久保存在数据库中，而不是使用内存中的对象。

聊天引用有一个 _conversation_ 属性，用于描述其中存在活动的聊天。 聊天有一个 _user_ 属性，用于列出参与聊天的用户；还有一个 _service URL_ 属性，供通道用来表示 URL（可以在其中发送当前活动的回复）。 若要向用户发送主动消息，需要有效的聊天引用。

## <a name="send-proactive-message"></a>发送主动消息

第二个控制器为 _notify_ 控制器，负责向机器人发送主动消息。 使用以下步骤生成主动消息。

1. 检索要向其发送主动消息的聊天的引用。
1. 调用适配器的 _continue conversation_ 方法，提供要使用的聊天引用和轮次处理程序委托。 continue conversation 方法会为引用的聊天生成一个轮次上下文，然后调用指定的轮次处理程序委托。
1. 在委托中，请根据轮次上下文来发送主动消息。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Controllers\NotifyController .cs**

每次请求机器人的 notify 页时，notify 控制器都会从字典中检索聊天引用。
该控制器然后使用 `ContinueConversationAsync` 和 `BotCallback` 方法来发送主动消息。

[!code-csharp[Notify logic](~/../botbuilder-samples/samples/csharp_dotnetcore/16.proactive-messages/Controllers/NotifyController.cs?range=17-59&highlight=28,40-43)]

适配器要求提供机器人的应用 ID 才能发送主动消息。 在生产环境中，可以使用机器人的应用 ID。 在本地测试环境中，可以使用任何 GUID。 如果目前未为机器人分配应用 ID，则 notify 控制器会自行生成一个可用于调用的占位符 ID。

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**index.js**

每次请求服务器的 `/api/notify` 页时，服务器都会从字典中检索聊天引用。
该服务器然后使用 `continueConversation` 方法来发送主动消息。
`continueConversation` 的参数是一个函数，该函数充当机器人此轮次的轮次处理程序。

[!code-javascript[Notify logic](~/../botbuilder-samples/samples/javascript_nodejs/16.proactive-messages/index.js?range=56-62&highlight=4-5)]

---

## <a name="test-your-bot"></a>测试机器人

1. 安装 [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme)（如果尚未安装）。
1. 在计算机本地运行示例。
1. 启动模拟器并连接到机器人。
1. 加载到机器人的 api/notify 页。 这样会在模拟器中生成主动消息。

## <a name="additional-information"></a>其他信息

除了本文中使用的示例，[GitHub](https://github.com/Microsoft/BotBuilder-Samples/) 中还提供了以 C# 和 JS 编写的其他示例。

### <a name="avoiding-401-unauthorized-errors"></a>避免 401“未授权”错误 

默认情况下，如果传入请求是通过 BotAuthentication 进行身份验证的，BotBuilder SDK 会将 `serviceUrl` 添加到受信任主机名列表中。 它们保留在内存中缓存中。 如果重启机器人，则等待主动消息的用户无法收到该消息，除非该用户在机器人重启后再次向机器人发送消息。 

为了避免这种情况，必须手动将 `serviceUrl` 添加到受信任主机名列表中，只需使用以下语句即可： 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp 
MicrosoftAppCredentials.TrustServiceUrl(serviceUrl); 
``` 

进行主动消息传送时，`serviceUrl` 是主动消息接收者使用的通道的 URL，可以在 `Activity.ServiceUrl` 中找到。 

需将上述代码直接添加到发送主动消息的代码前面。 此示例将它置于 `ProactiveBot.cs` 中接近 `CreateCallback()` 尾部的位置，但已将它注释掉，因为在没有 `appId` 和 `appPassword` 的情况下，它不能在模拟器中使用。

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```js
MicrosoftAppCredentials.trustServiceUrl(serviceUrl);
```

进行主动消息传送时，`serviceUrl` 是主动消息接收者使用的通道的 URL，可以在 `activity.serviceUrl` 中找到。

需将上述代码直接添加到发送主动消息的代码前面。 此示例将它置于 `bot.js` 中接近 `completeJob()` 尾部的位置，但已将它注释掉，因为在没有 `appId` 和 `appPassword` 的情况下，它不能在模拟器中使用。

---

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [实现顺序聊天流](bot-builder-dialog-manage-conversation-flow.md)
