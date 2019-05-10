---
title: 处理用户中断 | Microsoft Docs
description: 了解如何处理用户中断和直接聊天流。
keywords: 中断, 切换主题, 中断
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 04/18/2019
ms.reviewer: ''
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: bd8682966dbb2e33a536a72a4016ef23e9c1fc75
ms.sourcegitcommit: f84b56beecd41debe6baf056e98332f20b646bda
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/03/2019
ms.locfileid: "65032616"
---
# <a name="handle-user-interruptions"></a>处理用户中断

[!INCLUDE[applies-to](../includes/applies-to.md)]

如何处理中断是判断机器人是否可靠的一个重要因素。 用户不总是按照定义的聊天流逐步操作。 他们可能会在流程的中途提出问题，或者只是想要取消流程，而不要完成流程。 本主题将探讨一些在机器人中处理用户中断的常用方法。

## <a name="prerequisites"></a>先决条件

- 了解[机器人基础知识][concept-basics]、[管理状态][concept-state]、[对话库][concept-dialogs]，以及如何[重用对话][component-dialogs]。
- 以 [**CSharp**][cs-sample] 或 [**JavaScript**][js-sample] 编写的核心机器人示例副本。

## <a name="about-this-sample"></a>关于此示例

本文中使用的示例为某个航班预订机器人建模，该机器人使用对话从用户获取航班信息。 在与机器人聊天过程中的任何时候，用户都可以发出 _help_ 或 _cancel_ 命令来造成中断。 本文将处理两种类型的中断：

- **轮次级别**：绕过轮次级别的处理，但在堆栈中保留包含所提供信息的对话。 下一轮次将从上次离开的位置继续。 
- **对话级别**：完全取消处理，使机器人能够从头开始。

## <a name="define-and-implement-the-interruption-logic"></a>定义和实现中断逻辑

首先定义并实现 _help_ 和 _cancel_ 中断。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

若要使用对话，请安装 **Microsoft.Bot.Builder.Dialogs** NuGet 包。

**Dialogs\CancelAndHelpDialog.cs**

我们首先实现 `CancelAndHelpDialog` 类来处理用户中断。

[!code-csharp[Class signature](~/../botbuilder-samples/samples/csharp_dotnetcore/13.core-bot/Dialogs/CancelAndHelpDialog.cs?range=11)]

在 `CancelAndHelpDialog` 类中，`OnBeginDialogAsync` 和 `OnContinueDialogAsync` 方法调用 `InerruptAsync` 方法来检查用户是否中断了正常的流。 如果该流已中断，则调用基类方法；否则返回 `InterruptAsync` 中的返回值。

[!code-csharp[Overrides](~/../botbuilder-samples/samples/csharp_dotnetcore/13.core-bot/Dialogs/CancelAndHelpDialog.cs?range=18-27)]

如果用户键入“help”，`InterrupAsync` 方法将发送一条消息，然后调用 `DialogTurnResult (DialogTurnStatus.Waiting)` 来指示最前面的对话正在等待用户的响应。 这样只会中断某个轮次的聊天流，在下一轮次，我们将从上次离开的位置继续。

如果用户键入“cancel”，该方法将对其内部对话上下文调用 `CancelAllDialogsAsync`，这会清除其对话堆栈，导致堆栈退出并返回 cancelled（已取消）状态，但不返回结果值。 `MainDialog`（稍后会介绍）将显示预订对话已结束并返回了 null，就如同用户选择不确认其预订一样。

