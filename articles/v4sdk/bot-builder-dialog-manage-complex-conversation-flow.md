---
title: 使用分支和循环创建高级聊天流 | Microsoft Docs
description: 了解如何在 Bot Framework SDK 中使用对话管理复杂的聊天流。
keywords: 复杂的聊天流, 重复, 循环, 菜单, 对话, 提示, 瀑布, 对话集
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 07/05/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: b7ffa16c2f0a00043b12faec1d31bbfe5bfa250f
ms.sourcegitcommit: b498649da0b44f073dc5b23c9011ea2831edb31e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/05/2019
ms.locfileid: "67587477"
---
# <a name="create-advanced-conversation-flow-using-branches-and-loops"></a>使用分支和循环创建高级聊天流

[!INCLUDE[applies-to](../includes/applies-to.md)]

可以使用对话库来管理简单的和复杂的聊天流。
本文介绍如何管理可分支和循环的复杂聊天。
此外，介绍如何在对话的不同部分之间传递参数。

## <a name="prerequisites"></a>先决条件

- 了解[机器人基础知识][concept-basics], [managing state][concept-state]、[对话库][concept-dialogs]以及如何[实现有序聊天流][simple-dialog]。
- 以 [**CSharp**][cs-sample] or [**JavaScript**][js-sample] 编写的复杂对话示例副本。

## <a name="about-this-sample"></a>关于此示例

本示例演示一个可以注册用户，让其针对列表中的最多两家公司发表评论的机器人。

`DialogAndWelcomeBot` 扩展了 `DialogBot`，后者定义不同活动的处理程序以及机器人的轮次处理程序。 `DialogBot` 运行对话：

- _run_ 方法供 `DialogBot` 用来启动对话。
- `MainDialog` 是另两个对话的父项，这两个对话在对话的特定时间调用。 本文自始至终都会提供这些对话的详细信息。

这些对话可拆分成 `MainDialog`、`TopLevelDialog` 和 `ReviewSelectionDialog` 组件对话，这些组件对话共同完成以下操作：

- 它们会要求提供用户的姓名和年龄，然后根据用户的年龄来_分支_。
  - 如果用户太年轻，它们不会要求用户对任何公司进行评论。
  - 如果用户够年龄，它们就会开始收集用户的评论首选项。
    - 它们允许用户选择要评论的公司。
    - 如果用户选择了一家公司，它们会通过循环方式让用户选择第二家公司。 
- 最后，它们会感谢用户的参与。

它们使用瀑布对话和一些提示来管理复杂的聊天。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

![复杂的机器流](./media/complex-conversation-flow.png)

若要使用对话，项目需安装 **Microsoft.Bot.Builder.Dialogs** NuGet 包。

**Startup.cs**

在 `Startup` 中注册机器人的服务。 可以通过依赖项注入将这些服务用于其他代码部分。

- 机器人的基本服务：凭据提供程序、适配器和机器人实现。
- 用于管理状态的服务：存储、用户状态和聊天状态。
- 机器人使用的对话。

