---
title: 将机器人连接到 Twilio | Microsoft Docs
description: 了解如何配置到 Twilio 的机器人连接。
keywords: Twilio, 机器人通道, SMS, 应用, 电话, 配置 Twilio, 云通信, 文本
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 10/9/2018
ms.openlocfilehash: 7d7416940ccad4e62c98f4a386dac43189301b56
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998296"
---
# <a name="connect-a-bot-to-twilio"></a>将机器人连接到 Twilio

可以将机器人配置为与使用 Twilio 云通信平台的用户进行通信。

## <a name="log-in-to-or-create-a-twilio-account-for-sending-and-receiving-sms-messages"></a>登录或创建 Twilio 帐户用于发送和接收短信

如果没有 Twilio 帐户，请<a href="https://www.twilio.com/try-twilio" target="_blank">创建一个新帐户</a>。

## <a name="create-a-twiml-application"></a>创建 TwiML 应用程序

按说明<a href="https://support.twilio.com/hc/en-us/articles/223180928-How-Do-I-Create-a-TwiML-App-" target="_blank">创建 TwiML 应用程序</a>。

![创建应用](~/media/channels/twi-StepTwiml.png)

在“属性”下输入一个**友好名称**。 在本教程中，我们使用“我的 TwiML 应用”作为示例。 “语音”下的“请求 URL”可以留空。 “消息”下的请求 URL 应为 `https://sms.botframework.com/api/sms`。

## <a name="select-or-add-a-phone-number"></a>选择或添加电话号码

按照<a href = "https://support.twilio.com/hc/en-us/articles/223180048-Adding-a-Verified-Phone-Number-or-Caller-ID-with-Twilio" target="_blank">此处</a>的说明通过控制台站点添加经验证的呼叫方 ID。 完成后，会在“管理号码”下的“活动号码”中看到经验证的号码。

![设置电话号码](~/media/channels/twi-StepPhone.png)

## <a name="specify-application-to-use-for-voice-and-messaging"></a>指定用于语音和消息的应用程序

单击号码，转到“配置”。 在“语音”和“消息”下，将“配置方式”设置为“TwiML 应用”，将“TWIML 应用”设置为“我的 TwiML 应用”。 完成后，单击“保存”。

![指定应用程序](~/media/channels/twi-StepPhone2.png)

返回到“管理号码”，会看到“语音”和“消息”的配置都已更改为“TwiML 应用”。

![指定的号码](~/media/channels/twi-StepPhone3.png)


## <a name="gather-credentials"></a>收集凭据

返回到[控制台主页](https://www.twilio.com/console/)，此时会看到项目仪表板上的帐户 SID 和身份验证令牌，如下所示。

![收集应用凭据](~/media/channels/twi-StepAuth.png)

## <a name="submit-credentials"></a>提交凭据

在单独的窗口中，返回到 Bot Framework 站点 https://dev.botframework.com/。 

- 选择“我的机器人”，然后选择要连接到 Twilio 的机器人。 此时会转到 Azure 门户。
- 在“机器人管理”下选择“通道”。 单击“Twilio (短信)”图标。
- 输入电话号码、帐户 SID 和之前记录的身份验证令牌。 完成后，单击“保存”。

![提交凭据](~/media/channels/twi-StepSubmit.png)

完成这些步骤后，就能够将机器人成功配置为与使用 Twilio 的用户通信。