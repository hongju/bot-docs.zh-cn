---
title: Azure 机器人服务中的用户身份验证 | Microsoft Docs
description: 了解 Azure 机器人服务中的用户身份验证功能。
keywords: azure 机器人服务, 身份验证, bot framework token service
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: abs
ms.date: 05/31/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 6eea954f58096d89cd3278058146b93fa04f435f
ms.sourcegitcommit: e276008fb5dd7a37554e202ba5c37948954301f1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/05/2019
ms.locfileid: "66693757"
---
# <a name="user-authentication-within-a-conversation"></a>聊天中的用户身份验证

若要代表用户执行某些操作，例如查看电子邮件、引用日历、查看航班状态或者下单，机器人需要调用外部服务，例如 Microsoft Graph、GitHub 或某个公司的 REST 服务。
每项外部服务都可以通过某种方式来保护这些调用的安全。通常情况下，保护此类调用安全的方式是使用_用户令牌_（有时称为 JWT）发出这些请求，此令牌能够以唯一方式标识该外部服务上的用户。

为了安全地调用外部服务，机器人必须要求用户登录，以便获取用户用于该服务的令牌。
许多服务支持通过 OAuth 或 OAuth2 协议检索令牌。
Azure 机器人服务提供专用登录卡和服务来处理 OAuth 协议并管理令牌生命周期。机器人可以使用这些功能获取用户令牌。

- OAuth 连接设置  作为机器人配置的一部分注册到 Azure 中的 Azure 机器人服务资源。

    每项连接设置包含要使用的外部服务或标识提供者的相关信息、有效的 OAuth 客户端 ID 和机密、要启用的 OAuth 范围，以及该外部服务或标识提供者需要的任何其他连接元数据。

- 在机器人的代码中，OAuth 连接设置用于用户登录和获取用户令牌。

登录工作流涉及以下服务。

![身份验证概述](./media/bot-builder-concept-authentication.png)

## <a name="about-the-bot-framework-token-service"></a>关于 Bot Framework Token Service

Bot Framework Token Service 负责以下事项：

- 简化 OAuth 协议与各种外部服务的配合使用。
- 安全地存储特定机器人、通道、聊天和用户的令牌。
- 管理令牌生命周期，包括尝试刷新令牌。

例如，如果机器人可以使用 Microsoft Graph API 检查用户最近的电子邮件，则该机器人需要 Azure Active Directory 用户令牌。 在设计时，机器人开发人员会通过 Azure 门户将 Azure Active Directory 应用程序注册到 Bot Framework Token Service，然后为机器人配置名为 `GraphConnection` 的 OAuth 连接设置。 当用户与机器人交互时，工作流如下：

1. 用户向机器人提出要求：“请检查我的电子邮件”。
1. 系统会将包含此消息的活动从用户发送到 Bot Framework Channel Service。 此通道服务可确保活动中的 `userid` 字段已设置且消息会发送给机器人。

    用户 ID 特定于通道，例如用户的 Facebook ID 或其短信的手机号。

1. 机器人收到消息活动后，确定该消息的意向是检查用户的电子邮件。 机器人向 Bot Framework Token Service 提交请求，询问其是否已经有 OAuth 连接设置 `GraphConnection` 对应的 UserId 的令牌。
1. 由于这是该用户首次与机器人交互，Bot Framework Token Service 还没有该用户的令牌，因此为机器人返回 _NotFound_ 结果。
1. 机器人创建连接名称为 `GraphConnection` 的 OAuthCard 并回复用户，要求用户使用该卡登录。
1. 活动会传经 Bot Framework Channel Service，后者会调用 Bot Framework Token Service，为该请求创建有效的 OAuth 登录 URL。 此 URL 会添加到 OAuthCard 中，该卡会返回给用户。
1. 系统会向用户显示一条消息，要求其通过单击 OAuthCard 的登录按钮来登录。
1. 当用户单击登录按钮时，通道服务会打开 Web 浏览器，并调用外部服务来加载其登录页。
1. 用户登录到外部服务的该页面。 登录完成后，外部服务就完成了与 Bot Framework Token Service 的 OAuth 协议交换，结果就是外部服务向 Bot Framework Token Service 发送用户令牌。 Bot Framework Token Service 会安全地存储此令牌，并向机器人发送包含此令牌的活动。
1. 机器人在收到包含此令牌的活动后，就可以使用它针对 Graph API 进行调用。

## <a name="securing-the-sign-in-url"></a>确保登录 URL 的要求

通过 Bot Framework 方便用户登录的同时，需要考虑的重要事项是如何确保登录 URL 的安全。 系统为用户提供登录 URL 时，此 URL 与该机器人的特定聊天 ID 和用户 ID 相关联。 此 URL 不应共享，因为它会导致特定机器人聊天出现登录错误。 为了减少因共享登录 URL 而导致的安全攻击，必须确保单击登录 URL 的计算机用户是使用相应聊天窗口的用户。 

某些通道（例如 Cortana、Teams、Direct Line、WebChat）可以在用户注意不到的情况下实现这一点。 例如，WebChat 使用会话 Cookie 来确保登录流与 WebChat 聊天在同一浏览器中进行。 但是，在其他通道中，系统通常会为用户提供一个 6 位数的幻码。  这类似于内置的多重身份验证，因为 Bot Framework Token Service 在用户完成最终身份验证之前不会将令牌放出给机器人，而已登录的用户必须输入 6 位数的代码来证明自己能够访问聊天体验。

## <a name="azure-activity-directory-application-registration"></a>Azure Activity Directory 应用程序注册

每个注册为 Azure 机器人服务的机器人都使用一个 Azure Activity Directory (AD) 应用程序 ID。 **不得**重复使用此应用程序 ID 和密码来登录用户。 Azure 机器人服务的 Azure AD 应用程序 ID 用于确保在机器人和 Bot Framework Channel Service 之间进行安全的服务到服务通信。 若要将用户登录到 Azure AD，则应使用适当的权限和范围创建单独的 Azure AD 应用程序注册。

## <a name="configure-an-oauth-connection-setting"></a>配置 OAuth 连接设置

若要详细了解如何注册和使用 OAuth 连接设置，请参阅[向机器人添加身份验证](bot-builder-authentication.md)。