[!code-csharp[ConfigureServices](~/../botbuilder-samples/samples/csharp_dotnetcore/43.complex-dialog/Startup.cs?range=22-39)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

![复杂的机器流](./media/complex-conversation-flow-js.png)

若要使用对话，项目需要安装 **botbuilder-dialogs** npm 包。

**index.js**

我们为机器人创建代码其他部分需要的服务。

- 机器人的基本服务：适配器和机器人实现。
- 用于管理状态的服务：存储、用户状态和聊天状态。
- 机器人使用的对话。

[!code-javascript[ConfigureServices](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/index.js?range=25-38)]
[!code-javascript[ConfigureServices](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/index.js?range=43-45)]

---

> [!NOTE]
> 内存存储仅用于测试，不用于生产。
> 请务必对生产用机器人使用持久型存储。

## <a name="define-a-class-in-which-to-store-the-collected-information"></a>定义一个类，用于在其中存储收集的信息

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**UserProfile.cs**

[!code-csharp[UserProfile class](~/../botbuilder-samples/samples/csharp_dotnetcore/43.complex-dialog/UserProfile.cs?range=8-16)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**userProfile.js**

[!code-javascript[UserProfile class](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/userProfile.js?range=4-12)]

---

## <a name="create-the-dialogs-to-use"></a>创建要使用的对话

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Dialogs\MainDialog.cs**

我们定义了组件对话 `MainDialog`，该对话包含一些主要步骤，可以引导对话和提示。 初始步骤调用 `TopLevelDialog`，解释如下。

[!code-csharp[step implementations](~/../botbuilder-samples/samples/csharp_dotnetcore/43.complex-dialog/Dialogs/MainDialog.cs?range=31-50&highlight=3)]

**Dialogs\TopLevelDialog.cs**

初始的 top-level 对话包含四个步骤：

1. 请求用户的姓名。
1. 请求用户的年龄。
1. 根据用户的年龄分支。
1. 最后，感谢用户参与并返回收集的信息。

在第一步中，我们将清除用户的配置文件，这样对话每次启动时都可以使用空配置文件。 由于上一步在结束时会返回信息，因此 `AcknowledgementStepAsync` 在结束时会将其保存到用户状态，然后将该信息返回到主对话，以便在最后一步使用。

[!code-csharp[step implementations](~/../botbuilder-samples/samples/csharp_dotnetcore/43.complex-dialog/Dialogs/TopLevelDialog.cs?range=39-96&highlight=3-4,47-49,56-57)]

**Dialogs\ReviewSelectionDialog.cs**

review-selection 对话从 top-level 对话的 `StartSelectionStepAsync` 启动，包含两个步骤：

1. 请求用户选择要评论的公司，或选择 `done` 以完成操作。
1. 根据情况重复此对话或退出。

在此设计中，top-level 对话始终优先于堆栈中的 review-selection 对话，可将 review-selection 对话视为 top-level 对话的子级。

[!code-csharp[step implementations](~/../botbuilder-samples/samples/csharp_dotnetcore/43.complex-dialog/Dialogs/ReviewSelectionDialog.cs?range=42-106)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**dialogs/mainDialog.js**

我们定义了组件对话 `MainDialog`，该对话包含一些主要步骤，可以引导对话和提示。 初始步骤调用 `TopLevelDialog`，解释如下。

[!code-javascript[step implementations](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/dialogs/mainDialog.js?range=43-55&highlight=2)]

**dialogs/topLevelDialog.js**

初始的 top-level 对话包含四个步骤：

1. 请求用户的姓名。
1. 请求用户的年龄。
1. 根据用户的年龄分支。
1. 最后，感谢用户参与并返回收集的信息。

在第一步中，我们将清除用户的配置文件，这样对话每次启动时都可以使用空配置文件。 由于上一步在结束时会返回信息，因此 `acknowledgementStep` 在结束时会将其保存到用户状态，然后将该信息返回到主对话，以便在最后一步使用。

[!code-javascript[step implementations](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/dialogs/topLevelDialog.js?range=32-76&highlight=2-3,37-39,43-44)]

**dialogs/reviewSelectionDialog.js**

review-selection 对话从 top-level 对话的 `startSelectionStep` 启动，包含两个步骤：

1. 请求用户选择要评论的公司，或选择 `done` 以完成操作。
1. 根据情况重复此对话或退出。

在此设计中，top-level 对话始终优先于堆栈中的 review-selection 对话，可将 review-selection 对话视为 top-level 对话的子级。

[!code-javascript[step implementations](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/dialogs/reviewSelectionDialog.js?range=33-78)]

---

## <a name="implement-the-code-to-manage-the-dialog"></a>实现用于管理对话的代码

机器人的轮次处理程序重复这些对话定义的一个聊天流。
收到来自用户的消息时：

1. 继续活动的对话（如果有）。
   - 如果没有任何对话处于活动状态，则清除用户个人资料并启动 top-level 对话。
   - 如果活动的对话已完成，则收集并保存返回的信息，然后显示状态消息。
   - 否则，活动的对话仍在处理中途，我们暂时不需要执行其他任何操作。
1. 保存聊天状态，以持久保存对对话状态所做的任何更新。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

<!-- **DialogExtensions.cs**

In this sample, we've defined a `Run` helper method that we will use to create and access the dialog context.
Since component dialog defines an inner dialog set, we have to create an outer dialog set that's visible to the message handler code, and use that to create a dialog context.

- `dialog` is the main component dialog for the bot.
- `turnContext` is the current turn context for the bot.

[!code-csharp[Run method](~/../botbuilder-samples/samples/csharp_dotnetcore/43.complex-dialog/DialogExtensions.cs?range=13-24)]

-->

**Bots\DialogBot.cs**

消息处理程序调用 `RunAsync` 方法来管理对话，而我们已重写轮次处理程序，可以保存在轮次中对聊天和用户状态所做的任何更改。 基 `OnTurnAsync` 会调用 `OnMessageActivityAsync` 方法，确保在该轮次结束时进行保存调用。

[!code-csharp[Overrides](~/../botbuilder-samples/samples/csharp_dotnetcore/43.complex-dialog/Bots/DialogBot.cs?range=33-48&highlight=5-7)]

**Bots\DialogAndWelcome.cs**

`DialogAndWelcomeBot` 会扩展上面的 `DialogBot`，以便在用户加入聊天时提供欢迎消息，是 `Startup.cs` 调用的内容。

[!code-csharp[On members added](~/../botbuilder-samples/samples/csharp_dotnetcore/43.complex-dialog/Bots/DialogAndWelcome.cs?range=21-38)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**dialogs/mainDialog.js**

在此示例中，我们定义了一个 `run` 方法，用于创建和访问对话上下文。
由于组件对话定义内部对话集，因此我们必须创建可让消息处理程序代码看到的外部对话集，并使用它来创建对话上下文。

- `turnContext` 是机器人的当前轮次上下文。
- `accessor` 是我们创建的一个访问器，用于管理对话状态。

[!code-javascript[run method](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/dialogs/mainDialog.js?range=32-41)]

**bots/dialogBot.js**

消息处理程序调用 `run` 帮助程序方法来管理对话，而我们会实现一个轮次处理程序，可以保存在轮次中对聊天和用户状态所做的任何更改。 调用 `next` 时，会让基实现调用 `onDialog` 方法，确保在该轮次结束时进行保存调用。

[!code-javascript[Overrides](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/bots/dialogBot.js?range=30-47)]

**bots/dialogAndWelcomeBot.js**

`DialogAndWelcomeBot` 会扩展上面的 `DialogBot`，以便在用户加入聊天时提供欢迎消息，是 `Startup.cs` 调用的内容。

[!code-javascript[On members added](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/bots/dialogAndWelcomeBot.js?range=10-21)]

---

## <a name="branch-and-loop"></a>分支和循环

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Dialogs\TopLevelDialog.cs**

这是一个示例分支逻辑，来自 _top level_ 对话中的一个步骤：

[!code-csharp[branching logic](~/../botbuilder-samples/samples/csharp_dotnetcore/43.complex-dialog/Dialogs/TopLevelDialog.cs?range=68-80)]

**Dialogs\ReviewSelectionDialog.cs**

这是一个示例循环逻辑，来自 _review selection_ 对话中的一个步骤：

[!code-csharp[looping logic](~/../botbuilder-samples/samples/csharp_dotnetcore/43.complex-dialog/Dialogs/ReviewSelectionDialog.cs?range=96-105)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**dialogs/topLevelDialog.js**

这是一个示例分支逻辑，来自 _top level_ 对话中的一个步骤：

[!code-javascript[branching logic](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/dialogs/topLevelDialog.js?range=56-64)]

**dialogs/reviewSelectionDialog.js**

这是一个示例循环逻辑，来自 _review selection_ 对话中的一个步骤：

[!code-javascript[looping logic](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/dialogs/reviewSelectionDialog.js?range=71-77)]

---

## <a name="to-test-the-bot"></a>测试机器人

1. 安装 [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme)（如果尚未安装）。
1. 在计算机本地运行示例。
1. 按如下所示启动模拟器，连接到机器人，然后发送消息。

![测试复杂对话示例](~/media/emulator-v4/test-complex-dialog.png)

## <a name="additional-resources"></a>其他资源

有关如何实现对话的介绍，请参阅[实现有序的聊天流][simple-dialog]，其中使用了单个瀑布对话和一些提示来创建简单的交互，以便向用户提出一系列问题。

“对话”库包含提示的基本验证。 你也可以添加自定义验证。 有关详细信息，请参阅[使用对话提示收集用户输入][dialog-prompts]。

若要简化对话代码并将其重复用于多个机器人，可将对话集的某些部分定义为单独的类。
有关详细信息，请参阅[重复使用对话][component-dialogs]。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [重复使用对话](bot-builder-compositcontrol.md)

<!-- Footnote-style links -->

[concept-basics]: bot-builder-basics.md
[concept-state]: bot-builder-concept-state.md
[concept-dialogs]: bot-builder-concept-dialog.md

[simple-dialog]: bot-builder-dialog-manage-conversation-flow.md
[dialog-prompts]: bot-builder-prompts.md
[component-dialogs]: bot-builder-compositcontrol.md

[cs-sample]: https://aka.ms/cs-complex-dialog-sample
[js-sample]: https://aka.ms/js-complex-dialog-sample
