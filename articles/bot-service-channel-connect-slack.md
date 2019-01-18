---
title: 将机器人连接到 Slack | Microsoft Docs
description: 了解如何配置机器人与 Slack 的连接。
keywords: 连接机器人, 机器人通道, Slack 机器人, Slack 消息传递应用
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 01/09/2019
ms.openlocfilehash: 3573103e1d1c55e3ad648ad68d84674a98b397f7
ms.sourcegitcommit: 8161753641368567f239e24a35ad61768acccd8e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/10/2019
ms.locfileid: "54202554"
---
# <a name="connect-a-bot-to-slack"></a>将机器人连接到 Slack

可以对机器人进行配置，以与使用 Slack 消息传递应用的用户进行通信。

## <a name="create-a-slack-application-for-your-bot"></a>为机器人创建 Slack 应用程序

登录到 [Slack](https://slack.com/signin)，然后转到[创建 Slack 应用程序](https://api.slack.com/apps)通道。

![设置机器人](~/media/channels/slack-NewApp.png)

## <a name="create-an-app-and-assign-a-development-slack-team"></a>创建应用并分配 Slack 开发团队

输入“应用名称”并选择“Slack 开发团队”。 如果你不是 Slack 开发团队的成员，请[创建或加入一个](https://slack.com/)。

![创建应用](~/media/channels/slack-CreateApp.png)

单击“创建应用”。 Slack 将创建应用，并生成客户端 ID 和客户端密码。

## <a name="add-a-new-redirect-url"></a>添加新的重定向 URL

接下来将添加新的重定向 URL。

1. 选择“OAuth 和权限”选项卡。
2. 单击“添加新的重定向 URL”。
3. 输入 https://slack.botframework.com 。
4. 单击“添加”。
5. 单击“保存 URL”。

![添加重定向 URL](~/media/channels/slack-RedirectURL.png)

## <a name="create-a-slack-bot-user"></a>创建 Slack 机器人用户

添加机器人用户让你能够为机器人分配用户名，并选择是否始终显示为在线。

1. 选择“机器人用户”选项卡。
2. 单击“添加机器人用户”。

![创建机器人](~/media/channels/slack-CreateBot.png)

单击“添加机器人用户”以验证你的设置，将“始终将我的机器人显示为在线”设置为“开启”，然后单击“保存更改”。

![创建机器人](~/media/channels/slack-CreateApp-AddBotUser.png)

## <a name="subscribe-to-bot-events"></a>订阅机器人事件

按照以下步骤订阅六个特别的机器人事件。 通过订阅机器人事件，将在你指定的 URL 上向你的应用通知用户活动。

> [!TIP]
> 机器人句柄是机器人的名称。 若要查找机器人的句柄，请访问 [https://dev.botframework.com/bots](https://dev.botframework.com/bots)，选择一个机器人，然后记录该机器人的名称。

1. 选择“事件订阅”选项卡。
2. 将“启用事件”设置为“开启”。
3. 在“请求 URL”中，输入 `https://slack.botframework.com/api/Events/{YourBotHandle}`，其中 `{YourBotHandle}` 是你的机器人句柄（不带括号）。 此示例中使用的机器人句柄是 **ContosoBot**。

   ![订阅事件：顶部](~/media/channels/slack-SubscribeEvents-a.png)

4. 在“订阅机器人事件”中，单击“添加机器人用户事件”。
5. 在事件列表中，选择以下六种事件类型：
    * `member_joined_channel`
    * `member_left_channel`
    * `message.channels`
    * `message.groups`
    * `message.im`
    * `message.mpim`

   ![订阅事件：中部](~/media/channels/slack-SubscribeEvents-b.png)

6. 单击“保存更改”。

   ![订阅事件：底部](~/media/channels/slack-SubscribeEvents-c.png)

## <a name="add-and-configure-interactive-messages-optional"></a>添加和配置交互式消息（可选）

如果机器人将使用特定于 Slack 的功能（例如按钮），请按照以下步骤操作：

1. 选择“交互式组件”选项卡，然后单击“启用交互式组件”。
2. 输入 `https://slack.botframework.com/api/Actions` 作为“请求 URL”。
3. 单击“保存更改”按钮。

![启用消息](~/media/channels/slack-MessageURL.png)

## <a name="gather-credentials"></a>收集凭据

选择“基本信息”选项卡，然后滚动到“应用凭据”部分。
将显示配置 Slack 机器人所需的客户端 ID、客户端密码和验证令牌。

![收集凭据](~/media/channels/slack-AppCredentials.png)

## <a name="submit-credentials"></a>提交凭据

在单独的浏览器窗口中，返回到 Bot Framework 站点 `https://dev.botframework.com/`。

1. 选择“我的机器人”，然后选择要连接到 Slack 的机器人。
2. 在“通道”部分，单击 Slack 图标。
3. 在“输入 Slack 凭据”部分，将 Slack 网站上的应用凭据粘贴到相应的字段中。
4. “登录页 URL”是可选的。 可以省略或对其进行更改。
5. 单击“ **保存**”。

![提交凭据](~/media/channels/slack-SubmitCredentials.png)

按照说明授权 Slack 应用对 Slack 开发团队的访问权限。

## <a name="enable-the-bot"></a>启用机器人

在“配置 Slack 页面”上，通过“保存”按钮确认滑块“已启用”。
机器人配置为与 Slack 中的用户通信。

## <a name="create-an-add-to-slack-button"></a>创建“添加到 Slack”按钮

Slack 提供了可以用来帮助 Slack 用户在[本页](https://api.slack.com/docs/slack-button)的“添加 Slack 按钮”部分找到机器人的 HTML。
若要将此 HTML 用于机器人，请将 href 值（以 `https://` 开头）替换为机器人的 Slack 通道设置中的 URL。
请按照以下步骤获取替换 URL。

1. 在 [https://dev.botframework.com/bots](https://dev.botframework.com/bots) 上，单击机器人。
2. 单击“通道”，右键单击名为“Slack”的条目，然后单击“复制链接”。 此 URL 现在位于剪贴板中。
3. 将剪贴板中的此 URL 粘贴到为 Slack 按钮提供的 HTML 中。 此 URL 替换 Slack 为此机器人提供的 href 值。

授权用户可以单击此修改后的 HTML 提供的“添加到 Slack”按钮，以便在 Slack 上与机器人联系。
