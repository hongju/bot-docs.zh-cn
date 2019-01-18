---
title: 发送键入指示符 | Microsoft Docs
description: 了解如何使用 Bot Framework SDK for Node.js 添加“请稍候”指示符，告诉用户机器人正在处理请求
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 3852c0b25ea385301be11edd0a46ed5984510820
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/11/2019
ms.locfileid: "54224862"
---
# <a name="send-a-typing-indicator"></a>发送键入指示符 

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

用户希望发出的消息得到及时响应。 如果机器人执行一些长时间运行的任务（如调用服务器或执行查询），而不向用户指明机器人已听到其消息，用户可能会失去耐性，并发送其他消息或就此假设机器人出现故障。
许多通道支持发送键入指示，以向用户显示已接收并且正在处理消息。


## <a name="typing-indicator-example"></a>键入指示符示例

以下示例演示如何使用 [session.sendTyping()][SendTyping] 发送键入指示。  可使用 Bot Framework Emulator 进行测试。


```javascript

// Create bot and default message handler
var bot = new builder.UniversalBot(connector, function (session) {
    session.sendTyping();
    setTimeout(function () {
        session.send("Hello there...");
    }, 3000);
});
```

当插入消息延迟以防止无序发送包含图像的消息时，键入指示符也很有用。

若要了解详细信息，请参阅[如何发送资讯卡](bot-builder-nodejs-send-rich-cards.md)。


## <a name="additional-resources"></a>其他资源

* [sendTyping][SendTyping]


[SendTyping]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session#sendtyping
[IMessage]: http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.imessage
