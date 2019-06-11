---
title: 通道参考
description: Bot Framework 通道参考
keywords: 通道参考, 机器人生成器通道, Bot Framework 通道
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: tools
ms.date: 03/01/2019
ms.openlocfilehash: 28f284e4d69cbef7a1741d298b3ae9e6e127e9dd
ms.sourcegitcommit: 710d279898db587abb1e81d13628177a4e182293
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/11/2019
ms.locfileid: "59541083"
---
# <a name="categorized-activities-by-channel"></a>按通道分类的活动

以下各表显示了哪些事件（线路上的活动）可能会来自哪些通道。

这是表的关键：

符号              | 含义
:------------------:|:------------------------------------------------
:white_check_mark:  |机器人应当会收到此活动
:x:                 |机器人**决不**应当收到此活动
:white_large_square:|当前不确定机器人能否收到此活动

活动可以有意义地拆分为不同的类别。 对于每个类别，我们都提供了一个包含可能的活动的表。

<a name="conversational"></a>聊天
--------------

 \                      | Cortana            | Direct Line        | Direct Line（网上聊天） | 电子邮件              | Facebook           | GroupMe            | Kik                | 团队              | Slack              | Skype   | Skype Business | Telegram | Twilio  
:---------------------- | :-----:            | :----------------: | :--------------------: |:----:              | :------:           | :-----:            | :-----:            | :---:              | :---:              | :---:   | :------------: | :------: | :----:  
消息                 | :white_check_mark: | :white_check_mark: | :white_check_mark:     | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark:        | :white_check_mark:  | :white_check_mark: 
MessageReaction         | :x:                | :x:                | :x:                    | :x:                | :x:                | :x:                | :x:                | :white_check_mark: | :x:                | :x:      | :x:             | :x:       | :x:      

- 所有通道都会发送消息活动。
- 当使用对话时，消息活动通常应当始终传递到对话。
- 对于 MessageReaction，情况可能不是这样的，虽然它们在很大程度上是聊天的一部分。
- 从逻辑方面来看，有两种类型的 MessageReaction：“已添加”和“已删除”


> [!TIP]
> “消息反应”是对以前的评论执行的操作，例如“点赞”。 它们可能不按顺序发生，因此可以认为它们类似于按钮。 此活动当前是通过“团队”通道发送的。  


<a name="welcome"></a>欢迎使用
-------

 \                         | Cortana            | Direct Line        | Direct Line（网上聊天） | 电子邮件   | Facebook             | GroupMe | Kik     | 团队   | Slack   | Skype   | Skype Business | Telegram | Twilio  
:----------------------    | :-----:            | :---------:        | :--------------------: |:----:   | :------:             | :-----: | :-----: | :---:   | :---:   | :---:   | :------------: | :------: | :----:  
ConversationUpdate         | :white_check_mark: | :white_check_mark: | :white_check_mark:     | :x:     | :white_large_square: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :x:      | :x:             | :white_check_mark:  | :x:     
ContactRelationUpdate      | :x:                | :x:                | :x:                    | :x:     | :x:                  | :x:      | :x:      | :x:      | :x:      | :white_check_mark: | :white_check_mark:        | :x:       | :x:     

- 通道经常会发送 ConversationUpdate 活动。
- 从逻辑方面来看，有两种类型的 MessageReaction：“已添加”和“已删除”
- 如果可以通过绑定 ConversationUpdate.Added 来简单地实现机器人“欢迎”行为，这非常诱人，并且这有时候可行。
- 但是，这是一个简化版，为了产生可靠的“欢迎”行为，机器人实现可能还需要使用状态。


<a name="application-extensibility"></a>应用程序扩展性
-------------------------

 \                      | Cortana            | Direct Line        | Direct Line（网上聊天） | 电子邮件   | Facebook | GroupMe | Kik     | 团队   | Slack   | Skype   | Skype Business | Telegram | Twilio  
