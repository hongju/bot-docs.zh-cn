---
title: Bot Builder SDK for Node.js 中的重要概念 | Microsoft Docs
description: 了解用于构建和部署 Bot Builder SDK for Node.js 中可用的会话机器人的重要概念和工具。
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 43b6f669cdbc91b78094d3d0d9e7a54f97f9884f
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998024"
---
# <a name="key-concepts-in-the-bot-builder-sdk-for-nodejs"></a>Bot Builder SDK for Node.js 中的重要概念

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-concepts.md)
> - [Node.js](../nodejs/bot-builder-nodejs-concepts.md)

本文介绍了 Bot Builder SDK for Node.js 中的重要概念。 有关 Bot Framework 的简介，请参阅 [Bot Framework 概述](../overview-introduction-bot-framework.md)。

## <a name="connector"></a>连接器

Bot Framework Connector 是一项服务，用于将机器人连接到多个通道，这些通道是诸如 Skype、Facebook、Slack 和短信等的客户端。 Connector 通过从机器人到通道以及从通道到机器人的消息中继来促进机器人与用户之间的通信。 机器人的逻辑作为通过 Connector 服务接收用户消息的一项 Web 服务进行托管，并且使用 HTTPS POST 将机器人的回复发送到 Connector。 

Bot Builder SDK for Node.js 提供 [UniversalBot][UniversalBot] 和 [ChatConnector][ChatConnector] 类，用于将机器人配置为通过 Bot Framework Connector 发送和接收消息。 `UniversalBot` 类构成了机器人的大脑。 它负责管理机器人与用户的所有会话。 `ChatConnector` 类将机器人连接到 Bot Framework Connector 服务。
有关介绍如何使用这些类的示例，请参阅[使用 Bot Builder SDK for Node.js 创建机器人](bot-builder-nodejs-quickstart.md)。

Connector 还规范化机器人发送到通道的消息，使用户可以通过与平台无关的方式开发机器人。 规范化消息涉及从 Bot Framework 架构转换为通道架构。 如果通道不支持框架架构的所有方面，Connector 会尝试将消息转换为通道支持的格式。 例如，如果机器人向短信通道发送的消息中包含一张带有操作按钮的卡，Connector 可能会将该卡呈现为一个图像，并包含该操作作为消息文本中的链接。 [通道检查器][ChannelInspector]是一个 Web 工具，可显示 Connector 如何在不同的通道上呈现消息。

`ChatConnector` 要求在机器人中设置 API 终结点。 借助 Node.js SDK，这通常通过安装 `restify` Node.js 模块来实现。 还可以使用不要求设置 API 终结点的 [ConsoleConnector][ConsoleConnector] 为控制台创建机器人。

## <a name="messages"></a>消息

消息可以包含要显示的文本、要说出的文本、附件、富卡和建议的操作。 可以使用 [session.send][SessionSend] 方法发送消息作为对用户消息的响应。 机器人可以视需要多次调用 `send` 作为对用户消息的响应。 有关说明此内容的示例，请参阅[响应用户消息][RespondMessages]。

有关说明如何发送包含用户单击并发起操作的交互按钮的图形富卡的示例，请参阅[向消息添加富卡](bot-builder-nodejs-send-rich-cards.md)。 有关说明如何发送和接收附件的示例，请参阅[发送附件](bot-builder-nodejs-send-receive-attachments.md)。 有关说明如何在支持语音的通道中发送指定由机器人说出文本的消息的示例，请参阅[向消息添加语音](bot-builder-nodejs-text-to-speech.md)。 有关说明如何发送建议的操作的示例，请参阅[发送建议的操作](bot-builder-nodejs-send-suggested-actions.md)。

## <a name="dialogs"></a>对话框
对话框有助于组织机器人的会话逻辑，是[设计会话流](../bot-service-design-conversation-flow.md)的基础。 有关对话框的简介，请参阅[使用对话框管理会话](bot-builder-nodejs-dialog-manage-conversation.md)。

