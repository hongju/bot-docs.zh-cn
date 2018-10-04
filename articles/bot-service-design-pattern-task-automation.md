---
title: 创建任务自动化机器人 | Microsoft Docs
description: 了解如何设计无需进一步人工干预即可执行任务的机器人。
author: matvelloso
ms.author: mateusv
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 2/13/2018
ms.openlocfilehash: 3bf6bef805e4a86b6e070693660eb5cb20468ffd
ms.sourcegitcommit: f0b22c6286e44578c11c9f15d22b542c199f0024
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/27/2018
ms.locfileid: "47404003"
---
# <a name="create-task-automation-bots"></a>创建任务自动化机器人

[!INCLUDE [pre-release-label](./includes/pre-release-label-v3.md)]

任务自动化机器人使用户能在无任何人工帮助的情况下，完成特定任务或一组任务。 通常，这种类型的机器人非常类似于典型的应用或网站，主要通过大量的用户控件和文本与用户进行通信。 它可能具有自然语言理解功能，可与用户进行更多聊天。 

## <a name="example-use-case-password-reset"></a>示例用例：密码重置

为更好地理解任务机器人的本质，请考虑示例用例：密码重置。 

Contoso 公司每天都会有多个员工因需要重置密码而拨打支持人员电话。 Contoso 希望能自动完成重置员工密码这项简单的重复性任务，以便支持人员可将时间用于解决更复杂的问题。 

Contoso 公司经验丰富的开发人员 John 决定创建一个机器人来自动执行密码重置任务。 他首先为机器人编写了设计规范，就像创建新应用或网站一样。 

::: moniker range="azure-bot-service-3.0"

### <a name="navigation-model"></a>导航模型

规范定义了导航模型：

![对话结构](~/media/bot-service-design-pattern-task-automation/simple-task1.png)

用户从 `RootDialog` 开始。 当他们请求重置密码时，  
他们将被定向到 `ResetPasswordDialog`。 通过 `ResetPasswordDialog`，机器人将提示用户输入两条信息：电话号码和出生日期。 

> [!IMPORTANT]
> 本文中所述的机器人设计仅作为示例之用。 在实际情况下，密码重置机器人可能会实施更可靠的身份验证过程。

### <a name="dialogs"></a>对话

接下来，此规范描述了每个对话的外观和功能。 

#### <a name="root-dialog"></a>根对话

根对话为用户提供了两个选项： 

1. “更改密码”适用于用户知道其当前密码而只想更改密码的情况。
2. “重置密码”适用于用户忘记或输错密码而需要生成新密码的情况。

> [!NOTE]
> 为简单起见，本文仅介绍“重置密码”流程。

该规范描述了根对话，如以下屏幕截图所示。

![对话结构](~/media/bot-service-design-pattern-task-automation/simple-task2.png)

#### <a name="resetpassword-dialog"></a>“ResetPassword”对话

用户从根对话中选择“重置密码”时，将调用 `ResetPassword` 对话。 
然后，`ResetPassword` 对话将调用另外两个对话。 
首先，它调用 `PromptStringRegex` 对话，收集用户的电话号码。 
然后，它调用 `PromptDate` 对话，收集用户的出生日期。 

> [!NOTE]
> 在此示例中，John 选择使用两个单独的对话来实现收集用户电话号码和出生日期的逻辑。 此方法不仅简化了每个对话所需的代码，而且还增加了将来其他方案可使用这些对话的几率。 

该规范描述了 `ResetPassword` 对话。

![对话结构](~/media/bot-service-design-pattern-task-automation/simple-task3.png)

#### <a name="promptstringregex-dialog"></a>“PromptStringRegex”对话

`PromptStringRegex` 对话提示用户输入其电话号码，并验证用户提供的电话号码是否符合预期格式。 
它还考虑了用户重复提供无效输入的情况。 
该规范描述了 `PromptStringRegex` 对话。

![对话结构](~/media/bot-service-design-pattern-task-automation/simple-task4.png)

### <a name="prototype"></a>原型

最后，该规范提供了用户与机器人通信以成功完成密码重置任务的示例。

![对话结构](~/media/bot-service-design-pattern-task-automation/simple-task5.png)

::: moniker-end 

## <a name="bot-app-or-website"></a>机器人、应用或网站？

你可能想知道，如果任务自动化机器人与应用或网站非常类似，为什么不直接构建一个应用或网站呢？ 根据具体情况，构建应用或网站而非机器人可能是一个完全合理的选择。 甚至可选择使用 [Bot Framework Direct Line API][directLineAPI] 或<a href="https://aka.ms/BotFramework-WebChat" target="_blank">网上聊天控件</a>将机器人嵌入到应用中。 在应用的上下文中实现机器人堪称两全其美：集丰富的应用体验和聊天式体验于一身。 

然而，许多情况下，构建应用或网站可能比构建机器人更为复杂和昂贵。 应用或网站通常需要支持多个客户端和平台，打包和部署是一个繁琐且耗时的过程，并且必须下载和安装应用的用户体验不一定是理想之选。 出于这些原因，机器人通常可以提供一种更简单的方法来解决目前的问题。 

此外，机器人可轻松自由扩展。 例如，开发人员可选择向密码重置机器人添加自然语言和语音功能，以便可通过音频呼叫访问机器人，或者可以添加短信支持功能。 该公司可能会在整个建筑中设置网亭，并将密码重置机器人嵌入到该体验中。

::: moniker range="azure-bot-service-3.0"
<!-- TODO: SimpleTaskAutomation no longer exists
## Sample code

For a complete sample that shows how to implement simple task automation using the Bot Builder SDK for .NET, see the <a href="https://aka.ms/capability-SimpleTaskAutomation" target="_blank">Simple Task Automation sample</a> in GitHub.

For a complete sample that shows how to implement simple task automation using the Bot Builder SDK for Node.js, see the <a href="https://aka.ms/capability-SimpleTaskAutomation" target="_blank">Simple Task Automation sample</a> in GitHub.
-->

## <a name="additional-resources"></a>其他资源

- [对话框](~/dotnet/bot-builder-dotnet-dialogs.md)
- [使用对话管理聊天流 (.NET)](~/dotnet/bot-builder-dotnet-manage-conversation-flow.md)
- [使用对话管理聊天流 (Node.js)](~/nodejs/bot-builder-nodejs-manage-conversation-flow.md)

::: moniker-end

[directLineAPI]: https://docs.botframework.com/en-us/restapi/directline3/#navtitle
