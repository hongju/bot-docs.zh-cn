---
title: 将机器人连接到 Twilio | Microsoft Docs
description: 了解如何配置到 Twilio 的机器人连接。
keywords: Twilio, 机器人通道, SMS, 应用, 电话, 配置 Twilio, 云通信, 文本
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 7e09126d50cfbebfc0aad0ee7fcb71b4e7551a7d
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/18/2018
ms.locfileid: "39297717"
---
# <a name="connect-a-bot-to-twilio"></a>将机器人连接到 Twilio

可以将机器人配置为与使用 Twilio 云通信平台的用户进行通信。

## <a name="log-in-to-or-create-a-twilio-account-for-sending-and-receiving-sms-messages"></a>登录或创建 Twilio 帐户用于发送和接收短信

如果你没有 Twilio 帐户，请<a href="https://www.twilio.com/try-twilio" target="_blank">创建一个新帐户</a>

## <a name="create-a-twiml-application"></a>创建 TwiML 应用程序

<a href="https://www.twilio.com/user/account/messaging/dev-tools/twiml-apps/add" target="_blank">创建 TwiML 应用程序</a>

![创建应用](~/media/channels/twi-StepTwiml.png)

 “消息”下的请求 URL 应为 https://sms.botframework.com/api/sms。

## <a name="select-or-add-a-phone-number"></a>选择或添加电话号码

<a href="https://www.twilio.com/user/account/phone-numbers/incoming" target="_blank">选择或添加电话号码</a>。 单击数字，将其添加到你创建的 TwiML 应用程序。

![设置电话号码](~/media/channels/twi-StepPhone.png)

## <a name="specify-application-to-use-for-messaging"></a>指定用于“消息”的应用程序
在“消息”部分中，将“TwiML 应用”设置为刚创建的 TwiML 应用的名称。
复制电话号码值供以后使用。

![指定应用](~/media/channels/twi-StepPhone2.png)

## <a name="gather-credentials"></a>收集凭据

<a href="https://www.twilio.com/user/account/settings" target="_blank">收集凭据</a>，然后单击“眼睛”图标查看身份验证令牌。

![收集应用凭据](~/media/channels/twi-StepAuth.png)

## <a name="submit-credentials"></a>提交凭据

输入电话号码、accountSID 和之前复制的身份验证令牌，然后单击“提交 Twilio 凭据”。

## <a name="enable-the-bot"></a>启用机器人
勾选“在短信上启用此机器人”。 然后单击“我已经完成了短信配置”。

完成这些步骤后，就能够将机器人成功配置为与使用 Twilio 的用户通信。

