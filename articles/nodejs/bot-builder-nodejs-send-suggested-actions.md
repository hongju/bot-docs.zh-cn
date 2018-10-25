---
title: 向消息添加建议的操作 | Microsoft Docs
description: 了解如何使用 Bot Builder SDK for Node.js 在消息中发送建议的操作。
author: v-ducvo
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 06/06/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 1be5423de3859b32416b5acf9ba7b9d8e92ecb14
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997244"
---
# <a name="add-suggested-actions-to-messages"></a>向消息添加建议的操作

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-suggested-actions.md)
> - [Node.js](../nodejs/bot-builder-nodejs-send-suggested-actions.md)
> - [REST](../rest-api/bot-framework-rest-connector-add-suggested-actions.md)

[!INCLUDE [Introduction to suggested actions](../includes/snippet-suggested-actions-intro.md)]

> [!TIP]
> 使用 [Channel Inspector][channelInspector] 查看建议的操作在各种通道上的外观和工作方式。

## <a name="suggested-actions-example"></a>建议的操作示例

若要将建议的操作添加到消息中，请将消息的 `suggestedActions` 属性设置为表示要显示给用户的按钮的[卡操作][ICardAction]列表。

此代码示例演示如何向用户发送呈现三个建议操作的消息：

[!code-javascript[Send suggested actions](../includes/code/node-send-suggested-actions.js#sendSuggestedActions)]

当用户点击其中一个建议的操作时，机器人将从用户收到包含相应操作的 `value` 的消息。

请注意，`imBack` 方法会将 `value` 发布到你所使用的通道的聊天窗口中。 如果这不是所需的效果，你可以使用 `postBack` 方法，该方法仍然会将所选内容发布回机器人，但不会在聊天窗口中显示所选内容。 但是，某些通道不支持 `postBack`，在这些情况下，该方法的行为类似于 `imBack`。

## <a name="additional-resources"></a>其他资源

* [使用通道检查器预览功能][inspector]
* [IMessage][IMessage]
* [ICardAction][ICardAction]
* [session.send][SessionSend]

[IMessage]: http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.imessage

[SessionSend]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html#send

[ICardAction]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.icardaction.html

[inspector]: ../bot-service-channel-inspector.md

[channelInspector]: ../bot-service-channel-inspector.md
