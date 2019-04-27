---
title: Azure 机器人服务简介 | Microsoft Docs
description: 了解机器人服务，一种用于构建、连接、测试、部署、监视和管理机器人的服务。
keywords: 概述, 简介, SDK, 大纲
author: Kaiqb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 04/18/2019
ms.openlocfilehash: 86a75fb315ec303cb8a519bfa02063c4e6780307
ms.sourcegitcommit: aea57820b8a137047d59491b45320cf268043861
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/22/2019
ms.locfileid: "59904810"
---
# <a name="about-azure-bot-service"></a>关于 Azure 机器人服务

[!INCLUDE [applies-to-both](includes/applies-to-both.md)]

Azure 机器人服务提供的工具可用于在一个位置构建、测试、部署和管理智能机器人。 通过使用 SDK、工具、模板和 AI 服务提供的模块化可扩展框架，开发人员可以创建可使用语音、自然语言理解、问题和答案处理等功能的机器人。

## <a name="what-is-a-bot"></a>什么是机器人？
机器人提供的体验让你感觉不太像在使用计算机，而更像是在与人打交道，或者至少是在与智能机器人打交道。 可以使用机器人将简单的重复性任务（例如订餐或收集个人资料信息）转移给不再需要直接人为干预的自动化系统来完成。 用户使用文本、交互卡和语音与机器人聊天。 机器人交互可以是快速的问答式交互，也可以是复杂的聊天，通过聊天以智能方式提供对服务的访问权限。

机器人很像现代 Web 应用程序，驻留在 Internet 中，使用 API 发送和接收消息。 机器人中的内容差异很大，具体取决于机器人的类型。 现代机器人软件依赖一系列技术和工具，在各种平台上提供日益复杂的体验。 不过，简单的机器人可以只接收消息并将其回显给用户，基本不需要编写代码。 

机器人可以完成其他类型的软件可以完成的任务 - 读写文件、使用数据库和 API，以及执行常规的计算任务。 使机器人不同于其他软件的是，它们使用的通信机制通常是人与人之间通信才会使用的。 

Azure 机器人服务套餐：
- 用于开发机器人的 Bot Framework SDK
- 涵盖端到端机器人开发工作流的 Bot Framework 工具
- Bot Framework Service (BFS)，用于在机器人和通道之间发送和接收消息和事件
- Azure 中的机器人部署和通道配置

另外，机器人可能使用其他 Azure 服务，例如：
- 用于生成智能应用程序的 Azure 认知服务 
- 用于云存储解决方案的 Azure 存储

## <a name="building-a-bot"></a>构建机器人 

Azure 机器人服务提供一组集成的工具和服务来加快此过程。 请选择最喜欢的开发环境或命令行工具以创建机器人。 提供用于 C#、JavaScript 和 Typescript 的 SDK。 （用于 Java 和 Python 的 SDK 正在开发。）我们提供适合各个机器人开发阶段的工具，方便你设计和生成机器人。

![机器人概述](media/bot-service-overview.png) 

### <a name="plan"></a>计划
与任何类型的软件一样，若要创建成功的机器人，必须全面了解目标、流程和用户需求。 在编写代码之前，请参阅[设计指南](bot-service-design-principles.md) ，了解最佳做法并确定机器人的需求。 可以创建简单的机器人，也可以让机器人包含较复杂的功能，例如语音、自然语言理解和问题解答。

