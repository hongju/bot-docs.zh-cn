---
title: 将机器人连接到 Direct Line 语音（预览版）
titleSuffix: Bot Service
description: 概述如何通过必要的步骤将现有的 Bot Framework 机器人连接到 Direct Line 语音通道，以便进行双向语音交互，确保高可用性和低延迟。
services: bot-service
author: trrwilson
manager: nitinme
ms.service: bot-service
ms.subservice: bot-service
ms.topic: conceptual
ms.date: 05/02/2019
ms.author: travisw
ms.custom: ''
ms.openlocfilehash: 8e0d2939078e1e27162c7056373e95790a03eb88
ms.sourcegitcommit: 5042e31bc6b2762d7a6636e98c8f496b90ea33c1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65240431"
---
# <a name="connect-a-bot-to-direct-line-speech-preview"></a>将机器人连接到 Direct Line 语音（预览版）

[!INCLUDE[applies-to-v4](includes/applies-to.md)]

可以将机器人配置为允许客户端应用程序通过 Direct Line 语音通道与之通信。

生成机器人以后，通过 Direct Line 语音载入它即可使用[语音 SDK](https://aka.ms/speech/sdk) 与客户端应用程序进行低延迟、高可靠性连接。 这些连接已针对双向语音聊天体验进行优化。 若要详细了解 Direct Line 语音以及如何生成客户端应用程序，请访问[自定义语音优先虚拟助理](https://aka.ms/bots/speech/va)页。  

## <a name="sign-up-for-direct-line-speech-preview"></a>注册 Direct Line 语音预览版

Direct Line 语音目前为预览版，要求用户在 [Azure 门户](https://portal.azure.com)中进行快速注册。 请查看下面的详细信息。 获得批准后，即可访问通道。

## <a name="add-the-direct-line-speech-channel"></a>添加 Direct Line 语音通道

1. 若要添加 Direct Line 语音通道，请先在 [Azure 门户](https://portal.azure.com)中打开机器人，然后在配置边栏选项卡中单击“通道”。

    ![突出显示选择要连接的通道时的位置](media/voice-first-virtual-assistants/bot-service-channel-directlinespeech-selectchannel.png "选择通道")

1. 在通道选择页中找到并单击“`Direct Line Speech`”，以便选择通道。

    ![选择 Direct Line 语音通道](media/voice-first-virtual-assistants/bot-service-channel-directlinespeech-connectspeechchannel.png "连接 Direct Line 语音")

1. 如果你还未获允进行访问，则会看到一个请求访问权限的页面。 请填写系统请求的信息，然后单击“请求”。 此时会显示确认页。 在请求等待审批的时候，你只能停留在此页上。   

1. 获允进行访问以后，将会显示 Direct Line 语音的配置页。 查看使用条款以后，请单击“`Save`”，确认所选择的通道。

    ![保存 Direct Line 语音通道的启用设置](media/voice-first-virtual-assistants/bot-service-channel-directlinespeech-savechannel.png "保存通道配置")

## <a name="enable-the-bot-framework-protocol-streaming-extensions"></a>启用 Bot Framework 协议流式处理扩展

将 Direct Line 语音通道连接到机器人以后，现在需要启用 Bot Framework 协议流式处理扩展支持，以便进行优化的低延迟交互。

1. 在 [Azure 门户](https://portal.azure.com)中打开机器人的边栏选项卡（如果尚未打开）。 

1. 单击“机器人管理”类别下的“设置”（位于“通道”正下方）。 单击“启用流式处理终结点”所对应的复选框。

    ![启用流式处理协议](media/voice-first-virtual-assistants/bot-service-channel-directlinespeech-enablestreamingsupport.png "启用流式处理扩展支持")

1. 在页面顶部，单击“保存”。

1. 在同一边栏选项卡的“应用服务设置”类别下，单击“配置”。

    ![导航到应用服务设置](media/voice-first-virtual-assistants/bot-service-channel-directlinespeech-configureappservice.png "配置应用服务")

1. 单击“`General settings`”，然后选择启用 `Web socket` 支持所需的选项。

    ![为应用服务启用 websocket](media/voice-first-virtual-assistants/bot-service-channel-directlinespeech-enablewebsockets.png "启用 websocket")

1. 单击配置页顶部的“`Save`”。

1. 现在已为机器人启用 Bot Framework 协议流式处理扩展。 现在可以更新机器人代码并[将流式处理扩展支持集成](https://aka.ms/botframework/addstreamingprotocolsupport)到现有机器人项目了。

## <a name="manage-secret-keys"></a>管理密钥

客户端应用程序将需要一个通道机密，以便通过 Direct Line 语音通道连接到机器人。 保存通道选择设置以后，即可在 Azure 门户的“配置 Direct Line 语音”页中检索这些密钥。

![获取 Direct Line 语音的密钥](media/voice-first-virtual-assistants/bot-service-channel-directlinespeech-getspeechsecretkeys.png "获取 Direct Line 语音的密钥")

## <a name="adding-protocol-support-to-your-bot"></a>向机器人添加协议支持

连接 Direct Line 语音通道并启用对 Bot Framework 协议流式处理扩展的支持以后，剩下的就是向机器人添加代码，为优化的通信提供支持。 按说明[向机器人添加流式处理扩展支持](https://aka.ms/botframework/addstreamingprotocolsupport)，确保完全兼容 Direct Line 语音。

## <a name="known-issues"></a>已知问题

请注意，此服务为预览版并且可能会更改，因此可能会影响你的机器人开发和总体性能。 下面是已知问题的列表： 

1. 此服务目前部署到“美国西部 2”[Azure 区域](https://azure.microsoft.com/en-us/global-infrastructure/regions/)。 我们很快会将服务推广到其他区域，这样所有客户就能够使用其机器人进行低延迟的语音交互。

1. 以后会对控制字段（例如 [serviceUrl](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#service-url)）进行小的更改

1. 将会更新用于指示聊天的开始和结束的 [conversationUpdate](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#conversation-update-activity) 和 [endOfCoversation](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#end-of-conversation-activity) 活动（通常用于生成欢迎消息），使之与其他通道保持一致

1. 此通道目前不支持 [SigninCard](https://docs.microsoft.com/en-us/azure/bot-service/rest-api/bot-framework-rest-connector-add-rich-cards?view=azure-bot-service-4.0) 
