---
title: 配置语音启动 | Microsoft Docs
description: 了解如何使用 Azure 门户为机器人服务配置语音启动。
keywords: 语音启动, 语音识别, LUIS
author: v-royhar
ms.author: v-royhar
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: cognitive-services
ms.date: 12/13/2017
ms.openlocfilehash: 5cb47be530f9f82d83272684e6405730c72f3cb7
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997145"
---
# <a name="configure-speech-priming"></a>配置语音启动

语音启动改进了对机器人中常用的口头字词和短语的识别。 对于使用网上聊天和 Cortana 通道的启用语音的机器人，语音启动使用语言理解 (LUIS) 应用中指定的示例来提高重要字词的语音识别准确性。

你的机器人可能已经与 LUIS 应用集成，或者你可以选择创建 LUIS 应用与你的机器人关联以进行语音启动。 LUIS 应用包含你预期用户对你的机器人说的内容的示例。 应将你希望机器人识别的重要字词标记为实体。 例如，在国际象棋机器人中，你希望确保当用户说“Move knight”时，它不会被解释为“Move night”。 LUIS 应用应包含“knight”被标记为实体的示例。

> [!NOTE]
> 若要在网上聊天通道中使用语音启动，则必须使用必应语音服务。 有关如何使用必应语音服务的说明，请参阅[在网上聊天通道中启用语音](~/bot-service-channel-connect-webchat-speech.md)。

> [!IMPORTANT]
> 语音启动仅适用于为 Cortana 通道或网上聊天通道配置的机器人。

> [!IMPORTANT]
> 非美国区域 LUIS 应用不支持启动，包括：eu.luis.ai 和 au.luis.ai

## <a name="change-the-list-of-luis-apps-your-bot-uses"></a>更改你的机器人使用的 LUIS 应用列表

若要更改必应语音和机器人使用的 LUIS 应用列表，请执行以下操作：

1. 单击机器人服务边栏选项卡上的“语音启动”。 将显示你可以使用的 LUIS 应用列表。
2. 选择你希望必应语音使用的 LUIS 应用。
 
    a.在“解决方案资源管理器”中，右键单击项目文件夹下的“引用”文件夹，并单击“添加引用”。 若要在列表中选择 LUIS 应用，请将鼠标悬停在 LUIS 模型上，直到出现一个复选框，然后选中此复选框。
     
    b. 若要选择不在列表中的 LUIS 应用，请滚动到底部并在文本框中输入 LUIS 应用程序 ID GUID。
     
3. 单击“保存”以保存用于机器人的与必应语音关联的 LUIS 应用列表。

![语音启动面板](~/media/bot-service-manage-speech-priming/speech-priming.png)

## <a name="additional-resources"></a>其他资源

- 若要了解有关在网上聊天中启用语音的详细信息，请参阅[在网上聊天通道中启用语音](~/bot-service-channel-connect-webchat-speech.md)。
- 若要了解有关语音启动的详细信息，请参阅 [Bot Framework 中的语音支持 – 网上聊天到 Directline，再到 Cortana](https://blog.botframework.com/2017/06/26/Speech-To-Text/)。
- 若要了解有关 LUIS 应用的详细信息，请参阅[语言理解智能服务](https://www.luis.ai)。