:---------------------- | :-----:            | :---------:        | :--------------------: |:----:   | :------: | :-----: | :-----: | :---:   | :---:   | :---:   | :------------: | :------: | :----:  
Event.*                    | :white_large_square: | :white_check_mark: | :white_check_mark:    | :white_large_square: | :white_large_square:  | :white_large_square: | :white_large_square: | :white_large_square:  | :white_large_square:  | :white_large_square:  | :white_large_square:           | :white_large_square:     | :white_large_square:  
Event.CreateConversation   | :white_large_square: | :white_large_square: | :white_large_square:   | :white_large_square: | :white_large_square:  | :white_large_square: | :white_large_square: | :white_large_square:  | :white_large_square:  | :white_large_square:  | :white_large_square:           | :white_large_square:     | :white_large_square:  
Event.ContinueConversation | :white_large_square: | :white_large_square: | :white_large_square:   | :white_large_square: | :white_large_square:  | :white_large_square: | :white_large_square: | :white_large_square:  | :white_large_square:  | :white_large_square:  | :white_large_square:           | :white_large_square:     | :white_large_square:  

- “事件活动”是 Direct Line（又称为“网上聊天”  ）中的一种扩展性机制。
- 同时拥有客户端和服务器的应用程序可以选择借助使用此事件活动的服务来以隧道方式传输自己的事件。


<a name="microsoft-teams"></a>Microsoft Teams
------------------

 \                      | Cortana            | Direct Line        | Direct Line（网上聊天） | 电子邮件   | Facebook | GroupMe | Kik     | 团队   | Slack   | Skype   | Skype Business | Telegram | Twilio  
:---------------------- | :-----:            | :---------:        | :--------------------: |:----:   | :------: | :-----: | :-----: | :---:   | :---:   | :---:   | :------------: | :------: | :----:  
Invoke.TeamsVerification   | :x:      | :x:          | :x: | :x:   | :x:       | :x:      | :x:      | :white_check_mark: | :x:    | :x:    | :x:             | :x:       | :x:     
Invoke.ComposeResponse     | :x:      | :x:          | :x: | :x:   | :x:       | :x:      | :x:      | :white_check_mark: | :x:    | :x:    | :x:             | :x:       | :x:     

- 除了许多其他类型的活动外，Microsoft Teams 还定义了一些特定于团队的“调用活动”。
- “调用活动”特定于某个应用程序，而非特定于客户端将定义的事物。
- 当前不存在调用特定子类型的活动的常规方法。
- 调用当前是机器人上触发“请求-回复”行为的唯一活动。

这非常重要：如果使用对话，要想使 OAuth 提示起作用，必须将 Invoke.TeamsVerification 活动转发到对话。


<a name="message-update"></a>消息更新
--------------

 \                      | Cortana            | Direct Line        | Direct Line（网上聊天） | 电子邮件   | Facebook | GroupMe | Kik     | 团队   | Slack   | Skype   | Skype Business | Telegram | Twilio  
:---------------------- | :-----:            | :---------:        | :--------------------: |:----:   | :------: | :-----: | :-----: | :---:   | :---:   | :---:   | :------------: | :------: | :----:  
MessageUpdate | :x:      | :x:          | :x:    | :x: | :x:      | :x:      | :x:  | :white_check_mark: | :white_large_square:  | :x:    | :x:             | :x:       | :x:     
MessageDelete | :x:      | :x:          | :x:    | :x: | :x:      | :x:      | :x:  | :white_check_mark: | :white_large_square:  | :x:    | :x:             | :x:       | :x:     

- 消息更新当前由“团队”提供支持。


<a name="oauth"></a>OAuth
-------

 \                      | Cortana            | Direct Line        | Direct Line（网上聊天） | 电子邮件   | Facebook | GroupMe | Kik     | 团队   | Slack   | Skype   | Skype Business | Telegram | Twilio  
:---------------------- | :-----:            | :---------:        | :--------------------: |:----:   | :------: | :-----: | :-----: | :---:   | :---:   | :---:   | :------------: | :------: | :----:  
Event.TokenResponse| :white_large_square:  | :white_check_mark:   | :white_check_mark:    | :x:    | :white_large_square: | :white_large_square: | :white_large_square: | :x:    | :white_large_square: | :white_large_square: | :white_large_square:       | :white_large_square: | :white_large_square: 

这非常重要：如果使用对话，要想使 OAuth 提示起作用，必须将 Event.TokenResponse 活动转发到对话。


