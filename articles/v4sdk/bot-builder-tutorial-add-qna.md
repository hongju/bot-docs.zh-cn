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
ms.date: 01/15/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: b1c34531ee60b2ce9037f42e4f5a7093501cf83a
ms.sourcegitcommit: bdb981c0b11ee99d128e30ae0462705b2dae8572
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/17/2019
ms.locfileid: "54360939"
---
# <a name="tutorial-use-qna-maker-in-your-bot-to-answer-questions"></a>教程：在机器人中使用 QnA Maker 来回答问题

可以使用 QnA Maker 服务和知识库为机器人添加问答支持。 创建知识库时，请为其提供问题和回答。

本教程介绍如何执行下列操作：

> [!div class="checklist"]
> * 创建 QnA Maker 服务和知识库
> * 将知识库信息添加到 .bot 文件
> * 更新机器人，以便查询知识库
> * 重新发布机器人

如果还没有 Azure 订阅，可以在开始前创建一个[免费帐户](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

## <a name="prerequisites"></a>先决条件

* 在[上一教程](bot-builder-tutorial-basic-deploy.md)中创建的机器人。 我们将向机器人添加一项问答功能。
* 最好对 QnA Maker 有一些了解。 我们将通过 QnA Maker 门户创建、训练和发布可以与机器人配合使用的知识库。

你应该已经满足上一教程的先决条件：

[!INCLUDE [deployment prerequisites snippet](~/includes/deploy/snippet-prerequisite.md)]

## <a name="sign-in-to-qna-maker-portal"></a>登录到 QnA Maker 门户

<!-- This and the next step are close duplicates of what's in the QnA How-To -->

使用 Azure 凭据登录到 [QnA Maker 门户](https://qnamaker.ai/)。

## <a name="create-a-qna-maker-service-and-knowledge-base"></a>创建 QnA Maker 服务和知识库

我们将从 [Microsoft/BotBuilder-Samples](https://github.com/Microsoft/BotBuilder-Samples) 存储库中的 QnA Maker 示例导入现有的知识库定义。

1. 将 Samples 存储库克隆或复制到计算机。
1. 在 QnA Maker 门户中**创建知识库**。
   1. 如有必要，请创建 QnA 服务。 （就本教程来说，可以使用现有的 QnA Maker 服务，也可以创建一个新的。）如需更多详细的 QnA Maker 说明，请参阅[创建 QnA Maker 服务](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/how-to/set-up-qnamaker-service-azure)和[创建、训练和发布 QnA Maker 知识库](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/quickstarts/create-publish-knowledge-base)。
   1. 将 QnA 服务连接到知识库。
   1. 为知识库命名。
   1. 若要填充知识库，请使用 Samples 存储库中的 **BotBuilder-Samples\samples\csharp_dotnetcore\11.qnamaker\CognitiveModels\smartLightFAQ.tsv** 文件。
   1. 单击“创建知识库”，创建知识库。
1. **保存和训练**知识库。
1. **发布**知识库。

   知识库现在可以供机器人使用了。 记录知识库 ID、终结点密钥和主机名。 下一步需要用到这些值。

## <a name="add-knowledge-base-information-to-your-bot-file"></a>将知识库信息添加到 .bot 文件

向 .bot 文件添加访问知识库所需的信息。

1. 在编辑器中打开 .bot 文件。
1. 将 `qna` 元素添加到 `services` 数组。

    ```json
    {
        "type": "qna",
        "name": "<your-knowledge-base-name>",
        "kbId": "<your-knowledge-base-id>",
        "hostname": "<your-qna-service-hostname>",
        "endpointKey": "<your-knowledge-base-endpoint-key>",
        "subscriptionKey": "<your-azure-subscription-key>",
        "id": "<a-unique-id>"
    }
    ```

    | 字段 | 值 |
    |:----|:----|
    | type | 必须是 `qna`。 这表示此服务条目描述 QnA 知识库。 |
    | 名称 | 分配给知识库的名称。 |
    | kbId | QnA Maker 门户为你生成的知识库 ID。 |
    | hostname | QnA Maker 门户生成的主机 URL。 请使用完整的 URL，以 `https://` 开头，以 `/qnamaker` 结尾。 |
    | endpointKey | QnA Maker 门户为你生成的终结点密钥。 |
    | subscriptionKey | 在 Azure 中创建 QnA Maker 服务时使用的订阅的 ID。 |
    | id | 一个唯一的 ID，例如“201”，该 ID 尚未用于在 .bot 文件中列出的某个其他的服务。 |

1. 保存所做的编辑。

## <a name="update-your-bot-to-query-the-knowledge-base"></a>更新机器人，以便查询知识库

更新初始化代码，加载知识库的服务信息。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

1. 将 **Microsoft.Bot.Builder.AI.QnA** NuGet 包添加到项目中。
1. 将实现 **IBot** 的类重命名为 `QnaBot`。
1. 将包含机器人的访问器的类重命名为 `QnaBotAccessors`。
1. 在 **Startup.cs** 文件中，添加这些命名空间引用。
    ```csharp
    using System.Collections.Generic;
    using System.Linq;
    using Microsoft.Bot.Builder.AI.QnA;
    using Microsoft.Bot.Builder.Integration;
    ```
1. 修改 **ConfigureServices** 方法，以便初始化和注册在 **.bot** 文件中定义的知识库。 请注意，这头几行已从 `services.AddBot<QnaBot>(options =>` 调用的正文移到它的前面。
    ```csharp
    public void ConfigureServices(IServiceCollection services)
    {
        var secretKey = Configuration.GetSection("botFileSecret")?.Value;
        var botFilePath = Configuration.GetSection("botFilePath")?.Value;

        // Loads .bot configuration file and adds a singleton that your Bot can access through dependency injection.
        var botConfig = BotConfiguration.Load(botFilePath ?? @".\jfEchoBot.bot", secretKey);
        services.AddSingleton(sp => botConfig ?? throw new InvalidOperationException($"The .bot config file could not be loaded. ({botConfig})"));

        // Initialize the QnA knowledge bases for the bot.
        services.AddSingleton(sp => {
            var qnaServices = new List<QnAMaker>();
            foreach (var qnaService in botConfig.Services.OfType<QnAMakerService>())
            {
                qnaServices.Add(new QnAMaker(qnaService));
            }
            return qnaServices;
        });

        services.AddBot<QnaBot>(options =>
        {
            // Retrieve current endpoint.
            // ...
        });

        // Create and register state accessors.
        // ...
    }
    ```
1. 在 **QnaBot.cs** 文件中，添加这些命名空间引用。
    ```csharp
    using System.Collections.Generic;
    using Microsoft.Bot.Builder.AI.QnA;
    ```
1. 添加一个 `_qnaServices` 属性，在机器人的构造函数中将其初始化。
    ```csharp
    private readonly List<QnAMaker> _qnaServices;

    /// ...
    public QnaBot(QnaBotAccessors accessors, List<QnAMaker> qnaServices, ILoggerFactory loggerFactory)
    {
        // ...
        _qnaServices = qnaServices;
    }
    ```
1. 修改轮次处理程序，以便在用户的输入中查询任何已注册的知识库。 当机器人需要来自 QnAMaker 的答案时，请通过机器人代码调用 `GetAnswersAsync`，以便根据当前的上下文获取适当的答案。 若要访问自己的知识库，请更改下面的“没有答案”消息，为用户提供有用的说明。
    ```csharp
    public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
    {
        if (turnContext.Activity.Type == ActivityTypes.Message)
        {
            foreach(var qnaService in _qnaServices)
            {
                var response = await qnaService.GetAnswersAsync(turnContext);
                if (response != null && response.Length > 0)
                {
                    await turnContext.SendActivityAsync(
                        response[0].Answer,
                        cancellationToken: cancellationToken);
                    return;
                }
            }

            var msg = "No QnA Maker answers were found. This example uses a QnA Maker knowledge base that " +
                "focuses on smart light bulbs. Ask the bot questions like 'Why won't it turn on?' or 'I need help'.";

            await turnContext.SendActivityAsync(msg, cancellationToken: cancellationToken);
        }
        else
        {
            await turnContext.SendActivityAsync($"{turnContext.Activity.Type} event detected");
        }
    }
    ```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

1. 打开终端或命令提示符，转到项目的根目录。
1. 将 **botbuilder-ai** npm 包添加到项目。
    ```shell
    npm i botbuilder-ai
    ```
1. 在 **index.js** 文件中，添加此 require 语句。
    ```javascript
    const { QnAMaker } = require('botbuilder-ai');
    ```
1. 读取配置信息，以便生成 QnA Maker 服务。
    ```javascript
    // Read bot configuration from .bot file.
    // ...

    // Initialize the QnA knowledge bases for the bot.
    // Assume each QnA entry in the .bot file is well defined.
    const qnaServices = [];
    botConfig.services.forEach(s => {
        if (s.type == 'qna') {
            const endpoint = {
                knowledgeBaseId: s.kbId,
                endpointKey: s.endpointKey,
                host: s.hostname
            };
            const options = {};
            qnaServices.push(new QnAMaker(endpoint, options));
        }
    });

    // Get bot endpoint configuration by service name
    // ...
    ```
1. 更新机器人构造，以便传入 QnA 服务。
    ```javascript
    // Create the bot.
    const myBot = new MyBot(qnaServices);
    ```
1. 在 **bot.js** 文件中，添加一个构造函数。
    ```javascript
    constructor(qnaServices) {
        this.qnaServices = qnaServices;
    }
    ```
1. 更新轮次处理程序，以便在知识库中查询答案。
    ```javascript
    async onTurn(turnContext) {
        if (turnContext.activity.type === ActivityTypes.Message) {
            for (let i = 0; i < this.qnaServices.length; i++) {
                // Perform a call to the QnA Maker service to retrieve matching Question and Answer pairs.
                const qnaResults = await this.qnaServices[i].getAnswers(turnContext);

                // If an answer was received from QnA Maker, send the answer back to the user and exit.
                if (qnaResults[0]) {
                    await turnContext.sendActivity(qnaResults[0].answer);
                    return;
                }
            }
            // If no answers were returned from QnA Maker, reply with help.
            await turnContext.sendActivity('No QnA Maker answers were found. '
                + 'This example uses a QnA Maker Knowledge Base that focuses on smart light bulbs. '
                + `Ask the bot questions like "Why won't it turn on?" or "I need help."`);
        } else {
            await turnContext.sendActivity(`[${ turnContext.activity.type } event detected]`);
        }
    }
    ```

---

### <a name="test-the-bot-locally"></a>在本地测试机器人

目前，机器人应该能够回答一些问题。 在本地运行机器人，在模拟器中将其打开。

![测试 qna 示例](~/media/emulator-v4/qna-test-bot.png)

## <a name="re-publish-your-bot"></a>重新发布机器人

现在可以重新发布机器人。

[!INCLUDE [publish snippet](~/includes/deploy/snippet-publish.md)]

### <a name="test-the-published-bot"></a>测试已发布的机器人

发布机器人之后，请给 Azure 一到两分钟的时间来更新并启动机器人。

1. 使用模拟器测试机器人的生产终结点，或者使用 Azure 门户通过网上聊天测试机器人。

   不管采用哪一种方式，都会看到与本地测试相同的行为。

## <a name="clean-up-resources"></a>清理资源

<!-- In the first tutorial, we should tell them to use a new resource group, so that it is easy to clean up resources. We should also mention in this step in the first tutorial not to clean up resources if they are continuing with the sequence. -->

如果不打算继续使用此应用程序，请按以下步骤删除关联的资源：

1. 在 Azure 门户中打开机器人的资源组。
1. 单击“删除资源组”，删除该组及其包含的所有资源。
1. 在确认窗格中输入资源组名称，然后单击“删除”。

## <a name="next-steps"></a>后续步骤

若要了解如何为机器人添加功能，请参阅开发方法部分的文章。
> [!div class="nextstepaction"]
> [“后续步骤”按钮](bot-builder-howto-send-messages.md)
