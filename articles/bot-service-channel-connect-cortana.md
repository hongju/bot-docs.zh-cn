---
title: 将机器人连接到 Cortana | Microsoft Docs
description: 了解如何配置机器人以实现通过 Cortana 接口进行访问。
keywords: cortana, 机器人通道, 配置 cortana
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 04/30/2018
ms.openlocfilehash: 6e694ce8b54ebd2405d7496d333c2bb27eb344f1
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/18/2018
ms.locfileid: "39297852"
---
# <a name="connect-a-bot-to-cortana"></a>将机器人连接到 Cortana

Cortana 是支持语音的通道，除文本聊天外，它还可以发送和接收语音消息。 要连接到 Cortana 的机器人应支持语音和文本。 Cortana 技能是可使用 Cortana 客户端调用的机器人。 发布机器人会将机器人添加到可用技能列表中。

若要添加 Cortana 通道，在 [Azure 门户](https://portal.azure.com/)中打开机器人，单击“通道”边栏选项卡，然后单击“Cortana”。

![添加 Cortana 通道](~/media/channels/cortana-addchannel.png)

## <a name="configure-cortana"></a>配置 Cortana

将机器人与 Cortana 通道连接时，机器人的某些基本信息将预先填入到注册表单中。 仔细查看此信息。 此表单由以下字段组成。

| 字段 | Description |
|------|------|
| **技能图标** | 调用技能时 Cortana 画布中显示的图标。 这也适用于可发现这些技能的地方（如 Microsoft Store）。 （最大 32KB，仅限 PNG）。|
| **显示名称** | 在可视 UI 顶部向用户显示的 Cortana 技能名称。 （限 30 个字符） |
| **调用名** | 这是用户调用技能时所说的名字。 它不应超过三个单词，且需易于发音。 有关如何选择此名称的详细信息，请参阅[调用名指南][invocation]。|
| **说明** | Cortana 技能说明。 这用于可发现该技能的地方（如 Microsoft Store）。 |
| **简短说明** | 技能功能的简短说明，用于在 Cortana 的笔记本中描述该技能。 |

## <a name="general-bot-information"></a>常规机器人信息

在“通过连接服务管理用户标识”部分，按下该选项进行启用。 填写表单。

带有星号 (*) 的所有字段为必填项。 机器人必须先在 Bot Framework 上发布，然后才能连接到 Cortana。

![提供常规信息](~/media/channels/cortana-details.png)

### <a name="sign-in-at-invocation"></a>调用时登录

如果希望 Cortana 在用户调用你的技能时登录用户，请选中此选项。

### <a name="sign-in-when-required"></a>必要时登录

如果使用 Bot Framework 的登录卡登录用户，请选中此选项。 通常情况下，如果希望仅在用户使用需要身份验证的功能时才登录用户，则使用此选项。 如果技能发送的消息中包含登录卡附件，Cortana 将忽略登录卡并使用“连接帐户”设置执行授权流。

### <a name="connected-service-icon"></a>连接服务图标

希望在用户登录技能时显示的图标。

### <a name="account-name"></a>帐户名

希望在用户登录技能时显示的技能名称。

### <a name="client-id-for-third-party-services"></a>第三方服务的客户端 ID

机器人的应用程序 ID。 注册机器人时将收到此 ID。

### <a name="space-separated-list-of-scopes"></a>以空格分隔的作用域列表

指定服务所需的作用域（请参阅服务文档）。

### <a name="authorization-url"></a>授权 URL

设置为 `https://login.microsoftonline.com/common/oauth2/v2.0/authorize`。

### <a name="grant-type"></a>授权类型

选择“授权代码”，则使用代码授予流。 选择“隐式”，则使用隐式流。

### <a name="token-url"></a>令牌 URL

如果选择“授权代码”，请设置为 `https://login.microsoftonline.com/common/oauth2/v2.0/token`。

### <a name="client-secretpassword-for-third-party-services"></a>第三方服务的客户端机密/密码

机器人密码。 注册机器人时将收到此密码。

### <a name="client-authentication-scheme"></a>客户端身份验证方案

选择 HTTP 基本。

### <a name="token-options"></a>令牌选项

设置为 POST。

### <a name="request-user-profile-data-optional"></a>请求用户配置文件数据（可选）

Cortana 提供对几种不同类型的用户配置文件信息的访问，可使用这些信息为用户自定义机器人。 例如，如果技能可以访问用户的姓名和位置，那么技能可具有自定义响应，例如“你好，Kamran，我希望你在华盛顿州贝尔维尤度过愉快的一天”。

单击“添加用户配置文件请求”链接，然后从下拉列表中选择所需的用户配置文件信息。 添加友好名称以用于从机器人代码中访问此信息。

### <a name="save-skill"></a>保存技能

填写完 Cortana 技能注册表单后，单击“保存”以完成连接。 此操作将返回到机器人的“通道”边栏选项卡，可看到它现在已连接到 Cortana。

此时，机器人已作为 Cortana 技能自动部署到你的帐户。

## <a name="next-steps"></a>后续步骤

* [Cortana 技能套件](https://aka.ms/CortanaSkillsDocs)
* [启用调试](bot-service-debug-cortana-skill.md)
* [发布 Cortana 技能][publish]

[invocation]: https://docs.microsoft.com/en-us/cortana/skills/cortana-invocation-guidelines
[publish]: https://docs.microsoft.com/en-us/cortana/skills/publish-skill
[connected]: https://aka.ms/CortanaSkillsBotConnectedAccount
[CortanaEntity]: https://aka.ms/lgvcto
