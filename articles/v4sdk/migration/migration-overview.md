---
title: Bot Framework SDK 迁移概述 | Microsoft Docs
description: 概述对 SDK 所做的变更以及如何从 v3 迁移到 v4。
keywords: 机器人迁移
author: mmiele
ms.author: v-mimiel
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 06/11/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: b1082e16933da1fb4c20f51d4764ec1774aabdb6
ms.sourcegitcommit: 4f78e68507fa3594971bfcbb13231c5bfd2ba555
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/17/2019
ms.locfileid: "68292187"
---
# <a name="migration-overview"></a>迁移概述

Bot Framework SDK v4 基于客户的反馈以及旧版 SDK 的学习体验而构建。 新版本引入了适当的抽象级别，并允许用户使用机器人组件构建灵活的体系结构。 这样会给用户带来很大方便，例如，用户可以先创建一个简单的机器人，然后利用 Bot Framework SDK v4 的模块化和可扩展性来增强该机器人的复杂性。

> [!NOTE]
> Bot Framework SDK v4 的目标是使简单的事保持简单，使复杂的事成为可能。

我们采取的是开放式方法，因此 Bot Framework v4 SDK 的构建离不开社区的合作。 当你首次提交拉取请求时，[参与者许可协议](https://cla.opensource.microsoft.com/) (CLA) 会自动确定你是否需要许可证。 只需对所有存储库执行一次这样的操作。 通常情况下，建立一系列需要达成的目标需要有一个时间间隔。

## <a name="what-happens-to-bots-built-using-sdk-v3"></a>使用 SDK v3 构建的机器人会出现什么情况

Bot Framework SDK v3 将停用，但是现有的 V3 机器人工作负荷可以继续无中断地运行。 有关详细信息，请参阅：[Bot Framework SDK 版本 3 生存期支持](https://docs.microsoft.com/en-us/azure/bot-service/bot-service-resources-bot-framework-faq?view=azure-bot-service-4.0#bot-framework-sdk-version-3-lifetime-support)。

强烈建议你开始将 V3 机器人迁移到 V4。 为了支持该迁移，我们生成了相关的文档，并为用户通过标准通道实施迁移计划提供扩展支持。

## <a name="advantages"></a>优点

- 更丰富的体系结构，既灵活又开放：启用更灵活的聊天设计
- 可用性：引入其他支持新通道功能的方案
- 在开发周期中扩充了行业专家 (SME) 人员：新的 GUI 设计器允许非开发人员在聊天设计阶段进行协作
- 开发速度：新的调试和测试开发人员工具
- 性能见解：新的遥测功能，用于评估和改进聊天质量
- 智能：改进了认知服务功能

## <a name="why-migrate"></a>为何要迁移
<!-- [!] The declarative model introduced with Adaptive Dialogs would go great here (when ready).  -->
- 灵活的经改进的聊天管理
  - 用于处理活动的机器人适配器
  - 重构了状态管理
  - 新的对话库
  - 适用于可组合和可扩展设计的中间件：干净且一致的挂钩，用于自定义行为
- 为 .NET Core 而构建
  - 提高了性能
  - 跨平台兼容性 (Windows/Mac/Linux)
- 一致的编程模型，支持多种编程语言
- 改进的文档
- Bot Inspector 提供扩展的调试功能
- 虚拟助手
  - 一种全面的解决方案，通过基本聊天意向、Dispatch 集成、QnA Maker、Application Insights 和自动化部署简化了机器人的创建。
  - 可扩展的技能。 通过将可重用的聊天功能（称为技能）拼接到一起来构造聊天体验。
- 测试框架：现成的测试功能，提供新的独立于传输的适配器体系结构
- 遥测：使用 Conversational AI Analytics 获取机器人运行状况和行为的关键见解
- 即将推出（预览版）
  - 自适应对话：生成在聊天过程中可以动态变化的聊天
  - 语言生成：定义某个短语的多个变体
- 未来
  - 声明性设计允许适用于设计器的抽象级别
  - GUI 对话设计器
- Azure 机器人服务 
  - Direct Line 语音通道。 将 Bot Framework 和 Microsoft 的语音服务组合在一起。 这样就提供了一个通道，用于在客户端与机器人应用程序之间双向流式传输语音和文本

## <a name="whats-changed"></a>发生的更改

Bot Framework SDK v4 支持的底层 Bot Framework 服务与 v3 相同。 但是，v4 对以前的 SDK 作了重构，这样用户就可以在创建机器人方面拥有更高的灵活性和控制度。 其中包括：

- 引入了机器人适配器
  - 该适配器是活动处理堆栈的一部分
  - 它处理 Bot Framework 身份验证并初始化每个轮次的上下文
  - 它管理通道与机器人轮次处理程序之间的传入和传出流量，封装对 Bot Framework Connector 服务的调用
  - 如需更多详细信息，请参阅[机器人的工作原理](../bot-builder-basics.md)
- 重构了状态管理
  - 机器人中不再自动提供状态数据
  - 现在需要通过状态管理对象和属性访问器管理状态
  - 如需更多详细信息，请参阅[管理状态](../bot-builder-concept-state.md)
- 引入了新的对话库
  - 需要根据新的对话库重新编写 v3 对话
  - 如需更多详细信息，请参阅[对话库](../bot-builder-concept-dialog.md)

## <a name="whats-involved-in-migration-work"></a>迁移工作涉及到哪些东西

- 更新安装逻辑
- 移植任何关键的用户状态
  - 注意：敏感的用户状态不能保留在机器人状态中，只能存储在由你控制的单独存储中
- 移植机器人和对话逻辑（如需更多详细信息，请参阅特定于语言的主题）

### <a name="migration-estimation-worksheet"></a>迁移估算工作表

可以根据以下工作表来估算迁移工作负荷。 在“发生次数”列中，请将“计数”   替换为你的实际数字值。 在“T 恤”列中输入如下值：  小、中、大（根据估算）。   

步骤 | V3 | V4 | 出现次数 | 复杂性 | T 恤
-- | -- | -- | -- | -- | --
获取传入活动 | IDialogContext.Activity | ITurnContext.Activity | 计数 | 小型  
创建活动并将其发送给用户 | activity.CreateReply(“text”) IDialogContext.PostAsync | MessageFactory.Text(“text”) ITurnContext.SendActivityAsync | 计数 | 小型 |
状态管理 | UserData、ConversationData 和 PrivateConversationData context.UserData.SetValue context.UserData.TryGetValue botDataStore.LoadAsyn | UserState、ConversationState 和 PrivateConversationState  使用属性访问器 | context.UserData.SetValue - count context.UserData.TryGetValue - count botDataStore.LoadAsyn - count | 中到大（请参阅提供的[用户状态管理](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-concept-state?view=azure-bot-service-4.0#state-management)） |
处理对话的启动 | 实现 IDialog.StartAsync | 将此项设为瀑布对话的第一个步骤。 | 计数 | 小型 |  
发送活动 | IDialogContext.PostAsync | 调用 ITurnContext.SendActivityAsync。 | 计数 | 小型 |  
等待用户的响应 | 使用 IAwaitable<IMessageActivity>参数并调用 IDialogContext.Wait | Return await ITurnContext.PromptAsync 以开始提示对话。 然后在瀑布的下一步骤中检索结果。 | 计数 | 中（取决于流） |  
处理对话的持续 | IDialogContext.Wait | 将附加的步骤添加到瀑布对话，或实现 Dialog.ContinueDialogAsync | 计数 | 大型 |  
指示结束处理，直到用户发出下一条消息 | IDialogContext.Wait | 返回 Dialog.EndOfTurn。 | 计数 | 中型 |  
开始子对话 | IDialogContext.Call | Return await 步骤上下文的 BeginDialogAsync 方法。 如果子对话返回了值，将通过步骤上下文的 Result 属性在瀑布的下一步骤中提供该值。 | 计数 | 中型 |  
将当前对话替换为新对话 | IDialogContext.Forward | Return await ITurnContext.ReplaceDialogAsync。 | 计数 | 大型 |  
表示当前对话已完成的信号 | IDialogContext.Done | Return await 步骤上下文的 EndDialogAsync 方法。 | 计数 | 中型 |  
从对话中失败退出。 | IDialogContext.Fail | 引发一个可在机器人的另一级别捕获的异常，结束步骤并返回“已取消”状态，或者调用步骤或对话上下文的 CancelAllDialogsAsync。 | 计数 | 小型 |  

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

Bot Framework SDK v4 与 v3 基于相同的基础 REST API。 但是，v4 对以前的 SDK 版本作了重构，这样用户就可以在机器人操作方面拥有更高的灵活性和控制度。

建议迁移到 .NET Core，因为这样可以大大提高性能。 但是，某些现有的 V3 机器人使用的外部库没有 .NET Core 等效项。 在这种情况下，可以将 Bot Framework SDK v4 与 .NET Framework 4.6.1 或更高版本配合使用。 可以在 [corebot](https://github.com/microsoft/BotBuilder-Samples/tree/master/samples/csharp_webapi) 位置找到一个示例。

将项目从 v3 迁移到 v4 时，可以选择以下选项之一：针对 **.NET Framework** 就地进行转换，或者移植到一个适用于 **.NET Core** 的新项目。

#### <a name="net-framework"></a>.NET framework

- 更新和安装 NuGet 包
- 更新 Global.asax.cs 文件
- 更新 MessagesController 类
- 转换对话

有关详细信息，请参阅[将 .NET v3 机器人迁移到 .NET Framework v4 机器人](conversion-framework.md)。

#### <a name="net-core"></a>.NET Core

- 使用模板创建新项目
- 根据需要安装附加 NuGet 包
- 个性化机器人，更新 Startup.cs 文件，并更新控制器类
- 更新机器人类
- 复制并更新对话和模型

有关详细信息，请参阅[将 .NET v3 机器人迁移到 .NET Core v4 机器人](conversion-core.md)。

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**Bot Framework JavaScript SDK v4** 引入了多项有关如何创作机器人的基本变更。 这些变更影响在 Javascript 中开发机器人的语法，尤其是在创建机器人对象、定义对话和编码事件处理逻辑方面发生了变化。 Bot Framework SDK v4 与 v3 基于相同的基础 REST API。 但是，v4 对以前的 SDK 版本作了重构，这样用户就可以在机器人操作方面拥有更高的灵活性和控制度，尤其是以下方面：

- 对话和机器人实例已经进一步分离。 在 v3 中，对话直接注册到机器人的构造函数。 在 v4 中，现在可以将对话作为参数传递到机器人实例中，以便更灵活地进行组合
- v4 提供了一个 `ActivityHandler` 类，用于自动处理不同类型的活动，例如消息、聊天更新和事件活动
- 进行 NodeJS 机器人迁移时，需采用 TypeScript 或 JavaScript 创建新的 v4 NodeJS 机器人。 使用在相关迁移文档中介绍的全新 v4 构造来重新创建机器人逻辑

#### <a name="migrate-from-v3-to-v4"></a>从 v3 迁移到 v4

- 创建新项目并添加依赖项
- 更新入口点并定义常量
- 创建对话，使用 SDK v4 重新实现它们
- 更新运行对话所需的机器人代码
- 移植 `store.js` 实用程序文件

有关详细信息，请参阅[将 SDK v3 Javascript 机器人迁移到 v4](conversion-javascript.md)。

---

## <a name="additional-resources"></a>其他资源

下面的其他资源提供了可以在迁移过程中使用的更多信息。  

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

<!-- _Mini-TOC with explainer for .NET topics_ -->
以下主题介绍了 .NET v3 和 v4 Bot Framework SDK 之间的差异、这两个版本之间的主要变更，以及将机器人从 v3 迁移到 v4 的步骤。

| 主题 | 说明 |
| :--- | :--- |
| [v3 和 v4 .NET SDK 之间的差异](migration-about.md) |v3 和 v4 SDK 之间的常见差异 |
| [.NET 迁移快速参考](net-migration-quickreference.md) |v3 和 v4 SDK 中的主要变更 |
| [将 .NET v3 机器人迁移到 Framework v4 机器人](conversion-framework.md) |使用同一项目类型将 v3 机器人迁移到 v4 机器人 |
| [将 .NET v3 机器人迁移到 Core v4 机器人](conversion-core.md) | 在新的 .NET Core 项目中将 v3 机器人迁移到 v4 机器人|

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

<!-- _Mini-TOC with explainer for JavaScript topics_ -->
以下主题介绍了 JavaScript v3 和 v4 Bot Framework SDK 之间的差异、这两个版本之间的主要变更，以及将机器人从 v3 迁移到 v4 的步骤。

| 主题 | 说明 |
| :--- | :--- |
| [V3 和 v4 JavaScript SDK 之间的差异](migration-about-javascript.md) | v3 和 v4 SDK 之间的常见差异 |
| [JavaScript 迁移快速参考](javascript-migration-quickreference.md)| v3 和 v4 SDK 中的主要变更|
| [将 JavaScript v3 机器人迁移到 v4](conversion-javascript.md) |将 v3 机器人迁移到 v4 机器人 |

---

### <a name="code-samples"></a>代码示例

下面是可以用来学习 Bot Framework SDK V4 或启动项目的代码示例。

| 示例 | 说明 |
| :--- | :--- |
| [Bot Framework V3 到 V4 迁移示例](https://github.com/microsoft/BotBuilder-Samples/tree/master/MigrationV3V4) <img width="200">| Bot Framework V3 SDK 到 V4 SDK 迁移示例 |
| [Bot Builder .NET 示例](https://github.com/microsoft/BotBuilder-Samples/tree/master/samples/csharp_dotnetcore) | Bot Builder C# .NET Core 示例 |
| [Bot Builder JavaScript 示例](https://github.com/microsoft/BotBuilder-Samples/tree/master/samples/javascript_nodejs) | Bot Builder JavaScript (node.js) 示例 |
| [Bot Builder 所有示例](https://github.com/microsoft/BotBuilder-Samples/tree/master/samples) | Bot Builder 所有示例 |

### <a name="getting-help"></a>获取帮助

以下资源为开发机器人提供了额外的信息和支持。

[Bot Framework 的其他资源](https://docs.microsoft.com/en-us/azure/bot-service/bot-service-resources-links-help?view=azure-bot-service-4.0)

### <a name="references"></a>参考

有关更多详细信息和背景信息，请参阅以下资源。

| 主题 | 说明 |
| :--- | :--- |
| [Bot Framework 中的新增功能](https://docs.microsoft.com/en-us/azure/bot-service/what-is-new?view=azure-bot-service-4.0) | Bot Framework 和 Azure 机器人服务的关键功能和改进|
|[机器人工作原理](../bot-builder-basics.md)|机器人的内部机制|
|[管理状态](../bot-builder-concept-state.md)|通过抽象来简化状态管理|
|[对话框库](../bot-builder-concept-dialog.md)| 通过核心概念来管理聊天|
|[发送和接收文本消息](../bot-builder-howto-send-messages.md)|机器人与用户通信的主要方式|
|[发送媒体](../bot-builder-howto-add-media-attachments.md)|媒体附件，例如图像、视频、音频和文件| 
|[顺序聊天流](../bot-builder-dialog-manage-conversation-flow.md)| 将提问作为机器人与用户交互的主要方式|
|[保存用户和聊天数据](../bot-builder-howto-v4-state.md)|在无状态情况下跟踪聊天|
|[复杂流](../bot-builder-dialog-manage-complex-conversation-flow.md)|管理复杂的聊天流 |
|[重复使用对话](../bot-builder-compositcontrol.md)|创建独立的对话来处理特定的方案|
|[中断](../bot-builder-howto-handle-user-interrupt.md)| 通过处理中断创建强大的机器人|
|[活动架构](https://aka.ms/botSpecs-activitySchema)|人类和自动化软件的架构|
