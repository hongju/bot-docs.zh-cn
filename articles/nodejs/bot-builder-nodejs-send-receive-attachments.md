---
title: 发送和接收附件 | Microsoft Docs
description: 了解如何使用 Bot Framework SDK for Node.js 发送和接收包含附件的消息。
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: ac74fff5fa7635bf0ef585423b0f8663a1df41c4
ms.sourcegitcommit: a295a90eac461f8b96770dd902ba44919acf33fc
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/26/2019
ms.locfileid: "67404828"
---
# <a name="send-and-receive-attachments"></a>发送和接收附件

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-media-attachments.md)
> - [Node.js](../nodejs/bot-builder-nodejs-send-receive-attachments.md)
> - [REST](../rest-api/bot-framework-rest-connector-add-media-attachments.md)

用户与机器人之间的消息交换可以包含媒体附件，例如图像、视频、音频和文件。 可以发送的附件类型因通道而异，但以下是基本类型：

* **媒体和文件**：可以发送图像、音频和视频等文件，方法是将 contentType 设置为 [IAttachment object][IAttachment] 的 MIME 类型，然后将链接传递给 contentUrl 中的文件   。
* **卡片**：可以发送一组丰富的可视卡 <!-- and custom keyboards --> 方法是将 contentType 设置为所需的卡类型，然后传递卡的 JSON  。 如果使用其中一个资讯卡生成器类（如 HeroCard），则会自动为你填写附件  。 有关此示例，请参阅[发送资讯卡](bot-builder-nodejs-send-rich-cards.md)。

## <a name="add-a-media-attachment"></a>添加媒体附件
消息对象应为 [IMessage][IMessage] and it's most useful to send the user a message as an object when you’d like to include an attachment like an image. Use the [session.send()][SessionSend] 方法的实例，将消息以 JSON 对象的形式发送。 

## <a name="example"></a>示例

以下示例检查用户是否已发送附件，如果已发送，则会回响附件中包含的任何图像。 可通过向机器人发送图像来使用 Bot Framework Emulator 对此进行测试。

```javascript
// Create your bot with a function to receive messages from the user
var bot = new builder.UniversalBot(connector, function (session) {
    var msg = session.message;
    if (msg.attachments && msg.attachments.length > 0) {
     // Echo back attachment
     var attachment = msg.attachments[0];
        session.send({
            text: "You sent:",
            attachments: [
                {
                    contentType: attachment.contentType,
                    contentUrl: attachment.contentUrl,
                    name: attachment.name
                }
            ]
        });
    } else {
        // Echo back users text
        session.send("You said: %s", session.message.text);
    }
});
```
## <a name="additional-resources"></a>其他资源

* [使用 Channel Inspector 预览功能][inspector]
* [IMessage][IMessage]
* [发送资讯卡][SendRichCard]
* [session.send][SessionSend]

[IMessage]: http://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.imessage
[SendRichCard]: bot-builder-nodejs-send-rich-cards.md
[SessionSend]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session.html#send
[IAttachment]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.iattachment.html
[inspector]: ../bot-service-channel-inspector.md
