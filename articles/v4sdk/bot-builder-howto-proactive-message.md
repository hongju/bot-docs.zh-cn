---
title: 向用户发送主动通知 | Microsoft Docs
description: 了解如何发送通知消息
keywords: 主动消息, 通知消息, 机器人通知
author: jonathanfingold
ms.author: jonathanfingold
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 11/15/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 207dfaf71e8af7af3a36e496deb506ff9d0c13c8
ms.sourcegitcommit: cf3786c6e092adec5409d852849927dc1428e8a2
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 03/01/2019
ms.locfileid: "57224885"
---
# <a name="send-proactive-notifications-to-users"></a>向用户发送主动通知

[!INCLUDE [pre-release-label](~/includes/pre-release-label.md)]

通常，机器人向用户发送的每条消息与用户先前的输入直接相关。
在某些情况下，机器人可能需要向用户发送与当前聊天主题或用户发送的最后一条消息不直接相关的消息。 这些类型的消息称为主动消息。

## <a name="proactive-messages"></a>主动消息

主动消息在各种场景中都可以发挥作用。 如果机器人设置了计时器或提醒，它将需要在时间到来时通知用户。 或者，如果机器人从外部系统接收通知，则可能需要立即将该信息传达给用户。 例如，如果用户之前已经请求机器人监控产品的价格，则机器人可以在产品价格下降了 20% 时提醒用户。 或者，如果机器人需要一些时间来编译对用户问题的响应，则可以通知用户延迟并允许会话在此期间继续。 当机器人编译完对问题的响应时，将与用户共享该信息。

在机器人中实现主动消息时：

- 不要在短时间内发送多条主动消息。 某些通道强制限制机器人向用户发送消息的频率，并且如果违反这些限制，将禁用机器人。
- 不要通过其他方式（如电子邮件或短信）向之前未与机器人交互或未请求与机器人联系的用户发送主动消息。

临时主动消息是最简单的主动消息类型。 只要触发了消息，机器人就会简单地将消息插入到聊天中，而不考虑用户当前是否正在致力于与机器人进行的聊天的单独主题，并且不会尝试以任何方式更改聊天。

若要更顺畅地处理通知，请考虑将通知集成到聊天流中的其他方法，例如在聊天状态中设置标志或将通知添加到队列。

## <a name="prerequisites"></a>先决条件

- 了解[机器人基础知识](bot-builder-basics.md)以及如何[管理状态](bot-builder-concept-state.md)。
- 以 [C#](https://aka.ms/proactive-sample-cs) 或 [JS](https://aka.ms/proactive-sample-js) 语言编写的**主动消息示例**的副本。 此示例用于解释本文中所述的主动消息。 

## <a name="about-the-sample-code"></a>关于示例代码

“主动消息示例”对所需时间不确定的用户任务建模。 机器人会存储任务的相关信息，告诉用户它将在任务完成后再联系他们，并让聊天继续。 完成任务后，机器人会在原始聊天中主动发送确认消息。

## <a name="define-job-data-and-state"></a>定义作业数据和状态

在此方案中，我们要跟踪各个用户在不同的聊天中创建的任意作业。 需存储每项作业的相关信息，包括聊天、引用以及作业标识符。 我们需要：

- 聊天引用，以便可将主动消息发送到正确的聊天。
- 通过某种方式来标识作业。 例如，使用简单的时间戳。
- 存储独立于聊天或用户状态的作业状态。

我们将通过扩展 _bot state_ 来定义自己的机器人范围的状态管理对象。 Bot Framework 使用 _storage key_ 和轮次上下文来保留和检索状态。 有关详细信息，请参阅[管理状态](bot-builder-concept-state.md)。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

需定义作业数据和作业状态的类。 还需注册机器人并为作业日志设置状态属性访问器。

### <a name="define-a-class-for-job-data"></a>定义作业数据的类

`JobLog` 类跟踪按作业编号（时间戳）进行索引的作业数据。 `JobLog` 类跟踪所有未完成的作业。  每个作业由唯一键标识。 `JobData` 描述作业的状态，定义为字典的内部类。

**JobLog.cs**
```csharp
public class JobLog : Dictionary<long, JobLog.JobData>
{
    public class JobData
    {
        // Gets or sets the time-stamp for the job.
        public long TimeStamp { get; set; } = 0;

        // Gets or sets a value indicating whether indicates whether the job has completed.
        public bool Completed { get; set; } = false;

        // Gets or sets the conversation reference to which to send status updates.
        public ConversationReference Conversation { get; set; }
    }
}
```

### <a name="define-a-state-management-class"></a>定义状态管理类

`JobState` 类会管理独立于聊天或用户状态的作业状态。

**JobState.cs**
```csharp
using Microsoft.Bot.Builder;

/// A BotState for managing bot state for "bot jobs".
public class JobState : BotState
{
    // The key used to cache the state information in the turn context.
    private const string StorageKey = "ProactiveBot.JobState";

    // Initializes a new instance of the JobState class.
    public JobState(IStorage storage)
        : base(storage, StorageKey)
    {
    }

    // Gets the storage key for caching state information.
    protected override string GetStorageKey(ITurnContext turnContext) => StorageKey;
}
```

### <a name="register-the-bot-and-required-services"></a>注册机器人和所需服务

**Startup.cs** 文件注册机器人和关联的服务。

`ConfigureServices` 方法注册机器人和终结点服务，包括错误处理和状态管理。 它还会注册作业状态访问器。

**Startup.cs**
```csharp
public void ConfigureServices(IServiceCollection services)
{
    // The Memory Storage used here is for local bot debugging only. When the bot
    // is restarted, everything stored in memory will be gone.
    IStorage dataStore = new MemoryStorage();
    // ...

    // Create Job State object.
    // The Job State object is where we persist anything at the job-scope.
    // Note: It's independent of any user or conversation.
    var jobState = new JobState(dataStore);

    // Make it available to our bot
    services.AddSingleton(sp => jobState);

    // ...      
    }
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

机器人需要使用状态存储系统来保存消息之间的对话和用户状态，在本例中，该状态是使用内存中存储提供程序定义的。

**index.js**
```javascript
const memoryStorage = new MemoryStorage();
const botState = new BotState(memoryStorage, () => 'proactiveBot.botState');

// Create the main dialog, which serves as the bot's main handler.
const bot = new ProactiveBot(botState, adapter);

// Listen for incoming requests.
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (turnContext) => {
        // Route the message to the bot's main handler.
        await bot.onTurn(turnContext);
    });
});

