---
title: 保存状态和访问数据 | Microsoft Docs
description: 介绍了 Bot Builder SDK 中的状态管理器、聊天状态和用户状态。
keywords: LUIS, 聊天状态, 用户状态, 存储, 管理状态
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 02/15/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 56814ab12a85d18e52b0d5ec83fd81682f3b9f60
ms.sourcegitcommit: f95702d27abbd242c902eeb218d55a72df56ce56
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/19/2018
ms.locfileid: "39298554"
---
# <a name="save-state-and-access-data"></a>保存状态和访问数据
[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

对于出色的机器人设计，其关键是跟踪聊天上下文，使机器人能够记住上一问题答案等内容。
根据机器人的用途，甚至可能需要跟踪状态或长时间存储信息，使信息保留时间超过聊天生存期。
机器人的状态是它为了正确响应传入消息而记住的信息。 Bot Builder SDK 提供类，用于存储和检索作为与用户或聊天关联的对象的状态数据。

* 聊天属性可帮助机器人跟踪机器人与用户之间的当前聊天。 如果机器人需要完成一系列步骤或在聊天主题之间进行切换，可使用聊天属性来管理按顺序执行的步骤或跟踪当前主题。 由于聊天属性反映了当前聊天的状态，通常要在聊天结束时（机器人会收到“聊天结束”活动）清除它们。
* 用户属性可用于多种目的，例如确定用户之前聊天的中断位置或仅按名称来问候返回的用户。 如果存储了用户的首选项，则可在下次聊天时使用该信息自定义聊天。 例如，可提醒用户注意有关其感兴趣话题的新闻文章，或者在可进行预约时提醒用户。 如果机器人收到“删除用户数据”活动，则应清除这些信息。

可以使用[存储](bot-builder-howto-v4-storage.md)来读取和写入永久存储。 这样机器人便可以执行更新共享资源、记录 RSVP 或投票，或者读取历史天气数据等操作。 与应用使用存储来实现其目标的方式相同，机器人也可以在与用户的聊天中执行此操作。

<!-- 
*Conversation state* pertains to the current conversation that the user is having with your bot. When the conversation ends, your bot deletes this data.

You can also store *user state* that persists after a conversation ends. For example, if you store a user's preferences, you can use that information to customize the conversation the next time you chat. For example, you might alert the user to a news article about a topic that interests her, or alert a user when an appointment becomes available. 
-->

<!-- You should generally avoid saving state using a global variable or function closures.
Doing so will create issues when you want to scale out your bot. Instead, use the conversation state and user state middleware that the BotBuilder SDK provides --> 


## <a name="types-of-underlying-storage"></a>基础存储类型

SDK 提供机器人状态管理器中间件来持久存储聊天和用户状态。 可使用机器人上下文访问状态。 此状态管理器可将 Azure 表存储、文件存储或内存存储用作基础数据存储。 还可以为机器人创建自己的存储组件。

使用 Azure 表存储生成的机器人可以设计为无状态并可在多个计算节点之间缩放。

> [!NOTE] 
> 文件和内存存储无法在节点之间进行缩放。

## <a name="writing-directly-to-storage"></a>直接写入存储

还可以使用 Bot Builder SDK 将数据直接读取和写入到存储，而无需使用中间件或机器人上下文。 这适用于机器人所用数据来源于机器人聊天流外部的情况。

例如，假设机器人允许用户询问天气预报，而它通过读取外部数据库中的数据来检索指定日期的天气预报。 天气数据库的内容不依赖于用户信息或聊天上下文，因此可以直接从存储中读取，无需使用状态管理器。  有关示例，请参阅[如何直接写入存储](bot-builder-howto-v4-storage.md)。

## <a name="next-steps"></a>后续步骤

接下来，让我们深入了解如何处理活动以及如何对其作出响应。

> [!div class="nextstepaction"]
> [活动处理](bot-builder-concept-activity-processing.md)

## <a name="additional-resources"></a>其他资源

- [如何保存状态](bot-builder-howto-v4-state.md)
- [如何直接写入存储](bot-builder-howto-v4-storage.md)