## <a name="actions"></a>操作
你想要将机器人设计为能够在会话流中随时处理取消或帮助请求等中断情况。 Bot Builder SDK for Node.js 提供了全局消息处理程序，可触发取消或调用其他对话框等操作。 请参阅[处理用户操作](bot-builder-nodejs-dialog-actions.md)，获取有关如何使用 [triggerAction][triggerAction] 处理程序的示例。
<!--[Handling cancel](bot-builder-nodejs-manage-conversation-flow.md#handling-cancel), [Confirming interruptions](bot-builder-nodejs-manage-conversation-flow.md#confirming-interruptions) and-->


## <a name="recognizers"></a>识别器
当用户向你的机器人询问某些内容时，例如“帮助”或“查找新闻”，机器人需要了解用户所提问的内容，然后执行适当的操作。 可以将机器人设计为根据用户的输入来识别意向，并将该意向与操作关联。 

可以使用 Bot Builder SDK 提供的内置正则表达式识别器调用外部服务（如 LUIS API），也可以实现自定义识别器以确定用户意向。 请参阅[识别用户意向](bot-builder-nodejs-recognize-intent-messages.md)，查看说明如何向机器人添加识别器并使用它们触发操作的示例。


## <a name="saving-state"></a>保存状态

良好的机器人设计的关键是跟踪会话的上下文，使机器人能够记住用户提出的最后一个问题。 使用 Bot Builder SDK 生成的机器人设计为无状态，这样它们就可以轻松地进行缩放并跨多个计算节点运行。 Bot Framework 提供一个存储系统用于存储机器人数据，从而实现机器人 Web 服务的缩放。 因此，通常应避免使用全局变量或函数闭包来保存状态。 如果这样做，在扩展机器人时可能会出现问题。 转为使用机器人 [session][Session] 对象的以下属性来保存与用户或会话相关的数据：

* userData：全局存储所有会话中的用户信息。
* conversationData：全局存储单个会话的信息。 会话中的所有人都可以看到此数据，因此在将数据存储到此属性时要小心谨慎。 此属性默认启用，可以使用机器人的 [persistConversationData][PersistConversationData] 设置禁用它。
* privateConversationData：全局存储单个会话的信息，但这是特定于当前用户的专用数据。 此数据涵盖所有对话框，因此，如果用于存储你希望在会话结束时清理的临时状态，它非常有用。
* dialogData：保留单个对话实例的信息。 这对于在对话框中存储[瀑布图](bot-builder-nodejs-dialog-waterfall.md)各步骤之间的临时信息而言至关重要。

有关说明如何使用这些属性存储和检索数据的示例，请参阅[管理状态数据](bot-builder-nodejs-state.md)。

## <a name="natural-language-understanding"></a>自然语言理解

借助 Bot Builder，可以使用 LUIS 通过 [LuisRecognizer][LuisRecognizer] 类向机器人添加自然语言理解。 可以添加引用发布的语言模型的 **LuisRecognizer** 实例，然后添加处理程序，执行操作以响应用户的表达。 若要查看 LUIS 的实际用途，请观看下面的 10 分钟教程：

* [Microsoft LUIS 教程][LUISVideo]（视频）

## <a name="next-steps"></a>后续步骤
> [!div class="nextstepaction"]
> [对话框概述](bot-builder-nodejs-dialog-overview.md)



[PersistConversationData]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.iuniversalbotsettings.html#persistconversationdata
[UniversalBot]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.universalbot.html
[ChatConnector]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.chatconnector.html
[ConsoleConnector]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.consoleconnector.html

[ChannelInspector]: ../bot-service-channel-inspector.md

[Session]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html
[SessionSend]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session#send

[triggerAction]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#triggeraction
[waterfall]: bot-builder-nodejs-prompts.md

[RespondMessages]:bot-builder-nodejs-use-default-message-handler.md

[LUISRecognizer]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.luisrecognizer
[LUISVideo]: https://vimeo.com/145499419