// ...
```

---

## <a name="define-the-bot"></a>定义机器人

用户可以要求机器人为其创建并运行作业。 当作业完成时，可以通过单独的作业服务通知机器人。 机器人旨在执行以下操作：

- 创建用于响应用户的 `run` 或 `run job` 消息的作业。
- 显示用于响应用户的 `show` 或 `show jobs` 消息的所有已注册作业。
- 完成响应“作业已完成”事件的作业，该事件用于标识已完成的作业。
- 模拟用于响应 `done <jobIdentifier>` 消息的“作业已完成”事件。
- 当作业完成后，通过原始聊天向用户发送主动消息。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

机器人有多方面的内容：

- 初始化代码
- 轮次处理程序
- 创建和完成作业的方法

### <a name="declare-the-class"></a>声明类

用户的每次交互都会创建 `ProactiveBot` 类的实例。 每次创建用户所需服务的过程称为瞬时性生存期服务。 应该谨慎管理构造开销较高或者生存期超出单个轮次的对象。

**ProactiveBot.cs**
```csharp
namespace Microsoft.BotBuilderSamples
{
    public class ProactiveBot : IBot
    {
        // The name of events that signal that a job has completed.
        public const string JobCompleteEventName = "jobComplete";

        public const string WelcomeText = "Type 'run' or 'run job' to start a new job.\r\n" +
                                          "Type 'show' or 'show jobs' to display the job log.\r\n" +
                                          "Type 'done <jobNumber>' to complete a job.";
    }
}
```

### <a name="add-initialization-code"></a>添加初始化代码

**ProactiveBot.cs**
```csharp
private readonly JobState _jobState;
private readonly IStatePropertyAccessor<JobLog> _jobLogPropertyAccessor;

public ProactiveBot(JobState jobState, EndpointService endpointService)
{
    _jobState = jobState ?? throw new ArgumentNullException(nameof(jobState));
    _jobLogPropertyAccessor = _jobState.CreateProperty<JobLog>(nameof(JobLog));

    //...
}

