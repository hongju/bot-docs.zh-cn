---
redirect_url: /bot-framework/bot-builder-howto-v4-state
ms.openlocfilehash: a0d2b1295be1271e827d617ad09878ee8cfcd356
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/11/2019
ms.locfileid: "54223912"
---
<a name="--"></a><!--
---
title:状态和存储 | Microsoft Docs description:介绍 Bot Framework SDK 中的状态管理器、聊天状态和用户状态。
keywords:LUIS, 聊天状态, 用户状态, 存储, 管理状态 author:DeniseMak ms.author: v-demak manager: kamrani ms.topic: article ms.service: bot-service ms.subservice: sdk ms.date:02/15/2018 monikerRange: 'azure-bot-service-4.0'
---

# <a name="state-and-storage"></a>状态和存储
[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

对于出色的机器人设计，其关键是跟踪聊天上下文，使机器人能够记住上一问题答案等内容。
根据机器人的用途，甚至可能需要跟踪状态或长时间存储信息，使信息保留时间超过聊天生存期。
机器人的状态是它为了正确响应传入消息而记住的信息。 Bot Framework SDK 提供类，用于存储和检索作为与用户或聊天关联的对象的状态数据。

* 聊天属性可帮助机器人跟踪机器人与用户之间的当前聊天。 如果机器人需要完成一系列步骤或在聊天主题之间进行切换，可使用聊天属性来管理按顺序执行的步骤或跟踪当前主题。 由于聊天属性反映了当前聊天的状态，因此我们通常会在聊天结束时（此时机器人会收到“聊天结束”活动）清除它们。
* 用户属性可用于多种目的，例如确定用户之前聊天的中断位置或仅按名称来问候返回的用户。 如果存储了用户的首选项，则可在下次聊天时使用该信息自定义聊天。 例如，可提醒用户注意有关其感兴趣话题的新闻文章，或者在可进行预约时提醒用户。 如果机器人收到“删除用户数据”活动，则应清除这些信息。

可以使用[存储](bot-builder-howto-v4-storage.md)来读取和写入永久存储。 这样机器人便可以执行更新共享资源、记录 RSVP 或投票，或者读取历史天气数据等操作。 与应用使用存储来实现其目标的方式相同，机器人也可以在与用户的聊天中执行此操作。

<!-- 
*Conversation state* pertains to the current conversation that the user is having with your bot. When the conversation ends, your bot deletes this data.

You can also store *user state* that persists after a conversation ends. For example, if you store a user's preferences, you can use that information to customize the conversation the next time you chat. For example, you might alert the user to a news article about a topic that interests her, or alert a user when an appointment becomes available. 
-->

<!-- You should generally avoid saving state using a global variable or function closures.
Doing so will create issues when you want to scale out your bot. Instead, use the conversation state and user state middleware that the BotBuilder SDK provides --> 

<!--
## Types of underlying storage

The SDK provides bot state manager middleware to persist conversation and user state. State can be accessed using the bot's context. This state manager can use Azure Table Storage, file storage, or memory storage as the underlying data storage. You can also create your own storage components for your bot.

Bots built using Azure Table Storage can be designed to be stateless and scalable across multiple compute nodes.

> [!NOTE] 
> File and memory storage won't scale across nodes.

## Writing directly to storage

You can also use the Bot Framework SDK to read and write data directly to storage, without using middleware or without using the bot context. This can be appropriate to data that your bot uses, that comes from a source outside your bot's conversation flow.

For example, let's say your bot allows the user to ask for the weather report, and your bot retrieves the weather report for a specified date, by reading it from an external database. The content of the weather database isn't dependent on user information or the conversation context, so you could just read it directly from storage instead of using the state manager.  See [How to write directly to storage](bot-builder-howto-v4-storage.md) for an example.

## Next steps

Next, lets get into how activities are processed, in depth, and how we respond to them.

> [!div class="nextstepaction"]
> [Activity Processing](bot-builder-concept-activity-processing.md)

## Additional resources

- [How to save state](bot-builder-howto-v4-state.md)
- [How to write directly to storage](bot-builder-howto-v4-storage.md)

-->
