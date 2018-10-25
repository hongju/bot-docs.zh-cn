---
title: 对话框状态 | Microsoft Docs
description: 介绍 Bot Builder SDK 中状态的工作原理。
keywords: 状态, 机器人状态, 聊天状态, 用户状态
author: johnataylor
ms.author: johtaylo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 9/21/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 4da715650eb1c3e7b284aaa47aa5ce4ac7280f4c
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998754"
---
# <a name="dialog-state"></a>对话框状态

对话框是实现多轮次聊天逻辑的方法，因此，在依赖于跨会话持久化状态的 SDK 中，可以将它们作为功能的示例。 

基于对话框的机器人通常将 DialogSet 集合作为成员变量保存在其机器人实现中。 DialogSet 在创建时，会带有一个名为访问器的对象的句柄。 

在 SDK 状态模型中，访问器是核心概念。 访问器实现 IStatePropertyAccessor 接口，这意味着它必须为 Get、Set 和 Delete 方法提供实现。 若要创建访问器，请为其提供一个属性名称。 

使用访问器的代码可以调用其 Get、Set 和 Delete 方法，因为知道这些方法会引用该名称的属性。 更高级别的组件在需要某种程度的状态保持时，应使用访问器。 这样一来，不同的方案在进行自己的状态存储实现时，仍可使用相同的更高级别的代码。 例如，对话框可以使用访问器，这是其访问持久状态的唯一方式。

![对话框状态](media/bot-builder-dialog-state.png)

调用机器人的 OnTurn 时，机器人会通过在 DialogSet 上调用 CreateContext 来初始化对话框子系统。 创建 DialogContext 需要状态，因此 DialogSet 使用其访问器通过 Get 方法来获取相应的对话框状态 JSON。 SDK 以 BotState 类的形式提供访问器的实现。 需要利用状态实现的应用程序可以通过创建 BotState 子类来继承访问器的实现。 SDK 中包含 BotState 的两个子类：

- UserState
- ConversationState

UserState 和 ConversationState 使用密钥进行基础存储。 密钥会向下传递给物理存储。 在逻辑上，密钥充当访问器命名的属性的命名空间。 BotState 实现在内部使用 TurnContext 上的入站活动来动态生成所用的存储密钥。

- UserState 使用通道 ID 和源 ID 创建密钥。例如，_{Activity.ChannelId}/conversations/{Activity.From.Id}#DialogState_
- ConversationState 使用通道 ID 和聊天 ID 创建密钥。例如，_{Activity.ChannelId}/users/{Activity.Conversation.Id}#YourPropertyName_

应用程序需提供访问器，然后它就会在后台绑定到相应的动态创建的存储密钥和属性名称。 BotState 在实现访问器时包括一些优化： 

- 第一个优化是使用缓存形式的延迟加载。 对实际存储实现的加载会推迟到首次调用针对访问器的 Get 方法的时候，加载结果保存在缓存中。 将会使用相应的 BotState 提供的密钥将此缓存的引用添加到 TurnContext。 因此，与 UserState 对应的缓存会保存在名为“UserState”的字段中，而与 ConversationState 对应的缓存则保存在名为“ConversationState”的字段中。 调用各种访问器对象时，系统会将调用转给 TurnContext 来提取相应的缓存。

- 第二个优化是，BotState 类包含用于确定状态是否已更改的逻辑。 只有在已进行更改的情况下，它才会在基础存储上执行实际的保存操作。

- BotState 实现中的第三个优化存在于名为 BotStateSet 的类中。 BotStateSet 是 BotState 对象的集合，可以将对 SaveChanges 方法的调用并行委托给集合中的每个成员。

针对 BotState 的 SaveChanges 调用不能是 IStatePropertyAccessor 接口的一部分，这一点很重要。 原因在于，SaveChanges 是对 BotState 实现的特定优化，而不是模型的核心功能。 具体说来，对话框之类的代码无法知道 BotState，更不用说 SaveChanges 了。 事实上，对话框的代码仅耦合到访问器。 这样做的目的是确保在对话框系统执行完以后，只能在其外部调用 SaveChanges 方法。 例如，可以在机器人的 OnTurn 方法中调用它，如下图所示。

必须指出的是，BotState 实现带有某些特定的语义。 尤其是，它支持“以最后一次写入为准”行为，即最后一次写入会改写上一次写入的状态。 这适用于大多数应用情况，但在某些情况下会带来负面影响，尤其是在进行横向扩展的时候，需要进行某种程度的并发操作。 如果属于这种情况，则必须实现自己的访问器，并将其传递到对话框中。 可以使用许多替代方法，例如，可以在解决方案中选择利用 eTag 条件，这在 Azure 存储这样的云存储服务中很常用。 这种情况下，解决方案可能会实现其他重要的部件。 例如，解决方案可能会缓存出站活动，只有在成功地完成“保存”操作后才发送这些活动。 必须指出的是，此行为不是 BotState 实现的行为，而是应用程序需要在访问器级别提供并实施的行为。

BotState 实现本身有一个可插式存储模型。 它遵循简单的“加载/保存”模式，而 SDK 会提供两种替代的适用于生产的实现。 一种实现用于 Azure 存储，另一种实现用于 CosmosDB，而内存中实现则用于测试目的。 但是，这里必须指出，“以最后一次写入为准”的语义是 BotState 实现规定的。

## <a name="handling-state-in-middleware"></a>处理中间件中的状态
上图显示在机器人的 OnTurn 结束时发生的对 SaveAllChanges 的调用。 下面是同一张图，突出显示了该调用。

![状态中间件问题](media/bot-builder-dialog-state-problem.png)

此方法的问题是，在机器人的 OnTurn 方法返回后，从某些自定义中间件进行状态更新时，更新不会保存到持久存储。 解决方法是将 AutoSaveChangesMiddleware 添加到中间件堆栈末尾，这样就可以将对 SaveAllChanges 的调用操作移到自定义中间件的操作完成以后。 执行情况如下所示。

![状态中间件解决方案](media/bot-builder-dialog-state-solution.png)

## <a name="additional-resources"></a>其他资源
如需更多详细信息，请参阅 GitHub 上的 Bot Builder SDK [[C#](https://github.com/Microsoft/BotBuilder-dotnet) | [JavaScript](https://github.com/Microsoft/BotBuilder-js)]。
