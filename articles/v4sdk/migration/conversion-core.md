---
title: 迁移新 .NET Core 项目中的现有机器人 | Microsoft Docs
description: 使用新 .NET Core 项目将现有的 .NET v3 机器人迁移到 .NET v4 SDK。
keywords: 机器人, formflow, 对话, v3 机器人
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 06/17/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: d09b52760bf6ef15dac0205d9aef2d3906fcacfd
ms.sourcegitcommit: 41c8caf0e0c849beeeb50cdccf6dbc1ba7cce442
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/24/2019
ms.locfileid: "67344732"
---
# <a name="migrate-a-net-v3-bot-to-a-net-core-v4-bot"></a>将 .NET v3 机器人迁移到 .NET Core v4 机器人

在本文中，我们会将 [v3 ContosoHelpdeskChatBot](https://github.com/microsoft/BotBuilder-Samples/tree/master/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V3) 转换为新 .NET Core 项目中的  v4 机器人。
这种转换划分为以下步骤：

1. 使用模板创建新项目。
1. 根据需要安装附加 NuGet 包。
1. 个性化机器人，更新 Startup.cs 文件，并更新控制器类。
1. 更新机器人类。
1. 复制并更新对话和模型。
1. 最后的移植步骤。

此转换的结果是 [.NET Core v4 ContosoHelpdeskChatBot](https://github.com/microsoft/BotBuilder-Samples/tree/master/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore)。
若要在不转换项目类型的情况下迁移到 .NET Framework v4 机器人，  ，请参阅[将 .NET v3 机器人迁移到 .NET Framework v4 机器人](conversion-framework.md)。

Bot Framework SDK v4 与 SDK v3 基于相同的基础 REST API。 但是，SDK v4 对以前的 SDK 版本作了重构，使开发人员对其机器人拥有更高的灵活性和控制度。 该 SDK 的主要更改包括：

- 通过状态管理对象和属性访问器管理状态。
- 设置轮次处理程序以及向其传递活动的方式发生了变化。
- 可评分项不再存在。 在将控制权传递给对话之前，可以检查轮次处理程序中的“全局”命令。
- 采用新的对话库，该库与以前的版本有很大的不同。 需要使用组件和瀑布对话以及适用于 v4 的 Formflow 对话的社区实现，将旧对话转换为新的对话系统

有关特定更改的详细信息，请参阅 [v3 和 v4 .NET SDK 之间的差异](migration-about.md)。

## <a name="create-the-new-project-using-a-template"></a>使用模板创建新项目

为机器人创建新项目。

1. 请安装适用于 [C#](https://aka.ms/bot-vsix) 的 Bot Framework SDK v4 模板（如果尚未安装）。
1. 打开 Visual Studio，然后从模板创建一个新的 Echo 机器人项目。 将项目命名为 `ContosoHelpdeskChatBot`。

## <a name="install-additional-nuget-packages"></a>安装附加 NuGet 包

该模板安装你所需的大部分包，其中包括 Microsoft.Bot.Builder 和 Microsoft.Bot.Connector 包   。

1. 添加 **Bot.Builder.Community.Dialogs.Formflow**

    这是一个社区库，用于从 v3 Formflow 定义文件生成 v4 对话。 它有一个依赖项 **Microsoft.Bot.Builder.Dialogs**，因此系统也会安装此依赖项。

1. 添加 log4net 以支持日志记录  。

## <a name="personalize-your-bot"></a>个性化机器人

1. 将机器人文件从“Bots\EchoBot.cs”重命名为“Bots\DialogBot.cs”，并将 `EchoBot` 类重命名为 `DialogBot`   。
1. 将控制器从“Controllers\BotController.cs”重命名为“Controllers\MessagesController.cs”，并将 `BotController` 类重命名为 `MessagesController`   。

## <a name="update-your-startupcs-file"></a>更新 Startup.cs 文件

状态和机器人接收传入活动的方式已更改。 我们需要在 v4 中自行设置[状态管理](../bot-builder-concept-state.md)基础结构的部件。 例如，v4 使用机器人适配器来处理身份验证以及将活动转发到机器人代码，我们必须提前声明状态属性。

为 `DialogState` 创建状态属性，目前，在 v4 中支持对话需要提供此属性。 在控制器和机器人代码中使用依赖项注入获取所需的信息。

在 Startup.cs 中  ：

1. 更新 `using` 语句：[!code-csharp[Using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Startup.cs?range=4-13)]

1. 删除此构造函数：
    ```csharp
    public Startup(IConfiguration configuration)
    {
        Configuration = configuration;
    }
    ```

1. 删除 `Configuration` 属性。

1. 将 `ConfigureServices`方法更新为以下代码：[!code-csharp[Using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Startup.cs?range=19-41)]

此时将出现编译时错误。 我们将在下一步骤中对其进行修复。 

## <a name="messagescontroller-class"></a>MessagesController 类
此类处理一个请求。 依赖项注入将在运行时提供适配器和 IBot 实现。 此模板类保持不变。 

这就是机器人在 v4 中开始轮次的位置，这与 v3 消息控制器完全不同。 除了机器人的轮次处理程序本身外，大部分组件都可被视为样本。

机器人轮次处理程序将在 Bots\DialogBot.cs 中定义  。

### <a name="ignore-the-cancelscorable-and-globalmessagehandlersbotmodule-classes"></a>忽略 CancelScorable 和 GlobalMessageHandlersBotModule 类

因为 v4 中不存在可评分项，所以我们可以忽略这些类。 我们将更新轮次处理程序来响应 `cancel` 消息。

## <a name="update-your-bot-class"></a>更新机器人类

在 v4 中，轮次处理程序或消息循环逻辑主要位于机器人文件中。 我们的类派生自 `ActivityHandler`，后者为常见活动类型定义处理程序。

1. 更新 Bots\DialogBots.cs 文件  。

1. 更新 `using` 语句：[!code-csharp[Using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Bots/DialogBot.cs?range=4-8)]

1. 更新 `DialogBot` 以包括对话的泛型参数。
    [!code-csharp[Class definition](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Bots/DialogBot.cs?range=19)]

1. 添加以下字段和一个构造函数，将其初始化。 同样，ASP.NET 使用依赖项注入来获取参数值。
    [!code-csharp[Fields and constructor](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Bots/DialogBot.cs?range=21-28)]

1. 更新 `OnMessageActivityAsync` 实现，以调用主对话。 （我们很快就会定义 `Run` 扩展方法。）[!code-csharp[OnMessageActivityAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Bots/DialogBot.cs?range=38-47)]

1. 更新 `OnTurnAsync`，以便在轮次结束时保存对话状态。 在 v4 中，我们必须显式这样做才能将状态写入到持久层。 `ActivityHandler.OnTurnAsync` 方法根据已接收活动的类型调用特定的活动处理程序方法，因此我们会在调用基方法后保存状态。
    [!code-csharp[OnTurnAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Bots/DialogBot.cs?range=30-36)]

### <a name="create-the-run-extension-method"></a>创建 Run 扩展方法

我们将创建一个扩展方法，用于合并通过机器人运行裸机组件对话所需的代码。

创建 **DialogExtensions.cs** 文件并实现 `Run` 扩展方法。
[!code-csharp[The extension](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/DialogExtensions.cs?range=4-26)]

## <a name="copy-over-and-convert-your-dialogs"></a>复制并转换对话

我们将对原始 v3 对话做出大量更改，以将其迁移到 v4 SDK。 暂时不用担心编译器错误。 完成转换后，这些错误可自行解决。
为了避免多余地修改原始代码，完成迁移后，仍会出现一些编译器警告。

所有对话将派生自 `ComponentDialog`，而不是实现 v3 的 `IDialog<object>` 接口。

此机器人包含四个需要转换的对话：

| | |
|---|---|
| [RootDialog](#update-the-root-dialog) | 提供选项并启动其他对话。 |
| [InstallAppDialog](#update-the-install-app-dialog) | 处理在计算机上安装应用的请求。 |
| [LocalAdminDialog](#update-the-local-admin-dialog) | 处理本地计算机管理员权限的请求。 |
| [ResetPasswordDialog](#update-the-reset-password-dialog) | 处理重置密码的请求。 |

因为可评分项不再存在。所以不会复制 `CancelScorable` 类。 在将控制权传递给对话之前，可以检查轮次处理程序中的_全局_命令。

这些对话会收集输入，但不会在计算机上执行上述任何操作。

1. 在项目中创建“对话”文件夹  。
1. 将这些文件从 v3 项目的对话目录复制到新的对话框目录中。
    - **InstallAppDialog.cs**
    - **LocalAdminDialog.cs**
    - **ResetPasswordDialog.cs**
    - **RootDialog.cs**

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

- 在对话代码中，使用 `DialogContext.Context` 属性获取当前轮次上下文。
- 瀑布步骤包含派生自 `DialogContext` 的 `WaterfallStepContext` 参数。
- 所有具体对话和提示类派生自 `Dialog` 抽象类。
- 创建组件对话时分配 ID。 需要为对话集中的每个对话分配该集中唯一的 ID。

### <a name="update-the-root-dialog"></a>更新根对话

在此机器人中，根对话将提示用户从一组选项中做出选择，然后根据所做的选择启动子对话。 然后，在聊天的整个生存期内，此过程都会循环。

- 可将主要流设置为瀑布对话，这是 v4 SDK 中的一个新概念。 此流将按照一组固定的步骤依序运行。 有关详细信息，请参阅[实现顺序聊天流](~/v4sdk/bot-builder-dialog-manage-conversation-flow.md)。
- 现在会通过提示类处理提示。提示类是一些简短的子对话，它们提示提供输入、执行某种极简的处理和验证，然后返回值。 有关详细信息，请参阅[使用对话提示收集用户输入](~/v4sdk/bot-builder-prompts.md)。

在 **Dialogs/RootDialog.cs** 文件中：

1. 更新 `using` 语句：[!code-csharp[Using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Dialogs/RootDialog.cs?range=4-10)]

1. 需要将 `HelpdeskOptions` 选项从字符串列表转换为选项列表。 此项将与选项提示结合使用。选项提示接受选项编号（列表中）、选项值，或选项的任何同义词（作为有效输入）。
    [!code-csharp[HelpDeskOptions](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Dialogs/RootDialog.cs?range=28-33)]

1. 添加一个构造函数。 此代码将执行以下操作：
   - 创建对话时，将为该对话的每个实例分配一个 ID。 对话 ID 是对话所要添加到的对话集的一部分。 如前所述，机器人已使用 `MessageController` 类中的对话对象初始化。 每个 `ComponentDialog` 具有自身的内部对话集，该集具有自身的对话 ID 集。
   - 它会添加其他对话，包括选项提示（子对话）。 此处，我们只对每个对话 ID 使用类名。
   - 此类定义包括三个步骤的瀑布对话。 稍后我们将实现这些步骤。
     - 该对话首先提示用户选择要执行的任务。
     - 然后，启动与该选项关联的子对话。
     - 最后重启自身。
   - 每个瀑布步骤是一个委托，接下来，在可能的情况下，我们将采用原始对话中的现有代码实现这些步骤。
   - 启动组件对话框时，它会启动其 _initial dialog_。 默认情况下，这是添加到组件对话的第一个子对话。 我们将显式设置 `InitialDialogId` 属性，这意味着，主要瀑布对话不需要是添加到集中的第一个对话。 例如，如果你偏向于先添加提示，则你可以这样做，而不会导致运行时问题。
    [!code-csharp[Constructor](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Dialogs/RootDialog.cs?range=35-49)]

1. 我们可以删除 `StartAsync` 方法。 组件对话开始时，会自动启动其初始对话。  在这种情况下，它是我们在构造函数中定义的瀑布对话。 该对话还会自动启动其第一个步骤。

1. 我们将删除 `MessageReceivedAsync` 和 `ShowOptions` 方法，并将其替换为第一个瀑布步骤。 这两个方法问候用户，并要求他们选择可用的选项之一。
   - 在此处可以看到调用选项提示时提供的选项列表以及问候语和错误消息。
   - 无需指定对话中要调用的下一个方法，因为选项提示完成后，瀑布会转到下一步骤。
   - 选项提示将不断循环，直到收到有效输入或整个对话堆栈被取消。
    [!code-csharp[PromptForOptionsAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Dialogs/RootDialog.cs?range=51-65)]

1. 可将 `OnOptionSelected` 替换为第二个瀑布步骤。 仍然根据用户的输入启动一个子对话。
   - 选项提示返回 `FoundChoice` 值。 此值显示在步骤上下文的 `Result` 属性中。 对话堆栈将所有返回值视为对象。 如果返回值来自某个对话，则就能知道对象值的类型是什么。 有关返回的每种提示类型的列表，请参阅[提示类型](../bot-builder-concept-dialog.md#prompt-types)。
   - 由于选项提示不引发异常，因此我们可以删除 try-catch 块。
   - 我们需要添加一个失败代码，以便此方法始终返回适当的值。 此代码应该永远不会命中，但如果命中，它可以让对话“正常失败”。
    [!code-csharp[ShowChildDialogAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Dialogs/RootDialog.cs?range=67-102)]

1. 最后，将旧的 `ResumeAfterOptionDialog` 方法替换为最后一个瀑布步骤。
    - 通过将堆栈上的原始实例替换为对话自身的新实例来重启瀑布，而不要像在原始对话中那样结束对话并返回票证编号。 之所以可以这样做，是因为原始应用始终忽略返回值（票证编号）并重启根对话。
    [!code-csharp[ResumeAfterAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Dialogs/RootDialog.cs?range=104-138)]

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

1. 更新 `using` 语句：[!code-csharp[Using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Dialogs/InstallAppDialog.cs?range=4-11)]

1. 为用于跟踪所收集信息的键定义一个常量。
    [!code-csharp[Key ID](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Dialogs/InstallAppDialog.cs?range=17-18)]

1. 添加一个构造函数，并初始化组件的对话集。
    [!code-csharp[Constructor](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Dialogs/InstallAppDialog.cs?range=20-33)]

1. 我们可以将 `StartAsync` 替换为第一个瀑布步骤。
    - 我们必须自行管理状态，因此，将要跟踪对话状态中的安装应用对象。
    - 请求用户提供输入的消息成了提示调用中的一个选项。
    [!code-csharp[GetSearchTermAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Dialogs/InstallAppDialog.cs?range=35-50)]

1. 我们可将 `appNameAsync` 和 `multipleAppsAsync` 替换为第二个瀑布步骤。
    - 现在我们将获取提示结果，而不只是查看用户的最后一条消息。
    - 数据库查询和 if 语句的组织方式与在 `appNameAsync` 中相同。 if 语句的每个块中的代码已更新，可以配合 v4 对话运行。
        - 如果有一个命中项，则会更新对话状态并继续下一步。
        - 如果有多个命中项，则使用选项提示来要求用户从选项列表中做出选择。 这意味着可以删除 `multipleAppsAsync`。
        - 如果没有命中项，则结束此对话，并向根对话返回 null。
    [!code-csharp[ResolveAppNameAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Dialogs/InstallAppDialog.cs?range=52-91)]

1. 解析查询后，`appNameAsync` 还要求用户提供其计算机名。 我们将在下一个瀑布步骤中捕获该逻辑部分。
    - 同样，在 v4 中，我们必须自行管理状态。 此处唯一的棘手问题是，如何通过上一步骤中的两个不同逻辑分支转到此步骤。
    - 我们使用与前面一样的文本提示来请求用户指定计算机名（这一次只需提供不同的选项即可）。
    [!code-csharp[GetMachineNameAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Dialogs/InstallAppDialog.cs?range=93-114)]

1. `machineNameAsync` 中的逻辑包装在最后一个瀑布步骤中。
    - 从文本提示结果中检索计算机名，并更新对话状态。
    - 我们将删除更新数据库的调用，因为支持代码位于不同的项目中。
    - 然后，我们向用户发送成功消息并结束对话。
    [!code-csharp[SubmitRequestAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Dialogs/InstallAppDialog.cs?range=116-135)]

1. 模拟数据库调用时，我们会模拟 `getAppsAsync` 以查询静态列表而不是数据库。
    [!code-csharp[GetAppsAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Dialogs/InstallAppDialog.cs?range=137-200)]

### <a name="update-the-local-admin-dialog"></a>更新本地管理员对话

在 v3 中，此对话问候用户、启动 Formflow 对话，然后将结果保存到数据库。 这可以轻松转换为双步骤瀑布。

1. 更新 `using` 语句。 请注意，此对话包含一个 v3 Formflow 对话。 在 v4 中，我们可以使用社区 Formflow 库。
    [!code-csharp[Using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Dialogs/LocalAdminDialog.cs?range=4-8)]

1. 可以删除 `LocalAdmin` 的实例属性，因为结果将在对话状态中提供。

1. 添加一个构造函数，并初始化组件的对话集。 Formflow 对话以相同的方式创建。 只需在构造函数中将该对话添加到组件的对话集。
    [!code-csharp[Constructor](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Dialogs/LocalAdminDialog.cs?range=14-23)]

1. 我们可以将 `StartAsync` 替换为第一个瀑布步骤。 我们已在构造函数中创建 Formflow，其他两个语句将转换为此 Formflow。 请注意，`FormBuilder` 将模型的类型名称指定为已生成对话的 ID，该对话是此模型的 `LocalAdminPrompt`。
    [!code-csharp[BeginFormflowAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Dialogs/LocalAdminDialog.cs?range=25-35)]

1. 可将 `ResumeAfterLocalAdminFormDialog` 替换为第二个瀑布步骤。 必须从步骤上下文而不是实例属性中获取返回值。
    [!code-csharp[SaveResultAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Dialogs/LocalAdminDialog.cs?range=37-50)]

1. `BuildLocalAdminForm` 基本上保持不变，不过，我们没有让 Formflow 更新实例属性。
    [!code-csharp[BuildLocalAdminForm](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Dialogs/LocalAdminDialog.cs?range=52-76)]

### <a name="update-the-reset-password-dialog"></a>更新重置密码对话

在 v3 中，此对话问候用户、使用通行短语为用户授权、使 Formflow 对话失败或启动，然后重置密码。 这仍然可以转换为瀑布。

1. 更新 `using` 语句。 请注意，此对话包含一个 v3 Formflow 对话。 在 v4 中，我们可以使用社区 Formflow 库。
    [!code-csharp[Using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Dialogs/ResetPasswordDialog.cs?range=4-9)]

1. 添加一个构造函数，并初始化组件的对话集。 Formflow 对话以相同的方式创建。 只需在构造函数中将该对话添加到组件的对话集。
    [!code-csharp[Constructor](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Dialogs/ResetPasswordDialog.cs?range=15-25)]

1. 我们可以将 `StartAsync` 替换为第一个瀑布步骤。 我们已在构造函数中创建 Formflow。 否则，我们将保留相同的逻辑，只需将 v3 调用转换为 v4 等效项。
    [!code-csharp[BeginFormflowAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Dialogs/ResetPasswordDialog.cs?range=27-45)]

1. `sendPassCode` 主要是出于练习目的而保留的。 原始代码已注释掉，方法只返回 true。 此外，我们同样可以删除电子邮件地址，因为原始机器人中不使用它。
    [!code-csharp[SendPassCode](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Dialogs/ResetPasswordDialog.cs?range=47-81)]

1. `BuildResetPasswordForm` 无更改。

1. 可将 `ResumeAfterResetPasswordFormDialog` 替换为第二个瀑布步骤，我们将从步骤上下文中获取返回值。 我们已删除原始对话不对其做任何处理的电子邮件地址，并且我们提供了虚拟结果而不是查询数据库。 我们将保留相同的逻辑，只需将 v3 调用转换为 v4 等效项。
    [!code-csharp[ProcessRequestAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Dialogs/ResetPasswordDialog.cs?range=90-113)]

## <a name="copy-over-and-update-models-as-necessary"></a>根据需要复制和更新模型
可以将相同的 v3 模型用于 v4 社区表单流库。 

1. 在项目中创建“模型”文件夹  。
1. 将这些文件从 v3 项目的模型目录复制到新的模型目录中。
    - **InstallApp.cs**
    - **LocalAdmin.cs**
    - **LocalAdminPrompt.cs**
    - **ResetPassword.cs**
    - **ResetPassword.cs**

### <a name="update-using-statements"></a>更新 using 语句

我们需要更新模型类中的 `using` 语句，如下所示。

1. 在 InstallApps.cs 中将其改为：[!code-csharp[Using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Models/InstallApp.cs?range=4-5)] 

1. 在 LocalAdmin.cs 中将其改为：[!code-csharp[Using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Models/LocalAdmin.cs?range=4-5)] 

1. 在 LocalAdminPrompt.cs 中将其改为：[!code-csharp[Using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Models/LocalAdminPrompt.cs?range=4)] 

1. 在 ResetPassword.cs 中将其更改为：[!code-csharp[Using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Models/ResetPassword.cs?range=4-5)] 
    另外，删除命名空间内的 `using` 语句。

1. 在 ResetPasswordPrompt.cs 中将其改为：[!code-csharp[Using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Models/ResetPasswordPrompt.cs?range=4-5)] 

### <a name="additional-changes"></a>其他更改

在 ResetPassword.cs 中，更改 `MobileNumber` 的返回类型，如下所示：[!code-csharp[MobileNumber](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Models/ResetPassword.cs?range=17)] 

## <a name="final-porting-steps"></a>最终移植步骤 
若要完成移植过程，请执行以下步骤：

1. 创建 `AdapterWithErrorHandler` 类来定义适配器，该适配器包含可以捕获中间件或应用程序中异常的错误处理程序。 适配器处理传入的活动并通过机器人中间件管道将其定向到机器人的逻辑，然后再返回。 使用以下代码创建类：[!code-csharp[MobileNumber](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/AdapterWithErrorHandler.cs?range=4-46)]
1. 根据需要修改 wwwroot\default.htm 页面  。

## <a name="run-and-test-your-bot-in-the-emulator"></a>在仿真器中运行并测试机器人

此时，我们应该能够在 IIS 中本地运行机器人，并使用仿真器连接到该机器人。

1. 在 IIS 中运行机器人。
1. 启动仿真器并连接到机器人的终结点（例如 **http://localhost:3978/api/messages** ）。
    - 如果这是首次运行机器人，请单击“文件”>“新建机器人”  ，然后按照屏幕上的说明操作。 否则，单击“文件”>“打开机器人”  打开现有机器人。
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
- [使用模拟器进行调试](../../bot-service-debug-emulator.md)
- [将遥测功能添加到机器人](../bot-builder-telemetry.md)
