---
title: 使用机器人文件管理机器人资源 | Microsoft Docs
description: 介绍机器人文件的用途和用法。
keywords: 机器人文件, .bot, .bot 文件, msbot, 机器人资源, 管理机器人资源
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 11/13/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 0ff3f0e68d58a8768bb785a88ee7664ab430e453
ms.sourcegitcommit: 8b7bdbcbb01054f6aeb80d4a65b29177b30e1c20
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/14/2018
ms.locfileid: "51645627"
---
# <a name="manage-bot-resources-with-a-bot-file"></a>使用机器人文件管理机器人资源

机器人往往使用大量不同的服务，例如 [LUIS.ai](https://luis.ai) 或 [QnaMaker.ai](https://qnamaker.ai)。 开发机器人时，没有一个统一的位置用于存储有关所用服务的元数据。  这样，我们就无法构建可将机器人视为一个整体的工具。

为解决此问题，我们创建了 **.bot 文件**，它会将所有服务引用合并到一个位置，使我们能够构建工具。  例如，Bot Framework Emulator ([V4](https://aka.ms/Emulator-wiki-getting-started)) 通过 .bot 文件基于机器人使用的连接服务创建统一视图。  

使用 .bot 文件可以注册如下所述的服务：

* **Localhost** - 本地调试程序终结点
* [**Azure 机器人服务**](https://azure.microsoft.com/en-us/services/bot-service/) - Azure 机器人服务注册。
* [**LUIS.AI**](https://www.luis.ai/) - LUIS 使机器人能够使用自然语言与人类沟通。 
* [**QnA Maker**](https://qnamaker.ai/) - 在几分钟内基于 FAQ URL、结构化文档或编辑内容生成、训练和发布简单的问答机器人。
* [**Dispatch**](https://github.com/Microsoft/botbuilder-tools/tree/master/Dispatch) 模型 - 在多个服务之间进行调度。
* [**Azure Application Insights**](https://azure.microsoft.com/en-us/services/application-insights/) - 提供见解和机器人分析。
* [**Azure Blob 存储**](https://azure.microsoft.com/en-us/services/storage/blobs/) - 保存机器人状态。 
* [**Azure Cosmos DB**](https://azure.microsoft.com/en-us/services/cosmos-db/) - 用于保存机器人状态的全局分布式多模型数据库服务。

除此之外，机器人还可依赖于其他自定义服务。 可以利用[通用服务](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/MSBot/docs/add-services.md)功能连接通用服务配置。

## <a name="when-is-a-bot-file-created"></a>何时创建 .bot 文件？ 
- 如果使用 [Azure 机器人服务](https://ms.portal.azure.com/#blade/Microsoft_Azure_Marketplace/GalleryResultsListBlade/selectedSubMenuItemId/%7B%22menuItemId%22%3A%22gallery%2FCognitiveServices_MP%2FBotService%22%2C%22resourceGroupId%22%3A%22%22%2C%22resourceGroupLocation%22%3A%22%22%2C%22dontDiscardJourney%22%3Afalse%2C%22launchingContext%22%3A%7B%22source%22%3A%5B%22GalleryFeaturedMenuItemPart%22%5D%2C%22menuItemId%22%3A%22CognitiveServices_MP%22%2C%22subMenuItemId%22%3A%22BotService%22%7D%7D)创建机器人，则系统会自动创建一个 .bot 文件，其中包含预配的连接服务列表。 .bot 默认已加密。
- 如果使用适用于 Visual Studio 的 Bot Builder V4 SDK [模板](https://marketplace.visualstudio.com/items?itemName=BotBuilder.botbuilderv4)或使用 Bot Builder [Yeoman Generator](https://www.npmjs.com/package/generator-botbuilder) 创建机器人，则会自动创建 .bot 文件。 此流中不会预配连接服务，且机器人文件不会加密。
- 如果从 [BotBuilder 示例](https://github.com/Microsoft/botbuilder-samples)着手，Bot Builder V4 SDK 的每个示例会包含一个 .bot 文件，且该 .bot 文件不会加密。 
- 也可以使用 [MSBot](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/MSBot/README.md) 工具创建机器人文件。

## <a name="what-does-a-bot-file-look-like"></a>机器人文件的外观是怎样的？ 
请看看一个示例 [.bot](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/MSBot/docs/sample-bot-file.json) 文件。
若要了解如何加密和解密 .bot 文件，请参阅[机器人机密](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/MSBot/docs/bot-file-encryption.md)。
## <a name="why-do-i-need-a-bot-file"></a>为何需要 .bot 文件？

通过 Bot Builder SDK 生成机器人时**不一定**非要使用 .bot 文件。 可以继续使用 appsettings.json、web.config、env、keyvault 或其他任何机制，只要它们适合用于跟踪机器人依赖的服务引用和密钥即可。 但是，若要使用仿真器测试机器人，则需要 .bot 文件。 好消息是，仿真器可以创建用于测试的 .bot 文件。 为此，请启动仿真器，并在“欢迎”页上单击“创建新机器人配置”链接。 在显示的对话框中，键入**机器人名称**和**终结点 URL**。 然后建立连接。

使用 .bot 文件的优势包括：
- 不管使用哪种语言/平台，都能通过标准的方式存储存储资源。   
- Bot Framework Emulator 和 CLI 工具依赖于跟踪采用一致格式的连接服务（在 .bot 文件中），并且非常适合进行这种跟踪 
- 如果没有完善定义的架构（.bot 文件），则很难围绕服务的创建和管理开发精巧的工具解决方案。  


## <a name="using-bot-file-in-your-bot-builder-sdk-bot"></a>在 Bot Builder SDK 机器人中使用 .bot 文件
可以在机器人代码中使用 .bot 文件获取服务配置信息。 适用于 [C#](https://www.nuget.org/packages/Microsoft.Bot.Configuration) 和 [JS](https://www.npmjs.com/package/botframework-config) 的 BotFramework-Configuration 库可帮助你加载机器人文件，并支持通过多种方法来查询和获取相应的服务配置信息。

## <a name="additional-resources"></a>其他资源
有关使用机器人文件的详细信息，请参阅 [MSBot](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/MSBot/README.md) 自述文件。