<a name="uncategorized"></a>未分类的 
-------------

 \                      | Cortana  | Direct Line        | Direct Line（网上聊天） | 电子邮件 | Facebook | GroupMe | Kik     | 团队 | Slack | Skype | Skype Business | Telegram | Twilio  
:---------------------- | :-----:  | :---------:        | :--------------------: |:----: | :------: | :-----: | :-----: | :---: | :---: | :---: | :------------: | :------: | :----:  
EndOfConversation       | :x:      | :white_check_mark: | :white_check_mark:     | :x:   | :x:      | :x:     | :x:     | :x:   | :x:   | :x:   | :x:            | :x:      | :x:     
InstallationUpdate      | :x:      | :white_check_mark: | :white_check_mark:     | :x:   | :x:      | :x:     | :x:     | :x:   | :x:   | :x:   | :x:            | :x:      | :x:     
Typing                  | :x:      | :white_check_mark: | :white_check_mark:     | :x:   | :x:      | :x:     | :x:     | :x:   | :x:   | :x:   | :x:            | :x:      | :x:     
Handoff                 | :x:      | :x:                | :x:                    | :x:   | :x:      | :x:     | :x:     | :x:   | :x:   | :x:   | :x:            | :x:      | :x:     


<a name="out-of-use-includes-payment-specific-invoke"></a>停止使用（包括特定于付款的调用）
---------------------------------------------
- DeleteUserData 
- Invoke.PaymentRequest  
- Invoke.Address
- Ping

---

## <a name="summary-of-activities-supported-per-channel"></a>每个通道支持的活动的汇总

<a name="cortana"></a>Cortana
-------
- 消息
- ConversationUpdate
- _Event.TokenResponse_
- _EndOfConversation（当窗口关闭时？）_

<a name="direct-line"></a>Direct Line
--------
- 消息
- ConversationUpdate
- Event.TokenResponse
- Event.*
- _Event.CreateConversation_
- _Event.ContinueConversation_

<a name="email"></a>电子邮件
-----
- 消息

<a name="facebook"></a>Facebook
--------
- 消息
- _Event.TokenResponse_

<a name="groupme"></a>GroupMe
-------
- 消息
- ConversationUpdate
- _Event.TokenResponse_

<a name="kik"></a>Kik
---
- 消息
- ConversationUpdate
- _Event.TokenResponse_

<a name="teams"></a>团队
-----
- 消息
- ConversationUpdate
- MessageReaction
- MessageUpdate
- MessageDelete
- Invoke.TeamsVerification
- Invoke.ComposeResponse


<a name="slack"></a>Slack
-----
- 消息
- ConversationUpdate
- _Event.TokenResponse_


<a name="skype"></a>Skype
-----
- 消息
- ContactRelationUpdate
- _Event.TokenResponse_


<a name="skype-business"></a>Skype Business
--------------
- 消息
- ContactRelationUpdate 
- _Event.TokenResponse_


<a name="telegram"></a>Telegram
--------
- 消息
- ConversationUpdate
- _Event.TokenResponse_


<a name="twilio"></a>Twilio
------
- 消息

## <a name="summary-table-all-activities-to-all-channels"></a>发送到所有通道的所有活动的汇总表

 \                         | Cortana              | Direct Line          | Direct Line（网上聊天） | 电子邮件                | Facebook             | GroupMe | Kik     | 团队   | Slack   | Skype   | Skype Business | Telegram | Twilio  
