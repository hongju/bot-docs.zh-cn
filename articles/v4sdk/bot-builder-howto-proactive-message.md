---
title: 如何使用主动消息传递功能 | Microsoft Docs
description: 了解如何使用机器人主动传递消息。
keywords: 主动消息
author: jonathanfingold
ms.author: jonathanfingold
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 05/01/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: fd53a897d9847432fd337402d40edfcd6f4ff061
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/18/2018
ms.locfileid: "39298180"
---
# <a name="how-to-use-proactive-messaging"></a>如何使用主动消息传递功能

通常，机器人发送被动消息，但有时候我们也需要能够发送 [主动消息](bot-builder-proactive-messages.md)。 

主动传递消息的一种常见情况是当机器人正在执行耗时不确定的任务时。 在此情况下，你可存储任务的相关消息，告诉用户机器人将在任务完成时再联系他们，并保证继续聊天。 完成任务后，机器人可主动发送确认消息来恢复聊天。

# <a name="ctabcs"></a>[C#](#tab/cs)

## <a name="notes-about-this-sample"></a>关于此示例的说明

我们正在修改基本 EchoBot 示例。
- 我们将使用 `Microsoft.Samples.Proactive` 作为命名空间。
- 我们会将状态文件替换为 `JobData.cs` 文件。
- 我们会将机器人文件替换为 `ProactiveBot.cs` 文件。

> [!NOTE]
> 目前，要使用主动消息传递功能，机器人必须具备有效的 ApplicationID 和密码。


## <a name="define-task-data"></a>定义任务数据

在此情况下，我们要跟踪各个用户可在不同的聊天中创建的任意任务。 因此，我们要使用一般的机器人状态中间件，而不是用户或聊天状态中间件。

下面的类定义了我们将用于单个作业的数据结构。


```csharp
using Microsoft.Bot.Schema;
using System.Collections.Generic;

namespace Microsoft.Samples.Proactive
{
    /// <summary>
    /// Class for storing job state. 
    /// </summary>
    public class JobData
    {
        /// <summary>
        /// The name to use to read and write this bot state object to storage.
        /// </summary>
        public readonly static string PropertyName = $"BotState:{typeof(Dictionary<int, JobData>).FullName}";

        public int JobNumber { get; set; } = 0;
        public bool Completed { get; set; } = false;

        /// <summary>
        /// The conversation reference to which to send status updates.
        /// </summary>
        public ConversationReference Conversation { get; set; }
    }
}
```


我们还需要将状态中间件添加到启动代码中。


在 `StartUp.cs` 文件中，更新 `ConfigureServices` 方法，进而将作业字典添加到机器人状态。 在以下代码中，它是指对 `options.Middleware.Add` 的最后一个调用。
```csharp
// This method gets called by the runtime. Use this method to add services to the container.
public void ConfigureServices(IServiceCollection services)
{
    services.AddBot<ProactiveBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);

        // The CatchExceptionMiddleware provides a top-level exception handler for your bot. 
        // Any exceptions thrown by other Middleware, or by your OnTurn method, will be 
        // caught here. To facillitate debugging, the exception is sent out, via Trace, 
        // to the emulator. Trace activities are NOT displayed to users, so in addition
        // an "Ooops" message is sent. 
        options.Middleware.Add(new CatchExceptionMiddleware<Exception>(async (context, exception) =>
        {
            await context.TraceActivity($"{nameof(ProactiveBot)} Exception", exception);
            await context.SendActivity("Sorry, it looks like something went wrong!");
        }));

        // The Memory Storage used here is for local bot debugging only. When the bot
        // is restarted, anything stored in memory will be gone. 
        IStorage dataStore = new MemoryStorage();

        // Using the base BotState here, since the job log is not necessarily tied to a
        // specific user or conversation.
        options.Middleware.Add(
            new BotState<Dictionary<int, JobData>>(
                dataStore, JobData.PropertyName, (context) => $"jobs/{typeof(Dictionary<int, JobData>)}"));
    });
}
```


## <a name="update-your-bot-to-create-and-run-jobs"></a>更新机器人以创建和运行作业

在每个回合，我们都让用户通过键入 `run` 或 `run job` 来创建作业。

