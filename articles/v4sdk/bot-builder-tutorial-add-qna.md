---
title: 有关如何让机器人回答问题的 Azure 机器人服务教程 | Microsoft Docs
description: 有关如何在机器人中使用 QnA Maker 来回答问题的教程。
keywords: QnA Maker, 问题和回答, 知识库
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: tutorial
ms.service: bot-service
ms.subservice: sdk
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: aff1440f739150181ddc2d65d9b749b4eeda5d79
ms.sourcegitcommit: dbbfcf45a8d0ba66bd4fb5620d093abfa3b2f725
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/28/2019
ms.locfileid: "67464687"
---
# <a name="tutorial-use-qna-maker-in-your-bot-to-answer-questions"></a>教程：在机器人中使用 QnA Maker 来回答问题

[!INCLUDE [applies-to-v4](../includes/applies-to.md)]

可以使用 QnA Maker 服务和知识库为机器人添加问答支持。 创建知识库时，请为其提供问题和回答。

本教程介绍如何执行下列操作：

> [!div class="checklist"]
> * 创建 QnA Maker 服务和知识库
> * 将知识库信息添加到配置文件
> * 更新机器人，以便查询知识库
> * 重新发布机器人

如果还没有 Azure 订阅，可以在开始前创建一个[免费帐户](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

## <a name="prerequisites"></a>先决条件

* 在[上一教程](bot-builder-tutorial-basic-deploy.md)中创建的机器人。 我们将向机器人添加一项问答功能。
* 最好对 [QnA Maker](https://qnamaker.ai/) 有一些了解。 我们将通过 QnA Maker 门户创建、训练和发布可以与机器人配合使用的知识库。
* 熟悉使用 Azure 机器人服务来[创建 QnA 机器人](https://aka.ms/azure-create-qna)。

你还应该已经满足上一教程的先决条件。

## <a name="sign-in-to-qna-maker-portal"></a>登录到 QnA Maker 门户

<!-- This and the next step are close duplicates of what's in the QnA How-To -->

使用 Azure 凭据登录到 [QnA Maker 门户](https://qnamaker.ai/)。

## <a name="create-a-qna-maker-service-and-knowledge-base"></a>创建 QnA Maker 服务和知识库

我们将从 [Microsoft/BotBuilder-Samples](https://github.com/Microsoft/BotBuilder-Samples) 存储库中的 QnA Maker 示例导入现有的知识库定义。

1. 将 Samples 存储库克隆或复制到计算机。
1. 在 QnA Maker 门户中**创建知识库**。
   1. 如有必要，请创建 QnA 服务。 （就本教程来说，可以使用现有的 QnA Maker 服务，也可以创建一个新的。）如需更多详细的 QnA Maker 说明，请参阅[创建 QnA Maker 服务](https://docs.microsoft.com/azure/cognitive-services/qnamaker/how-to/set-up-qnamaker-service-azure)和[创建、训练和发布 QnA Maker 知识库](https://docs.microsoft.com/azure/cognitive-services/qnamaker/quickstarts/create-publish-knowledge-base)。
   1. 将 QnA 服务连接到知识库。
   1. 为知识库命名。
   1. 若要填充知识库，请使用 Samples 存储库中的 **BotBuilder-Samples\samples\csharp_dotnetcore\11.qnamaker\CognitiveModels\smartLightFAQ.tsv** 文件。
   1. 单击“创建知识库”，创建知识库。 
1. **保存和训练**知识库。
1. **发布**知识库。

发布 QnA Maker 应用以后，请选择“设置”选项卡，然后向下滚动到“部署详细信息”。  记录 _Postman_ 示例 HTTP 请求中的以下值。

```text
POST /knowledgebases/<knowledge-base-id>/generateAnswer
Host: <your-hostname>  // NOTE - this is a URL ending in /qnamaker.
Authorization: EndpointKey <qna-maker-resource-key>
```

主机名的完整 URL 字符串类似于“https://< >.azure.net/qnamaker”。

下一步将在 `appsettings.json` 或 `.env` 文件中使用这些值。

知识库现在可以供机器人使用了。

## <a name="add-knowledge-base-information-to-your-bot"></a>将知识库信息添加到机器人
从 Bot Framework v4.3 开始，Azure 不再在下载的机器人源代码中提供 .bot 文件。 请按以下说明将 CSharp 或 JavaScript 机器人连接到知识库。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

向 appsetting.json 文件添加以下值：

```json
{
  "MicrosoftAppId": "",
  "MicrosoftAppPassword": "",
  "ScmType": "None",
  
  "QnAKnowledgebaseId": "knowledge-base-id",
  "QnAAuthKey": "qna-maker-resource-key",
  "QnAEndpointHostName": "your-hostname" // This is a URL ending in /qnamaker
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

向 .env 文件添加以下值：

```
MicrosoftAppId=""
MicrosoftAppPassword=""
ScmType=None

QnAKnowledgebaseId="knowledge-base-id"
QnAAuthKey="qna-maker-resource-key"
QnAEndpointHostName="your-hostname" // This is a URL ending in /qnamaker
```

---

| 字段 | 值 |
|:----|:----|
| QnAKnowledgebaseId | QnA Maker 门户为你生成的知识库 ID。 |
| QnAAuthKey | QnA Maker 门户为你生成的终结点密钥。 |
| QnAEndpointHostName | QnA Maker 门户生成的主机 URL。 请使用完整的 URL，以 `https://` 开头，以 `/qnamaker` 结尾。 完整 URL 字符串看起来将类似于“https://< >.azure.net/qnamaker”。 |

现在保存所做的编辑。

## <a name="update-your-bot-to-query-the-knowledge-base"></a>更新机器人，以便查询知识库

更新初始化代码，加载知识库的服务信息。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

1. 将 **Microsoft.Bot.Builder.AI.QnA** NuGet 包添加到项目中。

   可通过 NuGet 包管理器或命令行执行此操作：

   ```cmd
   dotnet add package Microsoft.Bot.Builder.AI.QnA
   ```

   有关 NuGet 的详细信息，请参阅 [NuGet 文档](https://docs.microsoft.com/nuget/#pivot=start&panel=start-all)。

1. 将 **Microsoft.Extensions.Configuration** NuGet 包添加到项目中。

1. 在 **Startup.cs** 文件中，添加这些命名空间引用。

   **Startup.cs**

   ```csharp
   using Microsoft.Bot.Builder.AI.QnA;
   using Microsoft.Extensions.Configuration;
   ```

1. 修改 _ConfigureServices_ 方法，创建一个 QnAMakerEndpoint，以便连接到在 **appsettings.json** 文件中定义的知识库。

   **Startup.cs**

   ```csharp
   // Create QnAMaker endpoint as a singleton
   services.AddSingleton(new QnAMakerEndpoint
   {
      KnowledgeBaseId = Configuration.GetValue<string>($"QnAKnowledgebaseId"),
      EndpointKey = Configuration.GetValue<string>($"QnAAuthKey"),
      Host = Configuration.GetValue<string>($"QnAEndpointHostName")
    });

   ```

1. 在 **EchoBot.cs** 文件中，添加这些命名空间引用。

   **EchoBot.cs**

   ```csharp
   using System.Linq;
   using Microsoft.Bot.Builder.AI.QnA;
   ```

1. 添加一个 `EchoBotQnA` 连接器，在机器人的构造函数中将其初始化。

   **EchoBot.cs**

   ```csharp
   public QnAMaker EchoBotQnA { get; private set; }
   public EchoBot(QnAMakerEndpoint endpoint)
   {
      // connects to QnA Maker endpoint for each turn
      EchoBotQnA = new QnAMaker(endpoint);
   }
   ```

1. 在 _OnMembersAddedAsync( )_ 方法下方，通过添加以下代码创建 _AccessQnAMaker( )_ 方法：

   **EchoBot.cs**

   ```csharp
   private async Task AccessQnAMaker(ITurnContext<IMessageActivity> turnContext, CancellationToken cancellationToken)
   {
      var results = await EchoBotQnA.GetAnswersAsync(turnContext);
      if (results.Any())
      {
         await turnContext.SendActivityAsync(MessageFactory.Text("QnA Maker Returned: " + results.First().Answer), cancellationToken);
      }
      else
      {
         await turnContext.SendActivityAsync(MessageFactory.Text("Sorry, could not find an answer in the Q and A system."), cancellationToken);
      }
   }
   ```

1. 现在，在 _OnMessageActivityAsync( )_ 中调用新方法 _AccessQnAMaker( )_ ，如下所示：

   **EchoBot.cs**

   ```csharp
   protected override async Task OnMessageActivityAsync(ITurnContext<IMessageActivity> turnContext, CancellationToken cancellationToken)
   {
      // First send the user input to your QnA Maker knowledge base
      await AccessQnAMaker(turnContext, cancellationToken);
      ...
   }
   ```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

1. 打开终端或命令提示符，转到项目的根目录。
1. 将 **botbuilder-ai** npm 包添加到项目。
   ```shell
   npm i botbuilder-ai
   ```

1. 在 **index.js** 中的 // Create Adapter 节后面添加以下代码，以便读取生成 QnA Maker 服务所需的 .env 文件配置信息。

   **index.js**
   ```javascript
   // Map knowledge base endpoint values from .env file into the required format for `QnAMaker`.
   const configuration = {
      knowledgeBaseId: process.env.QnAKnowledgebaseId,
      endpointKey: process.env.QnAAuthKey,
      host: process.env.QnAEndpointHostName
   };

   ```

1. 更新机器人构造，以便传入 QnA 服务配置信息。

   **index.js**
   ```javascript
   // Create the main dialog.
   const myBot = new MyBot(configuration, {});
   ```

1. 在 **bot.js** 文件中，为 QnAMaker 添加此 require 语句

   **bot.js**
   ```javascript
   const { QnAMaker } = require('botbuilder-ai');
   ```

1. 修改构造函数，使之现在可以接收创建 QnAMaker 连接器所需的已传递的配置参数，在不提供这些参数的情况下引发错误。

   **bot.js**
   ```javascript
      class MyBot extends ActivityHandler {
         constructor(configuration, qnaOptions) {
            super();
            if (!configuration) throw new Error('[QnaMakerBot]: Missing parameter. configuration is required');
            // now create a qnaMaker connector.
            this.qnaMaker = new QnAMaker(configuration, qnaOptions);
   ```

1. 最后，更新 `onMessage` 函数，以便在知识库中查询答案。 将每个用户输入传递给 QnA Maker 知识库，并将第一个 QnA Maker 响应返回给用户。

    **bot.js**

    ```javascript
    this.onMessage(async (context, next) => {
        // send user input to QnA Maker.
        const qnaResults = await this.qnaMaker.getAnswers(context);

        // If an answer was received from QnA Maker, send the answer back to the user.
        if (qnaResults[0]) {
            await context.sendActivity(`QnAMaker returned response: ' ${ qnaResults[0].answer}`);
        }
        else {
            // If no answers were returned from QnA Maker, reply with help.
            await context.sendActivity('No QnA Maker response was returned.'
                + 'This example uses a QnA Maker Knowledge Base that focuses on smart light bulbs. '
                + `Ask the bot questions like "Why won't it turn on?" or "I need help."`);
        }
        await next();
    });
    ```

---

### <a name="test-the-bot-locally"></a>在本地测试机器人

目前，机器人应该能够回答一些问题。 在本地运行机器人，在模拟器中将其打开。

![测试 qna 示例](./media/qna-test-bot.png)

## <a name="republish-your-bot"></a>重新发布机器人

现在可以将机器人重新发布回 Azure。

> [!IMPORTANT]
> 在创建项目文件的 zip 文件之前，请确保在正确的文件夹中进行压缩。  
> - 对于 C# 机器人，正确的文件夹是包含 .csproj 文件的文件夹。 
> - 对于 JS 机器人，正确的文件夹是包含 app.js 或 index.js 文件的文件夹。 
>
> 在该文件夹中选择所有文件并将其压缩，然后运行此命令，此时仍在该文件夹中。
>
> 如果根文件夹位置不正确，**机器人将无法在 Azure 门户中运行**。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)
```cmd
az webapp deployment source config-zip --resource-group "resource-group-name" --name "bot-name-in-azure" --src "c:\bot\mybot.zip"
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

[!INCLUDE [publish snippet](~/includes/deploy/snippet-publish-js.md)]

---

### <a name="test-the-published-bot"></a>测试已发布的机器人

发布机器人之后，请给 Azure 一到两分钟的时间来更新并启动机器人。

使用模拟器测试机器人的生产终结点，或者使用 Azure 门户通过网上聊天测试机器人。
不管采用哪一种方式，都会看到与本地测试相同的行为。

## <a name="clean-up-resources"></a>清理资源

<!-- In the first tutorial, we should tell them to use a new resource group, so that it is easy to clean up resources. We should also mention in this step in the first tutorial not to clean up resources if they are continuing with the sequence. -->

如果不打算继续使用此应用程序，请按以下步骤删除关联的资源：

1. 在 Azure 门户中打开机器人的资源组。
1. 单击“删除资源组”，删除该组及其包含的所有资源。 
1. 在确认窗格中输入资源组名称，然后单击“删除”。 

## <a name="next-steps"></a>后续步骤

若要了解如何为机器人添加功能，请参阅开发方法部分中的“发送和接收文本消息”  一文以及其他文章。
> [!div class="nextstepaction"]
> [发送和接收文本消息](bot-builder-howto-send-messages.md)
