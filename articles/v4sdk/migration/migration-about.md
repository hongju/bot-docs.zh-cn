---
title: v3 与 v4 SDK 之间的差异 | Microsoft Docs
description: 介绍 v3 与 v4 SDK 之间的差异。
keywords: 机器人迁移, formflow, 对话, 状态
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 862fbf59cf33406e35e9051c0814489fdfdaa8f3
ms.sourcegitcommit: ea64a56acfabc6a9c1576ebf9f17ac81e7e2a6b7
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/24/2019
ms.locfileid: "66215581"
---
# <a name="differences-between-the-v3-and-v4-net-sdk"></a>v3 与 v4 .NET SDK 之间的差异

Bot Framework SDK 版本 4 支持的底层 Bot Framework 服务与版本 3 相同。 但是，v4 对以前的 SDK 版本作了重构，使开发人员对其机器人拥有更高的灵活性和控制度。 该 SDK 的主要更改包括：

- 引入了机器人适配器。 该适配器是活动处理堆栈的一部分。
  - 该适配器处理 Bot Framework 身份验证。
  - 该适配器管理通道与机器人轮次处理程序之间的传入和传出流量，封装对 Bot Framework 连接器的调用。
  - 该适配器初始化每个轮次的上下文。
  - 有关更多详细信息，请参阅[机器人工作原理][about-bots]。
- 重构了状态管理。
  - 机器人中不再自动提供状态数据。
  - 现在需要通过状态管理对象和属性访问器管理状态。
  - 有关更多详细信息，请参阅[管理状态][about-state]。
- 新的对话库。
  - 需要根据新的对话库重新编写 v3 对话。
  - 可评分对象不再存在。 在将控制权传递给对话之前，可以检查“全局”命令。 这可能位于消息处理程序中，也可能位于父对话中，具体取决于 v4 机器人的设计方式。 如需示例，请参阅如何[处理用户中断][interruptions]。
  - 有关更多详细信息，请参阅[对话库][about-dialogs]。
