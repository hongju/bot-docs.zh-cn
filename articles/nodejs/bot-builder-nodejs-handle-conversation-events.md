---
title: 处理用户和会话事件 | Microsoft Docs
description: 了解如何使用 Bot Framework SDK for Node.js 来处理事件，例如，用户加入会话。
author: DucVo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: a6149b750a4432f00268571df6d12b611114181f
ms.sourcegitcommit: a295a90eac461f8b96770dd902ba44919acf33fc
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/26/2019
ms.locfileid: "67404913"
---
# <a name="handle-user-and-conversation-events"></a>处理用户和会话事件

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

本文演示了机器人如何处理事件，例如用户加入会话、将机器人添加到联系人列表或者在从会话中移除机器人时说再见  。


## <a name="greet-a-user-on-conversation-join"></a>用户加入会话时发送问候
Bot Framework 提供 [conversationUpdate][conversationUpdate] 事件，用于在成员加入或离开会话时通知机器人。 会话成员可以是用户，也可以是机器人。

以下代码片段允许机器人向会话中的新成员发送问候，或者在从会话中移除机器人时说再见  。

[!INCLUDE [conversationUpdate sample Node.js](../includes/snippet-code-node-conversationupdate-1.md)]

## <a name="acknowledge-add-to-contacts-list"></a>确认添加到联系人列表

[contactRelationUpdate][contactRelationUpdate] 事件会在用户已将你添加到其联系人列表中时通知机器人。

[!INCLUDE [contactRelationUpdate sample Node.js](../includes/snippet-code-node-contactrelationupdate-1.md)]

## <a name="add-a-first-run-dialog"></a>添加首次运行对话框

由于不是所有通道都支持 conversationUpdate  和 contactRelationUpdate  事件，因此欢迎用户加入会话的通用方法是添加首次运行对话框。

在下面的示例中，我们添加了一个函数，可以在我们以前从未见过用户的任何时候触发对话框。 可通过为操作提供 [onFindAction][onFindAction] 处理程序来自定义触发操作的方式。 

[!INCLUDE [first-run sample Node.js](../includes/snippet-code-node-first-run-dialog-1.md)]

此外，还可以通过提供 [onSelectAction][onSelectAction]handler. For trigger actions you can provide an [onInterrupted][onInterrupted] 处理程序自定义操作触发后的操作，以便在中断发生前将其截获。 有关详细信息，请参阅[处理用户操作](bot-builder-nodejs-dialog-actions.md)

## <a name="additional-resources"></a>其他资源

* [conversationUpdate][conversationUpdate]
* [contactRelationUpdate][contactRelationUpdate]

[conversationUpdate]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.iconversationupdate.html
[contactRelationUpdate]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.icontactrelationupdate.html

[onFindAction]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.itriggeractionoptions#onfindaction
[onSelectAction]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.itriggeractionoptions#onselectaction
[onInterrupted]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.itriggeractionoptions#oninterrupted

[SendTyping]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session#sendtyping
[IMessage]: http://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.imessage
[ChatConnector]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.chatconnector.html
[session_userData]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session.html#userdata