[!code-csharp[Interrupt](~/../botbuilder-samples/samples/csharp_dotnetcore/13.core-bot/Dialogs/CancelAndHelpDialog.cs?range=40-61&highlight=11-12,16-17)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

若要使用对话，请安装 **botbuilder-dialogs** npm 包。

**dialogs/cancelAndHelpDialog.js**

我们首先实现 `CancelAndHelpDialog` 类来处理用户中断。

[!code-javascript[Class signature](~/../botbuilder-samples/samples/javascript_nodejs/13.core-bot/dialogs/cancelAndHelpDialog.js?range=10)]

在 `CancelAndHelpDialog` 类中，`onBeginDialog` 和 `onContinueDialog` 方法调用 `interrupt` 方法来检查用户是否中断了正常的流。 如果该流已中断，则调用基类方法；否则返回 `interrupt` 中的返回值。

[!code-javascript[Overrides](~/../botbuilder-samples/samples/javascript_nodejs/13.core-bot/dialogs/cancelAndHelpDialog.js?range=11-25)]

如果用户键入“help”，`interrupt` 方法将发送一条消息，然后返回一个 `{ status: DialogTurnStatus.waiting }` 对象来指示最前面的对话正在等待用户的响应。 这样只会中断某个轮次的聊天流，在下一轮次，我们将从上次离开的位置继续。

如果用户键入“cancel”，该方法将对其内部对话上下文调用 `cancelAllDialogs`，这会清除其对话堆栈，导致堆栈退出并返回 cancelled（已取消）状态，但不返回结果值。 `MainDialog`（稍后会介绍）将显示预订对话已结束并返回了 null，就如同用户选择不确认其预订一样。

[!code-javascript[Interrupt](~/../botbuilder-samples/samples/javascript_nodejs/13.core-bot/dialogs/cancelAndHelpDialog.js?range=27-40&highlight=7-8,11-12)]

---

## <a name="check-for-interruptions-each-turn"></a>检查每个轮次的中断

前面已介绍中断处理类的工作原理，接下来让我们回过头来了解一下当机器人收到用户的新消息时会发生什么情况。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Dialogs\MainDialog.cs**

当新的消息活动抵达时，机器人将运行 `MainDialog`。 `MainDialog` 提示用户需要哪种帮助。 然后，机器人将通过调用 `BeginDialogAsync` 来启动 `MainDialog.ActStepAsync` 方法中的 `BookingDialog`，如下所示。

[!code-csharp[ActStepAsync](~/../botbuilder-samples/samples/csharp_dotnetcore/13.core-bot/Dialogs/MainDialog.cs?range=54-68&highlight=13-14)]

接下来，在 `MainDialog` 类的 `FinalStepAsync` 方法中，预订对话将会结束，而预订被视为已完成或已取消。

[!code-csharp[FinalStepAsync](~/../botbuilder-samples/samples/csharp_dotnetcore/13.core-bot/Dialogs/MainDialog.cs?range=70-91)]

此处显示 `BookingDialog` 中的代码，因为这些代码不直接与中断处理相关。 它们用于提示用户输入预订详细信息。 可以在 **Dialogs\BookingDialogs.cs** 中找到这些代码。

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**dialogs/mainDialog.js**

当新的消息活动抵达时，机器人将运行 `MainDialog`。 `MainDialog` 提示用户需要哪种帮助。 然后，机器人将通过调用 `beginDialog` 来启动 `MainDialog.actStep` 方法中的 `bookingDialog`，如下所示。

[!code-javascript[Act step](~/../botbuilder-samples/samples/javascript_nodejs/13.core-bot/dialogs/mainDialog.js?range=71-88&highlight=16-17)]

接下来，在 `MainDialog` 类的 `finalStep` 方法中，预订对话将会结束，而预订被视为已完成或已取消。

[!code-javascript[Final step](~/../botbuilder-samples/samples/javascript_nodejs/13.core-bot/dialogs/mainDialog.js?range=93-110)]

此处显示 `BookingDialog` 中的代码，因为这些代码不直接与中断处理相关。 它们用于提示用户输入预订详细信息。 可以在 **dialogs/bookingDialogs.js** 中找到这些代码。

---

## <a name="handle-unexpected-errors"></a>处理意外错误

接下来，我们将处理可能发生的任何未经处理的异常。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**AdapterWithErrorHandler.cs**

在本示例中，适配器的 `OnTurnError` 处理程序接收机器人的轮次逻辑引发的任何异常。 如果引发了异常，该处理程序将删除当前聊天的聊天状态，以防止错误状态导致机器人陷入错误循环。

[!code-csharp[AdapterWithErrorHandler](~/../botbuilder-samples/samples/csharp_dotnetcore/13.core-bot/AdapterWithErrorHandler.cs?range=12-41)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**index.js**

在本示例中，适配器的 `onTurnError` 处理程序接收机器人的轮次逻辑引发的任何异常。 如果引发了异常，该处理程序将删除当前聊天的聊天状态，以防止错误状态导致机器人陷入错误循环。

[!code-javascript[AdapterWithErrorHandler](~/../botbuilder-samples/samples/javascript_nodejs/13.core-bot/index.js?range=28-38)]

---

## <a name="register-services"></a>注册服务

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Startup.cs**

最后，在 `Startup.cs` 中创建一个暂时性的机器人，并在每个轮次创建该机器人的新实例。

[!code-csharp[Add transient bot](~/../botbuilder-samples/samples/csharp_dotnetcore/13.core-bot/Startup.cs?range=46-47)]

下面提供了在用于创建上述机器人的调用中使用的类定义供参考。

[!code-csharp[MainDialog signature](~/../botbuilder-samples/samples/csharp_dotnetcore/13.core-bot/Dialogs/MainDialog.cs?range=15)]
[!code-csharp[DialogAndWelcomeBot signature](~/../botbuilder-samples/samples/csharp_dotnetcore/13.core-bot/Bots/DialogAndWelcomeBot.cs?range=16)]
[!code-csharp[DialogBot signature](~/../botbuilder-samples/samples/csharp_dotnetcore/13.core-bot/Bots/DialogBot.cs?range=18)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**index.js**

最后，在 `index.js` 中创建机器人。

[!code-javascript[Create bot](~/../botbuilder-samples/samples/javascript_nodejs/13.core-bot/index.js?range=55-56)]

下面提供了在用于创建上述机器人的调用中使用的类定义供参考。

[!code-javascript[MainDialog signature](~/../botbuilder-samples/samples/javascript_nodejs/13.core-bot/dialogs/mainDialog.js?range=12)]
[!code-javascript[DialogAndWelcomeBot signature](~/../botbuilder-samples/samples/javascript_nodejs/13.core-bot/bots/dialogAndWelcomeBot.js?range=8)]
[!code-javascript[DialogBot signature](~/../botbuilder-samples/samples/javascript_nodejs/13.core-bot/bots/dialogBot.js?range=6)]

---

## <a name="to-test-the-bot"></a>测试机器人

1. 安装 [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme)（如果尚未安装）。
1. 在计算机本地运行示例。
1. 按如下所示启动仿真器，连接到机器人，然后发送消息。

<!--![test dialog prompt sample](~/media/emulator-v4/test-dialog-prompt.png)-->

## <a name="additional-information"></a>其他信息

- [身份验证示例](https://aka.ms/logout)演示了如何处理注销，其中使用了此处所示的类似模式来处理中断。

- 应该发送默认响应，而非不采取任何措施，导致用户对当前情况感到困惑。 默认响应告诉用户机器人理解哪些命令，以便用户可以重新进行对话。

- 在轮次中的任意时间点，轮次上下文的 _responded_ 属性都会指示机器人在此轮次中是否向用户发送了消息。 在轮次结束之前，机器人应会向用户发送某条消息，即使该消息只是确认收到了用户的输入。

<!-- Footnote-style links -->

[concept-basics]: bot-builder-basics.md
[concept-state]: bot-builder-concept-state.md
[concept-dialogs]: bot-builder-concept-dialog.md

[using-luis]: bot-builder-howto-v4-luis.md
[using-qna]: bot-builder-howto-qna.md

[simple-flow]: bot-builder-dialog-manage-conversation-flow.md
[prompting]: bot-builder-prompts.md
[component-dialogs]: bot-builder-compositcontrol.md

[cs-sample]: https://aka.ms/cs-core-sample
[js-sample]: https://aka.ms/js-core-sample
