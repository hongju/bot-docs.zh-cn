---
title: 关于机器人服务 | Microsoft Docs
description: 了解机器人服务，一种用于构建、连接、测试、部署、监视和管理机器人的服务。
keywords: 概述, 简介, SDK, 大纲
author: Kaiqb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 05/03/2018
ms.openlocfilehash: b6326ac152112ff1df01470db1f525d4bf241af4
ms.sourcegitcommit: 67445b42796d90661afc643c6bb6533e9a662cbc
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/06/2018
ms.locfileid: "39574603"
---
::: moniker range="azure-bot-service-3.0"

# <a name="azure-bot-service"></a>Azure 机器人服务

[!INCLUDE [pre-release-label](includes/pre-release-label-v3.md)]

Azure 机器人服务提供的工具可用于在一个位置构建、测试、部署和管理智能机器人。 通过由 SDK 提供的模块化和可扩展框架，开发人员可以利用模板制作机器人，提供语音、语言理解、提问和解答等功能。  

## <a name="what-is-a-bot"></a>什么是机器人？
机器人是用户使用文本、图形（卡片）或语音通过聊天的方式进行交互的应用。 它可以是一个简单的问答对话，也可以是一个复杂的机器人，允许用户使用模式匹配、状态跟踪和与现有业务服务完美集成的人工智能技术通过智能的方式与服务进行交互。 请查看机器人[案例研究](https://azure.microsoft.com/services/bot-service/)。  

## <a name="building-a-bot"></a>构建机器人 
可以选择最喜欢的开发环境或命令行工具在 C# 或 Node.js 中构建机器人。 我们提供用户机器人开发各个阶段的工具，可用来构架机器人，帮助你入门。    

![机器人概述](media/bot-service-overview.png) 

## <a name="plan"></a>计划 
在编写代码之前，请参阅[设计指南](bot-service-design-principles.md) ，了解最佳做法并确定机器人的需求。 可以构建简单的机器人或添加更复杂的功能，例如，语音、语言理解、QnA 或从不同源中提取支持并提供智能答案的能力。  

> [!TIP] 
>
> 安装所需的模板：
>  - Bot Builder .NET SDK v3 [VSIX 模板](https://marketplace.visualstudio.com/items?itemName=BotBuilder.BotBuilderV3) 
>  - Bot Builder Node.js SDK v3 [Yeoman 模板](https://www.npmjs.com/package/generator-botbuilder) 
>
> 安装工具：
> - 下载 [CLI 工具](https://github.com/Microsoft/botbuilder-tools)，创建和管理机器人资产。 这些工具有助于通过命令行管理机器人配置文件、LUIS 应用、QnA 知识库等。 可在[自述文件](https://github.com/Microsoft/botbuilder-tools/blob/master/README.md)中找到更多详细信息。
> - [模拟器](https://github.com/Microsoft/BotFramework-Emulator/releases)，用于测试机器人
>
> 如果需要，请使用机器人组件，例如：  
> - [LUIS](https://www.luis.ai/)，以向机器人添加语言理解
> - [QnA Maker](https://qnamaker.ai/)，以更自然的聊天方式回答用户的问题
> - [语音](https://azure.microsoft.com/services/cognitive-services/speech/)，将音频转换为文本，了解目的，然后将文本转换回语音  
> - [拼写检查](https://azure.microsoft.com/services/cognitive-services/spell-check/)，帮助用户更正拼写错误、识别姓名、品牌名和俚语之间的区别 
> - [认知服务](bot-service-concept-intelligence.md)，针对其他各种智能组件 


## <a name="build-your-bot"></a>构建机器人 
机器人是一项 Web 服务，可实现聊天式界面并与机器人服务通信。 可以采用任意数量的环境和语言创建此解决方案，并且我们针对 Visual Studio 或 Yeoman，或者直接在 Azure 门户中提供了轻松入门工具。 请参阅以下内容，了解可以使用的部分工具和服务。

> [!TIP]
>
> 使用 [SDK](~/dotnet/bot-builder-dotnet-quickstart.md)、[Azure 门户](bot-service-quickstart.md)或使用 [CLI 工具](~/bot-builder-create-templates.md)构建机器人。
>
> 添加组件： 
> - 添加语言理解模型 [LUIS](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/home)。 
> - 添加 [QnA Maker](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/home) 知识库，回答用户提出的问题。  
> - 使用卡片、语音或转换增强用户体验。 
> - 使用 Bot Builder SDK 向机器人添加逻辑。   

## <a name="test-your-bot"></a>测试机器人 
机器人是复杂的应用，有大量不同的协同工作的部件。 就像其他复杂应用一样，这可能会导致出现一些需要关注的 Bug，或者会导致机器人的行为异常。 发布前，请先测试机器人。

> [!TIP] 
>
> - [使用模拟器测试机器人](bot-service-debug-emulator.md)
> - [通过网上聊天测试机器人](bot-service-manage-test-webchat.md)

## <a name="publish"></a>发布 
准备就绪后，可将机器人发布到 Azure 或自己的 Web 服务或数据中心。 可以设置持续部署，以便在本地开发机器人，而且这种部署在机器人签入到 GitHub 或 Visual Studio Team Services 等源控件时也非常有用。 返回源存储库查看更改时，所做的更改将自动部署到 Azure。

> [!Tip]
>
> - [部署到 Azure](bot-service-build-continuous-deployment.md)

## <a name="connect"></a>连接          
将机器人连接到 Facebook、Messenger、Kik、Skype、Slack、Microsoft Teams、Telegram、短信、Twilio、Cortana 和 Skype 等通道，以增加交互并吸引更多客户。  
  
> [!TIP]
>
> - [选择要添加的通道](bot-service-manage-channels.md)


## <a name="evaluate"></a>评估 
使用 Azure 门户中收集的数据确定改善机器人功能和性能的机会。 可以获得服务级和检测数据，如流量、延迟和集成。 此外，Analytics 还提供有关用户、消息和通道数据的聊天级报告。

> [!Tip]
>
> - [收集分析](bot-service-manage-analytics.md) 


::: moniker-end

::: moniker range="azure-bot-service-4.0"

# <a name="azure-bot-service"></a>Azure 机器人服务

[!INCLUDE [pre-release-label](includes/pre-release-label.md)]

Azure 机器人服务提供的工具可用于在一个位置构建、测试、部署和管理智能机器人。 通过由 SDK 提供的模块化和可扩展框架，开发人员可以利用模板制作机器人，提供语音、语言理解、提问和解答等功能。  

## <a name="what-is-a-bot"></a>什么是机器人？
机器人是用户使用文本、图形（卡片）或语音通过聊天的方式进行交互的应用。 它可以是一个简单的问答对话，也可以是一个复杂的机器人，允许用户使用模式匹配、状态跟踪和与现有业务服务完美集成的人工智能技术通过智能的方式与服务进行交互。 请查看机器人[案例研究](https://azure.microsoft.com/services/bot-service/)。  

## <a name="building-a-bot"></a>构建机器人 
可以选择最喜欢的开发环境或命令行工具在 C#、JavaScript、Java 和 Python 中构建机器人。 我们提供用户机器人开发各个阶段的工具，可用来构架机器人，帮助你入门。    

![机器人概述](media/bot-service-overview.png) 

## <a name="plan"></a>计划 
在编写代码之前，请参阅[设计指南](bot-service-design-principles.md) ，了解最佳做法并确定机器人的需求。 可以构建简单的机器人或添加更复杂的功能，例如，语音、语言理解、QnA 或从不同源中提取支持并提供智能答案的能力。 若要开始，可以使用 [CLI 工具](~/bot-builder-create-templates.md)、[Azure 门户](bot-service-quickstart.md)或下面的模板。

**获取所需的模板**

| .NET | JavaScript | 
| --- | --- | 
| [VSIX 模板](https://marketplace.visualstudio.com/items?itemName=BotBuilder.botbuilderv4) | [Yeoman 模板](https://www.npmjs.com/package/generator-botbuilder). 使用 @preview 获取 v4 模板。 |

如有必要，浏览或安装工具

- 下载 [CLI 工具](https://github.com/Microsoft/botbuilder-tools)，创建和管理机器人资产。 这些工具有助于通过命令行管理机器人配置文件、LUIS 应用、QnA 知识库等。 可在[自述文件](https://github.com/Microsoft/botbuilder-tools/blob/master/README.md)中找到更多详细信息。
- [模拟器](https://github.com/Microsoft/BotFramework-Emulator/releases)，用于测试机器人
- [LUIS](https://www.luis.ai/)，用于向机器人添加自然语言
- [QnA Maker](https://qnamaker.ai/)，以更自然的聊天方式回答用户的问题


## <a name="build-your-bot"></a>构建机器人 
机器人是一项 Web 服务，可实现聊天式界面并与机器人服务通信。 可以采用任意数量的环境和语言创建此解决方案，并且我们针对 Visual Studio 或 Yeoman，或者直接在 Azure 门户中提供了轻松入门工具。 请参阅以下内容，了解可以使用的部分工具和服务。

提供的部分组件包括：

| 组件 | Description |
| --- | --- |
| [LUIS](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/home) | 添加语言理解 |
| [QnA Maker](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/home)  | 添加知识库，回答用户提出的问题 |
| [Dispatch 工具](~/v4sdk/bot-builder-tutorial-dispatch.md) | 如果使用多个模型，智能判断何时使用哪一个 |
| [富媒体](v4sdk/bot-builder-howto-add-media-attachments.md) | 使用媒体卡或语音增强用户体验 |
| [语音](https://azure.microsoft.com/services/cognitive-services/speech/) | 将音频转换为文本，了解目的，然后将文本转换回语音 |
| [拼写检查](https://azure.microsoft.com/services/cognitive-services/spell-check/) | 帮助用户更正拼写错误、识别姓名、品牌名和俚语之间的区别 |

> [!NOTE]
> 上面的表格不是完整的列表。 浏览左侧的文章，从[发送消息](v4sdk/bot-builder-howto-send-messages.md)开始，了解更多机器人功能。

## <a name="test-your-bot"></a>测试机器人 
机器人是复杂的应用，有大量不同的协同工作的部件。 就像其他复杂应用一样，这可能会导致出现一些需要关注的 Bug，或者会导致机器人的行为异常。 发布前，请先测试机器人。 

[使用模拟器测试机器人](bot-service-debug-emulator.md)或[通过网上聊天测试机器人](bot-service-manage-test-webchat.md)。

## <a name="publish"></a>发布 
准备就绪后，可将机器人发布到 Azure 或自己的 Web 服务或数据中心。 可以设置持续部署，以便在本地开发机器人，而且这种部署在机器人签入到 GitHub 或 Visual Studio Team Services 等源控件时也非常有用。 返回源存储库查看更改时，所做的更改将自动部署到 Azure。

通过持续部署，[部署到 Azure](bot-service-build-continuous-deployment.md)。

## <a name="connect"></a>连接          
将机器人连接到 Facebook、Messenger、Kik、Skype、Slack、Microsoft Teams、Telegram、短信、Twilio、Cortana 和 Skype 等通道，以增加交互并吸引更多客户。

[选择要添加的通道](bot-service-manage-channels.md)。


## <a name="evaluate"></a>评估 
使用 Azure 门户中收集的数据确定改善机器人功能和性能的机会。 可以获得服务级和检测数据，如流量、延迟和集成。 此外，Analytics 还提供有关用户、消息和通道数据的聊天级报告。 

了解如何[收集分析](bot-service-manage-analytics.md)。

::: moniker-end