```

### <a name="add-a-turn-handler"></a>添加轮次处理程序

适配器将活动转发到轮次处理程序，后者检查 `Activity` 类型并调用相应的方法。 每个机器人必须实现一个轮次处理程序。

**ProactiveBot.cs**
```csharp
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
    if (turnContext.Activity.Type != ActivityTypes.Message)
    {
        // Handle non-message activities.
        await OnSystemActivityAsync(turnContext);
    }
    else
    {
        // Get the job log.
        // The job log is a dictionary of all outstanding jobs in the system.
        JobLog jobLog = await _jobLogPropertyAccessor.GetAsync(turnContext, () => new JobLog());

        // Get the user's text input for the message.
        var text = turnContext.Activity.Text.Trim().ToLowerInvariant();
        switch (text)
        {
            case "run":
            case "run job":

                // Start a virtual job for the user.
                JobLog.JobData job = CreateJob(turnContext, jobLog);

                // Set the new property
                await _jobLogPropertyAccessor.SetAsync(turnContext, jobLog);

                // Now save it into the JobState
                await _jobState.SaveChangesAsync(turnContext);

                await turnContext.SendActivityAsync(
                    $"We're starting job {job.TimeStamp} for you. We'll notify you when it's complete.");

                break;

            case "show":
            case "show jobs":
                // Display information for all jobs in the log.
                // ...
                break;

            default:
                // Check whether this is simulating a job completed event.
                string[] parts = text?.Split(' ', StringSplitOptions.RemoveEmptyEntries);
                if (parts != null && parts.Length == 2
                    && parts[0].Equals("done", StringComparison.InvariantCultureIgnoreCase)
                    && long.TryParse(parts[1], out long jobNumber))
                {
                    if (!jobLog.TryGetValue(jobNumber, out JobLog.JobData jobInfo))
                    {
                        await turnContext.SendActivityAsync($"The log does not contain a job {jobInfo.TimeStamp}.");
                    }
                    else if (jobInfo.Completed)
                    {
                        await turnContext.SendActivityAsync($"Job {jobInfo.TimeStamp} is already complete.");
                    }
                    else
                    {
                        await turnContext.SendActivityAsync($"Completing job {jobInfo.TimeStamp}.");

                        // Send the proactive message.
                        await CompleteJobAsync(turnContext.Adapter, AppId, jobInfo);
                    }
                }

                break;
        }

        if (!turnContext.Responded)
        {
            await turnContext.SendActivityAsync(WelcomeText);
        }
    }
}

