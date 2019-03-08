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
ms.date: 02/27/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 494af4cc7936c8b191280d3f3f6cd73e7bc7d364
ms.sourcegitcommit: cf3786c6e092adec5409d852849927dc1428e8a2
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 03/01/2019
ms.locfileid: "57224895"
---
# <a name="use-qna-maker-to-answer-questions"></a>使用 QnA Maker 回答问题

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

可以使用 QnA Maker 服务为机器人添加问答支持。 创建自己的 QnA Maker 服务的基本要求之一是为其提供问题和答案。 在许多情况下，问题和答案已经存在于常见问题解答或其他文档等内容中。 其他时候，你希望以更自然的会话方式自定义问题的答案。

在本主题中，我们将创建一个知识库并在机器人中使用它。

## <a name="prerequisites"></a>先决条件
- [QnA Maker](https://www.qnamaker.ai/) 帐户
- 本文中的代码基于 **QnA Maker** 示例。 需要获取 [C# 示例](https://aka.ms/cs-qna)或 [Javascript 示例](https://aka.ms/js-qna-sample)的副本。
- [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/blob/master/README.md#download)
- 了解[机器人基础知识](bot-builder-basics.md)、[QnA Maker](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/overview/overview) 和 [.bot](bot-file-basics.md) 文件。

## <a name="create-a-qna-maker-service-and-publish-a-knowledge-base"></a>创建 QnA Maker 服务并发布知识库
1. 首先需要创建 [QnA Maker 服务](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/how-to/set-up-qnamaker-service-azure)。
1. 接下来，使用项目的 CognitiveModels 文件夹中的 `smartLightFAQ.tsv` 文件创建一个知识库。 QnA Maker 文档中列出了创建、训练和发布 QnA Maker [知识库](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/quickstarts/create-publish-knowledge-base)的步骤。 在执行以下步骤的过程中，请将知识库命名为 `qna`，并使用 `smartLightFAQ.tsv` 文件来填充知识库。
> 注意。 也可以参考本文来访问用户自己开发的 QnA Maker 知识库。

## <a name="obtain-values-to-connect-your-bot-to-the-knowledge-base"></a>获取用于将机器人连接到知识库的值
1. 在 [QnA Maker](https://www.qnamaker.ai/) 站点中选择知识库。
1. 打开知识库以后，选择“设置”。 记录针对“服务名称”显示的值。 使用 QnA Maker 门户界面时，此值可用于查找所需的知识库。 此值不可用于将机器人应用连接到此知识库。 
1. 向下滚动，找到“部署详细信息”并记录以下值：
   - POST /knowledgebases/<知识库 ID>/getAnswers
   - Host: <主机名>/qnamaker
   - Authorization:EndpointKey <终结点密钥>
   
这三个值为应用提供所需的信息，使其能够通过 Azure QnA 服务连接到 QnA Maker 知识库。  

## <a name="update-the-bot-file"></a>更新 .bot 文件
首先，请将访问知识库所需的信息（包括主机名、终结点密钥和知识库 ID (kbId)）添加到 `qnamaker.bot` 中。 这些是在 QnA Maker 中通过知识库的“设置”保存的值。 
> 注意。 如果将 QnA Maker 知识库访问权限添加到现有的机器人应用程序中，请务必在 .bot 文件中添加如下所示的 "type": "qna" 节。 此节中的 "name" 值提供所需的密钥用于从应用内部访问此信息。

```json
{
  "name": "qnamaker",
  "services": [
    {
      "type": "endpoint",
      "name": "development",
      "endpoint": "http://localhost:3978/api/messages",
      "appId": "",
      "appPassword": "",
      "id": "25"    
    },
    {
      "type": "qna",
      "name": "QnABot",
      "kbId": "<Your_Knowledge_Base_Id>",
      "subscriptionKey": "",
      "endpointKey": "<Your_Endpoint_Key>",
      "hostname": "<Your_Hostname>",
      "id": "117"
    }
  ],
  "padlock": "",
   "version": "2.0"
}
```

# <a name="ctabcs"></a>[C#](#tab/cs)

请确保为项目安装 **Microsoft.Bot.Builder.AI.QnA** NuGet 包。

接下来，在 **BotServices.cs** 中初始化 `BotServices` 类的新实例，以便从 .bot 文件中获取上述信息。 使用 BotConfiguration 类配置外部服务。

```csharp
using Microsoft.Bot.Builder.AI.QnA;
using Microsoft.Bot.Configuration;
```

```csharp
public BotServices(BotConfiguration botConfiguration)
{
    foreach (var service in botConfiguration.Services)
    {
        switch (service.Type)
        {
            case ServiceTypes.QnA:
                {
                    // Create a QnA Maker that is initialized and suitable for passing
                    // into the IBot-derived class (QnABot).
                    var qna = service as QnAMakerService;

                    // ...

                    var qnaEndpoint = new QnAMakerEndpoint()
                    {
                        KnowledgeBaseId = qna.KbId,
                        EndpointKey = qna.EndpointKey,
                        Host = qna.Hostname,
                    };

                    var qnaMaker = new QnAMaker(qnaEndpoint);
                    QnAServices.Add(qna.Name, qnaMaker);
                    break;
                }
        }
    }
}
```

然后在 **QnABot.cs** 中，将此 QnAMaker 实例分配到机器人。 若要访问自己的知识库，请更改下面显示的“欢迎”消息，为用户提供有用的初始说明。 此类也是定义静态变量 `QnAMakerKey` 的位置。 此变量指向 .bot 文件中包含用于访问 QnA Maker 知识库的连接信息的节。

```csharp
public class QnABot : IBot
{
    public static readonly string QnAMakerKey = "QnABot";

    private const string WelcomeText = @"This bot will introduce you to QnA Maker.
                                         Ask a question to get started.";
    private readonly BotServices _services;

    public QnABot(BotServices services)
    {
        _services = services ?? throw new System.ArgumentNullException(nameof(services));
        if (!_services.QnAServices.ContainsKey(QnAMakerKey))
        {
            throw new System.ArgumentException(
                $"Invalid configuration. Please check your '.bot' file for a QnA service named '{QnAMakerKey}'.");
        }
    }
}
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

在示例中，启动代码位于 **index.js** 文件中，机器人逻辑代码位于 **bot.js** 文件中，其他配置信息位于 **qnamaker.bot** 文件中。

在 **index.js** 文件中，我们读取配置信息，以便生成 QnA Maker 服务并初始化机器人。

在 .bot 文件中，将 `QNA_CONFIGURATION` 的值更新为 "name": 值。 这是 .bot 文件中包含用于访问 QnA Maker 知识库的连接参数的 "type": "qna" 节中的密钥。

```js
// Name of the QnA Maker service stored in "name" field of .bot file. 
const QNA_CONFIGURATION = '<SERVICE_NAME_FROM_.BOT_FILE>';

// Get endpoint and QnA Maker configurations by service name.
const endpointConfig = botConfig.findServiceByNameOrId(BOT_CONFIGURATION);
const qnaConfig = botConfig.findServiceByNameOrId(QNA_CONFIGURATION);

// Map the contents to the required format for `QnAMaker`.
const qnaEndpointSettings = {
    knowledgeBaseId: qnaConfig.kbId,
    endpointKey: qnaConfig.endpointKey,
    host: qnaConfig.hostname
};

// Create adapter...

// Create the QnAMakerBot.
let bot;
try {
    bot = new QnAMakerBot(qnaEndpointSettings, {});
} catch (err) {
    console.error(`[botInitializationError]: ${ err }`);
    process.exit();
}
```

然后创建 HTTP 服务器并侦听传入请求，此类请求会生成对机器人逻辑的调用。

```js
// Create HTTP server.
let server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function() {
    console.log(`\n${ server.name } listening to ${ server.url }.`);
    console.log(`\nGet Bot Framework Emulator: https://aka.ms/botframework-emulator.`);
    console.log(`\nTo talk to your bot, open qnamaker.bot file in the emulator.`);
});

// Listen for incoming requests.
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (turnContext) => {
        await bot.onTurn(turnContext);
    });
});
```

---

## <a name="calling-qna-maker-from-your-bot"></a>通过机器人调用 QnA Maker

# <a name="ctabcs"></a>[C#](#tab/cs)

当机器人需要来自 QnAMaker 的答案时，请通过机器人代码调用 `GetAnswersAsync()`，以便根据当前的上下文获取适当的答案。 若要访问自己的知识库，请更改下面的“没有答案”消息，为用户提供有用的说明。

```csharp
// Check QnA Maker model
var response = await _services.QnAServices[QnAMakerKey].GetAnswersAsync(turnContext);
if (response != null && response.Length > 0)
{
    await turnContext.SendActivityAsync(response[0].Answer, cancellationToken: cancellationToken);
}
else
{
    var msg = @"No QnA Maker answers were found. This example uses a QnA Maker Knowledge Base that focuses on smart light bulbs.
                To see QnA Maker in action, ask the bot questions like 'Why won't it turn on?' or 'I need help'.";
    await turnContext.SendActivityAsync(msg, cancellationToken: cancellationToken);
}

    /// ...
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

