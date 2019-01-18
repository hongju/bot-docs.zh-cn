---
title: 对话框概述 | Microsoft Docs
description: 了解如何使用 Bot Framework SDK for .NET 中的对话框为聊天建模和管理聊天流。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 3089e7a073f6a6d9af3a3720954af3a915106888
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/11/2019
ms.locfileid: "54224992"
---
# <a name="dialogs-in-the-bot-framework-sdk-for-net"></a>Bot Framework SDK for .NET 中的对话框

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-dialogs.md)
> - [Node.js](../nodejs/bot-builder-nodejs-dialog-overview.md)

使用 Bot Framework SDK for .NET 创建机器人时，可以使用对话框为聊天建模并管理[聊天流](../bot-service-design-conversation-flow.md)。 每个对话框都是一个抽象，在实现 `IDialog` 的 C# 类中封装自己的状态。 对话框可以由其他对话框组成以最大限度地重用，并且对话框上下文可以维护在任何时间点都在会话中处于活动状态的[对话框堆栈](../bot-service-design-conversation-flow.md#dialog-stack)。 

包含对话框的会话可以跨计算机移植，这使得机器人实现可以扩展。 在 Bot Framework SDK for .NET 中使用对话框时，聊天状态（对话框堆栈和堆栈中每个对话框的状态）会自动存储到所选择的[状态数据](bot-builder-dotnet-state.md)存储中。 这让机器人的服务代码可以是无状态，类似于不需要在 Web 服务器内存中存储会话状态的 Web 应用程序。 

## <a name="echo-bot-example"></a>回显机器人示例

请考虑以下回显机器人示例，该示例介绍如何更改在[快速入门](bot-builder-dotnet-quickstart.md)教程中创建的机器人，以便它使用对话框与用户交换消息。

> [!TIP]
> 若要按照此示例，使用[快速入门](bot-builder-dotnet-quickstart.md)教程中的说明创建机器人，然后更新其 MessagesController.cs 文件，如下所述。

### <a name="messagescontrollercs"></a>MessagesController.cs 

在 Bot Framework SDK for .NET 中，可以使用[生成器][builderLibrary]库实现对话框。 若要访问相关类，导入 `Dialogs` 命名空间。