private static async Task SendWelcomeMessageAsync(ITurnContext turnContext)
{
    foreach (var member in turnContext.Activity.MembersAdded)
    {
        if (member.Id != turnContext.Activity.Recipient.Id)
        {
            await turnContext.SendActivityAsync($"Welcome to SuggestedActionsBot {member.Name}.\r\n{WelcomeText}");
        }
    }
}
```

### <a name="handle-non-message-activities"></a>处理非消息活动

在“作业已完成”事件中，将作业标记为已完成并通知用户。

**ProactiveBot.cs**
```csharp
private async Task OnSystemActivityAsync(ITurnContext turnContext)
{
    if (turnContext.Activity.Type is ActivityTypes.Event)
    {
        var jobLog = await _jobLogPropertyAccessor.GetAsync(turnContext, () => new JobLog());
        var activity = turnContext.Activity.AsEventActivity();
        if (activity.Name == JobCompleteEventName
            && activity.Value is long timestamp
            && jobLog.ContainsKey(timestamp)
            && !jobLog[timestamp].Completed)
        {
            await CompleteJobAsync(turnContext.Adapter, AppId, jobLog[timestamp]);
        }
    }
    else if (turnContext.Activity.Type is ActivityTypes.ConversationUpdate)
    {
        if (turnContext.Activity.MembersAdded.Any())
        {
            await SendWelcomeMessageAsync(turnContext);
        }
    }
}
```

### <a name="add-job-creation-and-completion-methods"></a>添加作业创建和完成方法

若要启动某个作业，机器人需创建作业并在作业日志中记录其相关信息和当前聊天。 机器人在任何聊天中收到“作业已完成”事件时，会先验证作业 ID，然后再调用完成作业所需的代码。

用于完成作业的代码从状态获取作业日志，然后将作业标记为完成，并使用适配器的 `ContinueConversationAsync` 方法发送一条主动消息。

- “继续聊天”调用会提示通道启动一个独立于用户的轮次。
- 适配器运行关联的回调来代替机器人正常的轮次处理程序。 此轮次具有自身的轮次上下文，可从中检索状态信息并向用户发送主动消息。

**ProactiveBot.cs**
```csharp
// Creates and "starts" a new job.
private JobLog.JobData CreateJob(ITurnContext turnContext, JobLog jobLog)
{
    JobLog.JobData jobInfo = new JobLog.JobData
    {
        TimeStamp = DateTime.Now.ToBinary(),
        Conversation = turnContext.Activity.GetConversationReference(),
    };

    jobLog[jobInfo.TimeStamp] = jobInfo;

    return jobInfo;
}
```

### <a name="sends-a-proactive-message-to-the-user"></a>将主动消息发送到用户

**ProactiveBot.cs**
```csharp
private async Task CompleteJobAsync(
    BotAdapter adapter,
    string botId,
    JobLog.JobData jobInfo,
    CancellationToken cancellationToken = default(CancellationToken))
{
    await adapter.ContinueConversationAsync(botId, jobInfo.Conversation, CreateCallback(jobInfo), cancellationToken);
}
```

### <a name="creates-the-turn-logic-to-use-for-the-proactive-message"></a>创建用于处理主动消息的轮次逻辑

**ProactiveBot.cs**
```csharp
private BotCallbackHandler CreateCallback(JobLog.JobData jobInfo)
{
    return async (turnContext, token) =>
    {
        // Get the job log from state, and retrieve the job.
        JobLog jobLog = await _jobLogPropertyAccessor.GetAsync(turnContext, () => new JobLog());

        // Perform bookkeeping.
        jobLog[jobInfo.TimeStamp].Completed = true;

        // Set the new property
        await _jobLogPropertyAccessor.SetAsync(turnContext, jobLog);

        // Now save it into the JobState
        await _jobState.SaveChangesAsync(turnContext);

        // Send the user a proactive confirmation message.
        await turnContext.SendActivityAsync($"Job {jobInfo.TimeStamp} is complete.");
    };
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

机器人在 **bot.js** 中定义，有多方面的内容：

- 初始化代码
- 轮次处理程序
- 创建和完成作业的方法

### <a name="declare-the-class-and-add-initialization-code"></a>声明类并添加初始化代码

```javascript
const { ActivityTypes, TurnContext } = require('botbuilder');

const JOBS_LIST = 'jobs';

class ProactiveBot {
    constructor(botState, adapter) {
        this.botState = botState;
        this.adapter = adapter;

        this.jobsList = this.botState.createProperty(JOBS_LIST);
    }

    // ...
};

// Helper function to check if object is empty.
function isEmpty(obj) {
    for (var key in obj) {
        if (obj.hasOwnProperty(key)) {
            return false;
        }
    }
    return true;
};

module.exports.ProactiveBot = ProactiveBot;
```

### <a name="the-turn-handler"></a>轮次处理程序

`onTurn` 和 `showJobs` 方法在 `ProactiveBot` 类中定义。 `onTurn` 处理用户的输入。 它还会从假设的作业履行系统接收事件活动。 `showJobs` 负责作业日志的格式化和发送。

**bot.js**
```javascript
/**
    *
    * @param {TurnContext} turnContext A TurnContext object representing an incoming message to be handled by the bot.
    */
async onTurn(turnContext) {
    // See https://aka.ms/about-bot-activity-message to learn more about the message and other activity types.
    if (turnContext.activity.type === ActivityTypes.Message) {
        const utterance = (turnContext.activity.text || '').trim().toLowerCase();
        var jobIdNumber;

        // If user types in run, create a new job.
        if (utterance === 'run') {
            await this.createJob(turnContext);
        } else if (utterance === 'show') {
            await this.showJobs(turnContext);
        } else {
            const words = utterance.split(' ');

            // If the user types done and a Job Id Number,
            // we check if the second word input is a number.
            if (words[0] === 'done' && !isNaN(parseInt(words[1]))) {
                jobIdNumber = words[1];
                await this.completeJob(turnContext, jobIdNumber);
            } else if (words[0] === 'done' && (words.length < 2 || isNaN(parseInt(words[1])))) {
                await turnContext.sendActivity('Enter the job ID number after "done".');
            }
        }

        if (!turnContext.responded) {
            await turnContext.sendActivity(`Say "run" to start a job, or "done <job>" to complete one.`);
        }
    } else if (turnContext.activity.type === ActivityTypes.Event && turnContext.activity.name === 'jobCompleted') {
        jobIdNumber = turnContext.activity.value;
        if (!isNaN(parseInt(jobIdNumber))) {
            await this.completeJob(turnContext, jobIdNumber);
        }
    }

    await this.botState.saveChanges(turnContext);
}

// Show a list of the pending jobs
async showJobs(turnContext) {
    const jobs = await this.jobsList.get(turnContext, {});
    if (Object.keys(jobs).length) {
        await turnContext.sendActivity(
            '| Job number &nbsp; | Conversation ID &nbsp; | Completed |<br>' +
            '| :--- | :---: | :---: |<br>' +
            Object.keys(jobs).map((key) => {
                return `${ key } &nbsp; | ${ jobs[key].reference.conversation.id.split('|')[0] } &nbsp; | ${ jobs[key].completed }`;
            }).join('<br>'));
    } else {
        await turnContext.sendActivity('The job log is empty.');
    }
}
```

### <a name="logic-to-start-a-job"></a>用于启动作业的逻辑

`createJob` 方法在 `ProactiveBot` 类中定义。 它创建并记录用户的新作业。 理论上，它还会将此信息转发给作业履行系统。

**bot.js**
```javascript
// Save job ID and conversation reference.
async createJob(turnContext) {
    // Create a unique job ID.
    var date = new Date();
    var jobIdNumber = date.getTime();

    // Get the conversation reference.
    const reference = TurnContext.getConversationReference(turnContext.activity);

    // Get the list of jobs. Default it to {} if it is empty.
    const jobs = await this.jobsList.get(turnContext, {});

    // Try to find previous information about the saved job.
    const jobInfo = jobs[jobIdNumber];

    try {
        if (isEmpty(jobInfo)) {
            // Job object is empty so we have to create it
            await turnContext.sendActivity(`Need to create new job ID: ${ jobIdNumber }`);

            // Update jobInfo with new info
            jobs[jobIdNumber] = { completed: false, reference: reference };

            try {
                // Save to storage
                await this.jobsList.set(turnContext, jobs);
                // Notify the user that the job has been processed
                await turnContext.sendActivity('Successful write to log.');
            } catch (err) {
                await turnContext.sendActivity(`Write failed: ${ err.message }`);
            }
        }
    } catch (err) {
        await turnContext.sendActivity(`Read rejected. ${ err.message }`);
    }
}
```

### <a name="logic-to-complete-a-job"></a>用于完成作业的逻辑

`completeJob` 方法在 `ProactiveBot` 类中定义。 它执行某些簿记操作并在用户的原始聊天中向用户发送主动消息，告知作业已完成。

**bot.js**
```javascript
async completeJob(turnContext, jobIdNumber) {
    // Get the list of jobs from the bot's state property accessor.
    const jobs = await this.jobsList.get(turnContext, {});

    // Find the appropriate job in the list of jobs.
    let jobInfo = jobs[jobIdNumber];

    // If no job was found, notify the user of this error state.
    if (isEmpty(jobInfo)) {
        await turnContext.sendActivity(`Sorry no job with ID ${ jobIdNumber }.`);
    } else {
        // Found a job with the ID passed in.
        const reference = jobInfo.reference;
        const completed = jobInfo.completed;

        // If the job is not yet completed and conversation reference exists,
        // use the adapter to continue the conversation with the job's original creator.
        if (reference && !completed) {
            // Since we are going to proactively send a message to the user who started the job,
            // we need to create the turnContext based on the stored reference value.
            await this.adapter.continueConversation(reference, async (proactiveTurnContext) => {
                // Complete the job.
                jobInfo.completed = true;
                // Save the updated job.
                await this.jobsList.set(turnContext, jobs);
                // Notify the user that the job is complete.
                await proactiveTurnContext.sendActivity(`Your queued job ${ jobIdNumber } just completed.`);
            });

            // Send a message to the person who completed the job.
            await turnContext.sendActivity('Job completed. Notification sent.');
        } else if (completed) { // The job has already been completed.
            await turnContext.sendActivity('This job is already completed, please start a new job.');
        };
    };
};
```

---

## <a name="test-your-bot"></a>测试机器人

在本地生成和运行机器人，并打开两个模拟器窗口。 如需分步说明，请参阅[自述文件](https://github.com/Microsoft/BotBuilder-Samples/blob/master/samples/csharp_dotnetcore/16.proactive-messages/README.md)。

1. 注意，两个窗口中的聊天 ID 不同。
1. 在第一个窗口中，键入 `run` 数次即可启动多个作业。
1. 在第二个窗口中，键入 `show` 可查看日志中作业的列表。
1. 在第二个窗口中键入 `done <jobNumber>`，其中 `<jobNumber>` 是日志中的一个作业编号，没有尖括号。 （根据设计，机器人代码会按照它是 jobComplete 事件的情况对此进行解释。）
1. 请注意，在第一个窗口中，机器人向用户发送一条主动消息。

从用户的角度来看，聊天看起来可能如下所示：

![用户的模拟器会话](~/v4sdk/media/how-to-proactive/user.png)

从模拟作业系统的角度来看，聊天看起来如下所示：

![作业系统的模拟器会话](~/v4sdk/media/how-to-proactive/job-system.png)

## <a name="additional-resources"></a>其他资源
查看 [GitHub](https://github.com/Microsoft/BotBuilder-Samples/blob/master/README.md) 中的其他 C# 和 JS 示例。