<!-- 4.2 uses `generateAnswer`, whereas 4.3 will use `getAnswers`. Change here and in the code when 4.3 goes live.
In the **bot.js** file, we pass the user's input to the QnA Maker service's `getAnswers` method to get answers from the knowledge base. If you are accessing your own knowledge base, change the _no answers_ and _welcome_ messages below to provide useful instructions for your users.
 -->
在 **bot.js** 文件中，我们将用户的输入传递给 QnA Maker 服务的 `generateAnswer` 方法，以便从知识库获取答案。 若要访问自己的知识库，请更改下面的“没有答案”和“欢迎”消息，为用户提供有用的说明。

```javascript
const { ActivityTypes, TurnContext } = require('botbuilder');
const { QnAMaker, QnAMakerEndpoint, QnAMakerOptions } = require('botbuilder-ai');

/**
 * A simple bot that responds to utterances with answers from QnA Maker.
 * If an answer is not found for an utterance, the bot responds with help.
 */
class QnAMakerBot {
    /**
     * The QnAMakerBot constructor requires one argument (`endpoint`) which is used to create an instance of `QnAMaker`.
     * @param {QnAMakerEndpoint} endpoint The basic configuration needed to call QnA Maker. In this sample the configuration is retrieved from the .bot file.
     * @param {QnAMakerOptions} config An optional parameter that contains additional settings for configuring a `QnAMaker` when calling the service.
     */
    constructor(endpoint, qnaOptions) {
        this.qnaMaker = new QnAMaker(endpoint, qnaOptions);
    }

    /**
     * Every conversation turn for our QnAMakerBot will call this method.
     * @param {TurnContext} turnContext Contains all the data needed for processing the conversation turn.
     */
    async onTurn(turnContext) {
        // By checking the incoming Activity type, the bot only calls QnA Maker in appropriate cases.
        if (turnContext.activity.type === ActivityTypes.Message) {
            // Perform a call to the QnA Maker service to retrieve matching Question and Answer pairs.
            const qnaResults = await this.qnaMaker.generateAnswer(turnContext);

            // If an answer was received from QnA Maker, send the answer back to the user.
            if (qnaResults[0]) {
                await turnContext.sendActivity(qnaResults[0].answer);

            // If no answers were returned from QnA Maker, reply with help.
            } else {
                await turnContext.sendActivity('No QnA Maker answers were found. This example uses a QnA Maker Knowledge Base that focuses on smart light bulbs. To see QnA Maker in action, ask the bot questions like "Why won\'t it turn on?" or "I need help."');
            }

        // If the Activity is a ConversationUpdate, send a greeting message to the user.
        } else if (turnContext.activity.type === ActivityTypes.ConversationUpdate &&
                   turnContext.activity.recipient.id !== turnContext.activity.membersAdded[0].id) {
            await turnContext.sendActivity('Welcome to the QnA Maker sample! Ask me a question and I will try to answer it.');

        // Respond to all other Activity types.
        } else if (turnContext.activity.type !== ActivityTypes.ConversationUpdate) {
            await turnContext.sendActivity(`[${ turnContext.activity.type }]-type activity detected.`);
        }
    }
}

module.exports.QnAMakerBot = QnAMakerBot;
```

---

## <a name="test-the-bot"></a>测试机器人

在计算机本地运行示例。 如需说明，请参阅 [C#](https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/csharp_dotnetcore/11.qnamaker) 或 [JS](https://github.com/Microsoft/BotBuilder-Samples/blob/master/samples/javascript_nodejs/11.qnamaker/README.md) 示例的自述文件。

在仿真器中，按如下所示将消息发送到机器人。

![测试 qna 示例](~/media/emulator-v4/qna-test-bot.png)


## <a name="next-steps"></a>后续步骤

QnA Maker 可与其他认知服务结合使用，使机器人更加强大。 调度工具提供了一种在机器人中将 QnA 与语言理解 (LUIS) 相结合的方法。

> [!div class="nextstepaction"]
> [使用调度工具组合 LUIS 和 QnA 服务](./bot-builder-tutorial-dispatch.md)
