---
title: 将机器人升级到 Bot Framework API v3 | Microsoft Docs
description: 了解如何将机器人升级到 Bot Framework API v3。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: 3e99828e7c26b10c39bef4c8db79f92ff5f2b30c
ms.sourcegitcommit: 49a76dd34d4c93c683cce6c2b8b156ce3f53280e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/26/2018
ms.locfileid: "50134707"
---
# <a name="upgrade-your-bot-to-bot-framework-api-v3"></a>将机器人升级到 Bot Framework API v3

在 Build 2016 大会上，Microsoft 宣布推出 Microsoft Bot Framework 及其 Bot Connector API 的初始迭代，以及 Bot Builder 和 Bot Connector SDK。 此后，我们一直在收集用户反馈，并积极改进 REST API 和 SDK。

2016 年 7 月发布了 Bot Framework API v3，并弃用了 Bot Framework API v1。 使用 API v1 的机器人已于 2016 年 12 月在 Skype 上停用，并于 2017 年 2 月 23 日在所有其余通道上停用。 如果使用 API v1 创建机器人，并且想要使它再次正常运行，必须按照本文中的说明将其升级到 API v3。 若要确保从端到端理解升级过程，请在开始之前完整阅读本文。 

## <a name="step-1-get-your-app-id-and-password-from-the-bot-framework-portal"></a>步骤 1：从 Bot Framework 门户获取应用 ID 和密码

