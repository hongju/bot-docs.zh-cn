---
title: 将机器人连接到 Facebook Messenger |Microsoft Docs
description: 了解如何配置机器人与 Facebook Messenger 的连接。
keywords: Facebook Messenger, 机器人通道, Facebook 应用, 应用 ID, 应用密码, Facebook 机器人, 凭据
author: RobStand
ms.author: RobStand
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 09/16/2018
ms.openlocfilehash: 60e39bb652ab5b7ffeeb5ba53bdf4c82f936553e
ms.sourcegitcommit: 3bf3dbb1a440b3d83e58499c6a2ac116fe04b2f6
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/23/2018
ms.locfileid: "46707863"
---
# <a name="connect-a-bot-to-facebook-messenger"></a>将机器人连接到 Facebook Messenger

若要了解有关 Facebook Messenger 开发的详细信息，请参阅 [Messenger 平台文档](https://developers.facebook.com/docs/messenger-platform)。 可能需要查看 Facebook 的[启动前准则](https://developers.facebook.com/docs/messenger-platform/product-overview/launch#app_public)、[快速入门](https://developers.facebook.com/docs/messenger-platform/guides/quick-start)和[设置指南](https://developers.facebook.com/docs/messenger-platform/guides/setup)。

若要将机器人配置为使用 Facebook Messenger 进行通信，请在 Facebook 页上启用 Facebook Messenger，然后将机器人连接到应用。

> [!NOTE]
> 显示的 Facebook UI 可能稍有不同，具体取决于所用版本。

## <a name="copy-the-page-id"></a>复制页 ID

可通过 Facebook 页访问机器人。 [创建新的 Facebook 页](https://www.facebook.com/bookmarks/pages)或转到现有页。

* 打开 Facebook 页的“关于”页，然后复制并保存页 ID。

## <a name="create-a-facebook-app"></a>创建 Facebook 应用

在页上[创建新的 Facebook 应用](https://developers.facebook.com/quickstarts/?platform=web)，并为其生成应用 ID 和应用密码。

![创建应用 ID](~/media/channels/FB-CreateAppId.png)

* 复制并保存应用 ID 和应用密码。

![保存应用 ID 和密码](~/media/channels/FB-get-appid.png)

将“允许对应用设置的 API 访问”滑块设置为“是”。

![应用设置](~/media/bot-service-channel-connect-facebook/api_settings.png)

## <a name="enable-messenger"></a>启用 Messenger


在新的 Facebook 应用中启用 Facebook Messenger。

* 在应用的“产品设置”页上，单击“开始”，然后再次单击“开始”。


![启用 Messenger](~/media/channels/FB-AddMessaging1.png)

## <a name="generate-a-page-access-token"></a>生成页访问令牌

在 Messenger 部分的“令牌生成”面板中，选择目标页。 此时将生成页访问令牌。

* 复制并保存“页访问令牌”。

![生成令牌](~/media/channels/FB-generateToken.png)

## <a name="enable-webhooks"></a>启用 Webhook

单击“设置 Webhook”，将消息事件从 Facebook Messenger 转发给机器人。

![启用 Webhook](~/media/channels/FB-webhook.png)

## <a name="provide-webhook-callback-url-and-verify-token"></a>提供 Webhook 回调 URL 及验证令牌

在 [Azure 门户](https://portal.azure.com/)中打开机器人，单击“通道”选项卡，然后单击“Facebook Messenger”。

* 从门户复制“回调 URL”和“验证令牌”值。

![复制值](~/media/channels/fb-callbackVerify.png)

1. 返回到 Facebook Messenger 并粘贴“回调 URL”和“验证令牌”值。

2. 在“订阅字段”下，选择“message\_deliveries”、“messages”、“messaging\_optins”和“messaging\_postbacks”。

3. 单击“验证并保存”。

![配置 Webhook](~/media/channels/FB-webhookConfig.png)

4. 将 Webhook 订阅到 Facebook 页。

![订阅 Webhook](~/media/bot-service-channel-connect-facebook/subscribe-webhook.png)


## <a name="provide-facebook-credentials"></a>提供 Facebook 凭据

在 Azure 门户中，粘贴之前从 Facebook Messenger 复制的“Facebook 应用 ID”、“Facebook 应用机密”、“页 ID”和“页访问令牌”值。 可以通过添加更多的页 ID 和访问令牌，在多个 Facebook 页上使用同一个机器人。

![输入凭据](~/media/channels/fb-credentials2.png)

## <a name="submit-for-review"></a>提交以供审阅

在 Facebook 基本应用设置页上，要求提供隐私策略 URL 和服务条款 URL。 [行为准则](https://aka.ms/bf-conduct)页包含第三方资源链接，用于帮助创建隐私策略。 [使用条款](https://aka.ms/bf-terms)页包含示例条款，用于帮助创建相应的服务条款文档。

机器人完成后，对于发布到 Messenger 的应用，Facebook 有其自己的[评审流程](https://developers.facebook.com/docs/messenger-platform/app-review)。 将对机器人进行测试，确保它符合 Facebook 的[平台策略](https://developers.facebook.com/docs/messenger-platform/policy-overview)。

## <a name="make-the-app-public-and-publish-the-page"></a>公开应用并发布页

> [!NOTE]
> 应用在发布之前处于[开发模式](https://developers.facebook.com/docs/apps/managing-development-cycle)。 插件和 API 功能仅适用于管理员、开发人员和测试人员。

评审成功后，请在“应用评审”下的“应用仪表板”中，将应用设置为“公开”。
确保发布了与此机器人关联的 Facebook 页。 状态会显示在页设置中。

> [!NOTE]
> 还可以使用 Facebook Workplace。 若要启用 Workplace，请为它创建[自定义集成](https://developers.facebook.com/docs/workplace/custom-integrations-new)，并使用其应用 ID、应用机密和访问令牌。 在应用的“关于”页上，不要使用传统的 pageID，而要使用编号后接集成名称。 可以使用 Azure 中显示的凭据连接 Webhook。
