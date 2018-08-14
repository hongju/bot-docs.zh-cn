---
title: 将机器人连接到网上聊天通道 | Microsoft Docs
description: 了解如何在网页中使用网上聊天控件将机器人连接到网上聊天通道。
keywords: 网上聊天, 机器人通道, 网页, 密钥, iframe, HTML
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: b560f9f43fc596bc8062676136819922d227d37b
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/18/2018
ms.locfileid: "39297727"
---
# <a name="connect-a-bot-to-web-chat"></a>将机器人连接到网上聊天
使用机器人服务[创建机器人](bot-service-quickstart.md)时，将自动配置网上聊天通道。 网上聊天通道包含网上聊天控件，用户可通过该控件直接在网页中与机器人进行交互。

![网上聊天示例](~/media/bot-service-channel-webchat/webchat-sample.png)

Bot Framework 门户中的网上聊天通道包含在网页中嵌入网上聊天控件所需的一切内容。 要使用网上聊天控件，只需获取机器人的密钥并将控件嵌入网页即可。

[!INCLUDE [Channel Inspector intro](~/includes/snippet-channel-inspector.md)]

## <a id="step-1"></a>获取机器人密钥

1. 在 [Azure 门户](http://portal.azure.com)中打开机器人，然后单击“通道”边栏选项卡。

2. 单击“网上聊天”通道的“编辑”。  
![网上聊天通道](~/media/bot-service-channel-webchat/bot-service-channel-list.png)

3. 在“密钥”下，单击第一个密钥下的“显示”。  
![密钥](~/media/bot-service-channel-webchat/secret-key.png)

4. 复制“密钥”和“嵌入代码”。

5. 单击“Done”（完成） 。

## <a name="embed-the-web-chat-control-in-your-website"></a>在网站中嵌入网上聊天控件

可通过以下两种方法将网上聊天控件嵌入网站中。

### <a name="option-1---keep-your-secret-hidden-exchange-your-secret-for-a-token-and-generate-the-embed"></a>方法 1：隐藏密码，将密码换成令牌，然后生成嵌入

如果可以执行服务器到服务器的请求来将网上聊天密码换成临时令牌，且希望其他开发人员无法轻易将你的机器人嵌入他们的网站，请使用此方法。 虽然使用此方法并不能完全阻止其他开发人员将你的机器人嵌入其网站，但这样确实能加大他们进行此操作的难度。

将密码换成令牌并生成嵌入：

1. 向 `https://webchat.botframework.com/api/tokens` 发出 GET 请求，并通过 `Authorization` 标头传递网上聊天密码。 `Authorization` 标头使用 `BotConnector` 方案并包含密码，如以下示例请求中所示。

2. 对 GET 请求的响应将包含令牌（用引号括起），该令牌可用于通过在 iframe 内中呈现网上聊天控件来启动聊天。 令牌仅对一个聊天有效；要启动另一个聊天，必须生成一个新令牌。

3. 在从 Bot Framework 门户的网上聊天通道复制的 `iframe` 嵌入代码（如上面[步骤 1](#step-1) 中所述）中，将 `s=` 参数更改为 `t=`，然后将“YOUR_SECRET_HERE”替换为令牌。 

> [!NOTE]
> 令牌将在过期之前自动续订。 

##### <a name="example-request"></a>示例请求

```requestGET https://webchat.botframework.com/api/tokens Authorization: BotConnector YOUR_SECRET_HERE
```

##### Example response 

```response
"IIbSpLnn8sA.dBB.MQBhAFMAZwBXAHoANgBQAGcAZABKAEcAMwB2ADQASABjAFMAegBuAHYANwA.bbguxyOv0gE.cccJjH-TFDs.ruXQyivVZIcgvosGaFs_4jRj1AyPnDt1wk1HMBb5Fuw"
```

##### <a name="example-iframe-using-token"></a>iframe 示例（使用令牌）

```html
<iframe src="https://webchat.botframework.com/embed/YOUR_BOT_ID?t=YOUR_TOKEN_HERE"></iframe>
```

##### <a name="example-html-code"></a>html 代码示例
```html
<!DOCTYPE html>
<html>
<body>
  <iframe id="chat" style="width: 400px; height: 400px;" src=''></iframe>
</body>
<script>

    var xhr = new XMLHttpRequest();
    xhr.open('GET', "https://webchat.botframework.com/api/tokens", true);
    xhr.setRequestHeader('Authorization', 'BotConnector ' + 'YOUR SECRET HERE');
    xhr.send();
    xhr.onreadystatechange = processRequest;

    function processRequest(e) {
      if (xhr.readyState == 4  && xhr.status == 200) {
        var response = JSON.parse(xhr.responseText);
        document.getElementById("chat").src="https://webchat.botframework.com/embed/lucas-direct-line?t="+response
      }
    }

  </script>
</html>
```

### <a id="option-2"></a>方法 2：使用密码将网上聊天控件嵌入网站中

如果希望其他开发人员能轻松将你的机器人嵌入其网站，请使用此方法。 

> [!WARNING]
> 如果使用此方法，其他开发人员只需复制嵌入代码即可将机器人嵌入其网站。

通过在 `iframe` 标记中指定密码，将机器人嵌入网站中：

1. 从 Bot Framework 门户中的网上聊天通道复制 `iframe` 嵌入代码（如上面[步骤 1](#step-1) 中所述）。

2. 在“嵌入代码”中，将“YOUR_SECRET_HERE”替换为从同一页面复制的“密钥值”。

##### <a name="example-iframe-using-secret"></a>iframe 示例（使用密码）

```html
<iframe src="https://webchat.botframework.com/embed/YOUR_BOT_ID?s=YOUR_SECRET_HERE"></iframe>
```

## <a name="style-the-web-chat-control"></a>设置网上聊天控件的样式

通过使用 `iframe` 的 `style` 属性指定 `height` 和 `width`，更改网上聊天控件的大小。

```html
<iframe style="height:480px; width:402px" src="... SEE ABOVE ..."></iframe>
```

![聊天控件客户端](~/media/chatwidget-client.png)

## <a name="additional-resources"></a>其他资源

可在 GitHub 上为网上聊天控件[下载源代码](https://github.com/Microsoft/BotFramework-WebChat)。
