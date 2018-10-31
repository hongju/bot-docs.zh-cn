---
title: Bot Builder SDK 中的对话框 | Microsoft Docs
description: 介绍什么是对话框，以及如何在 Bot Builder SDK 中使用对话框。
keywords: 聊天流, 识别意向, 单轮次, 多轮次, 机器人聊天, 对话框, 提示, 瀑布, 对话框集
author: johnataylor
ms.author: johtaylo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 9/22/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 45bca42ddce527826d2723bc9a20a3c3e6c5aebe
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998604"
---
# <a name="dialogs-library"></a>对话框库

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

在 SDK 中，用于管理聊天的核心概念是“对话框”。 对话框对象处理入站活动并生成出站响应。 机器人的业务逻辑在对话框类中直接或间接运行。

对话框实例在运行时以堆栈的形式排列。 堆栈顶部的对话框称为 ActiveDialog。 当前的活动对话框处理入站活动。 在每轮聊天（没有时限，可能持续数天以上）之间，会保留堆栈。 

对话框实现三个主要函数：
- BeginDialog
- ContinueDialog
- ResumeDialog

在运行时，可以配合使用 Dialogs 和 DialogContext 类来选择适用于处理活动的对话框。 DialogContext 类会将保留的对话框堆栈、入站活动和 DialogSet 类绑定在一起。 DialogSet 包含机器人可以调用的对话框。

DialogContext 的接口反映对话框的开始和继续这样的基础概念。 应用程序的常规模式始终是首先调用 ContinueDialog。 如果没有堆栈，因此也没有 ActiveDialog，应用程序就会在 DialogContext 上调用 BeginDialog，启动所选的对话框。 这会导致相应的对话框条目从 DialogSet 推送到堆栈（严格说来，添加到堆栈中的是对话框的 ID），然后会在特定的对话框对象上委托对 BeginDialog 的调用。 如果已经有 ActiveDialog，则会在处理过程中直接委托对该对话框的 ContinueDialog 的调用，为该对话框提供任何关联的保留属性。

请注意，**对话框的 BeginDialog** 是初始化代码，采用初始化属性（在代码中称为“options”）；**对话框的 ContinueDialog** 是运行起来就可以在暂留后的活动到达后继续执行的代码。 例如，假设一个对话框询问用户一个问题，则该问题会在 BeginDialog 中提出，回答会在 ContinueDialog 中提供。

若要支持对话框嵌套（对话框包含子对话框），可以使用另一类型的继续，即恢复。 当子对话框完成后，DialogContext 会在父对话框上调用 ResumeDialog 方法。