- 支持 ASP.NET Core。
  - 用于创建新 C# 机器人的模板面向 ASP.NET Core 框架。
  - 你仍可以对机器人使用 ASP.NET，但 v4 的重心是支持 ASP.NET Core 框架。
  - 有关此框架的详细信息，请参阅 [ASP.NET Core 简介](https://docs.microsoft.com/aspnet/core/)。

## <a name="activity-processing"></a>活动处理

为机器人创建适配器时，还需要提供一个消息处理程序委托，用于从通道和用户接收传入的活动。 该适配器将为收到的每个活动创建轮次上下文对象。 它将轮次上下文对象传递给机器人的轮次处理程序，并在轮次完成后处置该对象。

轮次处理程序可以接收多种类型的活动。 一般情况下，你只希望将消息活动转发到机器人包含的任何对话。  如果机器人派生自 `ActivityHandler`，则机器人的轮次处理程序会将所有消息活动转发到 `OnMessageActivityAsync`。 重写该方法即可添加消息处理逻辑。 有关活动类型的详细信息，请参阅[活动架构][]。

### <a name="handling-turns"></a>处理轮次

处理某条消息时，可使用轮次上下文获取有关传入活动的信息，并将活动发送给用户：

| | |
|-|-|
| 获取传入活动 | 获取轮次上下文的 `Activity` 属性。 |
| 创建活动并将其发送给用户 | 调用轮次上下文的 `SendActivityAsync` 方法。<br/>有关详细信息，请参阅[发送和接收文本消息][send-messages]以及[将媒体添加到消息][send-media]。 |

`MessageFactory` 类提供一些帮助器方法用于创建活动及设置其格式。

### <a name="scorables-is-gone"></a>可评分对象已弃用

请在机器人的消息循环中处理这些对象。 有关如何使用 v4 对话执行此操作的说明，请参阅如何[处理用户中断][interruptions]。

可组合的可评分对象调度树以及可组合的链对话（例如默认异常）也已弃用。  重现此功能的方法之一是在机器人的轮次处理程序中实现它。

## <a name="state-management"></a>状态管理

在 v3 中，可以在 Bot State 服务中存储聊天数据，该服务是 Bot Framework 提供的更大型服务套件的一部分。 不过，该服务在 2018 年 3 月 31 日以后已停用。 从 v4 开始，有关状态管理的设计注意事项就像任何 Web 应用一样，有许多可用选项。 将其缓存在内存中和同一进程中通常是最容易的；但是，对于生产型应用，应该对状态进行更持久的存储。例如，存储在 SQL 或 NoSQL 数据库中，或者以 Blob 形式存储。

v4 不使用 `UserData`、`ConversationData` 和 `PrivateConversationData` 属性以及数据袋来管理状态。
现在需要根据[管理状态][about-state]中所述，通过状态管理对象和属性访问器来管理状态。

v4 定义 `UserState`、`ConversationState` 和 `PrivateConversationState` 类用于管理机器人的状态数据。 需为你想要持久保留的每个属性创建一个状态属性访问器，而不仅仅是读取和写入预定义的数据袋。

### <a name="setting-up-state"></a>设置状态

应该尽可能地在 .NET Core 的 **Startup.cs** 或 .NET Framework 的 **Global.asax.cs** 中将状态配置为单一实例。

1. 初始化一个或多个 `IStorage` 对象。 这表示机器人数据的后备存储。
    v4 SDK 提供一些[存储层](../bot-builder-concept-state.md#storage-layer)。
    你也可以实现自己的存储层以连接到不同类型的存储。
1. 然后，根据需要创建并注册[状态管理](../bot-builder-concept-state.md#state-management)对象。
    v4 中的范围与 v3 中相同，如果需要，可以创建其他范围。
1. 然后，为机器人所需的属性创建并注册[状态属性访问器](../bot-builder-concept-state.md#state-property-accessors)。
    在状态管理对象中，每个属性访问器需要唯一的名称。

### <a name="using-state"></a>使用状态

每当创建机器人时，都可以使用依赖项注入来访问这些对象。
（在 ASP.NET 中，将为每个轮次创建机器人或消息控制器的新实例。）使用状态属性访问器获取和更新属性，使用状态管理对象将任何更改写入存储。 认识到应该考虑并发性问题之后，可参考下文来完成一些常见任务。

| | |
|-|-|
| 创建状态属性访问器 | 调用 `BotState.CreateProperty<T>`。<br/>`BotState` 是聊天、私人聊天和用户状态的抽象基类。 |
| 获取属性的当前值 | 调用 `IStatePropertyAccessor<T>.GetAsync`。<br/>如果以前未设置任何值，它将使用默认工厂参数来生成值。 |
| 更新当前缓存的属性值 | 调用 `IStatePropertyAccessor<T>.SetAsync`。<br/>这只会更新缓存，而不会更新后备存储层。 |
| 将状态更改持久保存到存储 | 针对退出轮次处理程序之前其状态已发生更改的状态管理对象调用 `BotState.SaveChangesAsync`。 |

### <a name="managing-concurrency"></a>管理并发

机器人可能需要管理状态并发性。 有关详细信息，请参阅“管理状态”的[保存状态](../bot-builder-concept-state.md#saving-state)部分，以及“直接写入到存储”的[使用 eTag 管理并发性](../bot-builder-howto-v4-storage.md#manage-concurrency-using-etags)部分。  

## <a name="dialogs-library"></a>对话框库

下面是对对话做出的一些重大更改：

- 对话库现在是单独的 NuGet 包：**Microsoft.Bot.Builder.Dialogs**。
- 对话类不再需要可序列化。 通过 `DialogState` 状态属性访问器管理对话状态。
  - 与对话对象本身不同，现在对话状态属性在完成每个轮次后会持久保存。
- `IDialogContext` 接口已由 `DialogContext` 类取代。 在轮次中，可为对话集创建对话上下文。 
  - 此对话上下文封装对话堆栈（旧堆栈帧）。 此信息将在对话状态属性中持久保存。
- `IDialog` 接口已由 `Dialog` 抽象类取代。

### <a name="defining-dialogs"></a>定义对话

虽然 v3 可以灵活地通过 `IDialog` 界面来实现对话，但这也意味着你必须为验证之类的功能实现你自己的代码。 在 v4 中，现在有可以自动为你验证用户输入的提示类，这些类可以将用户输入约束成特定的类型（例如整数）并会反复提示用户，直至用户提供有效的输入。 通常情况下，这意味着你作为开发人员可以少编写一些代码。

现在，可以使用几个选项来定义对话：

| | |
|:--|:--|
| 派生自 `ComponentDialog` 类的组件对话 | 允许你封装对话代码，而不会与外部上下文发生命名冲突。 请参阅[重复使用对话][reuse-dialogs]。 |
| 瀑布对话，即 `WaterfallDialog` 类的实例 | 非常适合提示对话，可以提示用户提供各种类型的用户输入，然后对其进行验证。 瀑布可将流程的大部分环节自动化，但强制要求对话代码采用特定的格式；请参阅[有序聊天流][sequential-flow]。 |
| 派生自 `Dialog` 抽象类的自定义对话 | 使用此选项可在控制对话行为方面获得最大灵活性，但同时要求你更详细地了解对话堆栈的实现方式。 |

在 v3 中，我们使用 `FormFlow` 针对任务执行一系列步骤。 在 v4 中，瀑布对话替代 FormFlow。 创建瀑布对话时，在构造函数中定义对话的步骤。 步骤的执行顺序严格按所声明的那样进行，并且会自动地一个接一个地向前推进。

也可使用多个对话创建复杂的控制流；请参阅[高级聊天流][complex-flow]。

若要访问某个对话，需要将它的某个实例放入对话集，然后生成该集的对话上下文。   创建对话集时需要提供对话状态属性访问器。 这样，框架便可以在完成每个轮次后持久保存对话状态。 [管理状态][about-state]中介绍了如何在 v4 中管理状态。

### <a name="using-dialogs"></a>使用对话

下面列出了 v3 中的常用操作，以及如何在瀑布对话中完成这些操作。 请注意，瀑布对话的每个步骤预期会返回 `DialogTurnResult` 值。 如果不是，则可能表示瀑布已提前结束。

| Operation | v3 | v4 |
|:---|:---|:---|
| 处理对话的启动 | 实现 `IDialog.StartAsync` | 将此项设为瀑布对话的第一个步骤。 |
| 发送活动 | 调用 `IDialogContext.PostAsync`。 | 调用 `ITurnContext.SendActivityAsync`。<br/>使用步骤上下文的 `Context` 属性获取轮次上下文。  |
| 等待用户的响应 | 使用 `IAwaitable<IMessageActivity>` 参数并调用 `IDialogContext.Wait` | Return await `ITurnContext.PromptAsync` 以开始提示对话。 然后在瀑布的下一步骤中检索结果。 |
| 处理对话的持续 | 调用 `IDialogContext.Wait`。 | 将附加的步骤添加到瀑布对话，或实现 `Dialog.ContinueDialogAsync` |
| 指示结束处理，直到用户发出下一条消息 | 调用 `IDialogContext.Wait`。 | 返回 `Dialog.EndOfTurn` |
| 开始子对话 | 调用 `IDialogContext.Call`。 | Return await 步骤上下文的 `BeginDialogAsync` 方法。<br/>如果子对话返回了值，将通过步骤上下文的 `Result` 属性在瀑布的下一步骤中提供该值。 |
| 将当前对话替换为新对话 | 调用 `IDialogContext.Forward`。 | Return await `ITurnContext.ReplaceDialogAsync`。 |
| 表示当前对话已完成的信号 | 调用 `IDialogContext.Done`。 | return await 步骤上下文的 `EndDialogAsync` 方法。 |
| 从对话中失败退出。 | 调用 `IDialogContext.Fail`。 | 引发一个可在机器人的另一级别捕获的异常，结束步骤并返回 `Cancelled` 状态，或者调用步骤或对话上下文的 `CancelAllDialogsAsync`。<br/>请注意，在 v4 中，对话内的异常将沿 C# 堆栈而不是对话堆栈传播。 |

有关 v4 代码的其他说明：

- v4 中的各种 `Prompt` 派生类将用户提示实现为单独的两步骤对话。 请参阅如何[实现顺序聊天流][sequential-flow]。
- 使用 `DialogSet.CreateContextAsync` 创建当前轮次的对话上下文。
- 在对话内部，使用 `DialogContext.Context` 属性获取当前轮次上下文。
- 瀑布步骤包含派生自 `DialogContext` 的 `WaterfallStepContext` 参数。
- 所有具体对话和提示类派生自 `Dialog` 抽象类。
- 创建组件对话时分配 ID。 需要为对话集中的每个对话分配该集中唯一的 ID。

### <a name="passing-state-between-and-within-dialogs"></a>在对话之间和内部传递状态

“对话库”一文的[对话状态](../bot-builder-concept-dialog.md#dialog-state)、[瀑布步骤上下文属性](../bot-builder-concept-dialog.md#waterfall-step-context-properties)和[使用对话](../bot-builder-concept-dialog.md#using-dialogs)部分介绍了如何在 v4 中管理对话状态。 

### <a name="iawaitable-is-gone"></a>IAwaitable 已弃用

若要获取轮次中的用户活动，可从轮次上下文获取。

提示用户并接收提示结果：

- 将相应的提示实例添加到对话集。
- 从瀑布对话中的某个步骤调用提示。
- 从下一步骤中的步骤上下文 `Result` 属性检索结果。

### <a name="formflow"></a>Formflow

在 v3 中，Formflow 是 C# SDK 的一部分，但不是 JavaScript SDK 的一部分。 它不是 v4 SDK 的一部分，但 C# 存在社区版。

| NuGet 包名称 | 社区 GitHub 存储库 |
|-|-|
| Bot.Builder.Community.Dialogs.Formflow | [BotBuilderCommunity/botbuilder-community-dotnet/libraries/Bot.Builder.Community.Dialogs.FormFlow](https://github.com/BotBuilderCommunity/botbuilder-community-dotnet/tree/master/libraries/Bot.Builder.Community.Dialogs.FormFlow) |

## <a name="additional-resources"></a>其他资源

- [将 .NET SDK v3 机器人迁移到 v4](conversion-framework.md)

<!-- -->

[about-bots]: ../bot-builder-basics.md
[about-state]: ../bot-builder-concept-state.md
[about-dialogs]: ../bot-builder-concept-dialog.md

[send-messages]: ../bot-builder-howto-send-messages.md
[send-media]: ../bot-builder-howto-add-media-attachments.md

[sequential-flow]: ../bot-builder-dialog-manage-conversation-flow.md
[complex-flow]: ../bot-builder-dialog-manage-complex-conversation-flow.md
[reuse-dialogs]: ../bot-builder-compositcontrol.md
[interruptions]: ../bot-builder-howto-handle-user-interrupt.md

[活动架构]: https://aka.ms/botSpecs-activitySchema