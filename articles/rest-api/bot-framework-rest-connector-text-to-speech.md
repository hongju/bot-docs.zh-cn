---
title: 向消息添加语音 | Microsoft Docs
description: 了解如何使用 Bot Connector 服务向消息添加语音。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: 2aac000b7e8dd52b00659ffecde5184df6c29991
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998684"
---
# <a name="add-speech-to-messages"></a>向消息添加语音
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-text-to-speech.md)
> - [Node.js](../nodejs/bot-builder-nodejs-text-to-speech.md)
> - [REST](../rest-api/bot-framework-rest-connector-text-to-speech.md)

如果要为支持语音的通道（如 Cortana）构建机器人，可以构造可指定机器人要说出的文本的消息。 还可以通过指定[输入提示](bot-framework-rest-connector-add-input-hints.md)来尝试影响客户端麦克风的状态，以指示机器人是在接受、期望还是忽略用户输入。

## <a name="specify-text-to-be-spoken-by-your-bot"></a>指定机器人要朗读的文本

若要在支持语音的通道上指定机器人要朗读的文本，请在表示消息的[ Activity][Activity] 对象中设置 `speak` 属性。 可以将 `speak` 属性设置为纯文本字符串或格式化为<a href="https://msdn.microsoft.com/en-us/library/hh378377(v=office.14).aspx" target="_blank">语音合成标记语言 (SSML)</a> 的字符串，后者是一种基于 XML 的标记语言，可用于控制机器人语音的各种特性（如声音、语速、音量、发音和音调等）。 

以下请求发送一条消息，指定要显示的文本和要朗读的文本，并指示机器人[期望用户输入](bot-framework-rest-connector-add-input-hints.md)。 它使用 <a href="https://msdn.microsoft.com/en-us/library/hh378377(v=office.14).aspx" target="_blank">SSML</a> 格式来指定 `speak` 属性，以指示在朗读“sure”一词时应适当加强语气。 在此示例请求中，`https://smba.trafficmanager.net/apis` 表示基本 URI；机器人发出的请求的基本 URI 可能不同。 有关设置基本 URI 的详细信息，请参阅 [API 参考](bot-framework-rest-connector-api-reference.md#base-uri)。

```http
POST https://smba.trafficmanager.net/apis/v3/conversations/abcd1234/activities/5d5cdc723
Authorization: Bearer ACCESS_TOKEN
Content-Type: application/json
```

```json
{
    "type": "message",
    "from": {
        "id": "12345678",
        "name": "sender's name"
    },
    "conversation": {
        "id": "abcd1234",
        "name": "conversation's name"
   },
   "recipient": {
        "id": "1234abcd",
        "name": "recipient's name"
    },
    "text": "Are you sure that you want to cancel this transaction?",
    "speak": "Are you <emphasis level='moderate'>sure</emphasis> that you want to cancel this transaction?",
    "inputHint": "expectingInput",
    "replyToId": "5d5cdc723"
}
```

## <a name="input-hints"></a>输入提示

在支持语音的通道上发送消息时，可以尝试通过同时包含输入提示来影响客户端麦克风的状态，以指示机器人是在接受、期望还是忽略用户输入。 有关详细信息，请参阅[向消息添加输入提示](bot-framework-rest-connector-add-input-hints.md)。

## <a name="additional-resources"></a>其他资源

- [创建消息](bot-framework-rest-connector-create-messages.md)
- [发送和接收消息](bot-framework-rest-connector-send-and-receive-messages.md)
- [向消息添加输入提示](bot-framework-rest-connector-add-input-hints.md)
- <a href="https://msdn.microsoft.com/en-us/library/hh378377(v=office.14).aspx" target="_blank">语音合成标记语言 (SSML)</a>

[Activity]: bot-framework-rest-connector-api-reference.md#activity-object