:----------------------    | :-----:              | :---------:          | :--------------------: |:----:                | :------:             | :-----: | :-----: | :---:   | :---:   | :---:   | :------------: | :------: | :----:  
消息                    | :white_check_mark:   | :white_check_mark:   | :white_check_mark:     | :white_check_mark:   | :white_check_mark:   | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark:        | :white_check_mark:  | :white_check_mark: 
MessageReaction            | :x:                  | :x:                  | :x:                    | :x:                  | :x:                  | :x:      | :x:      | :white_check_mark: | :x:      | :x:      | :x:             | :x:       | :x:      
ConversationUpdate         | :white_check_mark:   | :white_check_mark:   | :white_check_mark:     | :x:                  | :white_large_square: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :x:      | :x:             | :white_check_mark:  | :x:     
ContactRelationUpdate      | :x:                  | :x:                  | :x:                    | :x:                  | :x:                  | :x:      | :x:      | :x:      | :x:      | :white_check_mark: | :white_check_mark:        | :x:       | :x:     
Event.*                    | :white_large_square: | :white_check_mark:   | :white_check_mark:     | :white_large_square: | :white_large_square: | :white_large_square: | :white_large_square: | :white_large_square:  | :white_large_square:  | :white_large_square:  | :white_large_square:           | :white_large_square:     | :white_large_square:  
Event.CreateConversation   | :white_large_square: | :white_large_square: | :white_large_square:   | :white_large_square: | :white_large_square: | :white_large_square: | :white_large_square: | :white_large_square:  | :white_large_square:  | :white_large_square:  | :white_large_square:           | :white_large_square:     | :white_large_square:  
Event.ContinueConversation | :white_large_square: | :white_large_square: | :white_large_square:   | :white_large_square: | :white_large_square: | :white_large_square: | :white_large_square: | :white_large_square:  | :white_large_square:  | :white_large_square:  | :white_large_square:           | :white_large_square:     | :white_large_square:  
Invoke.TeamsVerification   | :x:                  | :x:                  | :x:                    | :x:                  | :x:                  | :x:      | :x:      | :white_check_mark: | :x:    | :x:    | :x:             | :x:       | :x:     
Invoke.ComposeResponse     | :x:                  | :x:                  | :x:                    | :x:                  | :x:                  | :x:      | :x:      | :white_check_mark: | :x:    | :x:    | :x:             | :x:       | :x:     
MessageUpdate              | :x:                  | :x:                  | :x:                    | :x:                  | :x:                  | :x:      | :x:  | :white_check_mark: | :white_large_square:  | :x:    | :x:             | :x:       | :x:     
MessageDelete              | :x:                  | :x:                  | :x:                    | :x:                  | :x:                  | :x:      | :x:  | :white_check_mark: | :white_large_square:  | :x:    | :x:             | :x:       | :x:     
Event.TokenResponse        | :white_large_square: | :white_check_mark:   | :white_check_mark:     | :x:                  | :white_large_square: | :white_large_square: | :white_large_square: | :x:    | :white_large_square: | :white_large_square: | :white_large_square:       | :white_large_square: | :white_large_square: 
EndOfConversation          | :x:                  | :white_check_mark:   | :white_check_mark:     | :x:                  | :x:      | :x:     | :x:     | :x:   | :x:   | :x:   | :x:            | :x:      | :x:     
InstallationUpdate         | :x:                  | :white_check_mark:   | :white_check_mark:     | :x:                  | :x:      | :x:     | :x:     | :x:   | :x:   | :x:   | :x:            | :x:      | :x:     
Typing                     | :x:                  | :white_check_mark:   | :white_check_mark:     | :x:                  | :x:      | :x:     | :x:     | :x:   | :x:   | :x:   | :x:            | :x:      | :x:     
Handoff                    | :x:                  | :x:                  | :x:                    | :x:                  | :x:      | :x:     | :x:     | :x:   | :x:   | :x:   | :x:            | :x:      | :x:     

## <a name="web-chat"></a>网上聊天 
网上聊天将发送：
- “消息”：带“文本”和/或“附件”
- “事件”：带“名称”和“值”（以 JSON/字符串形式）
- “键入中”：如果用户设置了名为“sendTypingIndicator”的选项，则网上聊天不会发送“contactRelationUpdate”。 并且网上聊天不支持“messageReaction”，没有人明确要求我们支持此功能。

默认情况下，网上聊天将呈现：
- “消息”：将以轮盘式或堆叠式呈现，具体取决于活动中的选项
- “键入中”：将呈现 5 秒后隐藏，或者一直呈现到下一个活动到来
- “conversationUpdate”：将隐藏
- “事件”：将隐藏
- 其他：将显示一个警告框（我们从未在生产中看到过它）。你可以修改此呈现管道来添加、删除或替换任何自定义呈现。

你可以使用网上聊天来发送任何活动类型和有效负载，但我们未将此功能记入文档，也不推荐使用此功能。 你应当改用“事件”活动。