### <a name="build"></a>构建
机器人是一项 Web 服务，可实现聊天式界面并与 Bot Framework Service 通信，以便发送和接收消息和事件。 Bot Framework Service 是 Azure 机器人服务的一个组件。 可在任意数目的环境和语言中创建机器人。 可在 [Azure 门户](bot-service-quickstart.md)中开始机器人开发，也可使用 [[C#](dotnet/bot-builder-dotnet-sdk-quickstart.md) | [JavaScript](javascript/bot-builder-javascript-quickstart.md)] 模板进行本地开发。

我们提供其他组件作为 Azure 机器人服务的一部分来扩展机器人的功能

| Feature | 说明 | 链接 |
| --- | --- | --- |
| 添加自然语言处理 | 可让机器人理解自然语言、了解拼写错误、使用语音和识别用户的意向 | 如何使用 [LUIS](~/v4sdk/bot-builder-howto-v4-luis.md) 
| 回答问题 | 添加知识库，以更自然的聊天形式回答用户的提问 | 如何使用 [QnA Maker](~/v4sdk/bot-builder-howto-qna.md) 
| 管理多个模型 | 如果使用多个模型（例如 LUIS 和 QnA Maker），在与机器人聊天过程中，机器人能够明智地确定何时使用哪个模型 | [Dispatch](~/v4sdk/bot-builder-tutorial-dispatch.md) 工具|
| 添加卡片和按钮 | 使用除文本以外的媒体（例如图形、菜单和卡片）来增强用户体验 | 如何[添加卡片](v4sdk/bot-builder-howto-add-media-attachments.md) |

> [!NOTE]
> 上面的表格不是完整的列表。 浏览左侧的文章，从[发送消息](~/v4sdk/bot-builder-howto-send-messages.md)开始，了解更多机器人功能。

此外，我们提供命令行工具来帮助你创建、管理和测试机器人资产。 这些工具可以管理机器人配置文件、配置 LUIS 应用、生成 QnA 知识库、模拟聊天，等等。 可在命令行工具[自述文件](https://aka.ms/botbuilder-tools-readme)中找到更多详细信息。

还可以访问各种[示例](https://github.com/microsoft/botbuilder-samples)，了解通过 SDK 提供的多项功能。 这些特别适用于希望从功能较丰富的示例着手的开发人员。

### <a name="test"></a>测试 
机器人是复杂的应用，有大量不同的协同工作的部件。 就像其他复杂应用一样，这可能会导致出现一些需要关注的 Bug，或者会导致机器人的行为异常。 发布前，请先测试机器人。 在发布机器人供用户使用之前，我们提供了多种方式来测试机器人：

- 使用[模拟器](bot-service-debug-emulator.md)在本地测试机器人。 Bot Framework Emulator 是独立的应用，不仅提供聊天界面，而且提供调试和询问工具来帮助理解机器人的工作方式和工作原理。  此模拟器可以在本地与正在开发的机器人应用程序一起运行。 
 
- 在 [Web](bot-service-manage-test-webchat.md) 上测试机器人。 通过 Azure 门户进行配置以后，机器人也可通过网上聊天界面进行访问。 测试者和其他无法直接访问机器人的运行代码的人员可以通过网上聊天界面访问机器人。

### <a name="publish"></a>发布 
做好在网上发布机器人的准备以后，请将机器人发布到 [Azure](bot-builder-howto-deploy-azure.md) 或你自己的 Web 服务或数据中心。 若要将机器人嵌入站点或聊天通道，第一步是获取公共 Internet 上的地址。

### <a name="connect"></a>连接          
将机器人连接到 Facebook、Messenger、Kik、Skype、Slack、Microsoft Teams、Telegram、短信、Twilio、Cortana 和 Skype 等通道。 在通过所有这些不同的平台发送和接收消息的过程中，Bot Framework 完成大多数必需的工作 - 不管连接到的通道的数目和类型如何，机器人应用程序都会收到统一且规范化的消息流。 有关如何添加通道的信息，请参阅[通道](bot-service-manage-channels.md)主题。

### <a name="evaluate"></a>评估 
使用 Azure 门户中收集的数据确定改善机器人功能和性能的机会。 可以获得服务级和检测数据，如流量、延迟和集成。 此外，Analytics 还提供有关用户、消息和通道数据的聊天级报告。 有关详细信息，请参阅[如何收集分析数据](bot-service-manage-analytics.md)。


## <a name="next-steps"></a>后续步骤
请查看这些机器人[案例研究](https://azure.microsoft.com/services/bot-service/)，或者单击下面的有关如何创建机器人的链接。
> [!div class="nextstepaction"]
> [创建机器人](bot-service-quickstart.md)
