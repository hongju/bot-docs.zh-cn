---
title: 实现顺序聊天流 | Microsoft Docs
description: 了解如何在 Bot Framework SDK 中使用对话管理简单的聊天流。
keywords: 简单聊天流, 顺序聊天流, 对话, 提示, 瀑布, 对话集
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 0f29520b993d12ce01c65cd29517b3a4b2aada84
ms.sourcegitcommit: a295a90eac461f8b96770dd902ba44919acf33fc
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/26/2019
ms.locfileid: "67404557"
---
# <a name="implement-sequential-conversation-flow"></a>实现顺序聊天流

[!INCLUDE[applies-to](../includes/applies-to.md)]

通过发布问题来收集信息是机器人与用户交互的主要方式之一。 对话库提供有用的内置功能（如 prompt 类），以轻松提问和验证答复，从而确保它与特定数据类型匹配或符合自定义验证规则  。 

可以使用对话库来管理简单的和复杂的聊天流。 在简单的交互中，机器人将按固定的顺序运行一组步骤，直到聊天完成。 一般情况下，当机器人需要从用户收集信息时，对话就非常有用。 本主题详细介绍如何通过创建提示并从瀑布对话调用这些提示来实现简单的聊天流。 

> [!TIP]
> 有关如何在不使用对话库的情况下编写自己的提示的示例，请参阅[创建自己的提示来收集用户输入](bot-builder-primitive-prompts.md)一文。 


## <a name="prerequisites"></a>先决条件

- 了解[机器人基础知识][concept-basics]、, [managing state][concept-state]和[对话库][概念对话]。
-  使用 [**CSharp**][cs-sample] or [**JavaScript**][js-sample] 的多轮次提示示例的副本。

## <a name="about-this-sample"></a>关于此示例

在多轮次提示示例中，我们将使用一个瀑布对话、几个提示和一个组件对话来创建简单的交互，向用户提出一系列问题。 代码使用对话来循环执行以下步骤：

| Steps        | 提示类型  |
|:-------------|:-------------| 
| 请求用户提供其交通方式 | 选项提示 |
| 请求用户输入其姓名 | 文本提示 |
| 询问用户是否愿意提供其年龄 | 确认提示 |
| 如果他们回答“是”，则请求他们提供年龄  | 附带验证的数字提示仅接受大于 0 且小于 150 的年龄。 |
| 询问收集的信息是否“正确” | 重用确认提示 |

最后，如果用户回答“是”，则显示收集的信息；否则，告知用户他们的信息不会保留。

## <a name="create-the-main-dialog"></a>创建主对话

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

若要使用对话，请安装 **Microsoft.Bot.Builder.Dialogs** NuGet 包。

机器人通过 `UserProfileDialog` 来与用户交互。 创建机器人的 `DialogBot` 类时，会将 `UserProfileDialog` 设置为其主对话。 然后，机器人使用 `Run` 帮助器方法来访问该对话。

![用户个人资料对话](media/user-profile-dialog.png)

**Dialogs\UserProfileDialog.cs**

我们首先创建派生自 `ComponentDialog` 类的 `UserProfileDialog`，这包括 6 个步骤。

在 `UserProfileDialog` 构造函数中创建瀑布步骤、提示和瀑布对话，然后将其添加到对话集。 提示需要位于使用这些提示的同一对话集中。

[!code-csharp[Constructor snippet](~/../botbuilder-samples/samples/csharp_dotnetcore/05.multi-turn-prompt/Dialogs/UserProfileDialog.cs?range=22-41)]

接下来，实现该对话使用的步骤。 若要使用提示，请从对话中的某个步骤调用它，然后使用 `stepContext.Result` 检索下一步骤中的提示结果。 在幕后，提示是由两个步骤组成的对话。 首先，提示会请求输入；其次，它会返回有效值，或者使用重新提示从头开始，直到收到有效的输入为止。



