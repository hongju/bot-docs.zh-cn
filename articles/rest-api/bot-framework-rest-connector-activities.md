---
title: 活动概述 | Microsoft Docs
description: 了解 Bot Connector 服务中提供的不同活动类型。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: b246e9e07243e4064f92e72ee3909541f642714e
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999924"
---
# <a name="activities-overview"></a>活动概述

Bot Connector 服务通过传递 [Activity][Activity] 对象在机器人和通道（用户）之间交换信息。 最常见的活动类型是“消息”，但是还有其他活动类型可用于将各种类型的信息传达给机器人或通道。 

## <a name="activity-types-in-the-bot-connector-service"></a>Bot Connector 服务中的活动类型

Bot Connector 服务支持以下活动类型。

| 活动类型 | Description |
|------|------|------|
| message | 表示机器人和用户之间的通信。 |
| conversationUpdate | 指示机器人已添加到聊天中、其他成员已添加到聊天或已从中删除，或者聊天元数据已更改。 |
| contactRelationUpdate | 指示已将机器人添加到用户的联系人列表或已从中删除。 |
| typing | 指示位于聊天另一端的用户或机器人正在编写答复。 | 
| deleteUserData | 向机器人表明用户已请求机器人删除其可能存储的所有用户数据。 |
| endOfConversation | 指示聊天结束。 |

## <a name="message"></a>message

机器人将发送“消息”活动向用户传达信息，并接收来自用户的“消息”活动。 一些消息可能是纯文本，而其他则可能包含更丰富的内容，如[媒体附件](bot-framework-rest-connector-add-media-attachments.md)[按钮和卡片](bot-framework-rest-connector-add-rich-cards.md)或[通道特定的数据](bot-framework-rest-connector-channeldata.md)。 有关常用消息属性的信息，请参阅[创建消息](bot-framework-rest-connector-create-messages.md)。 要了解如何发送和接收消息，请参阅[发送和接收消息](bot-framework-rest-connector-send-and-receive-messages.md)。 

## <a name="conversationupdate"></a>conversationUpdate

每次将机器人添加到聊天、其他成员已添加到聊天或已从中删除，或者聊天元数据已更改时，机器人都会收到 conversationUpdate 活动。 

如果成员已添加到聊天中，则活动的 `addedMembers` 属性将标识新成员。 如果成员已从聊天中删除，则 `removedMembers` 属性将标识已删除的成员。 

> [!TIP]
> 如果机器人收到 conversationUpdate 活动，指示用户已加入聊天，则可选择通过向该用户发送欢迎消息进行答复。 

## <a name="contactrelationupdate"></a>contactRelationUpdate

每次将机器人添加到用户的联系人列表或从中删除时，机器人都会收到 contactRelationUpdate 活动 。 活动的 `action` 属性的值 (add | remove) 指示是已将机器人添加到用户的联系人列表还是已将其从中删除。

## <a name="typing"></a>typing

机器人会收到 typing 活动，它指示用户正在键入答复。 机器人可发送 typing 活动向用户表明它正在积极处理请求或编写答复。 

## <a name="deleteuserdata"></a>deleteUserData

当用户请求删除机器人之前为其保留的任何数据时，机器人会收到 deleteUserData 活动。 如果机器人收到此类活动，则应删除之前为发出请求的用户存储的任何个人身份信息 (PII)。

## <a name="endofconversation"></a>endOfConversation 

机器人会收到 endOfConversation 活动，它指示用户已结束聊天。 机器人可发送 endOfConversation 活动向用户表明聊天即将结束。 

## <a name="additional-resources"></a>其他资源

- [创建消息](bot-framework-rest-connector-create-messages.md)
- [发送和接收消息](bot-framework-rest-connector-send-and-receive-messages.md)

[Activity]: bot-framework-rest-connector-api-reference.md#activity-object