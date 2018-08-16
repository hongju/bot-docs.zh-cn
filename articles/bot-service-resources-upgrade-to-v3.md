---
title: 将机器人升级到 Bot Framework API v3 | Microsoft Docs
description: 了解如何将机器人升级到 Bot Framework API v3。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: ca54abf89f8967b2b109895326d13a601030fdec
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/18/2018
ms.locfileid: "39297966"
---
# <a name="upgrade-your-bot-to-bot-framework-api-v3"></a>将机器人升级到 Bot Framework API v3

在 Build 2016 大会上，Microsoft 宣布推出 Microsoft Bot Framework 及其 Bot Connector API 的初始迭代，以及 Bot Builder 和 Bot Connector SDK。 此后，我们一直在收集用户反馈，并积极改进 REST API 和 SDK。

2016 年 7 月发布了 Bot Framework API v3，并弃用了 Bot Framework API v1。 使用 API v1 的机器人已于 2016 年 12 月在 Skype 上停用，并于 2017 年 2 月 23 日在所有其余通道上停用。 如果使用 API v1 创建机器人，并且想要使它再次正常运行，必须按照本文中的说明将其升级到 API v3。 若要确保从端到端理解升级过程，请在开始之前完整阅读本文。 

## <a name="step-1-get-your-app-id-and-password-from-the-bot-framework-portal"></a>步骤 1：从 Bot Framework 门户获取应用 ID 和密码

登录到 [Bot Framework 门户](https://dev.botframework.com/)，单击“我的机器人”，然后选择机器人以打开其仪表板。 接下来，单击页面右上角附近的“设置”链接。 

在设置页的“配置”部分，检查“应用 ID”字段的内容，然后继续执行后续步骤，具体取决于是否已填充“应用 ID”字段。

### <a name="case-1-app-id-field-is-already-populated"></a>案例 1：应用 ID 字段已填充

如果已填充“应用 ID“字段，完成以下步骤：

1. 单击“管理 Microsoft 应用 ID 和密码”。  
![配置](~/media/upgrade/manage-app-id.png)

2. 单击“生成新密码”。  
![生成新密码](~/media/upgrade/generate-new-password.png)

3. 复制并保存新密码以及 MSA 应用 ID；将在以后用到这些值。  
![新密码](~/media/upgrade/new-password-generated.png)

### <a name="case-2-app-id-field-is-empty"></a>案例 2：应用 ID 字段为空

如果“应用 ID”字段为空，完成以下步骤：

1. 单击“创建 Microsoft 应用 ID 和密码”。  
   ![创建应用 ID 和密码](~/media/upgrade/generate-appid-and-password.png)
   > [!IMPORTANT]
   > 不要选择“3.0 版”单选按钮。 将在[更新机器人代码](#update-code)后执行此操作。</div>

2. 单击“生成密码以继续”。  
   ![生成应用密码](~/media/upgrade/generate-a-password-to-continue.png)

3. 复制并保存新密码以及 MSA 应用 ID；将在以后用到这些值。  
   ![新密码](~/media/upgrade/new-password-generated.png)

4. 单击“完成并返回到 Bot Framework”。  
   ![完成并返回到门户](~/media/upgrade/finish-and-go-back-to-bot-framework.png)

5. 返回 Bot Framework 门户中的机器人设置页，滚动到页面底部，单击“保存更改”。  
   ![保存更改](~/media/upgrade/save-changes.png)

## <a id="update-code"></a> 步骤 2：将机器人更新到 3.0 版

若要将机器人更新到 3.0 版，请完成以下步骤：

1. 更新到机器人语言的最新版本的 [Bot Builder SDK](https://github.com/Microsoft/BotBuilder)。
2. 根据以下指南更新代码以应用必要的更改。
3. 使用 [Bot Framework Emulator](~/bot-service-debug-emulator.md) 依次在本地和云中测试机器人。

以下各节介绍 API v1 和 API v3 之间的主要差异。 将代码更新为 API v3 后，可以通过在 Bot Framework 门户中[更新机器人设置](#step-3)完成升级过程。

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

## <a id="step-3"></a> 步骤 3：在 Bot Framework 门户中更新机器人设置

在将机器人升级到 API v3 并部署到云后，请通过以下步骤完成升级过程： 

1. 登录 [Bot Framework 门户](https://dev.botframework.com/)。

2. 单击“我的机器人”，然后选择机器人打开其仪表板。 

3. 单击页面右上角附近的“设置”链接。 

4. 在“3.0 版”下的“配置”部分，将机器人的终结点粘贴到“消息终结点”字段。  
![版本 3 配置](~/media/upgrade/paste-new-v3-enpoint-url.png)

5. 选择“3.0 版”单选按钮。  
![选择 3.0 版](~/media/upgrade/switch-to-v3-endpoint.png)

6. 滚动到页面底部，单击“保存更改”。  
![保存更改](~/media/upgrade/save-changes.png)