应始终从瀑布步骤返回非 null 的 `DialogTurnResult`。 否则，对话可能不按设计意图运行。 此处演示了瀑布对话中 `NameStepAsync` 的实现。

[!code-csharp[Name step](~/../botbuilder-samples/samples/csharp_dotnetcore/05.multi-turn-prompt/Dialogs/UserProfileDialog.cs?range=56-61)]

在 `AgeStepAsync` 中，我们指定了当用户输入采用的是提示无法分析的格式，或者不符合验证条件，因而无法验证时，要使用的重试提示。 在这种情况下，如果未提供重试提示，则提示将使用初始提示文本来重新提示用户输入。

[!code-csharp[Age step](~/../botbuilder-samples/samples/csharp_dotnetcore/05.multi-turn-prompt/Dialogs/UserProfileDialog.cs?range=74-93&highlight=10)]

**UserProfile.cs**

用户的交通方式、姓名和年龄将保存在 `UserProfile` 类的实例中。

[!code-csharp[UserProfile class](~/../botbuilder-samples/samples/csharp_dotnetcore/05.multi-turn-prompt/UserProfile.cs?range=9-16)]

**Dialogs\UserProfileDialog.cs**

最后一个步骤检查前一瀑布步骤中调用的对话返回的 `stepContext.Result`。 如果返回值为 true，则我们将使用用户个人资料访问器来获取和更新用户个人资料。 为了获取用户个人资料，我们将调用 `GetAsync` 方法，然后设置 `userProfile.Transport`、`userProfile.Name` 和 `userProfile.Age` 属性的值。 最后，在调用用于结束对话的 `EndDialogAsync` 之前汇总用户的信息。 结束对话会从对话堆栈中弹出该对话，并将可选结果返回到该对话的父级。 父级是启动了刚刚结束的对话的对话或方法。

