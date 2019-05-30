---
title: 实现问候对话 | Microsoft Docs
description: 使用对话在用户加入聊天时问候用户。
keywords: 问候, 对话, 聊天流, 对话集
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 82e6273b8d6dc984e29bef891f3e8f67b1c53eed
ms.sourcegitcommit: ea64a56acfabc6a9c1576ebf9f17ac81e7e2a6b7
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/24/2019
ms.locfileid: "66215414"
---
# <a name="implement-a-greeting-dialog"></a>实现问候对话

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

可以使用对话来欢迎用户加入聊天。

若要详细了解如何欢迎用户，请参阅如何[向用户发送欢迎消息][send-welcome]。

## <a name="prerequisites"></a>先决条件

- 了解[管理状态][concept-state]、[对话库][concept-dialogs]、如何[管理聊天][simple-flow]，以及如何[使用对话提示收集用户输入][prompting]。
- ??? 示例副本，以 [**CSharp**][cs-sample] 或 [**JavaScript**][js-sample] 编写。

## <a name="task-as-in-to-do-x-do-these-things"></a>\<任务> [执行这些事项，就像在待办事项 X 中操作那样]

<!--The key lines of code for this task.
    here are the cool lines that do that.
    just the few lines of implementation without setup.
-->

## <a name="about-the-sample-code"></a>关于示例代码

<!--setup & implementation & discussion of the sample code-->

- 需要的其他包（AI.Luis、对话等）

<!--Any other key elements to get the code to work.
    Include setup for only the bits critical to the task at hand.
    don't go over all the code in the sample.
-->

## <a name="to-test-the-bot"></a>测试机器人

1. 安装 [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme)（如果尚未安装）。
1. 在计算机本地运行示例。
1. 按如下所示启动模拟器，连接到机器人，然后发送消息。

TODO：获取新的屏幕截图。

<!--![test dialog prompt sample](~/media/emulator-v4/test-dialog-prompt.png)-->

## <a name="discussion-optional"></a>讨论 [可选]

<!--Might be short and descriptive or include additional code for scenarios not covered in the samples repo
-->

## <a name="addition-information"></a>其他信息

<!--include cross-linking other articles about the same sample.-->

- 整体示例的“关于核心机器人”登陆页的链接（适用于 CoreBot 派生的文章）。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [处理用户中断](bot-builder-howto-handle-user-interrupt.md)

<!-- Footnote-style links -->

[concept-basics]: bot-builder-basics.md
[concept-state]: bot-builder-concept-state.md
[concept-dialogs]: bot-builder-concept-dialog.md

[send-welcome]: bot-builder-send-welcome-message.md

[simple-flow]: bot-builder-dialog-manage-conversation-flow.md
[prompting]: bot-builder-prompts.md
[component-dialogs]: bot-builder-compositcontrol.md

[cs-sample]: ???
[js-sample]: ???