我们的机器人将在该回合中采取以下步骤进行响应：
- 创建作业。
- 记录当前聊天的相关信息，以便我们稍后可发送主动消息。
- 让用户知道我们正在开始处理他们的作业，并将在完成后告知他们。
- 启动异步作业。
- 允许退出回合。

我们开始的作业是一个简单的 5 秒计时器，作业完成时发送主动消息。
- 如果调用适配器的“继续聊天”方法，会创建由机器人发起的新的回合。
- 此回合具有自己的回合上下文，其中包含状态消息。
- 我们使用此上下文向用户发送主动消息。



> [!NOTE]
> `GetAppId` 方法可用于在 .NET SDK 中启用主动消息传递功能。

```csharp
using Microsoft.Bot;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Connector.Authentication;
using Microsoft.Bot.Schema;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Security.Claims;
using System.Security.Principal;
using System.Threading;
using System.Threading.Tasks;

namespace Microsoft.Samples.Proactive
{
    public class ProactiveBot : IBot
    {
        /// <summary>
        /// Random number generator for job numbers.
        /// </summary>
        private static Random NumberGenerator = new Random();

        /// <summary>
        /// Gets the job log from the bot state.
        /// </summary>
        /// <param name="context">The current turn context.</param>
        /// <returns>The job log.</returns>
        private static Dictionary<int, JobData> GetJobLog(ITurnContext context)
        {
            return context.Services.Get<Dictionary<int, JobData>>(JobData.PropertyName);
        }

        /// <summary>
        /// Workaround to get the bot's app ID.
        /// </summary>
        /// <param name="context">The current turn context.</param>
        /// <returns>The application ID for the bot.</returns>
        private static string GetAppId(ITurnContext context)
        {
            // The BotFrameworkAdapter sets the identity provider on the context object.
            var claimsIdentity = context.Services.Get<IIdentity>("BotIdentity") as ClaimsIdentity;

            // For requests from a channel, the app ID is in the Audience claim of the JWT token.
            // For requests from the emulator, it is in the AppId claim.
            // For unauthenticated requests, we have anonymouse identity provided auth is disabled.
            // For Activities coming from Emulator AppId claim contains the Bot's AAD AppId.
            var botAppIdClaim =
                (claimsIdentity.Claims?.SingleOrDefault(claim => claim.Type == AuthenticationConstants.AudienceClaim)
                ?? claimsIdentity.Claims?.SingleOrDefault(claim => claim.Type == AuthenticationConstants.AppIdClaim));

            return botAppIdClaim?.Value;
        }

        /// <summary>
        /// Every Conversation turn calls this method.
        /// When the user types "run" or "run job", the bot starts a "job".
        /// When the job finishes, the bot proactively notifies the user.
        /// </summary>
        /// <param name="context">The turn context.</param>
        /// <remarks>When our virtual job finishes, it sends a proactive message
        /// to notify the user that the job completed.</remarks>
        public async Task OnTurn(ITurnContext context)
        {
            // This bot is only handling Messages
            if (context.Activity.Type is ActivityTypes.Message)
            {
                var text = context.Activity.AsMessageActivity()?.Text?.Trim().ToLower();
                switch (text)
                {
                    case "run":
                    case "run job":

                        var jobLog = GetJobLog(context);
                        var job = CreateJob(context, jobLog);
                        var appId = GetAppId(context);
                        var conversation = TurnContext.GetConversationReference(context.Activity);

                        await context.SendActivity($"We're starting job {job.JobNumber} for you. We'll notify you when it's complete.");

                        // Since the context is disposed at the end of the turn, extract and send the
                        // information we need to send the proactive message later.
                        var adapter = context.Adapter;
                        Task.Run(() =>
                        {
                            // Simulate a separate process to complete the user's job.
                            Thread.Sleep(5000);

                            // Perform bookkeeping and send the proactive message.
                            CompleteJob(adapter, appId, conversation, job.JobNumber);
                        });

                        break;

                    default:

                        await context.SendActivity("Type 'run' or 'run job' to start a new job.");

                        break;
                }
            }
        }

        /// <summary>
        /// Creates a simulated job and updates the job log.
        /// </summary>
        /// <param name="context">The current turn context.</param>
        /// <param name="jobLog">The job log.</param>
        /// <returns>A new job.</returns>
        private JobData CreateJob(ITurnContext context, Dictionary<int, JobData> jobLog)
        {
            // Generate a non-duplicate job number;
            int number;
            while (jobLog.ContainsKey(number = NumberGenerator.Next())) { }

            // Simulate creaing the job and logging it.
            var job = new JobData
            {
                JobNumber = number,
                Conversation = TurnContext.GetConversationReference(context.Activity)
            };
            jobLog.Add(job.JobNumber, job);

            // Return the created job.
            return job;
        }

        /// <summary>
        /// Performs bookkeeping and proactively notifies the user that their job completed.
        /// </summary>
        /// <param name="adapter">The bot adapter with which to send the message.</param>
        /// <param name="appId">The app ID of the bot to send the message from.</param>
        /// <param name="conversation">The conversation in which to put the message.</param>
        /// <param name="jobNumber">The number of the job that completed.</param>
        private async void CompleteJob(BotAdapter adapter, string appId, ConversationReference conversation, int jobNumber)
        {
            await adapter.ContinueConversation(appId, conversation, async context =>
            {
                // Get the job log from state, and retrieve the job.
                var jobLog = GetJobLog(context);
                var job = jobLog[jobNumber];

                // Perform bookkeeping.
                job.Completed = true;

                // Send the user a proactive confirmation message.
                await context.SendActivity($"Job {job.JobNumber} is complete.");
            });
        }
    }
}
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

用户必须至少向你的机器人发送一个被动样式的消息，然后你才可向该用户发送主动消息。 

你需要向机器人发送一条消息，因为它需要获取对活动对象的引用并将其保存在某处以备将来使用。 你可将活动对象视为用户地址，因为其所含信息涉及到其传入通道、用户 ID、聊天 ID，甚至是应接收任何后续消息的服务器。 此对象是简单的 JSON，应完整保存且不得篡改。

首先看一个简短的代码片段，它显示了如何在用户每次要求订阅时保存聊天引用：
```javascript
const { MemoryStorage } = require('botbuilder');

