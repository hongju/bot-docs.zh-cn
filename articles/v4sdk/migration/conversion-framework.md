---
title: 在同一 .NET Framework 项目中迁移现有的机器人 | Microsoft Docs
description: 使用同一个项目将现有的 v3 机器人迁移到 v4 SDK。
keywords: 机器人, formflow, 对话, v3 机器人
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 02/11/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 1904bb09d8bd387cc5cec0d85f82df24d1f6ec9d
ms.sourcegitcommit: 7f418bed4d0d8d398f824e951ac464c7c82b8c3e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/14/2019
ms.locfileid: "56240173"
---
# <a name="migrate-a-net-sdk-v3-bot-to-v4"></a>将.NET SDK v3 机器人迁移到 v4

在本文中，我们会将 v3 [ContosoHelpdeskChatBot](https://github.com/Microsoft/intelligent-apps/tree/master/ContosoHelpdeskChatBot/ContosoHelpdeskChatBot) 转换到 v4 机器人，_但不转换项目类型_。 它将仍然是 .NET Framework 项目。
这种转换划分为以下步骤：

1. 更新和安装 NuGet 包
1. 更新 Global.asax.cs 文件
1. 更新 MessagesController 类
1. 转换对话

<!--TODO: Link to the converted bot...[ContosoHelpdeskChatBot](https://github.com/EricDahlvang/intelligent-apps/tree/v4netframework/ContosoHelpdeskChatBot).-->

Bot Framework SDK v4 与 SDK v3 基于相同的基础 REST API。 但是，SDK v4 对以前的 SDK 版本作了重构，使开发人员对其机器人拥有更高的灵活性和控制度。 该 SDK 的主要更改包括：

- 通过状态管理对象和属性访问器管理状态。
- 设置轮次处理程序以及向其传递活动的方式发生了变化。
- 可评分对象不再存在。 在将控制权传递给对话之前，可以检查轮次处理程序中的“全局”命令。
- 采用新的对话库，该库与以前的版本有很大的不同。 需要使用组件和瀑布对话以及适用于 v4 的 Formflow 对话的社区实现，将旧对话转换为新的对话系统

有关特定更改的详细信息，请参阅 [v3 和 v4 .NET SDK 之间的差异](migration-about.md)。

## <a name="update-and-install-nuget-packages"></a>更新和安装 NuGet 包

1. 将 **Microsoft.Bot.Builder.Azure** 更新为最新稳定版本。

    这也会更新 **Microsoft.Bot.Builder** 和 **Microsoft.Bot.Connector** 包，因为它们是依赖项。

1. 删除 **Microsoft.Bot.Builder.History** 包。 此包不是 v4 SDK 的一部分。
1. 添加 **Autofac.WebApi2**

    我们将借助此包在 ASP.NET 中注入依赖项。

1. 添加 **Bot.Builder.Community.Dialogs.Formflow**

    这是一个社区库，用于从 v3 Formflow 定义文件生成 v4 对话。 它有一个依赖项 **Microsoft.Bot.Builder.Dialogs**，因此系统也会安装此依赖项。

如果现在就生成项目，则会收到编译器错误。 可以忽略这些错误。 完成转换后，将获得可正常运行的代码。

<!--
## Add a BotDataBag class

This file will contain wrappers to add a v3-style **IBotDataBag** to make dialog conversion simpler.

```csharp
using System.Collections.Generic;

namespace ContosoHelpdeskChatBot
{
    public class BotDataBag : Dictionary<string, object>, IBotDataBag
    {
        public bool RemoveValue(string key)
        {
            return base.Remove(key);
        }

        public void SetValue<T>(string key, T value)
        {
            this[key] = value;
        }

        public bool TryGetValue<T>(string key, out T value)
        {
            if (!ContainsKey(key))
            {
                value = default(T);
                return false;
            }

            value = (T)this[key];

            return true;
        }
    }

    public interface IBotDataBag
    {
        int Count { get; }

        void Clear();

        bool ContainsKey(string key);

        bool RemoveValue(string key);

        void SetValue<T>(string key, T value);

        bool TryGetValue<T>(string key, out T value);
    }
}
```
-->

## <a name="update-your-globalasaxcs-file"></a>更新 Global.asax.cs 文件

部分基架设计已发生更改，我们需要在 v4 中自行设置[状态管理](/articles/v4sdk/bot-builder-concept-state.md)基础结构的部件。 例如，v4 使用机器人适配器来处理身份验证以及将活动转发到机器人代码，我们必须提前声明状态属性。

为 `DialogState` 创建状态属性，目前，在 v4 中支持对话需要提供此属性。 在控制器和机器人代码中使用依赖项注入获取所需的信息。

在 **Global.asax.cs** 中：

1. 更新 `using` 语句：
    ```csharp
    using System.Configuration;
    using System.Reflection;
    using System.Web.Http;
    using Autofac;
    using Autofac.Integration.WebApi;
    using Microsoft.Bot.Builder;
    using Microsoft.Bot.Builder.Dialogs;
    using Microsoft.Bot.Connector.Authentication;
    ```
1. 从 **Application_Start** 方法中删除这些行
    ```csharp
    BotConfig.UpdateConversationContainer();
    this.RegisterBotModules();
    ```
    插入此行。
    ```csharp
    GlobalConfiguration.Configure(BotConfig.Register);
    ```
1. 删除不再引用的 **RegisterBotModules** 方法。
1. 将 **BotConfig.UpdateConversationContainer** 方法替换为此 **BotConfig.Register** 方法，我们将在此方法中注册所需的对象用于支持依赖项注入。
    > [!NOTE]
    > 此机器人不使用用户状态或私人聊天状态。 此处用于包含这些状态的代码行已注释掉。
    ```csharp
    public static void Register(HttpConfiguration config)
    {
        ContainerBuilder builder = new ContainerBuilder();
        builder.RegisterApiControllers(Assembly.GetExecutingAssembly());

        SimpleCredentialProvider credentialProvider = new SimpleCredentialProvider(
            ConfigurationManager.AppSettings[MicrosoftAppCredentials.MicrosoftAppIdKey],
            ConfigurationManager.AppSettings[MicrosoftAppCredentials.MicrosoftAppPasswordKey]);

        builder.RegisterInstance(credentialProvider).As<ICredentialProvider>();

        // The Memory Storage used here is for local bot debugging only. When the bot
        // is restarted, everything stored in memory will be gone.
        IStorage dataStore = new MemoryStorage();

        // Create Conversation State object.
        // The Conversation State object is where we persist anything at the conversation-scope.
        ConversationState conversationState = new ConversationState(dataStore);
        builder.RegisterInstance(conversationState).As<ConversationState>();

        //var userState = new UserState(dataStore);
        //var privateConversationState = new PrivateConversationState(dataStore);

        // Create the dialog state property acccessor.
        IStatePropertyAccessor<DialogState> dialogStateAccessor
            = conversationState.CreateProperty<DialogState>(nameof(DialogState));
        builder.RegisterInstance(dialogStateAccessor).As<IStatePropertyAccessor<DialogState>>();

        IContainer container = builder.Build();
        AutofacWebApiDependencyResolver resolver = new AutofacWebApiDependencyResolver(container);
        config.DependencyResolver = resolver;
    }
    ```

## <a name="update-your-messagescontroller-class"></a>更新 MessagesController 类

v4 中轮次处理程序在此类中执行，因此需要对此类进行一些更改。 除轮次处理程序本身外，大部分组件都可被视为样本。 在 **Controllers\MessagesController.cs** 文件中：

1. 更新 `using` 语句：
    ```csharp
    using System;
    using System.Net;
    using System.Net.Http;
    using System.Threading;
    using System.Threading.Tasks;
    using System.Web.Http;
    using Bot.Builder.Community.Dialogs.FormFlow;
    using ContosoHelpdeskChatBot.Dialogs;
    using Microsoft.Bot.Builder;
    using Microsoft.Bot.Builder.Dialogs;
    using Microsoft.Bot.Connector.Authentication;
    using Microsoft.Bot.Schema;
    ```
1. 从类中删除 `[BotAuthentication]` 属性。 在 v4 中，机器人的适配器将处理身份验证。
1. 添加这些字段。 **ConversationState** 将管理限定为聊天的状态，需要使用 **IStatePropertyAccessor\<DialogState>** 来支持 v4 中的对话。
    ```csharp
    private readonly ConversationState _conversationState;
    private readonly ICredentialProvider _credentialProvider;
    private readonly IStatePropertyAccessor<DialogState> _dialogData;

    private readonly DialogSet _dialogs;
    ```
1. 添加一个构造函数，用于：
    - 初始化实例字段。
    - 使用 ASP.NET 中的依赖项注入来获取参数值。 （我们已在 **Global.asax.cs** 中注册这些类的实例，以支持此功能。）
    - 创建并初始化对话集，我们可以从中创建对话上下文。 （需要在 v4 中显式执行此操作。）
    ```csharp
    public MessagesController(
        ConversationState conversationState,
        ICredentialProvider credentialProvider,
        IStatePropertyAccessor<DialogState> dialogData)
    {
        _conversationState = conversationState;
        _dialogData = dialogData;
        _credentialProvider = credentialProvider;

        _dialogs = new DialogSet(dialogData);
        _dialogs.Add(new RootDialog(nameof(RootDialog)));
    }
    ```
1. 替换 **Post** 方法的主体。 我们将在此处创建适配器，并使用它来调用消息循环（轮次处理程序）。 我们将在每个轮次的末尾使用 `SaveChangesAsync` 来保存机器人对状态所做的任何更改。

    ```csharp
    public async Task<HttpResponseMessage> Post([FromBody]Activity activity)
    {

        var botFrameworkAdapter = new BotFrameworkAdapter(_credentialProvider);

        var invokeResponse = await botFrameworkAdapter.ProcessActivityAsync(
            Request.Headers.Authorization?.ToString(),
            activity,
            OnTurnAsync,
            default(CancellationToken));

        if (invokeResponse == null)
        {
            return Request.CreateResponse(HttpStatusCode.OK);
        }
        else
        {
            return Request.CreateResponse(invokeResponse.Status);
        }
    }
    ```
1. 添加 **OnTurnAsync** 方法，其中包含机器人的[轮次处理程序](/articles/v4sdk/bot-builder-basics.md#the-activity-processing-stack)代码。
    > [!NOTE]
    > v4 中不存在此类可评分对象。 在继续运行任何活动对话之前，先在机器人的轮次处理程序中检查用户提供的 `cancel` 消息。
    ```csharp
    protected async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken)
    {
        // We're only handling message activities in this bot.
        if (turnContext.Activity.Type == ActivityTypes.Message)
        {
            // Create the dialog context for our dialog set.
            DialogContext dc = await _dialogs.CreateContextAsync(turnContext, cancellationToken);

            // Globally interrupt the dialog stack if the user sent 'cancel'.
            if (turnContext.Activity.Text.Equals("cancel", StringComparison.InvariantCultureIgnoreCase))
            {
                Activity reply = turnContext.Activity.CreateReply($"Ok restarting conversation.");
                await turnContext.SendActivityAsync(reply);
                await dc.CancelAllDialogsAsync();
            }

            try
            {
                // Continue the active dialog, if any. If we just cancelled all dialog, the
                // dialog stack will be empty, and this will return DialogTurnResult.Empty.
                DialogTurnResult dialogResult = await dc.ContinueDialogAsync();
                switch (dialogResult.Status)
                {
                    case DialogTurnStatus.Empty:
                        // There was no active dialog in the dialog stack; start the root dialog.
                        await dc.BeginDialogAsync(nameof(RootDialog));
                        break;

                    case DialogTurnStatus.Complete:
                        // The last dialog on the stack completed and the stack is empty.
                        await dc.EndDialogAsync();
                        break;

                    case DialogTurnStatus.Waiting:
                    case DialogTurnStatus.Cancelled:
                        // The active dialog is waiting for a response from the user, or all
                        // dialogs were cancelled and the stack is empty. In either case, we
                        // don't need to do anything here.
                        break;
                }
            }
            catch (FormCanceledException)
            {
                // One of the dialogs threw an exception to clear the dialog stack.
                await turnContext.SendActivityAsync("Cancelled.");
                await dc.CancelAllDialogsAsync();
                await dc.BeginDialogAsync(nameof(RootDialog));
            }
        }
    }
    ```
1. 由于我们只需处理消息活动，因此可以删除 **HandleSystemMessage** 方法。

### <a name="delete-the-cancelscorable-and-globalmessagehandlersbotmodule-classes"></a>删除 CancelScorable 和 GlobalMessageHandlersBotModule 类

由于 v4 中不存在可评分对象，并且我们已更新轮次处理程序，使之对 `cancel` 消息做出反应，因此，可以删除 **CancelScorable**（在 **Dialogs\CancelScorable.cs** 中）和 **GlobalMessageHandlersBotModule** 类。

## <a name="convert-your-dialogs"></a>转换对话

我们将对原始对话做出大量更改，以将其迁移到 v4 SDK。 暂时不用担心编译器错误。 完成转换后，这些错误可自行解决。
为了避免多余地修改原始代码，完成迁移后，仍会出现一些编译器警告。

所有对话将派生自 `ComponentDialog`，而不是实现 v3 的 `IDialog<object>` 接口。

此机器人包含四个需要转换的对话：

| | |
|---|---|
| [RootDialog](#update-the-root-dialog) | 提供选项并启动其他对话。 |
| [InstallAppDialog](#update-the-install-app-dialog) | 处理在计算机上安装应用的请求。 |
| [LocalAdminDialog](#update-the-local-admin-dialog) | 处理本地计算机管理员权限的请求。 |
| [ResetPasswordDialog](#update-the-reset-password-dialog) | 处理重置密码的请求。 |

这些对话会收集输入，但不会在计算机上执行上述任何操作。

### <a name="make-solution-wide-dialog-changes"></a>进行解决方案范围的对话更改

1. 对于整个解决方案，请将出现的所有 `IDialog<object>` 替换为 `ComponentDialog`。
1. 对于整个解决方案，请将出现的所有 `IDialogContext` 替换为 `DialogContext`。
1. 对于每个对话类，请删除 `[Serializable]` 属性。

对话中的控制流和消息传送不再以相同的方式进行处理，因此，在转换每个对话时，需要对处理方式进行修改。

| Operation | v3 代码 | v4 代码 |
| :--- | :--- | :--- |
| 处理对话的启动 | 实现 `IDialog.StartAsync` | 将此项设为瀑布对话的第一个步骤，或实现 `Dialog.BeginDialogAsync` |
| 处理对话的持续 | 调用 `IDialogContext.Wait` | 将附加的步骤添加到瀑布对话，或实现 `Dialog.ContinueDialogAsync` |
| 向用户发送消息 | 调用 `IDialogContext.PostAsync` | 调用 `ITurnContext.SendActivityAsync` |
| 启动子对话 | 调用 `IDialogContext.Call` | 调用 `DialogContext.BeginDialogAsync` |
| 表示当前对话已完成的信号 | 调用 `IDialogContext.Done` | 调用 `DialogContext.EndDialogAsync` |
| 获取用户的输入 | 使用 `IAwaitable<IMessageActivity>` 参数 | 从瀑布内部使用提示，或使用 `ITurnContext.Activity` |

有关 v4 代码的说明：

- 使用 `DialogContext.Context` 属性获取当前轮次上下文。
- 瀑布步骤包含派生自 `DialogContext` 的 `WaterfallStepContext` 参数。
- 所有具体对话和提示类派生自 `Dialog` 抽象类。
- 创建组件对话时分配 ID。 需要为对话集中的每个对话分配该集中唯一的 ID。

### <a name="update-the-root-dialog"></a>更新根对话

在此机器人中，根对话将提示用户从一组选项中做出选择，然后根据所做的选择启动子对话。 然后，在聊天的整个生存期内，此过程都会循环。

- 可将主要流设置为瀑布对话，这是 v4 SDK 中的一个新概念。 此流将按照一组固定的步骤依序运行。 有关详细信息，请参阅[实现顺序聊天流](/articles/v4sdk/bot-builder-dialog-manage-conversation-flow)。
- 现在会通过提示类处理提示。提示类是一些简短的子对话，它们提示提供输入、执行某种极简的处理和验证，然后返回值。 有关详细信息，请参阅[使用对话提示收集用户输入](/articles/v4sdk/bot-builder-prompts.md)。

在 **Dialogs/RootDialog.cs** 文件中：

1. 更新 `using` 语句：
    ```csharp
    using System;
    using System.Collections.Generic;
    using System.Threading;
    using System.Threading.Tasks;
    using Microsoft.Bot.Builder;
    using Microsoft.Bot.Builder.Dialogs;
    using Microsoft.Bot.Builder.Dialogs.Choices;
    ```
1. 需要将 `HelpdeskOptions` 选项从字符串列表转换为选项列表。 此项将与选项提示结合使用。选项提示接受选项编号（列表中）、选项值，或选项的任何同义词（作为有效输入）。
    ```csharp
    private static List<Choice> HelpdeskOptions = new List<Choice>()
    {
        new Choice(InstallAppOption) { Synonyms = new List<string>(){ "install" } },
        new Choice(ResetPasswordOption) { Synonyms = new List<string>(){ "password" } },
        new Choice(LocalAdminOption)  { Synonyms = new List<string>(){ "admin" } }
    };
    ```
1. 添加一个构造函数。 此代码将执行以下操作：
   - 创建对话时，将为该对话的每个实例分配一个 ID。 对话 ID 是对话所要添加到的对话集的一部分。 如前所述，机器人具有一个在 **MessageController** 类中初始化的对话集。 每个 `ComponentDialog` 具有自身的内部对话集，该集具有自身的对话 ID 集。
   - 它会添加其他对话，包括选项提示（子对话）。 此处，我们只对每个对话 ID 使用类名。
   - 此类定义包括三个步骤的瀑布对话。 稍后我们将实现这些步骤。
     - 该对话首先提示用户选择要执行的任务。
     - 然后，启动与该选项关联的子对话。
     - 最后重启自身。
   - 每个瀑布步骤是一个委托，接下来，在可能的情况下，我们将采用原始对话中的现有代码实现这些步骤。
   - 启动组件对话框时，它会启动其 _initial dialog_。 默认情况下，这是添加到组件对话的第一个子对话。 若要分配与初始对话不同的子级，需手动设置组件的 `InitialDialogId` 属性。
    ```csharp
    public RootDialog(string id)
        : base(id)
    {
        AddDialog(new WaterfallDialog("choiceswaterfall", new WaterfallStep[]
            {
                PromptForOptionsAsync,
                ShowChildDialogAsync,
                ResumeAfterAsync,
            }));
        AddDialog(new InstallAppDialog(nameof(InstallAppDialog)));
        AddDialog(new LocalAdminDialog(nameof(LocalAdminDialog)));
        AddDialog(new ResetPasswordDialog(nameof(ResetPasswordDialog)));
        AddDialog(new ChoicePrompt("options"));
    }
    ```
1. 可以删除 **StartAsync** 方法。 组件对话开始时，会自动启动其初始对话。 在这种情况下，它是我们在构造函数中定义的瀑布对话。 该对话还会自动启动其第一个步骤。
1. 我们将删除 **MessageReceivedAsync** 和 **ShowOptions** 方法，并将其替换为第一个瀑布步骤。 这两个方法问候用户，并要求他们选择可用的选项之一。
   - 在此处可以看到调用选项提示时提供的选项列表以及问候语和错误消息。
   - 无需指定对话中要调用的下一个方法，因为选项提示完成后，瀑布会转到下一步骤。
   - 选项提示将不断循环，直到收到有效输入或整个对话堆栈被取消。
    ```csharp
    private async Task<DialogTurnResult> PromptForOptionsAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // Prompt the user for a response using our choice prompt.
        return await stepContext.PromptAsync(
            "options",
            new PromptOptions()
            {
                Choices = HelpdeskOptions,
                Prompt = MessageFactory.Text(GreetMessage),
                RetryPrompt = MessageFactory.Text(ErrorMessage)
            },
            cancellationToken);
    }
    ```
1. 可将 **OnOptionSelected** 替换为第二个瀑布步骤。 仍然根据用户的输入启动一个子对话。
   - 选项提示返回 `FoundChoice` 值。 此值显示在步骤上下文的 `Result` 属性中。 对话堆栈将所有返回值视为对象。 如果返回值来自某个对话，则就能知道对象值的类型是什么。 有关返回的每种提示类型的列表，请参阅[提示类型](/articles/v4sdk/bot-builder-concept-dialog.md#prompt-types)。
   - 由于选项提示不引发异常，因此可以删除 try-catch 块。
   - 我们需要添加一个失败代码，以便此方法始终返回适当的值。 此代码应该永远不会命中，但如果命中，它可以让对话“正常失败”。
    ```csharp
    private async Task<DialogTurnResult> ShowChildDialogAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // string optionSelected = await userReply;
        string optionSelected = (stepContext.Result as FoundChoice).Value;

        switch (optionSelected)
        {
            case InstallAppOption:
                //context.Call(new InstallAppDialog(), this.ResumeAfterOptionDialog);
                //break;
                return await stepContext.BeginDialogAsync(
                    nameof(InstallAppDialog),
                    cancellationToken);
            case ResetPasswordOption:
                //context.Call(new ResetPasswordDialog(), this.ResumeAfterOptionDialog);
                //break;
                return await stepContext.BeginDialogAsync(
                    nameof(ResetPasswordDialog),
                    cancellationToken);
            case LocalAdminOption:
                //context.Call(new LocalAdminDialog(), this.ResumeAfterOptionDialog);
                //break;
                return await stepContext.BeginDialogAsync(
                    nameof(LocalAdminDialog),
                    cancellationToken);
        }

        // We shouldn't get here, but fail gracefully if we do.
        await stepContext.Context.SendActivityAsync(
            "I don't recognize that option.",
            cancellationToken: cancellationToken);
        // Continue through to the next step without starting a child dialog.
        return await stepContext.NextAsync(cancellationToken: cancellationToken);
    }
    ```
1. 最后，将旧的 **ResumeAfterOptionDialog** 方法替换为最后一个瀑布步骤。
    - 通过将堆栈上的原始实例替换为对话自身的新实例来重启瀑布，而不要像在原始对话中那样结束对话并返回票证编号。 之所以可以这样做，是因为原始应用始终忽略返回值（票证编号）并重启根对话。
    ```csharp
    private async Task<DialogTurnResult> ResumeAfterAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        try
        {
            //var message = await userReply;
            var message = stepContext.Context.Activity;

            int ticketNumber = new Random().Next(0, 20000);
            //await context.PostAsync($"Thank you for using the Helpdesk Bot. Your ticket number is {ticketNumber}.");
            await stepContext.Context.SendActivityAsync(
                $"Thank you for using the Helpdesk Bot. Your ticket number is {ticketNumber}.",
                cancellationToken: cancellationToken);

            //context.Done(ticketNumber);
        }
        catch (Exception ex)
        {
            // await context.PostAsync($"Failed with message: {ex.Message}");
            await stepContext.Context.SendActivityAsync(
                $"Failed with message: {ex.Message}",
                cancellationToken: cancellationToken);

            // In general resume from task after calling a child dialog is a good place to handle exceptions
            // try catch will capture exceptions from the bot framework awaitable object which is essentially "userReply"
            logger.Error(ex);
        }

        // Replace on the stack the current instance of the waterfall with a new instance,
        // and start from the top.
        return await stepContext.ReplaceDialogAsync(
            "choiceswaterfall",
            cancellationToken: cancellationToken);
    }
    ```

### <a name="update-the-install-app-dialog"></a>更新安装应用对话

安装应用对话执行几个逻辑任务，我们将这些任务设置为 4 步骤瀑布对话。 将现有代码构造成瀑布步骤的方式涉及到每个对话的逻辑行为。 对于每个步骤，都注标了代码的来源原始方法。

1. 要求用户提供搜索字符串。
1. 在数据库中查询潜在的匹配项。
   - 如果有一个命中项，请将其选中并继续。
   - 如果有多个命中项，此步骤会要求用户选择一个命中项。
   - 如果没有命中项，对话将会退出。
1. 要求用户指定一台用于安装应用的计算机。
1. 将信息写入数据库并发送确认消息。

在 **Dialogs/InstallAppDialog.cs** 文件中：

1. 更新 `using` 语句：
    ```csharp
    using System.Collections.Generic;
    using System.Linq;
    using System.Threading;
    using System.Threading.Tasks;
    using ContosoHelpdeskChatBot.Models;
    using Microsoft.Bot.Builder;
    using Microsoft.Bot.Builder.Dialogs;
    using Microsoft.Bot.Builder.Dialogs.Choices;
    ```
1. 定义用于提示和对话的 ID 的常量。 这样就可以更轻松地维护对话代码，因为在一个位置定义了要使用的字符串。
    ```csharp
    // Set up our dialog and prompt IDs as constants.
    private const string MainId = "mainDialog";
    private const string TextId = "textPrompt";
    private const string ChoiceId = "choicePrompt";
    ```
1. 为用于跟踪对话状态的键定义常量。
    ```csharp
    // Set up keys for managing collected information.
    private const string InstallInfo = "installInfo";
    ```
1. 添加一个构造函数，并初始化组件的对话集。 此时我们将显式设置 `InitialDialogId` 属性，这意味着，主要瀑布对话不需要是添加到集中的第一个对话。 例如，如果你偏向于先添加提示，则你可以这样做，而不会导致运行时问题。
    ```csharp
    public InstallAppDialog(string id)
        : base(id)
    {
        // Initialize our dialogs and prompts.
        InitialDialogId = MainId;
        AddDialog(new WaterfallDialog(MainId, new WaterfallStep[] {
            GetSearchTermAsync,
            ResolveAppNameAsync,
            GetMachineNameAsync,
            SubmitRequestAsync,
        }));
        AddDialog(new TextPrompt(TextId));
        AddDialog(new ChoicePrompt(ChoiceId));
    }
    ```
1. 可将 **StartAsync** 替换为第一个瀑布步骤。
    - 我们必须自行管理状态，因此，将要跟踪对话状态中的安装应用对象。
    - 请求用户提供输入的消息成了提示调用中的一个选项。
    ```csharp
    private async Task<DialogTurnResult> GetSearchTermAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // Create an object in dialog state in which to track our collected information.
        stepContext.Values[InstallInfo] = new InstallApp();

        // Ask for the search term.
        return await stepContext.PromptAsync(
            TextId,
            new PromptOptions
            {
                Prompt = MessageFactory.Text("Ok let's get started. What is the name of the application? "),
            },
            cancellationToken);
    }
    ```
1. 可将 **appNameAsync** 和 **multipleAppsAsync** 替换为第二个瀑布步骤。
    - 现在我们将获取提示结果，而不只是查看用户的最后一条消息。
    - 数据库查询和 if 语句的组织方式与在 **appNameAsync** 中相同。 if 语句的每个块中的代码已更新，可以配合 v4 对话运行。
        - 如果有一个命中项，则会更新对话状态并继续下一步。
        - 如果有多个命中项，则使用选项提示来要求用户从选项列表中做出选择。 这意味着可以删除 **multipleAppsAsync**。
        - 如果没有命中项，则结束此对话，并向根对话返回 null。
    ```csharp
    private async Task<DialogTurnResult> ResolveAppNameAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // Get the result from the text prompt.
        var appname = stepContext.Result as string;

        // Query the database for matches.
        var names = await this.getAppsAsync(appname);

        if (names.Count == 1)
        {
            // Get our tracking information from dialog state and add the app name.
            var install = stepContext.Values[InstallInfo] as InstallApp;
            install.AppName = names.First();

            return await stepContext.NextAsync();
        }
        else if (names.Count > 1)
        {
            // Ask the user to choose from the list of matches.
            return await stepContext.PromptAsync(
                ChoiceId,
                new PromptOptions
                {
                    Prompt = MessageFactory.Text("I found the following applications. Please choose one:"),
                    Choices = ChoiceFactory.ToChoices(names),
                },
                cancellationToken);
        }
        else
        {
            // If no matches, exit this dialog.
            await stepContext.Context.SendActivityAsync(
                $"Sorry, I did not find any application with the name '{appname}'.",
                cancellationToken: cancellationToken);

            return await stepContext.EndDialogAsync(null, cancellationToken);
        }
    }
    ```
1. 解决查询后，**appNameAsync** 还要求用户指定其计算机名。 我们将在下一个瀑布步骤中捕获该逻辑部分。
    - 同样，在 v4 中，我们必须自行管理状态。 此处唯一的棘手问题是，如何通过上一步骤中的两个不同逻辑分支转到此步骤。
    - 我们使用与前面一样的文本提示来请求用户指定计算机名（这一次只需提供不同的选项即可）。
    ```csharp
    private async Task<DialogTurnResult> GetMachineNameAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // Get the tracking info. If we don't already have an app name,
        // Then we used the choice prompt to get it in the previous step.
        var install = stepContext.Values[InstallInfo] as InstallApp;
        if (install.AppName is null)
        {
            install.AppName = (stepContext.Result as FoundChoice).Value;
        }

        // We now need the machine name, so prompt for it.
        return await stepContext.PromptAsync(
            TextId,
            new PromptOptions
            {
                Prompt = MessageFactory.Text(
                    $"Found {install.AppName}. What is the name of the machine to install application?"),
            },
            cancellationToken);
    }
    ```
1. **machineNameAsync** 中的逻辑包装在最后一个瀑布步骤中。
    - 从文本提示结果中检索计算机名，并更新对话状态。
    - 我们将删除更新数据库的调用，因为支持代码位于不同的项目中。
    - 然后，我们向用户发送成功消息并结束对话。
    ```csharp
    private async Task<DialogTurnResult> SubmitRequestAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        if (stepContext.Reason != DialogReason.CancelCalled)
        {
            // Get the tracking info and add the machine name.
            var install = stepContext.Values[InstallInfo] as InstallApp;
            install.MachineName = stepContext.Context.Activity.Text;

            //TODO: Save to this information to the database.
        }

        await stepContext.Context.SendActivityAsync(
            $"Great, your request to install {install.AppName} on {install.MachineName} has been scheduled.",
            cancellationToken: cancellationToken);

        return await stepContext.EndDialogAsync(null, cancellationToken);
    }
    ```
1. 为了模拟数据库，我已更新 **getAppsAsync** 以查询静态列表而不是数据库。
    ```csharp
    private async Task<List<string>> getAppsAsync(string Name)
    {
        List<string> names = new List<string>();

        // Simulate querying the database for applications that match.
        return (from app in AppMsis
            where app.ToLower().Contains(Name.ToLower())
            select app).ToList();
    }

    // Example list of app names in the database.
    private static readonly List<string> AppMsis = new List<string>
    {
        "µTorrent 3.5.0.44178",
        "7-Zip 17.1",
        "Ad-Aware 9.0",
        "Adobe AIR 2.5.1.17730",
        "Adobe Flash Player (IE) 28.0.0.105",
        "Adobe Flash Player (Non-IE) 27.0.0.130",
        "Adobe Reader 11.0.14",
        "Adobe Shockwave Player 12.3.1.201",
        "Advanced SystemCare Personal 11.0.3",
        "Auslogics Disk Defrag 3.6",
        "avast! 4 Home Edition 4.8.1351",
        "AVG Anti-Virus Free Edition 9.0.0.698",
        "Bonjour 3.1.0.1",
        "CCleaner 5.24.5839",
        "Chmod Calculator 20132.4",
        "CyberLink PowerDVD 17.0.2101.62",
        "DAEMON Tools Lite 4.46.1.328",
        "FileZilla Client 3.5",
        "Firefox 57.0",
        "Foxit Reader 4.1.1.805",
        "Google Chrome 66.143.49260",
        "Google Earth 7.3.0.3832",
        "Google Toolbar (IE) 7.5.8231.2252",
        "GSpot 2701.0",
        "Internet Explorer 903235.0",
        "iTunes 12.7.0.166",
        "Java Runtime Environment 6 Update 17",
        "K-Lite Codec Pack 12.1",
        "Malwarebytes Anti-Malware 2.2.1.1043",
        "Media Player Classic 6.4.9.0",
        "Microsoft Silverlight 5.1.50907",
        "Mozilla Thunderbird 57.0",
        "Nero Burning ROM 19.1.1005",
        "OpenOffice.org 3.1.1 Build 9420",
        "Opera 12.18.1873",
        "Paint.NET 4.0.19",
        "Picasa 3.9.141.259",
        "QuickTime 7.79.80.95",
        "RealPlayer SP 12.0.0.319",
        "Revo Uninstaller 1.95",
        "Skype 7.40.151",
        "Spybot - Search & Destroy 1.6.2.46",
        "SpywareBlaster 4.6",
        "TuneUp Utilities 2009 14.0.1000.353",
        "Unlocker 1.9.2",
        "VLC media player 1.1.6",
        "Winamp 5.56 Build 2512",
        "Windows Live Messenger 2009 16.4.3528.331",
        "WinPatrol 2010 31.0.2014",
        "WinRAR 5.0",
    };
    ```

### <a name="update-the-local-admin-dialog"></a>更新本地管理员对话

在 v3 中，此对话问候用户、启动 Formflow 对话，然后将结果保存到数据库。 这可以轻松转换为双步骤瀑布。

1. 更新 `using` 语句。 请注意，此对话包含一个 v3 Formflow 对话。 在 v4 中，我们可以使用社区 Formflow 库。
    ```csharp
    using System.Threading;
    using System.Threading.Tasks;
    using Bot.Builder.Community.Dialogs.FormFlow;
    using ContosoHelpdeskChatBot.Models;
    using Microsoft.Bot.Builder.Dialogs;
    ```
1. 可以删除 `LocalAdmin` 的实例属性，因为结果将在对话状态中提供。
1. 定义用于对话的 ID 的常量。 在社区库中，构造的对话 ID 始终设置为对话生成的类的名称。
    ```csharp
    // Set up our dialog and prompt IDs as constants.
    private const string MainDialog = "mainDialog";
    private static string AdminDialog { get; } = nameof(LocalAdminPrompt);
    ```
1. 添加一个构造函数，并初始化组件的对话集。 Formflow 对话以相同的方式创建。 只需在构造函数中将该对话添加到组件的对话集。
    ```csharp
    public LocalAdminDialog(string dialogId) : base(dialogId)
    {
        InitialDialogId = MainDialog;

        AddDialog(new WaterfallDialog(MainDialog, new WaterfallStep[]
        {
            BeginFormflowAsync,
            SaveResultAsync,
        }));
        AddDialog(FormDialog.FromForm(BuildLocalAdminForm, FormOptions.PromptInStart));
    }
    ```
1. 可将 **StartAsync** 替换为第一个瀑布步骤。 我们已在构造函数中创建 Formflow，其他两个语句将转换为此 Formflow。
    ```csharp
    private async Task<DialogTurnResult> BeginFormflowAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        await stepContext.Context.SendActivityAsync("Great I will help you request local machine admin.");

        // Begin the Formflow dialog.
        return await stepContext.BeginDialogAsync(AdminDialog, cancellationToken: cancellationToken);
    }
    ```
1. 可将 **ResumeAfterLocalAdminFormDialog** 替换为第二个瀑布步骤。 必须从步骤上下文而不是实例属性中获取返回值。
    ```csharp
    private async Task<DialogTurnResult> SaveResultAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // Get the result from the Formflow dialog when it ends.
        if (stepContext.Reason != DialogReason.CancelCalled)
        {
            var admin = stepContext.Result as LocalAdminPrompt;

            //TODO: Save to this information to the database.
        }

        return await stepContext.EndDialogAsync(null, cancellationToken);
    }
    ```
1. **BuildLocalAdminForm** 基本上保持不变，不过，我们没有让 Formflow 更新实例属性。
    ```csharp
    // Nearly the same as before.
    private IForm<LocalAdminPrompt> BuildLocalAdminForm()
    {
        //here's an example of how validation can be used in form builder
        return new FormBuilder<LocalAdminPrompt>()
            .Field(nameof(LocalAdminPrompt.MachineName),
            validate: async (state, value) =>
            {
                ValidateResult result = new ValidateResult { IsValid = true, Value = value };
                //add validation here

                //this.admin.MachineName = (string)value;
                return result;
            })
            .Field(nameof(LocalAdminPrompt.AdminDuration),
            validate: async (state, value) =>
            {
                ValidateResult result = new ValidateResult { IsValid = true, Value = value };
                //add validation here

                //this.admin.AdminDuration = Convert.ToInt32((long)value) as int?;
                return result;
            })
            .Build();
    }
    ```

### <a name="update-the-reset-password-dialog"></a>更新重置密码对话

在 v3 中，此对话问候用户、使用通行短语为用户授权、使 Formflow 对话失败或启动，然后重置密码。 这仍然可以转换为瀑布。

1. 更新 `using` 语句。 请注意，此对话包含一个 v3 Formflow 对话。 在 v4 中，我们可以使用社区 Formflow 库。
    ```csharp
    using System;
    using System.Threading;
    using System.Threading.Tasks;
    using Bot.Builder.Community.Dialogs.FormFlow;
    using ContosoHelpdeskChatBot.Models;
    using Microsoft.Bot.Builder.Dialogs;
    ```
1. 定义用于对话的 ID 的常量。 在社区库中，构造的对话 ID 始终设置为对话生成的类的名称。
    ```csharp
    // Set up our dialog and prompt IDs as constants.
    private const string MainDialog = "mainDialog";
    private static string ResetDialog { get; } = nameof(ResetPasswordPrompt);
    ```
1. 添加一个构造函数，并初始化组件的对话集。 Formflow 对话以相同的方式创建。 只需在构造函数中将该对话添加到组件的对话集。
    ```csharp
    public ResetPasswordDialog(string dialogId) : base(dialogId)
    {
        InitialDialogId = MainDialog;

        AddDialog(new WaterfallDialog(MainDialog, new WaterfallStep[]
        {
            BeginFormflowAsync,
            ProcessRequestAsync,
        }));
        AddDialog(FormDialog.FromForm(BuildResetPasswordForm, FormOptions.PromptInStart));
    }
    ```
1. 可将 **StartAsync** 替换为第一个瀑布步骤。 我们已在构造函数中创建 Formflow。 否则，我们将保留相同的逻辑，只需将 v3 调用转换为 v4 等效项。
    ```csharp
    private async Task<DialogTurnResult> BeginFormflowAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        await stepContext.Context.SendActivityAsync("Alright I will help you create a temp password.");

        // Check the passcode and fail out or begin the Formflow dialog.
        if (sendPassCode(stepContext))
        {
            return await stepContext.BeginDialogAsync(ResetDialog, cancellationToken: cancellationToken);
        }
        else
        {
            //here we can simply fail the current dialog because we have root dialog handling all exceptions
            throw new Exception("Failed to send SMS. Make sure email & phone number has been added to database.");
        }
    }
    ```
1. **sendPassCode** 主要是出于练习目的而保留的。 原始代码已注释掉，方法只返回 true。 此外，我们同样可以删除电子邮件地址，因为原始机器人中不使用它。
    ```csharp
    private bool sendPassCode(DialogContext context)
    {
        //bool result = false;

        //Recipient Id varies depending on channel
        //refer ChannelAccount class https://docs.botframework.com/en-us/csharp/builder/sdkreference/dd/def/class_microsoft_1_1_bot_1_1_connector_1_1_channel_account.html#a0b89cf01fdd73cbc00a524dce9e2ad1a
        //as well as Activity class https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html
        //int passcode = new Random().Next(1000, 9999);
        //Int64? smsNumber = 0;
        //string smsMessage = "Your Contoso Pass Code is ";
        //string countryDialPrefix = "+1";

        // TODO: save PassCode to database
        //using (var db = new ContosoHelpdeskContext())
        //{
        //    var reset = db.ResetPasswords.Where(r => r.EmailAddress == email).ToList();
        //    if (reset.Count >= 1)
        //    {
        //        reset.First().PassCode = passcode;
        //        smsNumber = reset.First().MobileNumber;
        //        result = true;
        //    }

        //    db.SaveChanges();
        //}

        // TODO: send passcode to user via SMS.
        //if (result)
        //{
        //    result = Helper.SendSms($"{countryDialPrefix}{smsNumber.ToString()}", $"{smsMessage} {passcode}");
        //}

        //return result;
        return true;
    }
    ```
1. **BuildResetPasswordForm** 没有变化。
1. 可将 **ResumeAfterLocalAdminFormDialog** 替换为第二个瀑布步骤，我们将从步骤上下文中获取返回值。 我们已删除原始对话不对其做任何处理的电子邮件地址，并且我们提供了虚拟结果而不是查询数据库。 我们将保留相同的逻辑，只需将 v3 调用转换为 v4 等效项。
    ```csharp
    private async Task<DialogTurnResult> ProcessRequestAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // Get the result from the Formflow dialog when it ends.
        if (stepContext.Reason != DialogReason.CancelCalled)
        {
            var prompt = stepContext.Result as ResetPasswordPrompt;
            int? passcode;

            // TODO: Retrieve the passcode from the database.
            passcode = 1111;

            if (prompt.PassCode == passcode)
            {
                string temppwd = "TempPwd" + new Random().Next(0, 5000);
                await stepContext.Context.SendActivityAsync(
                    $"Your temp password is {temppwd}",
                    cancellationToken: cancellationToken);
            }
        }

        return await stepContext.EndDialogAsync(null, cancellationToken);
    }
    ```

### <a name="update-models-as-necessary"></a>根据需要更新模型

需要更新引用 Formflow 库的某些模型中的 `using` 语句。

1. 在 `LocalAdminPrompt` 中，将其更改为：
    ```csharp
    using Bot.Builder.Community.Dialogs.FormFlow;
    ```
1. 在 `ResetPasswordPrompt` 中，将其更改为：
    ```csharp
    using Bot.Builder.Community.Dialogs.FormFlow;
    using System;
    ```

## <a name="update-webconfig"></a>更新 Web.config

注释掉 **MicrosoftAppId** 和 **MicrosoftAppPassword** 的配置键。 这样，便可以在本地调试机器人，而无需在仿真器中提供这些值。

## <a name="run-and-test-your-bot-in-the-emulator"></a>在仿真器中运行并测试机器人

此时，我们应该能够在 IIS 中本地运行机器人，并使用仿真器连接到该机器人。

1. 在 IIS 中运行机器人。
1. 启动仿真器并连接到机器人的终结点（例如 **http://localhost:3978/api/messages**）。
    - 如果这是首次运行机器人，请单击“文件”>“新建机器人”，然后按照屏幕上的说明操作。 否则，单击“文件”>“打开机器人”打开现有机器人。
    - 仔细检查配置中的端口设置。 例如，如果在浏览器中通过 `http://localhost:3979/` 打开机器人，请在仿真器中将机器人的终结点设置为 `http://localhost:3979/api/messages`。
1. 所有四个对话应会正常运行，并且你可以在瀑布步骤中设置断点，以检查这些断点处的对话上下文和对话状态。

## <a name="additional-resources"></a>其他资源

v4 概念主题：

- [机器人工作原理](../bot-builder-basics.md)
- [管理状态](../bot-builder-concept-state.md)
- [对话框库](../bot-builder-concept-dialog.md)

v4 操作指南主题：

- [发送和接收文本消息](../bot-builder-howto-send-messages.md)
- [保存用户和聊天数据](../bot-builder-howto-v4-state.md)
- [实现顺序聊天流](../bot-builder-dialog-manage-conversation-flow.md)
- [使用对话提示收集用户输入](../bot-builder-prompts.md)

<!-- TODO:
- The conceptual piece
- The migration to a .NET Core project
-->
