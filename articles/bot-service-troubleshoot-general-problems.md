---
title: 故障排除机器人 | Microsoft Docs
description: 使用技术常见问题排查机器人开发中的常见问题。
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 09/26/2018
ms.openlocfilehash: 410f50f02dcea2bb64ccf0389e20f5cb76e2fd6b
ms.sourcegitcommit: 3cb288cf2f09eaede317e1bc8d6255becf1aec61
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/27/2018
ms.locfileid: "47389836"
---
# <a name="troubleshooting-general-problems"></a>排查常见问题
这些常见问题有助于排查常见的机器人开发或操作问题。

## <a name="how-can-i-troubleshoot-issues-with-my-bot"></a>如何排查机器人的问题？

1. 通过 [Visual Studio Code](debug-bots-locally-vscode.md) 或 [Visual Studio](https://docs.microsoft.com/en-us/visualstudio/debugger/navigating-through-code-with-the-debugger?view=vs-2017) 调试机器人的源代码。
2. 先使用[模拟器](bot-service-debug-emulator.md)测试机器人，然后再将其部署到云。
3. 将机器人部署到类似于 Azure 的云托管平台，然后使用 <a href="https://dev.botframework.com" target="_blank">Bot Framework 门户</a>中机器人仪表板上的内置 Web 聊天控件测试与机器人的连接性。 如果在将机器人部署到 Azure 后遇到问题，可以考虑使用此博客文章：[了解 Azure 故障排除和支持](https://azure.microsoft.com/en-us/blog/understanding-azure-troubleshooting-and-support/)。
4. 排除[身份验证][TroubleshootingAuth]可能存在问题的可能性。
5. 在 Skype 上测试机器人。 这将有助于验证端到端的用户体验。
6. 考虑在具有其他身份验证要求的渠道（例如 Direct Line 或网上聊天）上测试机器人。

## <a name="how-can-i-troubleshoot-authentication-issues"></a>如何排查身份验证问题？

有关排查机器人身份验证问题的详细信息，请参阅[故障排除][TroubleshootingAuth] Bot Framework 身份验证。

## <a name="im-using-the-bot-builder-sdk-for-net-how-can-i-troubleshoot-issues-with-my-bot"></a>我使用的是 Bot Builder SDK for .NET。 如何排查机器人的问题？

**查找异常。**  
在 Visual Studio 2017 中，转到“调试” > “Windows” > “异常设置”。 在“异常设置”窗口中，选中“公共语言运行时异常”旁边的“引发时中断”复选框。 当存在引发的异常或未处理的异常时，还可能会在“输出”窗口中看到诊断输出。

**查看调用堆栈。**  
在 Visual Studio 中，可以选择是否调试[仅我的代码](https://msdn.microsoft.com/en-us/library/dn457346.aspx)。 检查完整的调用堆栈可能会对问题提供其他见解。

**确保所有对话框方法都以处理下一条消息的计划结束。**  
所有 `IDialog` 方法均应通过 `IDialogStack.Call`、`IDialogStack.Wait` 或 `IDialogStack.Done` 完成。 这些 `IDialogStack` 方法通过传递给每个 `IDialog` 方法的 `IDialogContext` 公开。 通过 `PromptDialog` 静态方法调用 `IDialogStack.Forward` 并使用系统提示将在其实现中调用其中一种方法。

**确保所有对话框均可序列化。**  
这可以像在 `IDialog` 实现上使用 `[Serializable]` 属性一样简单。 但是，请注意，如果匿名方法闭包引用其外部环境来捕获变量，则它们不可序列化。 Bot Framework 支持基于反射的序列化代理，以帮助序列化未标记为可序列化的类型。

## <a name="why-doesnt-the-typing-activity-do-anything"></a>为什么键入活动不会执行任何操作？
某些渠道不支持其客户端中的临时性键入更新。

## <a name="what-is-the-difference-between-the-connector-library-and-builder-library-in-the-sdk"></a>SDK 中的连接器库和生成器库之间的区别是什么？

连接器库是 REST API 的展示。 生成器库添加了会话对话框编程模型和其他功能，如提示、瀑布图、链和指导表单完成。 生成器库还提供对 LUIS 等认知服务的访问。

## <a name="what-causes-an-error-with-http-status-code-429-too-many-requests"></a>导致出现 HTTP 状态代码 429“请求过多”错误的原因是什么？

HTTP 状态代码 429 的错误响应表示在给定的时间内发出了太多请求。 响应的正文应包含问题的说明，也可以指定请求之间所需的最小间隔。 此错误的一个可能来源是 [ngrok](https://ngrok.com/)。 如果你处于免费计划并且遇到 ngrok 的限制，请访问其网站上的定价和限制页以获取更多[选项](https://ngrok.com/product#pricing)。 
 

## <a name="how-can-i-run-background-tasks-in-aspnet"></a>如何在 ASP.NET 中运行后台任务？ 

在某些情况下，你可能希望启动一个需要等待几秒钟的异步任务，然后执行一些代码以清除用户配置文件或重置会话/对话状态。 有关如何实现此目的的详细信息，请参阅[如何在 ASP.NET 中运行后台任务](https://www.hanselman.com/blog/HowToRunBackgroundTasksInASPNET.aspx)。 具体而言，请考虑使用[HostingEnvironment.QueueBackgroundWorkItem](https://msdn.microsoft.com/en-us/library/dn636893(v=vs.110).aspx)。 


## <a name="how-do-user-messages-relate-to-https-method-calls"></a>用户消息如何与 HTTPS 方法调用关联？

当用户通过通道发送消息时，Bot Framework Web 服务将向机器人的 Web 服务终结点发出 HTTPS POST。 机器人通过针对其发送的每条消息向 Bot Framework 发送单独的 HTTPS POST，从而在该通道上向用户发送零个、一个或多个消息。

## <a name="my-bot-is-slow-to-respond-to-the-first-message-it-receives-how-can-i-make-it-faster"></a>我的机器人对收到的第一条消息的响应速度很慢。 怎样才能加快它的响应速度？

机器人是 Web 服务和一些托管平台（包括 Azure），会在一段时间内没有收到流量时自动将服务置于休眠状态。 如果你的机器人发生这种情况，它必须在下次收到消息时重启，这会使其响应速度比其已在运行的响应速度慢得多。

借助某些托管平台，你可以配置服务，以使其不会进入休眠状态。 要在 Azure 中执行此操作，请导航到 [Azure 门户](https://portal.azure.com)中的机器人服务，选择“应用程序设置”，然后选择“Always on”。 大多数（但不是全部）服务计划都提供此选项。

## <a name="how-can-i-guarantee-message-delivery-order"></a>如何保证消息传递顺序？

Bot Framework 将尽可能地保留消息顺序。 例如，如果先发送消息 A 并等待该 HTTP 操作完成，然后再发起另一个 HTTP 操作以发送消息 B，则 Bot Framework 将自动认为消息 A 应该在消息 B 之前。但是，在一般情况下，无法保证消息传递顺序，因为通道最终负责消息传递并可能对消息重新排序。 为了降低以错误顺序传递消息的风险，可以选择在消息之间实施时间延迟。

## <a name="how-can-i-intercept-all-messages-between-the-user-and-my-bot"></a>如何截获用户和我的机器人之间的所有消息？

使用 Bot Builder SDK for .NET，可以为 `Autofac` 依赖关系注入容器提供 `IPostToBot` 和 `IBotToUser` 接口的实现。 使用 Bot Builder SDK for Node.js，可以将中间件用于相同的目的。 [BotBuilder-Azure](https://github.com/Microsoft/BotBuilder-Azure) 存储库包含将此数据记录到 Azure 表的 C# 和 Node.js 库。

## <a name="why-are-parts-of-my-message-text-being-dropped"></a>为什么我的消息文本被部分删除？

Bot Framework 和许多通道就像使用 [Markdown](https://en.wikipedia.org/wiki/Markdown) 对文本进行格式化一样来解释文本。 检查文本是否包含可能被解释为 Markdown 语法的字符。

## <a name="how-can-i-support-multiple-bots-at-the-same-bot-service-endpoint"></a>如何在同一个机器人服务终结点支持多个机器人？ 

此[示例](https://github.com/Microsoft/BotBuilder/issues/2258#issuecomment-280506334)演示了如何使用正确的 `MicrosoftAppCredentials` 配置 `Conversation.Container` 并使用简单的 `MultiCredentialProvider` 来验证多个应用 ID 和密码。

## <a name="identifiers"></a>标识符

## <a name="how-do-identifiers-work-in-the-bot-framework"></a>标识符如何在 Bot Framework 中运行？

有关 Bot Framework 中标识符的详细信息，请参阅 Bot Framework [标识符指南][BotFrameworkIDGuide]。

## <a name="how-can-i-get-access-to-the-user-id"></a>如何访问用户 ID？

短信和电子邮件消息将提供 `from.Id` 属性中的原始用户 ID。 在 Skype 消息中，`from.Id` 属性将包含用户的唯一 ID，该 ID 与用户的 Skype ID 不同。 如果需要连接到现有帐户，可以使用登录卡并实施自己的 OAuth 流，以将用户 ID 连接到自己服务的用户 ID。

## <a name="why-are-my-facebook-user-names-not-showing-anymore"></a>为什么不再显示我的 Facebook 用户名？

是否更改了 Facebook 密码？ 这样做会使访问令牌无效，将需要在 <a href="https://dev.botframework.com" target="_blank">Bot Framework 门户</a>中更新机器人对 Facebook Messenger 通道的配置设置。

## <a name="why-is-my-kik-bot-replying-im-sorry-i-cant-talk-right-now"></a>为什么我的 Kik 机器人回复“我很抱歉，我现在不能说话”？

在 Kik 上开发的机器人允许 50 个订阅者。 在 50 个不同用户与你的机器人交互后，任何尝试与你的机器人聊天的新用户都会收到消息“我很抱歉，我现在不能说话。” 有关详细信息，请参阅 [Kik 文档](https://botsupport.kik.com/hc/en-us/articles/225764648-How-can-I-share-my-bot-with-Kik-users-while-in-development-)。

## <a name="how-can-i-use-authenticated-services-from-my-bot"></a>如何通过我的机器人使用经过身份验证的服务？

有关 Azure Active Directory 身份验证，请参阅添加身份验证 [V3](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-tutorial-authentication?view=azure-bot-service-3.0&tabs=csharp) | [V4](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-tutorial-authentication?view=azure-bot-service-4.0&tabs=csharp)。 

> [!NOTE] 
> 如果向机器人添加身份验证和安全功能，则应确保在代码中实现的模式符合应用程序相应的安全标准。

## <a name="how-can-i-limit-access-to-my-bot-to-a-pre-determined-list-of-users"></a>如何仅限预先确定的用户列表访问我的机器人？

某些通道（例如 SMS 和电子邮件）提供未区分范围的地址。 在这些情况下，来自用户的消息将包含 `from.Id` 属性中的原始用户 ID。

其他通道（例如 Skype、Facebook 和 Slack）提供作用域内地址或租用地址，以防止机器人提前预测用户的 ID。 在这些情况下，需要通过登录链接或共享机密对用户进行身份验证，以确定他们是否有权使用机器人。

## <a name="why-does-my-direct-line-11-conversation-start-over-after-every-message"></a>为什么我的 Direct Line 1.1 会话会在每条消息后重新开始？

如果你的 Direct Line 会话似乎在每条消息后重新开始，则 `from` 属性可能缺失或 Direct Line 客户端发送给机器人的消息中包含 `null`。 当 Direct Line 客户端发送缺少 `from` 属性或包含 `null` 的消息时，Direct Line 服务会自动分配一个 ID，因此客户端发送的每条消息都将显示为来自不同的新用户。

若要解决此问题，请将 Direct Line 客户端发送的每条消息中的 `from` 属性设置为唯一表示发送消息的用户的稳定值。 例如，如果用户已登录到网页或应用，则可以将该现有的用户 ID 用作用户发送的消息中 `from` 属性的值。 或者，可以选择在页面加载或应用程序加载时生成随机用户 ID，将该 ID 存储在 Cookie 或设备状态中，并将该 ID 用作用户发送的消息中 `from` 属性的值。

## <a name="what-causes-the-direct-line-30-service-to-respond-with-http-status-code-502-bad-gateway"></a>导致 Direct Line 3.0 服务使用 HTTP 状态代码 502“错误的网关”进行响应的原因是什么？
当 Direct Line 3.0 尝试联系你的机器人但请求未成功完成时会返回 HTTP 状态代码 502。 此错误表示机器人返回错误或请求超时。有关机器人生成的错误的详细信息，请转到 <a href="https://dev.botframework.com" target="_blank">Bot Framework 门户</a>中的机器人仪表板，然后单击受影响通道的“问题”链接。 如果为机器人配置了 Application Insights，则还可以在那里找到详细的错误信息。 

## <a name="what-is-the-idialogstackforward-method-in-the-bot-builder-sdk-for-net"></a>Bot Builder SDK for .NET 中的 IDialogStack.Forward 方法是什么？

`IDialogStack.Forward` 的主要目的是重用一个通常为“被动”的现有子对话框，其中子对话框（在 `IDialog.StartAsync` 中）等待带有一些 `ResumeAfter` 处理程序的对象 `T`。 特别是，如果你有一个等待 `IMessageActivity` `T` 的子对话框，则可以使用 `IDialogStack.Forward` 方法转发传入的 `IMessageActivity`（已由某个父对话框接收）。 例如，要将传入的 `IMessageActivity` 转发到 `LuisDialog`，请调用 `IDialogStack.Forward` 以将 `LuisDialog` 推送到对话框堆栈，运行 `LuisDialog.StartAsync` 中的代码，直到它安排等待下一条消息，然后立即通过转发的 `IMessageActivity` 满足该等待。

`T` 通常是一个 `IMessageActivity`，因为通常会构造 `IDialog.StartAsync` 以等待该类型的活动。 在将消息转发到现有的 `LuisDialog` 之前，可以对 `LuisDialog` 使用 `IDialogStack.Forward` 作为一种机制，截获来自用户的消息进行某些处理。 或者，还可以将 `DispatchDialog` 与 `ContinueToNextGroup` 结合使用来实现此目的。

你可能希望在由 `StartAsync` 计划的第一个 `ResumeAfter` 处理程序（例如，`LuisDialog.MessageReceived`）中找到转发项。

## <a name="what-is-the-difference-between-proactive-and-reactive"></a>“主动”和“被动”之间的区别是什么？

从机器人的角度来看，“被动”意味着用户通过向机器人发送消息来启动会话，并且机器人通过响应该消息来做出反应。 相反，“主动”意味着机器人通过先向用户发送第一条消息来启动会话。 例如，机器人可以发送主动消息以在计时器过期或事件发生时通知用户。

## <a name="how-can-i-send-proactive-messages-to-the-user"></a>如何向用户发送主动消息？

有关演示如何发送主动消息的示例，请参阅 GitHub 上的 BotBuilder-Samples 存储库中的 [C# V4 示例](https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/csharp_dotnetcore/16.proactive-messages)和 [Node.js V4 示例](https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/javascript_nodejs/16.proactive-messages)。

## <a name="how-can-i-reference-non-serializable-services-from-my-c-dialogs"></a>如何从 C# 对话框引用不可序列化的服务？

可提供多种方法：

* 通过 `Autofac` 和 `FiberModule.Key_DoNotSerialize` 解决依赖关系。 这是最明确的解决方案。
* 使用 [NonSerialized](https://msdn.microsoft.com/en-us/library/system.nonserializedattribute(v=vs.110).aspx) 和 [OnDeserialized](https://msdn.microsoft.com/en-us/library/system.runtime.serialization.ondeserializedattribute(v=vs.110).aspx) 属性还原反序列化的依赖关系。 这是最简单的解决方案。
* 不要存储该依赖关系，以便不对其进行序列化。 不建议在技术上可行的情况下使用此解决方案。
* 使用反射序列化代理。 在某些情况下，此解决方案可能不可行，并且存在序列化过多的风险。

::: moniker range="azure-bot-service-3.0"

## <a name="where-is-conversation-state-stored"></a>在哪里存储会话状态？

使用 Connector 的 `IBotState` 接口存储用户、会话和私人会话属性包中的数据。 按机器人的 ID 划分每个属性包的作用范围。 用户属性包由用户 ID 键入，会话属性包由会话 ID 键入，私人会话属性包由用户 ID 和会话 ID 键入。 

如果使用 Bot Builder SDK for .NET 或 Bot Builder SDK for Node.js 来构建机器人，则对话框堆栈和对话框数据将自动存储为私人会话属性包中的条目。 C# 实现使用二进制序列化，Node.js 实现使用 JSON 序列化。

`IBotState` REST 接口由两个服务实现。

* Bot Framework Connector 提供实现此接口并在 Azure 中存储数据的云服务。  此数据进行静态加密，并且不会故意过期。
* Bot Framework Emulator 提供此接口的内存中实现，用于调试机器人。 当模拟器进程退出时，此数据将过期。

如果要将此数据存储在数据中心内，可以提供状态服务的自定义实现。 可以通过至少两种方法来实现此目的：

* 使用 REST 层来提供自定义 `IBotState` 服务。
* 使用语言（Node.js 或 C#）层中的生成器接口。

> [!IMPORTANT]
> 建议不要将 Bot Framework State Service API 用于生产环境，该 API 可能会在将来的版本中弃用。 建议更新机器人代码以使用内存中存储进行测试，或者将 Azure 扩展之一用于生产机器人。 有关详细信息，请参阅针对 [.NET](~/dotnet/bot-builder-dotnet-state.md) 或 [Node](~/nodejs/bot-builder-nodejs-state.md) 实现的“管理状态数据”主题。

::: moniker-end

## <a name="what-is-an-etag--how-does-it-relate-to-bot-data-bag-storage"></a>什么是 ETag？  它与机器人数据包存储有何关系？

[ETag](https://en.wikipedia.org/wiki/HTTP_ETag) 是一种用于[乐观并发控制](https://en.wikipedia.org/wiki/Optimistic_concurrency_control)的机制。 机器人数据包存储使用 ETag 来防止数据更新冲突。 ETag 错误并显示 HTTP 状态代码 412“不满足前提条件”指示为该机器人数据包同时执行了多个“读取-修改-写入”序列。

对话框堆栈和状态均存储在机器人数据包中。 例如，如果机器人在收到该会话的新消息时仍在处理上一条消息，则可能会看到“不满足前提条件”ETag 错误。

## <a name="what-causes-an-error-with-http-status-code-412-precondition-failed-or-http-status-code-409-conflict"></a>导致出现 HTTP 状态代码 412“不满足前提条件”或 HTTP 状态代码 409“冲突”错误的原因是什么？

Connector 的 `IBotState` 服务用于存储机器人数据包（即，用户、会话和机器人专用数据包，其中机器人专用数据包包含对话框堆栈“控制流”状态）。 `IBotState` 服务中的并发控制通过 ETag 由乐观并发进行管理。 如果在“读取-修改-写入"序列期间（由于单个机器人数据包的并发更新）存在更新冲突，则：

* 如果保留 ETag，则会从 `IBotState` 服务引发 HTTP 状态代码 412“不满足前提条件”错误。 这是 Bot Builder SDK for .NET 中的默认行为。
* 如果未保留 ETag（即，ETag 设置为 `\*`），则“最后写入为准”策略将生效，这可防止“不满足前提条件”错误，但存在导致数据丢失的风险。 这是 Bot Builder SDK for Node.js 中的默认行为。

## <a name="how-can-i-fix-precondition-failed-412-or-conflict-409-errors"></a>如何修复“不满足前提条件”（412）或“冲突”（409）错误？

这些错误表明机器人同时为同一个会话处理了多条消息。 如果机器人连接到需要精确排序消息的服务，则应考虑锁定会话状态以确保不会并行处理这些消息。 

::: moniker range="azure-bot-service-3.0"

Bot Builder SDK for .NET 提供一种机制（实现 `IScope` 的类 `LocalMutualExclusion`）以保守地使用内存中的信号量来序列化单个会话的处理。 可以扩展此实现以使用 Redis 租用，按会话地址划分其作用范围。

如果机器人未连接到外部服务，或者如果并行处理来自同一会话的消息是可接受的操作，则可以添加此代码以忽略 Bot State API 中发生的任何冲突。 这将允许最后一个回复设置会话状态。

```cs
var builder = new ContainerBuilder();
builder
    .Register(c => new CachingBotDataStore(c.Resolve<ConnectorStore>(), CachingBotDataStoreConsistencyPolicy.LastWriteWins))
    .As<IBotDataStore<BotData>>()
    .AsSelf()
    .InstancePerLifetimeScope();
builder.Update(Conversation.Container);
```
::: moniker-end

## <a name="is-there-a-limit-on-the-amount-of-data-i-can-store-using-the-state-api"></a>可以使用状态 API 存储的数据量是否有限制？

有限制，每个状态存储（即，用户、会话和机器人专用数据包）最多可包含 64 kb 的数据。 有关详细信息，请参阅[管理状态数据][StateAPI]。

::: moniker range="azure-bot-service-3.0"

## <a name="how-do-i-version-the-bot-data-stored-through-the-state-api"></a>如何对通过状态 API 存储的机器人数据进行版本控制？

> [!IMPORTANT]
> 建议不要将 Bot Framework State Service API 用于生产环境或 v4 机器人，该 API 可能会在将来的版本中完全弃用。 建议更新机器人代码以使用内存中存储进行测试，或者将 Azure 扩展之一用于生产机器人。 有关详细信息，请参阅[管理状态数据](v4sdk/bot-builder-howto-v4-state.md)主题。

借助状态服务，可以通过会话中的对话框保留进度，以便用户稍后可以返回到与机器人的会话，而不会丢失其位置。 为保留这功能，修改机器人代码时，不会自动清除通过状态 API 存储的机器人数据属性包。 应根据修改后的代码是否与旧版数据兼容来决定是否应清除机器人数据。 

* 如果要在开发机器人期间手动重置会话的对话框堆栈和状态，可以使用 ` /deleteprofile` 命令删除状态数据。 确保在此命令中包含前导空格，以防通道解释该命令。
* 将机器人部署到生产环境后，可以对机器人数据进行版本控制，以便在版本冲突时清除关联的状态数据。 对于 Bot Builder SDK for Node.js，这可以使用中间件完成，对于 Bot Builder SDK for .NET，这可以使用 `IPostToBot` 实现来完成。

> [!NOTE]
> 如果无法正确反序列化对话框堆栈，则由于序列化格式更改或因为代码更改太多，将重置会话状态。

::: moniker-end

## <a name="what-are-the-possible-machine-readable-resolutions-of-the-luis-built-in-date-time-duration-and-set-entities"></a>LUIS 内置日期、时间、持续时间和设置实体的可能的机器可读解决方案是什么？

有关示例列表，请参阅 LUIS 文档的[预建实体部分][LUISPreBuiltEntities]。

## <a name="how-can-i-use-more-than-the-maximum-number-of-luis-intents"></a>如何使用超过 LUIS 最大意向数的意向数？

可以考虑拆分模型并以串行或并行方式调用 LUIS 服务。

## <a name="how-can-i-use-more-than-one-luis-model"></a>如何使用多个 LUIS 模型？

Bot Builder SDK for Node.js 和 Bot Builder SDK for .NET 均支持从单个 LUIS 意向对话框调用多个 LUIS 模型。 请注意以下几个警告：

* 使用多个 LUIS 模型假设 LUIS 模型具有非重叠的意向集。
* 使用多个 LUIS 模型假设来自不同模型的分数是可比较的，以在多个模型中选择“最佳匹配意向”。
* 使用多个 LUIS 模型意味着，如果一个意向与一个模型匹配，它也将完全匹配其他模型的“none”意向。 在这种情况下，可以避免选择“none”意向；Bot Builder SDK for Node.js 将自动缩小“none”意向的分数以避免此问题。

## <a name="where-can-i-get-more-help-on-luis"></a>从哪里可以获得有关 LUIS 的详细帮助信息？

* [语言理解 (LUIS) 简介 - Microsoft 认知服务](https://www.youtube.com/watch?v=jWeLajon9M8)（视频）
* [语言理解 (LUIS) 的高级学习会话](https://www.youtube.com/watch?v=39L0Gv2EcSk)（视频）
* [LUIS 文档](/azure/cognitive-services/LUIS/Home)
* [语言理解论坛](https://social.msdn.microsoft.com/forums/azure/en-US/home?forum=LUIS) 


## <a name="what-are-some-community-authored-dialogs"></a>社区创作的对话框有哪些？

* [BotAuth](https://www.nuget.org/packages/BotAuth) - Azure Active Directory 身份验证
* [BestMatchDialog](http://www.garypretty.co.uk/2016/08/01/bestmatchdialog-for-microsoft-bot-framework-now-available-via-nuget/) - 用户文本到对话框方法的基于正则表达式的调度

## <a name="what-are-some-community-authored-templates"></a>社区创作的模板有哪些？

* [ES6 BotBuilder](https://github.com/brene/botbuilder-es6-template) -ES6 机器人生成器模板

## <a name="why-do-i-get-an-authorizationrequestdenied-exception-when-creating-a-bot"></a>创建机器人时，为什么会收到 Authorization_RequestDenied 异常？

创建 Azure 机器人服务机器人的权限是通过 Azure Active Directory (AAD) 门户进行管理的。 如果未在 [AAD 门户](http://aad.portal.azure.com)中正确配置权限，则在尝试创建机器人服务时，用户将收到 Authorization_RequestDenied 异常。

首先检查你是否是目录的“来宾”：

1. 登录到 [Azure 门户](http://portal.azure.com)。
2. 单击“所有服务”，然后搜索“可用”。
3. 选择“Azure Active Directory”。
4. 单击“用户”。
5. 从列表中查找用户，并确保“用户类型”不是“来宾”。

![Azure Active Directory 用户类型](~/media/azure-active-directory/user_type.png)

确认你不是“来宾”后，然后确保 Active Directory 中的用户可以创建机器人服务，目录管理员需要配置以下设置：

1. 登录到 [AAD 门户](http://aad.portal.azure.com)。 转到“用户和组”，然后选择“用户设置”。
2. 在“应用注册”部分下，将“用户可以注册应用程序”设置为“是”。 这允许目录中的用户创建机器人服务。
3. 在“外部用户”部分下，将“来宾用户权限受限”设置为“否”。 这允许目录中的来宾用户创建机器人服务。

![Azure Active Directory 管理中心](~/media/azure-active-directory/admin_center.png)

## <a name="why-cant-i-migrate-my-bot"></a>为什么不能迁移我的机器人？

如果在迁移机器人时遇到问题，可能是因为机器人属于默认目录以外的目录。 请尝试以下步骤：

1. 从目标目录添加不是默认目录成员的新用户（通过电子邮件地址），在作为迁移目标的订阅上授予用户参与者角色。

2. 从[开发人员门户](https://dev.botframework.com)，添加用户的电子邮件地址，作为应迁移的机器人的共有者。 然后注销。

3. 以新用户身份登录到[开发人员门户](https://dev.botframework.com)然后继续迁移机器人。

## <a name="where-can-i-get-more-help"></a>从哪里可以获得更多帮助？

* 利用 [Stack Overflow](https://stackoverflow.com/questions/tagged/botframework) 上以前回答过的问题中的信息，或使用 `botframework` 标记发布你自己的问题。 请注意，Stack Overflow 有一些准则，例如要求具有描述性的标题、完整而简明的问题陈述以及重现问题的足够详细信息。 功能请求或过于宽泛的问题都是偏离主题的；新用户应访问 [Stack Overflow 帮助中心](https://stackoverflow.com/help/how-to-ask)了解更多详细信息。
* 请查阅 GitHub 中的 [BotBuilder 问题](https://github.com/Microsoft/BotBuilder/issues)，了解有关机器人生成器 SDK 的已知问题的信息，或报告新问题。
* 利用 BotBuilder 社区中有关 [Gitter ](https://gitter.im/Microsoft/BotBuilder) 讨论的信息。




[LUISPreBuiltEntities]: /azure/cognitive-services/luis/pre-builtentities
[BotFrameworkIDGuide]: bot-service-resources-identifiers-guide.md
[StateAPI]: ~/rest-api/bot-framework-rest-state.md
[TroubleshootingAuth]: bot-service-troubleshoot-authentication-problems.md

