---
title: 使用 QnA Maker | Microsoft Docs
description: 了解如何在机器人中使用 QnA maker。
keywords: 问题和答案, QnA, 常见问题解答, 中间件
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 03/13/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 78bc2c849a2c1900da33c7419693a7ff84c43cb0
ms.sourcegitcommit: dcbc8ad992a3e242a11ebcdf0ee99714d919a877
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/30/2018
ms.locfileid: "39352946"
---
# <a name="how-to-use-qna-maker"></a>如何使用 QnA Maker

要为机器人添加简单的问答支持，你可以使用 [QnA Maker](https://qnamaker.ai/) 服务。

编写自己的 QnA Maker 服务的基本要求之一是为其提供问题和答案。 在许多情况下，问题和答案已经存在于常见问题解答或其他文档等内容中。 其他时候，你希望以更自然的会话方式自定义问题的答案。 

## <a name="create-a-qna-maker-service"></a>创建 QnA Maker 服务
首先创建一个帐户并登录 [QnA Maker](https://qnamaker.ai/)。 导航到“创建知识库”。 单击“创建 QnA 服务”，然后按照创建 Azure QnA 服务的说明进行操作。

![Qna 图像 1](media/QnA_1.png)

此时重定向到[创建 QnA Maker](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesQnAMaker)。 完成表单，并单击“创建”。

![Qna 图像 2](media/QnA_2.png)
 
在 Azure 门户中创建 QnA 服务后，你将获得“资源管理”标题下的密钥，你可以忽略这些密钥。 继续执行步骤 2 以连接 Azure QnA 服务。 刷新页面以选择刚刚创建的 Azure 服务，并输入知识库的名称。

![Qna 图像 3](media/QnA_3.png)

单击“创建知识库”。 输入你自己的问题和答案，或复制以下示例。 

![Qna 图像 4](media/QnA_4.png)

或者，你可以选择“填充知识库”并提供文件或 URL。 用于生成简单 QnA Maker 服务的示例源文件在[此处](https://aka.ms/qna-tsv)。

添加新的 QnA 对或填充知识库后，单击“保存并定型”。 完成后，在“发布”选项卡上，单击“发布”。

要将 QnA 服务连接到机器人，你将需要包含知识库 ID 和 QnA Maker 订阅密钥的 HTTP 请求字符串。 从发布的结果复制示例 HTTP 请求。 

![Qna 图像 5](media/QnA_5.png)

## <a name="installing-packages"></a>安装程序包

在我们编码之前，请确保你拥有 QnA Maker 所需的程序包。

# <a name="ctabcs"></a>[C#](#tab/cs)

在以下 NuGet 包的 v4 预发布版本中[添加引用](https://docs.microsoft.com/en-us/nuget/tools/package-manager-ui)：

* `Microsoft.Bot.Builder.Ai.QnA`

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

可以使用 botbuilder-ai 程序包将这些服务中的任何一项添加到机器人。 可以通过 npm 将此包添加到项目中：

* `npm install --save botbuilder@preview`
* `npm install --save botbuilder-ai@preview`

---


## <a name="using-qna-maker"></a>使用 QnA Maker

QnA Maker 首先作为中间件添加。 然后我们可以在机器人逻辑中使用结果。

# <a name="ctabcs"></a>[C#](#tab/cs)

更新 `Startup.cs` 文件中的 `ConfigureServices` 方法以添加 `QnAMakerMiddleware` 对象。 只需将知识库添加到机器人的中间件堆栈，即可将机器人配置为检查从用户收到的每条消息的知识库。


```csharp
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Bot.Builder.Ai.Qna;
using Microsoft.Bot.Builder.BotFramework;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Builder.Integration.AspNet.Core;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Logging;
using System;

public void ConfigureServices(IServiceCollection services)
{
    services.AddSingleton(_ => Configuration);
    services.AddBot<AiBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);

        var endpoint = new QnAMakerEndpoint
        {
           KnowledgeBaseId = "YOUR-KB-ID",
           // Get the Host from the HTTP request example at https://www.qnamaker.ai
           // For GA services: https://<Service-Name>.azurewebsites.net/qnamaker
           // For Preview services: https://westus.api.cognitive.microsoft.com/qnamaker/v2.0           
           Host = "YOUR-HTTP-REQUEST-HOST",
           EndpointKey = "YOUR-QNA-MAKER-SUBSCRIPTION-KEY"
        };
        options.Middleware.Add(new QnAMakerMiddleware(endpoint));
    });
}
```



编辑 EchoBot.cs 文件中的代码，以便 `OnTurn` 在 QnA Maker 中间件未向用户的问题发送响应的情况下发送回退消息：

```csharp
using System.Threading.Tasks;
using Microsoft.Bot;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Schema;

namespace Bot_Builder_Echo_Bot_QnA
{
    public class EchoBot : IBot
    {    
        public async Task OnTurn(ITurnContext context)
        {
            // This bot is only handling Messages
            if (context.Activity.Type == ActivityTypes.Message)
            {             
                if (!context.Responded)
                {
                    // QnA didn't send the user an answer
                    await context.SendActivity("Sorry, I couldn't find a good match in the KB.");

                }
            }
        }
    }
}
```


有关示例机器人，请参阅 [QnA Maker 示例](https://aka.ms/qna-cs-bot-sample)。

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

首先请求/导入 [QnAMaker](https://github.com/Microsoft/botbuilder-js/tree/master/doc/botbuilder-ai/classes/botbuilder_ai.qnamaker.md) 类：

```js
const { QnAMaker } = require('botbuilder-ai');
```

通过基于 QnA 服务的 HTTP 请求使用字符串初始化 `QnAMaker` 来创建它。 可以从“设置”>“部署详细信息”下的 [QnA Maker 门户](https://qnamaker.ai)复制服务的示例请求。


字符串的格式取决于 QnA Maker 服务是使用 GA 还是 QnA Maker 的预览版本。 复制示例 HTTP 请求，并在初始化 `QnAMaker` 时获取要使用的知识库 ID、订阅密钥和主机。

<!--
**Preview**
```js
const qnaEndpointString = 
    // Replace xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx with your knowledge base ID
    "POST /knowledgebases/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/generateAnswer\r\n" + 
    "Host: https://westus.api.cognitive.microsoft.com/qnamaker/v2.0\r\n" +
    // Replace xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx with your QnAMaker subscription key
    "Ocp-Apim-Subscription-Key: xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx\r\n"
const qna = new QnAMaker(qnaEndpointString);
```

**GA**
```js
const qnaEndpointString = 
    // Replace xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx with your knowledge base ID
    "POST /knowledgebases/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/generateAnswer\r\n" + 
    // Replace <Service-Name> to match the Azure URL where your service is hosted
    "Host: https://<Service-Name>.azurewebsites.net/qnamaker\r\n" +
    // Replace xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx with your QnAMaker subscription key
    "Authorization: EndpointKey xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx\r\n"
const qna = new QnAMaker(qnaEndpointString);
```
-->
**预览**
```js
const qna = new QnAMaker(
    {
        knowledgeBaseId: '<KNOWLEDGE-BASE-ID>',
        endpointKey: '<QNA-SUBSCRIPTION-KEY>',
        host: 'https://westus.api.cognitive.microsoft.com/qnamaker/v2.0'
    },
    {
        // set this to true to send answers from QnA Maker
        answerBeforeNext: true
    }
);
```
**GA**
```js
const qna = new QnAMaker(
    {
        knowledgeBaseId: '<KNOWLEDGE-BASE-ID>',
        endpointKey: '<QNA-SUBSCRIPTION-KEY>',
        host: 'https://<Service-Name>.azurewebsites.net/qnamaker'
    },
    {
        answerBeforeNext: true
    }
);
```
通过简单地将 QNA maker 添加到机器人的中间件堆栈，可以将机器人配置为自动调用它：

```js
// Add QnA Maker as middleware
adapter.use(qna);

// Listen for incoming requests 
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        // If `!context.responded`, that means an answer wasn't found for the user's utterance.
        // In this case, we send the user a fallback message.
        if (context.activity.type === 'message' && !context.responded) {
            await context.sendActivity('No QnA Maker answers were found.');
        } else if (context.activity.type !== 'message') {
            await context.sendActivity(`[${context.activity.type} event detected]`);
        }
    });
});
```

初始化 `QnAMaker` 时，将 `answerBeforeNext` 初始化为 `true` 意味着如果 QnA Maker 中间件找到答案，则它会自动响应，然后才能在 `processActivity` 中获取机器人的主逻辑。 如果与此相反，将 `answerBeforeNext` 设置为 `false`，则机器人只会在 `processActivity` 中运行所有机器人主逻辑后调用 QnA Maker，并且仅适用于机器人未回复用户的情况。 

## <a name="calling-qna-maker-without-using-middleware"></a>不使用中间件调用 QnA Maker

在前面的示例中，`adapter.use(qna);` 语句表示 QnA 作为中间件运行，因此响应机器人收到的每条消息。 为了更好地控制调用 QnA Maker 的方式和时间，可以直接从机器人的逻辑中调用 `qna.answer()`，而不是将其作为一个中间件安装。

删除 `adapter.use(qna);` 语句并使用以下代码直接调用 QnA Maker。

```js
// Listen for incoming activity 
server.post('/api/messages', (req, res) => {
    // Route received activity to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type === 'message') {
            var handled = await qna.answer(context)
                if (!handled) {
                    await context.sendActivity(`I'm sorry. I didn't understand.`);
                }
        }
    });
});
```

另一种自定义 QnA Maker 的方法是使用 `qna.generateAnswer()`。 此方法允许从 QnA Maker 获得有关答案的更多详细信息。


```js
// Listen for incoming activity 
server.post('/api/messages', (req, res) => {
    // Route received activity to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type === 'message') {
            // Get all the answers QnA Maker finds
            var results = await qna.generateAnswer(context.activity.text);
                if (results && results.length > 0) {
                    await context.sendActivity(results[0].answer);
                } else {
                    await context.sendActivity(`I don't know.`);
                }
    
        }
    });
});
```
---

询问机器人问题，以查看来自 QnA Maker 服务的回复。

![Qna 图像 6](media/QnA_6.png)



## <a name="next-steps"></a>后续步骤

QnA Maker 可与其他认知服务结合使用，使机器人更加强大。 调度工具提供了一种在机器人中将 QnA 与语言理解 (LUIS) 相结合的方法。

> [!div class="nextstepaction"]
> [使用调度工具组合 LUIS 应用和 QnA 服务](./bot-builder-tutorial-dispatch.md)
