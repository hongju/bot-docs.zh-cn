---
title: 重复使用对话 | Microsoft Docs
description: 了解如何使用组件对话在 Bot Framework SDK 中模块化机器人逻辑。
keywords: 复合控件, 模块化机器人逻辑
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 07/05/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 77f1c154af5821b1e476546f307a01be27f568c0
ms.sourcegitcommit: b498649da0b44f073dc5b23c9011ea2831edb31e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/05/2019
ms.locfileid: "67587494"
---
# <a name="reuse-dialogs"></a>重复使用对话

[!INCLUDE[applies-to](../includes/applies-to.md)]

可以通过组件对话创建独立的对话来处理特定的方案，将大型对话集分解成更易于管理的片段。 其中的每个片段有自身的对话集，可避免与其外的对话集发生名称冲突。

## <a name="prerequisites"></a>先决条件

- 了解[机器人基础知识][concept-basics], the [dialogs library][concept-dialogs]，以及如何[管理聊天][simple-flow]。
- 使用 [**CSharp**][cs-sample] or [**JavaScript**][js-sample] 的多轮次提示示例的副本。

## <a name="about-the-sample"></a>关于本示例

在多轮次提示示例中，我们将使用一个瀑布对话、几个提示和一个组件对话来创建简单的交互，向用户提出一系列问题。 代码使用对话来循环执行以下步骤：

| Steps        | 提示类型  |
|:-------------|:-------------|
| 请求用户提供其交通方式 | 选项提示 |
| 请求用户输入其姓名 | 文本提示 |
| 询问用户是否愿意提供其年龄 | 确认提示 |
| 如果他们回答“是”，则请求他们提供年龄  | 附带验证的数字提示仅接受大于 0 且小于 150 的年龄。 |
| 询问收集的信息是否“正确” | 重用确认提示 |

最后，如果用户回答“是”，则显示收集的信息；否则，告知用户他们的信息不会保留。

## <a name="implement-the-component-dialog"></a>实现组件对话

在多轮次提示示例中，我们将使用一个瀑布对话、几个提示和一个组件对话来创建简单的交互，向用户提出一系列问题。   

组件对话可封装一个或多个对话。 组件对话有一个内部对话集，而添加到此内部对话集的对话和提示有其自己的 ID，这些 ID 只能在组件对话内部查看。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

若要使用对话，请安装 **Microsoft.Bot.Builder.Dialogs** NuGet 包。

**Dialogs\UserProfileDialog.cs**

在这里，`UserProfileDialog` 类派生自 `ComponentDialog` 类。

[!code-csharp[Class](~/../botbuilder-samples/samples/csharp_dotnetcore/05.multi-turn-prompt/Dialogs/UserProfileDialog.cs?range=13)]

在构造函数中，`AddDialog` 方法会将对话和提示添加到组件对话。 使用此方法添加的第一个项将设置为初始对话，但你可以显式设置 `InitialDialogId` 属性，对其进行更改。 启动组件对话框时，它会启动其 _initial dialog_。

[!code-csharp[Constructor](~/../botbuilder-samples/samples/csharp_dotnetcore/05.multi-turn-prompt/Dialogs/UserProfileDialog.cs?range=17-42)]

这是实现瀑布对话的第一步。

[!code-csharp[First step](~/../botbuilder-samples/samples/csharp_dotnetcore/05.multi-turn-prompt/Dialogs/UserProfileDialog.cs?range=44-54)]

若要详细了解如何实现瀑布对话，请参阅如何[实现顺序聊天流](bot-builder-dialog-manage-complex-conversation-flow.md)。

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

若要使用对话，项目需要安装 **botbuilder-dialogs** npm 包。

**dialogs/userProfileDialog.js**

在这里，`UserProfileDialog` 类扩展了 `ComponentDialog`。

[!code-javascript[Class](~/../botbuilder-samples/samples/javascript_nodejs/05.multi-turn-prompt/dialogs/userProfileDialog.js?range=24)]