[!code-csharp[SummaryStepAsync](~/../botbuilder-samples/samples/csharp_dotnetcore/05.multi-turn-prompt/Dialogs/UserProfileDialog.cs?range=108-134&highlight=5-10,25-26)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

若要使用对话，项目需要安装 **botbuilder-dialogs** npm 包。

机器人通过 `UserProfileDialog` 来与用户交互。 创建机器人的 `DialogBot` 时，会将 `UserProfileDialog` 设置为其主对话。 然后，机器人使用 `run` 帮助器方法来访问该对话。

![用户个人资料对话](media/user-profile-dialog-js.png)

**dialogs\userProfileDialog.js**

我们首先创建派生自 `ComponentDialog` 类的 `UserProfileDialog`，这包括 6 个步骤。

在 `UserProfileDialog` 构造函数中创建瀑布步骤、提示和瀑布对话，然后将其添加到对话集。 提示需要位于使用这些提示的同一对话集中。

[!code-javascript[Constructor snippet](~/../botbuilder-samples/samples/javascript_nodejs/05.multi-turn-prompt/dialogs/userProfileDialog.js?range=25-47)]

接下来，实现该对话使用的步骤。 若要使用提示，请从对话中的某个步骤调用它，然后从步骤上下文检索下一步骤中的提示结果（本例使用 `step.result`）。 在幕后，提示是由两个步骤组成的对话。 首先，提示会请求输入；其次，它会返回有效值，或者使用重新提示从头开始，直到收到有效的输入为止。

应始终从瀑布步骤返回非 null 的 `DialogTurnResult`。 否则，对话可能不按设计意图运行。 此处演示了瀑布对话中 `nameStep` 的实现。

[!code-javascript[name step](~/../botbuilder-samples/samples/javascript_nodejs/05.multi-turn-prompt/dialogs/userProfileDialog.js?range=75-78)]

在 `ageStep` 中，我们指定了当用户输入采用的是提示无法分析的格式，或者不符合上述构造函数中指定的验证条件，因而无法验证时，要使用的重试提示。 在这种情况下，如果未提供重试提示，则提示将使用初始提示文本来重新提示用户输入。

[!code-javascript[age step](~/../botbuilder-samples/samples/javascript_nodejs/05.multi-turn-prompt/dialogs/userProfileDialog.js?range=90-101&highlight=5)]

**userProfile.js**

用户的交通方式、姓名和年龄将保存在 `UserProfile` 类的实例中。

[!code-javascript[user profile](~/../botbuilder-samples/samples/javascript_nodejs/05.multi-turn-prompt/userProfile.js?range=4-10)]

**Dialogs\UserProfileDialog.cs**

最后一个步骤检查前一瀑布步骤中调用的对话返回的 `step.result`。 如果返回值为 true，则我们将使用用户个人资料访问器来获取和更新用户个人资料。 为了获取用户个人资料，我们将调用 `get` 方法，然后设置 `userProfile.transport`、`userProfile.name` 和 `userProfile.age` 属性的值。 最后，在调用用于结束对话的 `endDialog` 之前汇总用户的信息。 结束对话会从对话堆栈中弹出该对话，并将可选结果返回到该对话的父级。 父级是启动了刚刚结束的对话的对话或方法。

[!code-javascript[summary step](~/../botbuilder-samples/samples/javascript_nodejs/05.multi-turn-prompt/dialogs/userProfileDialog.js?range=115-136&highlight=4-8,20-21)]

---

## <a name="create-the-extension-method-to-run-the-waterfall-dialog"></a>创建用于运行瀑布对话的扩展方法

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

我们定义了一个 `Run` 扩展方法用于创建和访问对话上下文。 此处的 `accessor` 是对话状态属性的状态属性访问器，`dialog` 是用户个人资料组件对话。 由于组件对话定义内部对话集，因此我们必须创建可让消息处理程序代码看到的外部对话集，并使用它来创建对话上下文。

对话上下文是通过调用 `CreateContext` 方法创建的，用于从机器人轮次处理程序内部来与对话集交互。 对话上下文包含当前轮次上下文、父对话和对话状态。对话状态提供一种方法用于在对话中保留信息。

借助对话上下文可以使用对话的字符串 ID 来启动该对话，或继续当前的对话（例如，包含多个步骤的瀑布对话）。 对话上下文将传递到机器人的所有对话和瀑布步骤。

**DialogExtensions.cs**

[!code-csharp[Run method](~/../botbuilder-samples/samples/csharp_dotnetcore/05.multi-turn-prompt/DialogExtensions.cs?range=13-24)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

我们在 `userProfileDialog` 中定义了一个 `run` 帮助器方法用于创建和访问对话上下文。 此处的 `accessor` 是对话状态属性的状态属性访问器，`this` 是用户个人资料组件对话。 由于组件对话定义内部对话集，因此我们必须创建可让消息处理程序代码看到的外部对话集，并使用它来创建对话上下文。

对话上下文是通过调用 `createContext` 方法创建的，用于从机器人轮次处理程序内部来与对话集交互。 对话上下文包含当前轮次上下文、父对话和对话状态。对话状态提供一种方法用于在对话中保留信息。

借助对话上下文可以使用对话的字符串 ID 来启动该对话，或继续当前的对话（例如，包含多个步骤的瀑布对话）。 对话上下文将传递到机器人的所有对话和瀑布步骤。

[!code-javascript[run method](~/../botbuilder-samples/samples/javascript_nodejs/05.multi-turn-prompt/dialogs/userProfileDialog.js?range=55-64)]

---

## <a name="run-the-dialog"></a>运行对话

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Bots\DialogBot.cs**

`OnMessageActivityAsync` 处理程序使用扩展方法来启动或继续对话。 在 `OnTurnAsync` 中，我们使用了机器人的状态管理对象将所有状态更改保存到存储中。 （`ActivityHandler.OnTurnAsync` 方法调用各种活动处理程序方法，例如 `OnMessageActivityAsync`。 这样，在消息处理程序完成之后、轮次本身完成之前，我们将会保存状态。）

[!code-csharp[overrides](~/../botbuilder-samples/samples/csharp_dotnetcore/05.multi-turn-prompt/Bots/DialogBot.cs?range=33-48&highlight=5-7)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

`onMessage` 处理程序使用帮助器方法来启动或继续对话。 在 `onDialog` 中，我们使用了机器人的状态管理对象将所有状态更改保存到存储中。 （在运行其他定义的处理程序（例如 `onMessage`）之后，最后会调用 `onDialog` 方法。 这样，在消息处理程序完成之后、轮次本身完成之前，我们将会保存状态。）

[!code-javascript[overrides](~/../botbuilder-samples/samples/javascript_nodejs/05.multi-turn-prompt/bots/dialogBot.js?range=30-44)]

---

## <a name="register-services-for-the-bot"></a>注册机器人的服务

此机器人使用以下服务。 

- 机器人的基本服务：凭据提供程序、适配器和机器人实现。
- 用于管理状态的服务：存储、用户状态和聊天状态。
- 机器人使用的对话。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Startup.cs**

在 `Startup` 中注册机器人的服务。 可以通过依赖项注入将这些服务用于其他代码部分。

[!code-csharp[ConfigureServices](~/../botbuilder-samples/samples/csharp_dotnetcore/05.multi-turn-prompt/Startup.cs?range=17-41)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**index.js**

在 `index.js` 中注册机器人的服务。 

[!code-javascript[overrides](~/../botbuilder-samples/samples/javascript_nodejs/05.multi-turn-prompt/index.js?range=18-49)]

---

> [!NOTE]
> 内存存储仅用于测试，不用于生产。
> 请务必对生产用机器人使用持久型存储。

## <a name="to-test-the-bot"></a>测试机器人

1. 安装 [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme)（如果尚未安装）。
1. 在计算机本地运行示例。
1. 按如下所示启动模拟器，连接到机器人，然后发送消息。

![多轮次提示对话的示例运行](../media/emulator-v4/multi-turn-prompt.png)

## <a name="additional-information"></a>其他信息

### <a name="about-dialog-and-bot-state"></a>关于对话和机器人状态

在此机器人中，我们定义了两个状态属性访问器：

- 一个访问器是在对话状态属性的聊天状态中创建的。 对话状态跟踪用户在对话集的对话中所处的位置，由对话上下文更新，例如，当我们调用 begin dialog 或 continue dialog 方法时，就会更新对话状态。
- 一个访问器是在用户配置文件属性的用户状态中创建的。 机器人使用此访问器来跟踪有关用户的信息，我们将在对话代码中显式管理此状态。

状态属性访问器的 _get_ 和 _set_ 方法在状态管理对象的缓存中获取和设置属性值。 首次在某个轮次中请求状态属性的值时，将填充该缓存，但是，必须显式持久保存该值。 为了持久保存对这两个状态属性所做的更改，我们将调用相应状态管理对象的 _save changes_ 方法。

此示例从对话内部更新用户配置文件状态。 这种做法适用于简单的机器人；如果你想要在多个机器人中重复使用某个对话，则此做法不适用。

有多种选项可将对话步骤与机器人状态相分离。 例如，在对话收集完整信息后，你可以：

- 使用 end dialog 方法将收集的数据作为返回值返回给父上下文。 此上下文可能是机器人的轮次处理程序，或对话堆栈中以前的某个活动对话。 这就是提示类的设计方式。
- 向相应的服务生成请求。 如果机器人充当较大服务的前端，此选项可能很适合。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [向机器人添加自然语言理解](bot-builder-howto-v4-luis.md)

<!-- Footnote-style links -->

[concept-basics]: bot-builder-basics.md
[concept-state]: bot-builder-concept-state.md
[concept-dialogs]: bot-builder-concept-dialog.md

[prompting]: bot-builder-prompts.md
[component-dialogs]: bot-builder-compositcontrol.md

[cs-sample]: https://aka.ms/cs-multi-prompts-sample
[js-sample]: https://aka.ms/js-multi-prompts-sample