const storage = new MemoryStorage();

// Listen for incoming activity 
server.post('/api/messages', (req, res) => {
    // Route received activity to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type === 'message') {
            const utterances = (context.activity.text || '').trim().toLowerCase()
            if (utterances === 'subscribe') {
                var userId = await saveReference(TurnContext.getConversationReference(context.activity));
                await subscribeUser(userId)
                await context.sendActivity(`Thank You! We will message you shortly.`);
               
            } else{
                await context.sendActivity("Say 'subscribe' to start proactive message");
            }
    
        }
    });
});
```
上述片段调用 `saveReference()` 函数，该函数将使用 `MemoryStorage` 保存用户的引用并返回 `userId`。 该引用成功保存后，我们将调用 `subscribeUser()`，它会通知用户其已获得订阅。 

`subscribeUser()` 函数用于设置实际订阅。 让我们看一个简单的实现，它启动一个 2 秒计时器，并在计时器结束后立即向用户主动发送消息：

```javascript
// Persist info to storage
async function saveReference(reference){
    const userId = reference.activityId
    const changes = {};
    changes['reference/' + userId] = reference;
    await storage.write(changes); // Write reference info to persisted storage
    return userId;
}

// Subscribe user to a proactive call. In this case, we are using a setTimeOut() to trigger the proactive call
async function subscribeUser(userId) {
    setTimeout(async () => {
        const reference = await findReference(userId);
        if (reference) {
            await adapter.continueConversation(reference, async (context) => {
                await context.sendActivity("You have been notified");
            });
            
        }
    }, 2000); // Trigger after 2 secs
}

// Read the stored reference info from storage
async function findReference(userId){
    const referenceKey = 'reference/' + userId;
    var rows = await storage.read([referenceKey])
    var reference = await rows[referenceKey]

    return reference;
}
```

`subscribeUser()` 函数设置一个计时器，它通过从存储中读取引用对象来找到此对象。 如果找到引用对象，则可继续与用户聊天。 通过 `continueConversation` 方法，机器人可主动向已与之通信的聊天或用户发送消息。

---

## <a name="test-your-bot"></a>测试机器人

要测试智能机器人，请将其作为唯一注册的机器人部署到 Azure，并在网上聊天中进行测试，或使用模拟器在本地测试。
