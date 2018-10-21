---
title: 使用 QnA Maker 回答问题 | Microsoft Docs
description: 了解如何在机器人中使用 QnA maker。
keywords: 问题和答案, QnA, 常见问题解答, 中间件
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 10/08/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: e3d7c0a541a4b7f8c2065c5db724e5d79ced54b8
ms.sourcegitcommit: 54ed5000c67a5b59e23b667547565dd96c7302f9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/13/2018
ms.locfileid: "49315173"
---
# <a name="use-qna-maker-to-answer-questions"></a>使用 QnA Maker 回答问题

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

可以使用 QnA Maker 服务为机器人添加问答支持。 创建自己的 QnA Maker 服务的基本要求之一是为其提供问题和答案。 在许多情况下，问题和答案已经存在于常见问题解答或其他文档等内容中。 其他时候，你希望以更自然的会话方式自定义问题的答案。

## <a name="prerequisites"></a>先决条件
- 创建 [QnA Maker](https://www.qnamaker.ai/) 帐户
- 下载 QnA Maker 示例 [[C#](https://aka.ms/cs-qna) | [JavaScript](https://aka.ms/js-qna-sample)]

## <a name="create-a-qna-maker-service-and-publish-a-knowledge-base"></a>创建 QnA Maker 服务并发布知识库

创建 QnA Maker 帐户之后，请根据说明创建 [QnA Maker 服务](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/how-to/set-up-qnamaker-service-azure)和[知识库](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/quickstarts/create-publish-knowledge-base)。 

发布知识库以后，需记录以下值，以便通过编程方式将机器人连接到知识库。
- 在 [QnA Maker](https://www.qnamaker.ai/) 站点中选择知识库。
- 打开知识库以后，选择“设置”。 将针对“服务名称”显示的值记录为 <your_kb_name>
- 向下滚动，找到“部署详细信息”并记录以下值：
   - POST /knowledgebases/<your_knowledge_base_id>/generateAnswer
   - 主机: https://<you_hostname>.azurewebsites.net/qnamaker
   - 授权: EndpointKey <your_endpoint_key>

## <a name="installing-packages"></a>安装程序包

在我们编码之前，请确保你拥有 QnA Maker 所需的程序包。

# <a name="ctabcs"></a>[C#](#tab/cs)

将以下 [NuGet 包](https://docs.microsoft.com/en-us/nuget/tools/package-manager-ui)添加到机器人。

* `Microsoft.Bot.Builder.AI.QnA`

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

QnA Maker 功能位于 `botbuilder-ai` 包中。 可通过 npm 向项目添加以下包：

```shell
npm install --save botbuilder-ai
```

---

## <a name="using-cli-tools-to-update-your-bot-configuration"></a>使用 CLI 工具更新 .bot 配置

若要获取知识库访问值，另一种方法是使用 [qnamaker](https://aka.ms/botbuilder-tools-qnaMaker) 和 [msbot](https://aka.ms/botbuilder-tools-msbot-readme) BotBuilder CLI 工具获取知识库的元数据并将其添加到 .bot 文件。

1. 打开终端或命令提示符，导航到机器人项目的根目录。
2. 运行 `qnamaker init`，创建一个 QnA Maker 资源文件 (**.qnamakerrc**)。 它会提示输入 QnA Maker 订阅密钥。
3. 运行以下命令，下载元数据并将其添加到机器人的配置文件。

    ```shell
    qnamaker get kb --kbId <your-kb-id> --msbot | msbot connect qna --stdin [--secret <your-secret>]
    ```
如果已加密配置文件，则需提供密钥才能更新该文件。

## <a name="using-qna-maker"></a>使用 QnA Maker
初始化机器人时，先添加对 QnA Maker 的引用。 然后，我们可以在机器人逻辑中调用它。

# <a name="ctabcs"></a>[C#](#tab/cs)
打开此前下载的 QnA Maker 示例。 我们会根据需要修改此代码。
首先，请将访问知识库所需的信息（包括主机名、终结点密钥和知识库 ID (KbId)）添加到 `BotConfiguration.bot` 中。 这些是在 QnA Maker 中通过知识库的“设置”保存的值。

```json
{
  "name": "QnABotSample",
  "services": [
    {
      "type": "endpoint",
      "name": "development",
      "endpoint": "http://localhost:3978/api/messages",
      "appId": "",
      "id": "1",
      "appPassword": ""
    },
    {
      "type": "qna",
      "name": "QnABot",
      "KbId": "<YOUR_KNOWLEDGE_BASE_ID>",
      "Hostname": "https://<YOUR_HOSTNAME>.azurewebsites.net/qnamaker",
      "EndpointKey": "<YOUR_ENDPOINT_KEY>"
    }
  ],
  "version": "2.0",
  "padlock": ""
}
```

接下来，在 `Startup.cs` 中创建 QnA Maker 实例。 这样会从 `BotConfiguration.bot` 文件中获取上述信息。 这些字符串还可以进行测试所需的硬编码。

```csharp
private static BotServices InitBotServices(BotConfiguration config)
{
    var qnaServices = new Dictionary<string, QnAMaker>();
    foreach (var service in config.Services)
    {
        switch (service.Type)
        {
            case ServiceTypes.QnA:
            {
                // Create a QnA Maker that is initialized and suitable for passing
                // into the IBot-derived class (QnABot).
                var qna = (QnAMakerService)service;
                if (qna == null)
                {
                    throw new InvalidOperationException("The QnA service is not configured correctly in your '.bot' file.");
                }

                if (string.IsNullOrWhiteSpace(qna.KbId))
                {
                    throw new InvalidOperationException("The QnA KnowledgeBaseId ('kbId') is required to run this sample. Please update your '.bot' file.");
                }

                if (string.IsNullOrWhiteSpace(qna.EndpointKey))
                {
                    throw new InvalidOperationException("The QnA EndpointKey ('endpointKey') is required to run this sample. Please update your '.bot' file.");
                }

                if (string.IsNullOrWhiteSpace(qna.Hostname))
                {
                    throw new InvalidOperationException("The QnA Host ('hostname') is required to run this sample. Please update your '.bot' file.");
                }

                var qnaEndpoint = new QnAMakerEndpoint()
                {
                    KnowledgeBaseId = qna.KbId,
                    EndpointKey = qna.EndpointKey,
                    Host = qna.Hostname,
                };

                var qnaMaker = new QnAMaker(qnaEndpoint);
                qnaServices.Add(qna.Name, qnaMaker);
                break;
            }
        }
    }
    var connectedServices = new BotServices(qnaServices);
    return connectedServices;
}
```

然后，我们需向机器人提供此 QnAMaker 实例。 打开 `QnABot.cs`，并在文件顶部添加以下代码。 若要访问自己的知识库，请更改下面显示的“欢迎”消息，为用户提供有用的初始说明。

```csharp
public class QnABot : IBot
{
    public static readonly string QnAMakerKey = "QnABot";
    private const string WelcomeText = @"This bot will introduce you to QnA Maker.
                                         Ask a quesiton to get started.";
    private readonly BotServices _services;
    public QnABot(BotServices services)
    {
        _services = services ?? throw new System.ArgumentNullException(nameof(services));
        Console.WriteLine($"{_services}");
        if (!_services.QnAServices.ContainsKey(QnAMakerKey))
        {
            throw new System.ArgumentException($"Invalid configuration. Please check your '.bot' file for a QnA service named '{QnAMakerKey}'.");
        }
    }
}
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
打开此前下载的 QnA Maker 示例。 我们会根据需要修改此代码。
在示例中，启动代码位于 **index.js** 文件中，机器人逻辑代码位于 **bot.js** 文件中，其他配置信息位于 **qnamaker.bot** 文件中。

在按说明创建知识库并更新 **.bot** 文件以后，**qnamaker.bot** 文件应包含一个适用于 QnA Maker 知识库的服务条目。

```json
{
    "name": "qnamaker",
    "description": "",
    "services": [
        {
            "type": "endpoint",
            "name": "development",
            "id": "1",
            "appId": "",
            "appPassword": "",
            "endpoint": "http://localhost:3978/api/messages"
        },
        {
            "type": "qna",
            "name": "<YOUR_KB_NAME>",
            "kbId": "<YOUR_KNOWLEDGE_BASE_ID>",
            "endpointKey": "<YOUR_ENDPOINT_KEY>",
            "hostname": "https://<YOUR_HOSTNAME>.azurewebsites.net/qnamaker",
            "id": "221"
        }
    ],
    "padlock": "",
    "version": "2.0"
}
```

在 **index.js** 文件中，我们读取配置信息，以便生成 QnA Maker 服务并初始化机器人。

将 `QNA_CONFIGURATION` 的值更新为知识库的名称，正如其显示在配置文件中的那样。

```js
// QnA Maker knowledge base name as specified in .bot file.
const QNA_CONFIGURATION = '<YOUR_KB_NAME>';

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
            const qnaResults = await this.qnaMaker.generateAnswer(turnContext.activity.text);

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

询问机器人问题，以查看来自 QnA Maker 服务的回复。 若要详细了解如何测试和发布 QnA 服务，请参阅有关如何[测试知识库](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/how-to/test-knowledge-base)的 QnA Maker 文章。

## <a name="next-steps"></a>后续步骤

QnA Maker 可与其他认知服务结合使用，使机器人更加强大。 调度工具提供了一种在机器人中将 QnA 与语言理解 (LUIS) 相结合的方法。

> [!div class="nextstepaction"]
> [使用调度工具组合 LUIS 和 QnA 服务](./bot-builder-tutorial-dispatch.md)
