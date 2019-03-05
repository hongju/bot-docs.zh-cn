---
title: 将机器人连接到 LINE | Microsoft Docs
description: 了解如何配置机器人与 LINE 的连接。
keywords: 连接机器人, 机器人通道, LINE 机器人, 凭据, 配置, 手机
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 1/7/2019
ms.openlocfilehash: f6ed40c5ad3999ea5a123bf051de849917f22b42
ms.sourcegitcommit: e41dabe407fdd7e6b1d6b6bf19bef5f7aae36e61
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/27/2019
ms.locfileid: "56893507"
---
# <a name="connect-a-bot-to-line"></a>将机器人连接到 LINE

可以对机器人进行配置，以便通过 LINE 应用与人通信。

## <a name="log-into-the-line-console"></a>登录到 LINE 控制台

通过“使用 Line 登录”登录到 LINE 帐户的 [LINE 开发人员控制台](https://developers.line.biz/console/register/messaging-api/provider/)。 

> [!NOTE]
> [下载 LINE](https://line.me/)（如果尚未下载），然后转到设置，注册电子邮件地址。

### <a name="register-as-a-developer"></a>注册为开发人员

如果这是第一次进入 LINE 开发人员控制台，请输入名称和电子邮件地址，创建一个开发人员帐户。

![LINE 屏幕截图：注册开发人员](./media/channels/LINE-screenshot-1.png)

## <a name="create-a-new-provider"></a>创建新的提供商

首先，为机器人创建一个提供商（如果尚未设置）。 提供商是提供应用的实体（个人或公司）。

![LINE 屏幕截图：创建提供商](./media/channels/LINE-screenshot-2.png)

## <a name="create-a-messaging-api-channel"></a>创建消息传送 API 通道

接下来，创建新的消息传送 API 通道。 

![LINE 屏幕截图：通道类型](./media/channels/LINE-channel-type-selection.png)

通过单击绿色方块，创建新的消息传送 API 通道。

![LINE 屏幕截图：创建通道](./media/channels/LINE-create-channel.png)

此名称不能包含“LINE”或某些类似的字符串。 填写必填字段，确认通道设置。

![LINE 屏幕截图：通道设置](./media/channels/LINE-screenshot-4.png)

## <a name="get-necessary-values-from-your-channel-settings"></a>或者通道设置中的必需值

待你确认通道设置以后，系统会将你定向到一个类似于以下页面的页面。

![LINE 屏幕截图：通道页面](./media/channels/LINE-screenshot-5.png)

单击创建的通道以访问通道设置，向下滚动，找到“基本信息”>“通道机密”。 将该内容暂时保存在某个位置。 验证“可用功能”是否包含 `PUSH_MESSAGE`。

![LINE 屏幕截图：通道机密](./media/channels/LINE-screenshot-6.png)

然后继续向下滚动，找到“消息传送设置”。 在该处会看到“通道访问令牌”字段和“问题”按钮。 单击该按钮以获取访问令牌，将其也暂时保存一下。

![LINE 屏幕截图：通道令牌](./media/channels/LINE-screenshot-8.png)

## <a name="connect-your-line-channel-to-your-azure-bot"></a>将 LINE 通道连接到 Azure 机器人

登录到 [Azure 门户](https://portal.azure.com/)并找到机器人，然后单击“通道”。 

![LINE 屏幕截图：Azure 设置](./media/channels/LINE-channel-setting-2.png)

在该处选择 LINE 通道，并将上面的通道机密和访问令牌粘贴到相应字段中。 确保保存所做的更改。

复制 Azure 提供的自定义 Webhook URL。

![LINE 屏幕截图：Azure 设置](./media/channels/LINE-channel-setting-1.png)

## <a name="configure-line-webhook-settings"></a>配置 LINE Webhook 设置

接下来回到 LINE 开发人员控制台，将 Webhook URL 从 Azure 粘贴到“消息设置”>“Webhook URL”，然后单击“验证”对连接进行验证。 如果刚在 Azure 中创建通道，可能需要几分钟才能生效。

然后，启用“消息设置”>“使用 Webhook”。

> [!IMPORTANT]
> 在 LINE 开发人员控制台中，必须先设置 Webhook URL，然后再将“使用 Webhook”设置为“启用”。 第一次使用空 URL 来启用 Webhook 不会设置“启用”状态，即使 UI 显示其他内容。

添加 Webhook URL 并启用 Webhook 以后，请务必重新加载此页，验证这些更改是否已正确设置。

![LINE 屏幕截图：Webhook](./media/channels/LINE-screenshot-9.png)

## <a name="test-your-bot"></a>测试机器人

完成这些步骤后，机器人就已经成功配置为与 LINE 上的用户通信，可以进行测试了。

### <a name="add-your-bot-to-your-line-mobile-app"></a>将机器人添加到 LINE 移动应用

在 LINE 开发人员控制台中导航到设置页，此时会看到机器人的 QR 码。 

在移动 LINE 应用中，转到最右侧的带有三个点 [**...**] 的导航选项卡，点击 QR 码图标。 

![LINE 屏幕截图：移动应用](./media/channels/LINE-screenshot-12.jpg)

将 QR 码读取器对准开发人员控制台中的 QR 码。 现在应该能够与移动 LINE 应用中的机器人交互并测试机器人。

### <a name="automatic-messages"></a>自动消息

开始测试机器人时，可能会发现机器人会发送意外消息，这些意外消息不是你在 `conversationUpdate` 活动中指定的。  对话可能会如下所示：

![LINE 屏幕截图：聊天](./media/channels/LINE-screenshot-conversation.jpg)

为了避免发送这些消息，需关闭自动响应消息。

![LINE 屏幕截图：自动响应](./media/channels/LINE-screenshot-10.png)

也可选择保留这些消息。 在这种情况下，可以单击“设置消息”并对其进行编辑。

![LINE 屏幕截图：设置自动响应](./media/channels/LINE-screenshot-11.png)

## <a name="troubleshooting"></a>故障排除

* 如果机器人根本不响应你的任何消息，请在 Azure 门户中导航到机器人，然后选择“在 Web 聊天中测试”。  
    * 如果机器人可以在该处正常使用，但在 LINE 中不响应，请重新加载 LINE 开发人员控制台页面，并按上面的 Webhook 说明重新操作。 确保在启用 Webhook 之前设置“Webhook URL”。
    * 如果机器人在“Web 聊天”中无法使用，请调试机器人的该问题，然后返回来完成 LINE 通道的配置。