在构造函数中，`AddDialog` 方法会将对话和提示添加到组件对话。 使用此方法添加的第一个项将设置为初始对话，但你可以显式设置 `InitialDialogId` 属性，对其进行更改。 启动组件对话框时，它会启动其 _initial dialog_。

[!code-javascript[Constructor](~/../botbuilder-samples/samples/javascript_nodejs/05.multi-turn-prompt/dialogs/userProfileDialog.js?range=25-47)]

这是实现瀑布对话的第一步。

[!code-javascript[First step](~/../botbuilder-samples/samples/javascript_nodejs/05.multi-turn-prompt/dialogs/userProfileDialog.js?range=66-73)]

若要详细了解如何实现瀑布对话，请参阅如何[实现顺序聊天流](bot-builder-dialog-manage-complex-conversation-flow.md)。

---

在运行时，组件对话保留其自己的对话堆栈。 启动组件对话时：

- 会创建一个实例并将其添加到外部对话堆栈
- 它会创建一个内部对话堆栈并将其添加到状态中
- 它会启动初始对话并将其添加到内部对话堆栈。

在父上下文中，看起来组件是活动对话。 在组件内部，看起来初始对话是活动对话。

### <a name="implement-the-rest-of-the-dialog-and-add-it-to-the-bot"></a>实现对话的余下部分，然后将其添加到机器人

在外部对话集（需向其添加组件对话）中，组件对话有一个用于创建对话的 ID。 在外部集中，组件看起来像单个对话，与提示特别相似。

若要使用组件对话，请将其实例添加到机器人的对话集 - 这是外部对话集。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Bots\DialoBot.cs**

在该示例中，这是使用从机器人的 `OnMessageActivityAsync` 方法调用的 `RunAsync` 方法完成的。

[!code-csharp[OnMessageActivityAsync](~/../botbuilder-samples/samples/csharp_dotnetcore/05.multi-turn-prompt/Bots/DialogBot.cs?range=42-48)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**dialogs/userProfileDialog.js**

在示例中，我们向用户配置文件对话添加了 `run` 方法。

[!code-javascript[run method](~/../botbuilder-samples/samples/javascript_nodejs/05.multi-turn-prompt/dialogs/userProfileDialog.js?range=55-64)]

**bots/dialogBot.js**

`run` 方法从机器人的 `onMessage` 方法中调用。

[!code-javascript[onMessage](~/../botbuilder-samples/samples/javascript_nodejs/05.multi-turn-prompt/bots/dialogBot.js?range=30-37)]

---

## <a name="to-test-the-bot"></a>测试机器人

1. 安装 [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme)（如果尚未安装）。
1. 在计算机本地运行示例。
1. 按如下所示启动模拟器，连接到机器人，然后发送消息。

![多轮次提示对话的示例运行](../media/emulator-v4/multi-turn-prompt.png)

## <a name="additional-information"></a>其他信息

### <a name="how-cancellation-works-for-component-dialogs"></a>如何针对组件对话执行取消操作

如果从组件对话的上下文调用“取消所有对话”，  组件对话会取消其内部堆栈上的所有对话，然后结束，将控制返回给外部堆栈上的下一对话。

如果从外部上下文调用“取消所有对话”，则会取消该组件以及外部上下文中的其余对话  。

在机器人中管理嵌套式组件对话时，请注意这一点。

## <a name="next-steps"></a>后续步骤

可以增强机器人，以针对附加的输入（例如，可能会中断正常聊天流的“help”或“cancel”）做出反应。

> [!div class="nextstepaction"]
> [处理用户中断](bot-builder-howto-handle-user-interrupt.md)

<!-- Footnote-style links -->

[concept-basics]: bot-builder-basics.md
[concept-state]: bot-builder-concept-state.md
[concept-dialogs]: bot-builder-concept-dialog.md

[simple-flow]: bot-builder-dialog-manage-conversation-flow.md
[prompting]: bot-builder-prompts.md
[component-dialogs]: bot-builder-compositcontrol.md

[cs-sample]: https://aka.ms/cs-multi-prompts-sample
[js-sample]: https://aka.ms/js-multi-prompts-sample