[!code-csharp[Using statement](../includes/code/dotnet-dialogs.cs#usingStatement)]

接下来，将此 `EchoDialog` 类添加到 MessagesController.cs 来表示会话。 

[!code-csharp[EchoDialog class](../includes/code/dotnet-dialogs.cs#echobot1)]

然后，通过调用 `Conversation.SendAsync` 方法将 `EchoDialog` 类绑定到 `Post` 方法。

[!code-csharp[Post method](../includes/code/dotnet-dialogs.cs#echobot2)]

### <a name="implementation-details"></a>实现详细信息 

`Post` 方法标记为 `async`，因为 Bot Builder 使用 C# 设施来处理异步通信。 它将返回 `Task` 对象，代表负责向传入消息发送答复的任务。 如果出现异常，该方法返回的 `Task` 将包含异常信息。 

`Conversation.SendAsync` 方法是使用 Bot Framework SDK for .NET 实现对话框的关键。 它遵循<a href="https://en.wikipedia.org/wiki/Dependency_inversion_principle" target="_blank">依赖关系反转原则</a>并执行以下步骤：

1. 实例化所需组件  
2. 从 `IBotDataStore` 反序列化会话状态（对话框堆栈和堆栈中的每个对话框状态）
3. 恢复机器人在其中挂起的会话过程并等待消息
4. 发送答复
5. 序列化已更新的会话状态并将其保存回 `IBotDataStore`

第一次启动会话时，对话框不包含状态，因此 `Conversation.SendAsync` 构造 `EchoDialog` 并调用其 `StartAsync` 方法。 `StartAsync` 方法使用延续委托调用 `IDialogContext.Wait` 来指定在接收新消息时应调用的方法 (`MessageReceivedAsync`)。 

`MessageReceivedAsync` 方法等待消息、发布响应并等待下一条消息。 每次调用 `IDialogContext.Wait`，机器人就会进入挂起状态，并且可以在接收消息的任何计算机上重新启动。 

使用上面的代码示例创建的机器人将答复用户发送的每条消息，只需在用户消息前面加前缀文本“您说：”进行响应。 因为机器人使用对话框创建，所以它可以演变为支持更复杂的会话，而不必显式管理状态。

## <a name="echo-bot-with-state-example"></a>带有状态的回显机器人示例

下一个示例基于上一个示例通过添加跟踪对话框状态功能构建而成。 更新 `EchoDialog` 类时（如下面的代码示例所示），机器人将答复用户发送的每条消息，方法是通过在用户消息前面加数字 (`count`) 后跟文本“您说：”进行响应。 机器人将继续通过每个答复递增 `count`，直到用户选择重置计数。

### <a name="messagescontrollercs"></a>MessagesController.cs 

[!code-csharp[EchoDialog class](../includes/code/dotnet-dialogs.cs#echobot3)]

### <a name="implementation-details"></a>实现详细信息

在第一个示例中，收到新消息时调用 `MessageReceivedAsync` 方法。 不过，这次 `MessageReceivedAsync` 方法会在响应之前评估用户消息。 如果用户消息为“重置”，内置 `PromptDialog.Confirm` 提示会生成一个子对话框，提示用户确认计数重置。 子对话框有其自己的私有状态，该状态不会影响父对话框状态。 当用户响应提示，子对话框的结果会传递给 `AfterResetAsync` 方法，该方法将消息发送给用户以指示是否已重置计数，然后在下一条消息中调用 `IDialogContext.Wait` 并延续回 `MessageReceivedAsync`。

## <a name="dialog-context"></a>对话框上下文

传递到每个对话框方法的 `IDialogContext` 接口提供对服务的访问权限，对话框需要这些服务来保存状态并与该通道进行通信。 `IDialogContext` 接口包含三个接口：[Internals.IBotData][iBotData]、[Internals.IBotToUser][iBotToUser] 和 [Internals.IDialogStack][iDialogStack]。 

### <a name="internalsibotdata"></a>Internals.IBotData

`Internals.IBotData` 接口提供对每个用户、每个会话以及由 Connector 维护的私人会话状态数据的访问。 每个用户状态数据可用于存储与特定会话无关的用户数据，而每个会话数据用于存储有关会话的常规数据，私人会话数据则用于存储与特定会话相关的用户数据。 

### <a name="internalsibottouser"></a>Internals.IBotToUser

`Internals.IBotToUser` 提供方法向用户发送来自机器人的消息。 消息可能使用对 Web API 方法调用的响应以内联方式发送，或直接通过 [Connector 客户端](bot-builder-dotnet-connector.md#create-a-connector-client)发送。 通过对话框上下文发送和接收消息可确保 `Internals.IBotData` 状态通过 Connector 传递。

### <a name="internalsidialogstack"></a>Internals.IDialogStack

`Internals.IDialogStack` 提供方法来管理[对话框堆栈](../bot-service-design-conversation-flow.md#dialog-stack)。 大多数情况下会自动管理对话框堆栈。 但是，也有可能你希望显式管理堆栈。 例如，你可能想要调用子对话框并将其添加到对话框堆栈顶部，将当前对话框标记为“完成”（从而在对话框堆栈中将其删除，并将结果返回到堆栈中的先前对话框）、挂起当前对话框直到收到用户消息，甚至完全重置对话框堆栈。

## <a name="serialization"></a>序列化

对话框堆栈和所有活动对话框的状态都会序列化到每个用户，每个会话 [IBotDataBag][iBotDataBag]。 序列化机器人在机器人发送到 [Connector](bot-builder-dotnet-concepts.md#connector) 和从中接收的消息中保持不变。 若要进行序列化，`Dialog` 类必须包含 `[Serializable]` 属性。 [Builder][builderLibrary] 库中的所有 `IDialog` 实现都标记为可序列化。 

[Chain 方法](#dialog-chains)提供在 LINQ 查询语法中可用的流畅对话框接口。 LINQ 查询语法的已编译形式通常使用匿名方法。 如果这些匿名方法不引用局部变量的环境，这些匿名方法就没有状态，并且完全可序列化。 但是，如果匿名方法捕获环境中的任何局部变量，生成的闭包对象（由编译器生成）并没有被标记为可序列化。 在这种情况下，Bot Builder 会引发 `ClosureCaptureException` 来标识问题。

为了使用反射来序列化未标记为可序列化的类，Builder 库包含了一个基于反射的序列化代理，可以使用它来注册 [Autofac][autofac]。

[!code-csharp[Serialization](../includes/code/dotnet-dialogs.cs#serialization)]

## <a id="dialog-chains"></a> 对话框链

虽然可以使用 `IDialogStack.Call<R>` 和 `IDialogStack.Done<R>` 来显式管理活动对话框堆栈，还是可以通过使用这些流畅的 [Chain][chain] 方法隐式管理活动对话框堆栈。


|           方法            |  类型   |                                 说明                                  |
|-----------------------------|---------|------------------------------------------------------------------------|
|     Chain.Select<T, R>      |  LINQ   |           支持 LINQ 查询语法中的“select”和“let”。            |
|  Chain.SelectMany<T, C, R>  |  LINQ   |            支持 LINQ 查询语法中的连续“from”。            |
|       Chain.Where<T>        |  LINQ   |                 支持 LINQ 查询语法中的“where”。                 |
|        Chain.From<T>        | Chains  |                实例化一个新的对话框实例。                |
|       Chain.Return<T>       | Chains  |                向链返回常量值。                |
|         Chain.Do<T>         | Chains  |               允许链中的副作用。                |
|  Chain.ContinueWith<T, R>   | Chains  |                      简单对话框链。                       |
|       Chain.Unwrap<T>       | Chains  |                  解包嵌套在对话框中的对话框。                   |
| Chain.DefaultIfException<T> | Chains  | 吞并前面结果中的异常，并返回 default(T)。 |
|        Chain.Loop<T>        | 分支  |                   循环整个对话框链。                   |
|        Chain.Fold<T>        | 分支  |   将对话框枚举中的结果折叠到单个结果中。   |
|     Chain.Switch<T, R>      | 分支  |            支持分支到不同对话框链。            |
|     Chain.PostToUser<T>     | 消息 |                      将消息发送给用户。                      |
|     Chain.WaitToBot<T>      | 消息 |                    等待消息发送给机器人。                     |
|    Chain.PostToChain<T>     | 消息 |              从用户消息开始一个链。              |

### <a name="examples"></a>示例 

LINQ 查询语法使用 `Chain.Select<T, R>` 方法。

[!code-csharp[Chain.Select](../includes/code/dotnet-dialogs.cs#chain1)]

或 `Chain.SelectMany<T, C, R>` 方法。

[!code-csharp[Chain.SelectMany](../includes/code/dotnet-dialogs.cs#chain2)]

`Chain.PostToUser<T>` 和 `Chain.WaitToBot<T>` 方法将消息从机器人发送给用户，反之亦然。

[!code-csharp[Chain.PostToUser](../includes/code/dotnet-dialogs.cs#chain3)]

`Chain.Switch<T, R>` 方法对会话对话框流进行分支。

[!code-csharp[Chain.Switch](../includes/code/dotnet-dialogs.cs#chain4)]

如果 `Chain.Switch<T, R>` 返回嵌套的 `IDialog<IDialog<T>>`，则内部 `IDialog<T>` 可以使用 `Chain.Unwrap<T>` 解包。 这允许将会话分支到对话框链的不同路径，可能是不等长度。 此示例演示了一个更完整的分支对话框，它用流畅的链样式编写，包含隐式堆栈管理。

[!code-csharp[Chain.Switch](../includes/code/dotnet-dialogs.cs#chain5)]

## <a name="next-steps"></a>后续步骤

对话框管理机器人和用户之间的会话流。 对话框定义如何与用户进行交互。 机器人可以使用堆栈中组织的许多对话框来指导与用户的会话。 在下节中，了解在堆栈中创建和删除对话框时，对话框堆栈如何增长和缩减。

> [!div class="nextstepaction"]
> [使用对话框管理会话流](bot-builder-dotnet-manage-conversation-flow.md)


[builderLibrary]: /dotnet/api/microsoft.bot.builder.dialogs

[iBotData]: /dotnet/api/microsoft.bot.builder.dialogs.internals.ibotdata

[iBotToUser]: /dotnet/api/microsoft.bot.builder.dialogs.internals.ibottouser

[iDialogStack]: /dotnet/api/microsoft.bot.builder.dialogs.internals.idialogstack

[iBotDataBag]: /dotnet/api/microsoft.bot.builder.dialogs.ibotdatabag

[autofac]: /dotnet/api/microsoft.bot.builder.autofac.base

[chain]: /dotnet/api/microsoft.bot.builder.dialogs.chain