登录到 [Bot Framework 门户](https://dev.botframework.com/)，单击“我的机器人”，然后选择机器人以打开其仪表板。 接下来，单击页面左侧“机器人管理”下的“设置”链接。 

在设置页的“配置”部分，检查“Microsoft 应用 ID”字段的内容，然后继续执行后续步骤。

<!-- TODO: Remove this 
### Case 1: App ID field is already populated

If the **App ID** field is already populated, complete these steps:
-->

1. 单击“管理 Microsoft 应用 ID 和密码”。  
![配置](./media/upgrade/manage-app-id.png)

2. 单击“生成新密码”。  
![生成新密码](./media/upgrade/generate-new-password.png)

3. 复制并保存新密码以及 MSA 应用 ID；将在以后用到这些值。  
![新密码](./media/upgrade/new-password-generated.png)

可以按照这些[说明](https://blog.botframework.com/2018/07/03/find-your-azure-bots-appid-and-appsecret/)操作，以另一种方法检索 **Microsoft 应用 ID 和密码**。

<!-- TODO: These steps are no longer valid. AppID will always be generated, confirmed with Support Engineers
### Case 2: App ID field is empty

If the **App ID** field is empty, complete these steps:

1. Click **Create Microsoft App ID and password**.  
   ![Create App ID and password](~/media/upgrade/generate-appid-and-password.png)
   > [!IMPORTANT]
   > Do not select the **Version 3.0** radio button yet. You will do this later, after you have [updated your bot code](#update-code).</div>

2. Click **Generate a password to continue**.  
   ![Generate app password](~/media/upgrade/generate-a-password-to-continue.png)

3. Copy and save the new password along with the MSA App Id; you will need these values in the future.  
   ![New password](~/media/upgrade/new-password-generated.png)

4. Click **Finish and go back to Bot Framework**.  
   ![Finish and go back to Portal](~/media/upgrade/finish-and-go-back-to-bot-framework.png)

5. Back on the bot settings page in the Bot Framework Portal, scroll to the bottom of the page and click **Save changes**.  
   ![Save changes](~/media/upgrade/save-changes.png)
-->

## <a id="update-code"></a> 步骤 2：将机器人代码更新到 4.0 版

不再兼容 V1 机器人。 若要更新机器人，需改为在 V3 中创建新的机器人。 若要保留任何旧的代码，需手动迁移代码。

最简单的解决方案是使用新的 [SDK](https://docs.microsoft.com/en-us/azure/bot-service/?view=azure-bot-service-4.0) 重新创建机器人，然后进行部署。 

如果希望保留旧的代码，请按以下步骤操作：

1. 创建新的机器人应用程序。
2. 将旧代码复制到新机器人应用程序中。
3. 通过 Nuget 包管理器将 SDK 升级到最新版本。
4. 修复出现的任何错误，参考新的 [SDK](https://docs.microsoft.com/en-us/azure/bot-service/?view=azure-bot-service-4.0)。
5. 按照这些[说明](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-howto-deploy-azure?view=azure-bot-service-4.0)将机器人部署到 Azure

<!-- TODO: Remove outdated code 
To update your bot code to version 3.0, complete these steps:

1. Update to the latest version of the [Bot Builder SDK](https://github.com/Microsoft/BotBuilder) for your bot's language.
2. Update your code to apply the necessary changes, according the guidance below.
3. Use the [Bot Framework Emulator](~/bot-service-debug-emulator.md) to test your bot locally and then in the cloud.

The following sections describe the key differences between API v1 and API v3. After you have updated your code to API v3, you can finish the upgrade process by [updating your bot settings](#step-3) in the Bot Framework Portal.
-->

### <a name="botbuilder-and-connector-are-now-one-sdk"></a>BotBuilder 和 Connector 现在是一个 SDK

无需使用多个 NuGet 包（或 NPM 模块）为 Builder 和 Connector 安装单独的 SDK，现在可以在单个 Bot Builder SDK 中获得这两个库：

- Bot Builder SDK for .NET：`Microsoft.Bot.Builder` NuGet 包
- Bot Builder SDK for Node.js：`botbuilder` NPM 模块

独立 `Microsoft.Bot.Connector` SDK 现已过时，并不再保留。

### <a name="message-is-now-activity"></a>Message 现为 Activity

`Message` 对象已替换为 API v3 中的 `Activity` 对象。 最常见的活动类型是“message”，但是还有其他活动类型可用于将各种类型的信息传达给机器人或通道。 有关消息的详细信息，请参阅[创建消息](~/dotnet/bot-builder-dotnet-create-messages.md)和[发送和接收活动](~/dotnet/bot-builder-dotnet-connector.md)。

### <a name="activity-types--events"></a>活动类型和事件

某些事件已重命名和/或在 API v3 中重构。 此外，新 `ActivityTypes` 枚举已添加到 Connector，无需记住特定活动类型。

- `conversationUpdate` Activity 类型将 Bot/User Added/Removed To/From Conversation 替换为单个方法。
- 新 `typing` Activity 类型让机器人能够指示它正在编译响应，并知道用户何时输入响应。
- 新 `contactRelationUpdate` Activity 类型让机器人知道它是否已被添加到用户的联系人列表或是从中删除。

当机器人收到 `conversationUpdate` 活动，`MembersRemoved` 属性和 `MembersAdded` 属性将指示已添加到会话或从中删除的对象。 当机器人收到 `contactRelationUpdate` 活动，`Action` 属性将指示用户是将机器人添加到了其联系人列表还是从中删除了机器人。 有关活动类型的详细信息，请参阅[活动概述](~/dotnet/bot-builder-dotnet-activities.md)。

### <a name="addressing-messages"></a>处理消息

在消息中指定的发件人、收件人和通道信息在 API v3 中略有变化：

|API v1 字段 | API v3 字段|
|--------|--------|
| `From` 对象 | `From` 对象 |
| `To` 对象 | `Recipient` 对象 |
| `ChannelConversationID` 属性 | `Conversation` 对象|
| `ChannelId` 属性 | `ChannelId` 属性 |

有关处理消息的详细信息，请参阅[发送和接收活动](~/dotnet/bot-builder-dotnet-connector.md)。

### <a name="sending-replies"></a>发送答复

在 Bot Framework API v3，对用户的所有答复都将以异步方式通过单独启动的 HTTP 请求发送，而不是使用 HTTP POST 以内联方式向机器人传入消息。 由于没有消息会通过 Connector 以内联方式返回给用户，机器人的 post 方法的返回类型将为 `HttpResponseMessage`。 这意味着机器人不同步“返回”你希望发送给用户的字符串，而是在代码的任意位置发送答复消息，而不需要作为对传入 POST 的响应进行答复。 这两种方法将同时向会话发送消息：

- `SendToConversation`
- `ReplyToConversation`

`SendToConversation` 方法会将指定消息附加到会话末尾，而 `ReplyToConversation` 方法（适用于支持它的会话）会在会话中将指定消息添加为上一个消息的直接答复。 有关这些方法的详细信息，请参阅[发送和接收活动](~/dotnet/bot-builder-dotnet-connector.md)。

### <a name="starting-conversations"></a>开始会话

在 Bot Framework API v3 中，可以通过使用新方法 `CreateDirectConversation` 开始与单个用户进行私人会话，或通过使用新方法 `CreateConversation` 与多个用户展开群组会话。 有关开始会话的详细信息，请参阅[发送和接收活动](~/dotnet/bot-builder-dotnet-connector.md#start-a-conversation)。

### <a name="attachments-and-options"></a>附件和选项

Bot Framework API v3 引入了更可靠的附件和卡实现。 `Options` 类型在 API v3 中不再受支持，而是替换为卡。 有关使用 .NET 向消息添加附件的详细信息，请参阅[向消息添加媒体附件](~/dotnet/bot-builder-dotnet-add-media-attachments.md)和[向消息添加富卡附件](~/dotnet/bot-builder-dotnet-add-rich-card-attachments.md)。

### <a name="bot-data-storage-bot-state"></a>机器人数据存储（机器人状态）

在 Bot Framework API v1 中，用于管理机器人状态数据的 API 已折叠到消息 API。 在 Bot Framework API v3 中，这些 API 都是独立的。 现在必须使用 Bot State 服务来获取状态数据（而不是假设它将包含在 `Message` 对象中），并存储状态数据（而不是作为 `Message` 对象的一部分传递）。 有关使用 Bot State 服务管理机器人状态数据的信息，请参阅[管理状态数据](~/dotnet/bot-builder-dotnet-state.md)。

> [!IMPORTANT]
> 建议不要将 Bot Framework State Service API 用于生产环境，该 API 可能会在将来的版本中弃用。 建议更新机器人代码以使用内存中存储进行测试，或者将 Azure 扩展之一用于生产机器人。 有关详细信息，请参阅针对 [.NET](~/dotnet/bot-builder-dotnet-state.md) 或 [Node](~/nodejs/bot-builder-nodejs-state.md) 实现的“管理状态数据”主题。

### <a name="webconfig-changes"></a>Web.config 更改

Bot Framework API v1 在 Web.Config 中使用这些密钥存储身份验证属性：

- `AppID`
- `AppSecret`

Bot Framework API v3 在 Web.Config 中使用这些密钥存储身份验证属性：

- `MicrosoftAppID`
- `MicrosoftAppPassword`

## <a id="step-3"></a> 步骤 3：将更新的机器人部署到 Azure。

将机器人代码升级到 API v3 以后，请直接按照这些[说明](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-howto-deploy-azure?view=azure-bot-service-4.0)将机器人部署到 Azure。 由于 V1 不再受支持，因此所有机器人在部署到 Azure 服务时都会自动使用 V3 API。

<!-- TODO: Documentation set for removal 
1. Sign in to the [Bot Framework Portal](https://dev.botframework.com/).

2. Click **My bots** and select your bot to open its dashboard. 

3. Click the **SETTINGS** link that is located near the top-right corner of the page. 

4. Under **Version 3.0** within the **Configuration** section, paste your bot's endpoint into the **Messaging endpoint** field.  
![Version 3 configuration](~/media/upgrade/paste-new-v3-enpoint-url.png)

5. Select the **Version 3.0** radio button.  
![Select version 3.0](~/media/upgrade/switch-to-v3-endpoint.png)

6. Scroll to the bottom of the page and click **Save changes**.  
![Save changes](~/media/upgrade/save-changes.png)
-->