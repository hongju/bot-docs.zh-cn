---
title: 应答模板 | Microsoft Docs
description: 了解如何设置聊天设计器机器人的应答模板。
author: vkannan
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ROBOTS: NoIndex, NoFollow
ms.openlocfilehash: b09b7fa5f5672c121711deb2edcdc9718a79983f
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/18/2018
ms.locfileid: "39298444"
---
# <a name="response-template-for-conversation-designer-bots"></a>聊天设计器机器人的应答模板
> [!IMPORTANT]
> 聊天设计器尚无法供所有客户使用。 关于聊天设计器可用性的更多详细信息将在今年晚些时候发布。

利用语言生成功能，机器人能够使用可变的答复消息以自然、多样的方式与用户沟通。 这些消息通过聊天设计器中的应答模板进行管理。

利用应答模板，可重复使用答复，并使机器人答复中的语调和语言保持一致。 

## <a name="create-response-templates"></a>创建应答模板

可在聊天设计器中创建应答模板；在任何需要向用户发送答复的情况下都可重复使用这些模板。 

简单的应答模板会定义一个一次性的集合，其中包含可能的语音和显示话语。 然后，可在机器人的答复、提示状态或自适应卡片中使用这些模板，从而重新构造完全解析的字符串。

要添加简单的应答模板，请执行以下操作：
1. 从左面板中，单击“添加”。 随即显示上下文菜单。
2. 单击“简单答复”。 主内容面板中会显示编辑窗口。
3. 在“简单答复名称”字段中，输入此简单答复的名称。
4. 在“机器人对用户的答复”字段中，输入答复（一次输入一个短语）。
5. 简单答复设置完毕后，请在右上角单击“保存”。 

例如，可创建具有以下内容的确认短语模板（称其为“AckPhrase”）：

- 正常
- 当然
- 没问题
- 我很乐意为你提供帮助
- 很乐意效劳

使用此模板，聊天运行时会随机解析为其中一个字符串，从而让机器人的答复听起来更加自然，而不是单调乏味。

## <a name="conditional-response-templates"></a>带条件的应答模板

带条件的应答模板会指定多个带条件的答复。 你编写的回调函数可帮助语言生成引擎根据你指定的条件决定使用哪个答复字符串。 

例如，“一天中的时间”模板应基于一天中的实际时间解析为“早上好”或“晚上好”。 

要添加带条件的应答模板，请执行以下操作：
1. 从左面板中，单击“添加”。 随即显示上下文菜单。
2. 单击“带条件的答复”。 主内容面板中会显示编辑窗口。
3. 在“带条件的答复名称”字段中，输入此模板的名称。
4. 在“带条件的答复”字段中，一次输入一个短语。 例如，对于“timeOfDayTemplate”，输入“早上好”和“晚上好”作为模板中两种可能的答复。
5. 对于“早上好”这一答复，将其“条件名称”指定为“早上”。 同样，对于“晚上好”这一答复，将其“条件名称”指定为“晚上”。 你要编写的自定义脚本将返回上述条件中的任一一种。
6. 在“运行时要执行的代码”字段中，输入回调函数名称（例如 `fnResolveTimeOfDayTemplate`）。 然后，单击“查看代码”以加载脚本*编辑器。 此时，可定义此回调函数的实现。
7. 在右上角中，单击“保存”以保存所做的更改并创建此模板。

fnResolveTimeOfDayTemplate 回调函数的示例代码。 此函数将返回与答复指定的“条件名称”（即“早上”或“晚上”）相匹配的字符串。

```javascript
module.exports.fnTimeOfDayTemplate = function(context) {
    var currentTime = new Date().getHours();
    if(currentTime >= 12) {
        return "evening!";
    } else {
        return "morning!";
    }
}
```

## <a name="send-a-response-to-user"></a>向用户发送答复

要使用应答模板向用户发送答复，请将任一模板名称用方括号括起来。 例如，要在反馈状态中使用“AckPhrase”这一简单的应答模板，请将“机器人对用户的答复”短语输入为 `[AckPhrase], I will get that done for you right away`

如果想要在答复中使用方括号，请使用 "\" 作为转义字符，指示聊天运行时不要解析方括号。

如果已定义实体，则还可在对用户的答复中使用它们。 可将实体名称用大括号括起来，以便引用语言生成中的实体。 例如，如果机器人具有 `location` 实体，则可在答复中引用它，如下所示：`[AckPhrase], I can help you find a table at {location}.`

## <a name="nesting-response-templates"></a>嵌套应答模板

应答模板可进行“嵌套”；一个应答模板可引用另一应答模板。 例如，可使用 `AckPhrase` 这一简单的应答模板和 `timeOfDayTemplate` 这一带条件的应答模板构造一个新的 `timeBasedGreeting` 应答模板，而该新模板使用前两个模板解析最终答复。 

> [!TIP]
> 尽管模板嵌套这一功能非常强大，但请务必检查确保嵌套模板不会导致无限循环。 也就是说，要避免 `AckPhrase` 调用 `timeOfDayTemplate` 和 `timeOfDayTemplate` 回调 `AckPhrase` 的情况。

例如，创建一个新的“简单答复”名称 `timeBasedGreeting`，并输入以下文本作为此模板可能的“机器人对用户的答复”：`[timeOfDayTemplate] [AckPhrase], ... `

## <a name="define-user-utterance-as-help-tips"></a>将用户话语定义为帮助提示

定义用户“话语”时，还可将话语设置为“帮助提示”。 要将话语设置为“帮助提示”，请单击话语右侧的 `...`，并选择“用作帮助提示”。 

在任何位置定义“机器人对用户的答复”字段时，内置语言生成模板 `[builtin-tasktips]` 中都会使用帮助提示。 例如，可撰写如下表述的答复：`Sorry I did not understand that. Here are some things you can try - [builtin.tasktips]`

如果任务尚未定义“帮助提示”，则 `[builtin-tasktips]` 将解析为空字符串。 如果任务已定义多个“帮助提示”，则 `[builtin-tasktips]` 将在每次给出答复时从中随机选择一个。

## <a name="next-step"></a>后续步骤
> [!div class="nextstepaction"]
> [自适应卡片](conversation-designer-adaptive-cards.md)
