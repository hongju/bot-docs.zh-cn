---
title: 使用 QnA Maker 回答问题 | Microsoft Docs
description: 了解如何在机器人中使用 QnA maker。
keywords: 问题和解答, QnA, 常见问题解答, qna maker
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: cognitive-services
ms.date: 05/20/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 10ae35f51a072a1af6cf7d4bdf2fd2f4cb3d66ee
ms.sourcegitcommit: 72cc9134bf50f335cbb33265b048bf6b76252ce4
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/21/2019
ms.locfileid: "65973858"
---
# <a name="use-qna-maker-to-answer-questions"></a>使用 QnA Maker 回答问题

[!INCLUDE[applies-to](../includes/applies-to.md)]

QnA Maker 提供基于数据的聊天式问答层。 这样机器人就可以向 QnA Maker 发送问题并接收答案，不需要你来分析和解释问题的意图。 

创建自己的 QnA Maker 服务的基本要求之一是为其提供问题和答案。 在许多情况下，问题和答案已经存在于常见问题解答或其他文档等内容中；其他情况下，可能需要以更自然、更具聊天风格的方式来自定义问题的解答。 

## <a name="prerequisites"></a>先决条件

- 本文中的代码基于 QnA Maker 示例。 需要以 **[CSharp](https://aka.ms/cs-qna) 或 [JavaScript](https://aka.ms/js-qna-sample)** 编写的示例的副本。
- [QnA Maker](https://www.qnamaker.ai/) 帐户
- 了解[机器人基础知识](bot-builder-basics.md)、[QnA Maker](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/overview/overview) 和[管理机器人资源](bot-file-basics.md)。

## <a name="about-this-sample"></a>关于此示例

若要让机器人利用 QnA Maker，需先在 [QnA Maker](https://www.qnamaker.ai/) 上创建一个知识库，这一点我们会在下一部分介绍。 然后，机器人就可以向它发送用户的查询，它会在响应中提供问题的最佳解答。

## <a name="ctabcs"></a>[C#](#tab/cs)
![QnABot 逻辑流](./media/qnabot-logic-flow.png)

每次收到用户输入都会调用 `OnMessageActivityAsync`。 被调用时，它会访问存储在示例代码的 `appsetting.json` 文件中的 `_configuration` 信息，以便查找连接到预配置的 QnA Maker 知识库所需的值。 

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)
![QnABot JS 逻辑流](./media/qnabot-js-logic-flow.png)

每次收到用户输入都会调用 `OnMessage`。 被调用时，它会访问 `qnamaker` 连接器，该连接器已使用示例代码的 `.env` 文件中提供的值进行预配置。  qnamaker 方法 `getAnswers` 可将机器人连接到外部 QnA Maker 知识库。

---
用户的输入将发送到知识库，而返回的最佳解答将回显给用户。

## <a name="create-a-qna-maker-service-and-publish-a-knowledge-base"></a>创建 QnA Maker 服务并发布知识库
第一步是创建 QnA Maker 服务。 按照 QnA Maker [文档](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/how-to/set-up-qnamaker-service-azure)中列出的步骤在 Azure 中创建此服务。

接下来，使用示例项目的 CognitiveModels 文件夹中的 `smartLightFAQ.tsv` 文件创建一个知识库。 QnA Maker 文档中列出了创建、训练和发布 QnA Maker [知识库](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/quickstarts/create-publish-knowledge-base)的步骤。 在执行以下步骤的过程中，请将知识库命名为 `qna`，并使用 `smartLightFAQ.tsv` 文件来填充知识库。

> 注意。 也可以参考本文来访问用户自己开发的 QnA Maker 知识库。

## <a name="obtain-values-to-connect-your-bot-to-the-knowledge-base"></a>获取用于将机器人连接到知识库的值
1. 在 [QnA Maker](https://www.qnamaker.ai/) 站点中选择知识库。
1. 打开知识库以后，选择“设置”。 记录针对“服务名称”显示的值。 使用 QnA Maker 门户界面时，此值可用于查找所需的知识库。 此值不可用于将机器人应用连接到此知识库。 
1. 向下滚动，找到“部署详细信息”并记录 Postman 示例 HTTP 请求中的以下值：
   - POST /knowledgebases/\<knowledge-base-id>/generateAnswer
   - Host: \<your-hostname> // 以 /qnamaker 结尾的完整 URL
   - Authorization:EndpointKey \<your-endpoint-key>
   
主机名的完整 URL 字符串类似于“https://< >.azure.net/qnamaker”。 这三个值将为应用提供所需的信息，使其能够通过 Azure QnA 服务连接到 QnA Maker 知识库。  

## <a name="update-the-settings-file"></a>更新设置文件

首先，请将访问知识库所需的信息（包括主机名、终结点密钥和知识库 ID (kbId)）添加到设置文件中。 这些是在 QnA Maker 中通过知识库的“设置”选项卡保存的值。 

如果不是针对生产环境进行此部署，则应用 ID 和密码字段可以留空。

> [!NOTE]
> 如果将 QnA Maker 知识库访问权限添加到现有的机器人应用程序中，请务必为 QnA 条目添加说明性标题。 此节中的 "name" 值提供所需的密钥用于从应用内部访问此信息。

## <a name="ctabcs"></a>[C#](#tab/cs)

### <a name="update-your-appsettingsjson-file"></a>更新 appsettings.json 文件

```json
{
  "MicrosoftAppId": "",
  "MicrosoftAppPassword": "",
  
  "QnA-sample-qna-kbId": "<knowledge-base-id>",
  "QnA-sample-qna-endpointKey": "<your-endpoint-key>",
  "QnA-sample-qna-hostname": "<your-hostname>"
}
```

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

### <a name="update-your-env-file"></a>更新 .env 文件

```file
MicrosoftAppId=""
MicrosoftAppPassword=""

QnAKnowledgebaseId="<knowledge-base-id>"
QnAAuthKey="<your-endpoint-key>"
QnAEndpointHostName="<your-hostname>"
```

---

## <a name="set-up-the-qna-maker-instance"></a>设置 QnA Maker 实例

首先，创建一个用于访问 QnA Maker 知识库的对象。

## <a name="ctabcs"></a>[C#](#tab/cs)

请确保为项目安装 **Microsoft.Bot.Builder.AI.QnA** NuGet 包。

在 **QnABot.cs** 的 `OnMessageActivityAsync` 方法中，我们创建一个 QnAMaker 实例。 `QnABot` 类也是在其中拉入连接信息名称（保存在上面的 `appsettings.json` 中）的地方。 如果在设置文件中为知识库连接信息选择了其他名称，请务必根据你所选择的名称在此更新这些名称。

**Bots/QnABot.cs** [!code-csharp[qna connection](~/../botbuilder-samples/samples/csharp_dotnetcore/11.qnamaker/Bots/QnABot.cs?range=32-37)]

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

确保为项目安装 npm 包 **botbuilder-ai**。

在我们的示例中，机器人逻辑的代码位于 **QnABot.js** 文件中。

在 **QnABot.js** 文件中，我们使用 .env 文件提供的连接信息建立到 QnA Maker 服务的连接：_this.qnaMaker_。

**QnAMaker.js** [!code-javascript[QnAMaker](~/../botbuilder-samples/samples/javascript_nodejs/11.qnamaker/bots/QnABot.js?range=19-23)]


---

## <a name="calling-qna-maker-from-your-bot"></a>通过机器人调用 QnA Maker

## <a name="ctabcs"></a>[C#](#tab/cs)

当机器人需要来自 QnAMaker 的答案时，请通过机器人代码调用 `GetAnswersAsync()`，以便根据当前的上下文获取适当的答案。 若要访问自己的知识库，请更改下面的“找不到答案”消息，为用户提供有用的说明。

**QnABot.cs** [!code-csharp[qna connection](~/../botbuilder-samples/samples/csharp_dotnetcore/11.qnamaker/Bots/QnABot.cs?range=43-52)]

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

在 **QnABot.js** 文件中，我们将用户的输入传递给 QnA Maker 服务的 `getAnswers` 方法，以便从知识库获取答案。 如果 QnA Maker 返回响应，则会将其显示给用户。 否则，用户会收到消息“找不到 QnA Maker 答案”。 

**QnABot.js** [!code-javascript[OnMessage](~/../botbuilder-samples/samples/javascript_nodejs/11.qnamaker/bots/QnABot.js?range=43-59)]

---

## <a name="test-the-bot"></a>测试机器人

在计算机本地运行示例。 安装 [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/blob/master/README.md#download)（如果尚未安装）。 如需更多说明，请参阅 [C# 示例](https://aka.ms/cs-qna)或 [Javascript 示例](https://aka.ms/js-qna-sample)的自述文件。

按如下所示启动模拟器，连接到机器人，然后发送消息。

![测试 qna 示例](../media/emulator-v4/qna-test-bot.png)

## <a name="next-steps"></a>后续步骤

QnA Maker 可与其他认知服务结合使用，使机器人更加强大。 调度工具提供了一种在机器人中将 QnA 与语言理解 (LUIS) 相结合的方法。

> [!div class="nextstepaction"]
> [使用调度工具组合 LUIS 和 QnA 服务](./bot-builder-tutorial-dispatch.md)
