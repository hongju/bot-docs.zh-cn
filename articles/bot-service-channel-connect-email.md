---
title: 将机器人连接到 Office 365 电子邮件 | Microsoft Docs
description: 了解如何配置机器人，让其可以使用 Office 365 发送和接收邮件。
keywords: Office 365, 机器人通道, 电子邮件, 电子邮件凭据, azure 门户, 自定义电子邮件
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 02/08/2019
ms.openlocfilehash: e77f6cddac07cdcc06d6d35cda98544f33dd1d43
ms.sourcegitcommit: 980612a922b8290b2faadaca193496c4117e415a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/26/2019
ms.locfileid: "64563422"
---
# <a name="connect-a-bot-to-office-365-email"></a>将机器人连接到 Office 365 电子邮件

除了其他[通道](~/bot-service-manage-channels.md)，机器人还可以通过 Office 365 电子邮件与其他用户通信。 当机器人经过配置，可以访问电子邮件时，如果有新邮件到达，它会收到一条消息。 然后机器人可以按照其业务逻辑做出响应。 例如，机器人可能发送确认电子邮件已收到的电子邮件回复，邮件内容为“你好！ 感谢你下达的订单！ 我们将立即开始处理。”

> [!WARNING]
> 创建“spambots”（包括发送不需要或未经请求的批量电子邮件的机器人）将被视为违反 Bot Framework [行为准则](https://www.botframework.com/Content/Microsoft-Bot-Framework-Preview-Online-Services-Agreement.htm)。

## <a name="configure-email-credentials"></a>配置电子邮件凭据

可以通过在电子邮件通道配置中输入 Office 365 凭据，将机器人连接到电子邮件通道。
不支持使用任何替换 AAD 的供应商的联合身份验证。

> [!NOTE]
> 不应该将自己的个人电子邮件帐户用于机器人，因为发送到该电子邮件帐户的每个邮件都将转发给机器人。 这可能导致机器人不恰当地向发件人发送响应。 因此，机器人应仅使用专用的 O365 电子邮件帐户。

若要添加电子邮件通道，在 [Azure 门户](https://portal.azure.com/)中打开机器人，单击“通道”边栏选项卡，然后单击“电子邮件”。 输入有效的电子邮件凭据，然后单击“保存”。

![输入电子邮件凭据](~/media/bot-service-channel-connect-email/bot-service-channel-connect-email-credentials.png)

电子邮件通道当前仅使用 Office 365。 目前不支持其他电子邮件服务。

## <a name="customize-emails"></a>自定义电子邮件

电子邮件通道支持发送自定义属性以使用 `channelData` 属性创建更高级的自定义电子邮件。

[!INCLUDE [Email channelData table](~/includes/snippet-channelData-email.md)]

下面的示例邮件显示了包含这些 `channelData` 属性的 JSON 文件。

```json
{
    "type": "message",
    "locale": "en-Us",
    "channelID": "email",
    "from": { "id": "mybot@mydomain.com", "name": "My bot"},
    "recipient": { "id": "joe@otherdomain.com", "name": "Joe Doe"},
    "conversation": { "id": "123123123123", "topic": "awesome chat" },
    "channelData":
    {
        "htmlBody": "<html><body style = /"font-family: Calibri; font-size: 11pt;/" >This is more than awesome.</body></html>",
        "subject": "Super awesome message subject",
        "importance": "high",
        "ccRecipients": "Yasemin@adatum.com;Temel@adventure-works.com"
    }
}
```

::: moniker range="azure-bot-service-3.0"
若要详细了解如何使用 `channelData`，请参阅 [Node.js](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-ChannelData) 示例或 [.NET](~/dotnet/bot-builder-dotnet-channeldata.md) 文档。
::: moniker-end

::: moniker range="azure-bot-service-4.0"
有关使用 `channelData` 的详细信息，请参阅[如何实现特定于通道的功能](~/v4sdk/bot-builder-channeldata.md)。
::: moniker-end

## <a name="other-considerations"></a>其他注意事项

如果机器人没有在 15 秒内返回 HTTP 状态代码“200 正常”作为对传入电子邮件的响应，则电子邮件通道会尝试重新发送该邮件，因此机器人可能会多次收到同一电子邮件活动。 有关详细信息，请参阅**机器人工作原理**一文和[如何排查超时错误](https://github.com/daveta/analytics/blob/master/troubleshooting_timeout.md)一文中的 [HTTP 详细信息](v4sdk/bot-builder-basics.md#http-details)部分。

## <a name="additional-resources"></a>其他资源

<!-- Put whole list in monikers, even though it's just the second item that needs to be different. -->
::: moniker range="azure-bot-service-3.0"
* 将机器人连接到[通道](~/bot-service-manage-channels.md)
* 使用 Bot Framework SDK for .NET [实现特定于通道的功能](dotnet/bot-builder-dotnet-channeldata.md)
* 使用[通道检查器](bot-service-channel-inspector.md)查看通道如何显示机器人应用程序的某个功能
::: moniker-end
::: moniker range="azure-bot-service-4.0"
* 将机器人连接到[通道](~/bot-service-manage-channels.md)
* 使用 Bot Framework SDK for .NET [实现特定于通道的功能](~/v4sdk/bot-builder-channeldata.md)
* 使用[通道检查器](bot-service-channel-inspector.md)查看通道如何显示机器人应用程序的某个功能
::: moniker-end
