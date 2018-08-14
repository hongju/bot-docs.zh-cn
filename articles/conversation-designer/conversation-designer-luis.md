---
title: 将 LUIS 识别器定义为任务触发器 | Microsoft Docs
description: 了解如何使用 LUIS.ai 将语言理解识别器设置为任务触发器
author: vkannan
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ROBOTS: NoIndex, NoFollow
ms.openlocfilehash: 39fe222fb1d54346b33617c425b1fdf2d56daa0d
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/18/2018
ms.locfileid: "39298441"
---
# <a name="define-a-luis-recognizer-as-task-trigger"></a>将 LUIS 识别器定义为任务触发器
> [!IMPORTANT]
> 聊天设计器尚无法供所有客户使用。 关于聊天设计器可用性的更多详细信息将在今年晚些时候发布。

通常，用户用自然语言表达执行任务的意向。 借助聊天设计器，可轻松设置由 <a href="https://luis.ai" target="_blank">LUIS.ai</a> 助力的机器人自然语言理解模型。

为此，请选择触发类型“用户说话或键入内容”。 这将提供用于指定意向名称的选项。 

可搜索现有意向或在“语言意向是什么?”字段中创建一个意向。

## <a name="create-a-new-intent"></a>创建新意向

要创建新的意向，请键入意向的名称并单击“创建新意向”。 输入认为用户可能说的示例话语，该话语应触发此特定任务。

例如，咖啡馆机器人应该能够执行查找用户附近咖啡馆位置的任务。 要处理此方案，请选择“用户说话或键入内容”，并将“意向名称”设置为 LocationNearMe。 然后提供此意向的示例话语。 例如： 
- 查找我附近的位置
- 查找我附近的咖啡馆位置
- Redmond 现在在营业吗？
- 西雅图是否有这种商店？
- 哪儿的咖啡馆现在在营业？
- 我可以去哪里买一些吃的？
- 我想去吃点东西
- 我饿了

尽可能多地输入你认为可帮助表述出用户要触发此特定任务的意向的话语。

## <a name="default-intents-provisioned-for-your-bot"></a>为机器人预配的默认意向

默认情况下，机器人预配有 4 个意向。 
- 无：这是机器人的回退（默认）意向。 使用此意向帮助了解机器人尚不知道如何答复的内容。
- 帮助：设置示例话语，该话语可帮助确定用户是否需要帮助。 例如“我需要帮助”、“我要说什么？”、“我很疑惑”等等。
- 问候：设置使用可帮助匹配问候意向示例话语，例如“嗨”、“你好”、“早上好”、“你好吗，机器人”等等。
- 取消：设置用于取消意向的示例话语。 例如“停止”、“取消”、“不执行此操作”、“还原”等等。

## <a name="create-and-label-entities"></a>创建并标记实体

除了帮助确定用户意向外，语言理解还可以帮助你确定与任务相关的感兴趣的特定实体。 例如，当用户说“查找 Redmond 附近的咖啡馆位置”，你可能需要提取“Redmond”作为实体位置的可能值。 

要为任务设置实体，请从话语字符串中选择应作为实体值示例的话语部分。 将其分配给现有实体或创建一个新实体。 要创建新实体，请将实体的名称键入“搜索或创建”字段，然后单击“创建新实体”。 

# <a name="supported-entity-types"></a>支持的实体类型

语言理解可用于创建不同类型的实体。 在创建实体时，必须指定 `type`。 

可用类型如下：

- **Simple**：这是默认类型。
- **List**：如果实体具有一组有限的可能值，请使用此类型。 示例：Color、City。
- **Hierarchical**：使用此类型创建具有父子级关系的实体。 示例：fromCity 和 toCity 都将 City 实体作为父级
- **Composite**：使用此类型创建几组构成有意义的单位的值。 示例：City 和 State 一起组成 Location 实体。

<!-- # pre-built entity types TBD -->

# <a name="entities-in-use"></a>正在使用的实体

创建实体并将其添加到语言理解部分时，“正在使用的实体”表上将更新有此特定任务使用的实体列表。 可将此任务中使用的其他实体手动添加到列表。 

可用选项包括：

- **Code**：这是在自定义脚本中创建的实体。 可在此处指定它，以帮助创建诸如 intellisense 之类的功能。

<!-- # Use as help tip TBD  -->

## <a name="next-step"></a>后续步骤
> [!div class="nextstepaction"]
> [代码识别器](conversation-designer-code-recognizer.md)