提示和瀑布框都是 SDK 提供的对话框的具体示例。 许多方案是通过将这些抽象的东西组合在一起来生成的，但实际上，执行的逻辑自始至终都没有变，这就是此处介绍的继续和恢复模式。 从头开始实现一个对话框类是相对高级的主题，此处不做介绍，但你可以查看[示例](https://github.com/Microsoft/BotBuilder-samples)中提供的示例。

Bot Builder SDK 中的“对话框”库包含内置功能（例如提示、瀑布对话框、组件对话框），有助于管理机器人的聊天。 可以使用提示来要求用户提供不同类型的信息，使用瀑布框将多个步骤组合到一个序列中，使用组件对话框将对话框逻辑打包到单独的类中，然后将这些类集成到其他机器人中。
## <a name="waterfall-dialogs-and-prompts"></a>瀑布对话框和提示

**对话框**库提供一组提示类型，这些类型可以用来收集不同类型的用户输入。 例如，要求用户输入文本，可以使用 **TextPrompt**；要求用户输入数字，可以使用 **NumberPrompt**；要求用户输入日期和时间，可以使用 **DateTimePrompt**。 提示是特定类型的对话框。 若要使用瀑布对话框中的提示，请将瀑布对话框和提示都添加到同一对话框集。 

考虑到提示-响应交互的性质，实现一个提示至少需要在瀑布对话框中执行两个步骤 - 一个步骤用于发送提示，另一个步骤用于捕获和处理响应。  如果有其他提示，有时可以使用单个函数先处理用户的响应，然后启动下一提示，将这些操作组合在一起。

`WaterfallDialog` 是特定的对话框实现，用于从用户那里收集信息或指导用户完成一系列任务。 任务作为一组函数来实现，第一个函数的结果作为参数传递到下一个函数，依此类推。 每个函数通常表示整个过程中的一步。 在每个步骤中，机器人会提示用户输入，等待响应，然后将结果传递到下一步。 

提示和瀑布框均属对话框，如以下类层次结构所示。 

![对话框类](media/bot-builder-dialog-classes.png)

瀑布对话框由一系列瀑布步骤组成。 每个步骤都是一个异步委托，该委托采用瀑布步骤上下文 (`step`) 参数。 此模式是：瀑布步骤中的最后一项操作不是启动子对话框（通常为提示），就是终止瀑布框本身。 下图显示一系列瀑布步骤以及发生的堆栈操作。

![对话框概念](media/bot-builder-dialog-concept.png)

可以在对话框的瀑布步骤中处理对话框的返回值，也可以通过机器人的轮次处理程序来处理。
在瀑布步骤中，对话框在瀑布步骤上下文的 _result_ 属性中提供返回值。
通常只需通过机器人的轮次逻辑检查对话框轮次结果的状态。

## <a name="repeating-a-dialog"></a>重复某个对话框

若要重复某个对话框，请使用 *replace dialog* 方法。 对话框上下文的 *replace dialog* 方法会将当前对话框从堆栈中弹出，并将用于替换的对话框推送到堆栈顶层，然后启动该对话框。 可以使用此方法并将某个对话替换为其自身，来创建循环。 请注意，如需暂存当前对话框的内部状态，则需在调用 _replace dialog_ 方法时将信息传递给对话框的新实例，然后适当地初始化对话框。 传递到新对话框中的选项可以在对话框的任何步骤中通过步骤上下文的 _options_ 属性进行访问。 这是处理复杂聊天流或管理菜单的极佳方法。

## <a name="branch-a-conversation"></a>将聊天分支

对话上下文维护对话堆栈，对于堆栈中的每个对话，将跟踪下一个步骤是什么。 它的 _begin dialog_ 方法将对话框推送到堆栈的顶层，_end dialog_ 方法从堆栈中弹出顶层对话框。

一个对话框可以调用对话框上下文的 _begin dialog_ 方法并提供新对话框的 ID，以便启动同一对话框集中的新对话框，随后使新对话框成为当前的活动对话框。 原始对话框仍保留在堆栈中，但对对话框上下文的 _continue dialog_ 方法的调用只会发送到位于堆栈顶层的对话框，即“活动对话框”。 将某个对话框从堆栈中弹出后，对话框上下文会在堆栈中弹出原始对话框的位置处继续执行瀑布框的下一步。

因此，可以在一个对话框中包含一个步骤（该步骤可按条件选择一个对话框来启动一组可用对话框），以便在聊天流中创建分支。

## <a name="component-dialog"></a>组件对话框
有时需编写一个可重用的对话框，在不同的场景中使用。 例如，编写一个地址对话框，要求用户提供邮政编码、城市和街道的值。 

ComponentDialog 可以进行某种程度的隔离，因为它有单独的 DialogSet。 有单独的 DialogSet 意味着，它可以避免与包含对话框的父项发生名称冲突，可以通过创建自己的 DialogContext 来创建自己的独立的内部对话框运行时，还可以将活动调度到该运行时。 这个辅助调度意味着，它有机会截获活动。 这特别适用于需要实现“帮助”和“取消”之类功能的情况。  请参阅企业机器人模板示例。 

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [使用对话库收集用户输入](bot-builder-prompts.md)
