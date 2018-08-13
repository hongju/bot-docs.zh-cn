---
title: 将对话定义为“执行”操作 | Microsoft Docs
description: 了解如何将对话设置为“执行”操作。
author: vkannan
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ROBOTS: NoIndex, NoFollow
ms.openlocfilehash: d59df20821a7f63eb9ee5dea365597b1af839f75
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/18/2018
ms.locfileid: "39297790"
---
# <a name="define-a-dialogue-as-a-do-action"></a>将对话定义为“执行”操作
> [!IMPORTANT]
> 聊天设计器尚未可供所有客户使用。 关于聊天设计器可用性的更多详细信息将在今年晚些时候发布。

对话涵盖特定任务的聊天模型。 例如，帮助用户订桌的咖啡店机器人可以定义一项名为“订桌”的任务。 “执行”操作将会绑定到一个对话，而该对话则为机器人和用户之间的聊天流建模。 

对话特别适用于在完成某个任务时，机器人与用户进行你一句我一句的聊天的情况。  用户很少能够在单个话语中表述完成某个任务所需的所有值。 

订桌时，需提供位置、参加人数、日期和时间首选项。 订桌机器人需理解并处理用户可能说出的各种可能的短语。 

- 订桌：在这种情况下，不捕获任何所需的实体。 机器人现在必须与用户聊天。
- 预订本周六晚 7 点的桌：在这种情况下，日期和时间首选项已指定，但机器人仍然必须收集预期的位置和参加人数。

在聊天过程中，某些实体可能会随用户输入的变化而变化。 例如，如果用户说“预订本周日晚 7 点雷德蒙德的桌”， 但雷德蒙德的店在周日晚 6 点关门，则机器人的对话应该处理此类无效请求。 

聊天设计器提供了一个拖放式对话设计器，用于将聊天流可视化。 对话设计器提供七种对话状态，这些状态可以用于为聊天流建模。

## <a name="dialogue-states"></a>对话状态

对话由聊天状态组成。 对话本身作为一个结构化的定向流来建模，该流为聊天运行时提供的结构用于展示如何执行聊天流。

对话带有许多内置的可供使用的状态。 支持的内置状态包括：

- [**启动**](#start-state)：表示聊天流的启动状态。 所有对话都必须定义至少一个启动状态。
- [**返回**](#return-state)：表示已完成特定的聊天流。 由于聊天流是可以组合的，因此返回状态会指示聊天运行时将执行返回到此对话的任何可能的调用方。
- [**决策**](#decision-state)：表示聊天流中的某个分支点。
- [**处理**](#process-state)：表示机器人正在执行业务逻辑的状态。
- [**提示**](#prompt-state)：表示可以提示用户输入的状态。 
- [**反馈**](#feedback-state)：表示可以向用户提供反馈或确认的状态。 例如，一个确认已进行预订的对话。
- [**模块**](#module-state)：表示对另一对话的调用。 由于对话流在默认情况下是可以组合的，因此此状态可以调用某个共享对话或者在此任务下定义的某个其他的对话。

每个聊天状态都使用对话设计器中的对话连接器连接到另一状态。

每个对话状态都有一个关联的状态编辑器，用于指定该状态的属性，包括自定义脚本的回调函数名称。 **状态编辑器**位于对话设计器主视图端口底部，充当一个重设大小窗格。 若要打开该编辑器，请双击对话设计器中的某个状态，然后**状态编辑器**就会显示该状态的属性。
<!-- TODO: insert screenshot of the wrench in horizontal menu -->

以下子节更详细地介绍了每个这样的内置对话状态。

### <a name="start-state"></a>启动状态

启动状态表示对话的起始点。 所需值为状态**名称**。 此名称默认为“启动”，可以通过编辑来重命名此状态。

### <a name="return-state"></a>返回状态

返回状态表示已完成聊天流的特定分支。 由于对话是可以组合的，因此此状态也会指示聊天运行时将执行返回到调用方对话。 所需值为状态**名称**。 此名称默认为“返回”，可以通过编辑来重命名此状态。

### <a name="decision-state"></a>决策状态

决策状态表示聊天流中的分支。 可以编写自定义脚本来评估要执行哪个分支。 脚本会根据用户输入和业务逻辑，返回某个可能的转换值。 每个转换值都会提示聊天运行时运行另一对话分支。

决策状态的必需属性：
- **名称**：决策状态的唯一名称。
- **要在运行时执行的代码**：回调函数的名称，该函数通过实现业务逻辑来确定执行聊天的哪个分支。 

#### <a name="example-code-for-decision-state"></a>决策状态的示例代码

以下示例回调函数返回一项决策，该决策指示聊天运行时执行哪个分支。

```javascript
module.exports.fnDecisionState = function(context) {
    var a = context.taskEntities['a'];
    if (a[0].value === '0') {
        return 'yes';
    }
    else if (a[0].value === '1') {
        return 'no';
    }
}
```

### <a name="process-state"></a>处理状态

处理状态表示对话中的某个点，机器人在该点驱动聊天继续进行，或者尝试执行最终任务完成操作。 

处理状态的必需属性：
- **名称**：处理状态的唯一名称
- **要在运行时执行的代码**：用于实现业务逻辑的回调函数的名称。

#### <a name="example-code-for-process-state"></a>处理状态的示例代码

以下示例回调函数获取天气并将天气信息返回给用户。

```javascript
module.exports.fnGetWeather = function(context) {
    var options =  {
        host: 'mock',
        path: '/get?a=b',
        method: 'get'
    };
    return http.request(options).then(function(response) {
        context.contextEntities['x'].value= response.statusCode;
        var jsonBody = JSON.parse(response.body);
          // understand response
        if (response.statusCode != "200") {
            // error
        }
    });
}
```

### <a name="prompt-state"></a>提示状态

提示状态要求用户提供特定的信息。 提示状态包含一个子对话系统，因此按定义来说属于复杂状态。 

在提示状态中，可以定义需提供给用户的实际响应，并可选择包括一张自适应卡。 然后即可指定一个触发器来分析并理解用户的响应。 该触发器可以是 LUIS，也可以是使用正则表达式的自定义代码识别器。  

如果用户提供的输入无效，机器人可以重新提示用户提供该信息。 此行为也可在提示状态编辑器中定义。 

#### <a name="prompting-the-user"></a>提示用户

可以通过提示响应来指定在**提示用户**输入时使用的消息。 例如，若要收集日期和时间，可以将响应设置为“你想要何时进来？” 或“你想要何时就座？”

#### <a name="prompt-listening-for-user-input"></a>提示侦听用户输入

在提示用户进行响应以后，聊天运行时会自动侦听用户输入，并尝试理解用户所说的内容。 根据 LUIS 或自定义代码识别器配置触发器，尝试理解用户的输入和意图。 该触发器类似于任务触发器。

#### <a name="re-prompting-the-user"></a>重新提示用户

使用重新提示节来指定每个尝试的响应。 重新提示节中的每一行对应于用于该特定轮次的重新提示字符串。 第一个响应会用于第一个重新提示，第二个响应会用于第二个重新提示，依此类推。 例如：

抱歉，我没有听懂，你想要什么时候进来？
对不起，我难以理解你的话。我们再试一遍 - 你想要什么时候进来？

#### <a name="prompt-callback-functions"></a>提示回调函数

可以根据一个提示状态指定两个不同的回调函数。 

1. **在每个提示和重新提示之前**：在每个提示或重新提示之前执行此函数。 此回调函数预期会有一个布尔返回值，该值为 true 表明执行此提示或重新提示，为 false 表明不执行此提示或重新提示。 可以使用 `getTurnIndex()` 获取该提示执行操作的当前轮次索引。
2. **在响应时**：在每次生成某个提示后执行此函数，此时尚未将该提示发送给用户（包括重新提示响应）。 这样就可以通过脚本来修改发送给用户的消息。

#### <a name="sample-code"></a>代码示例

以下代码片段演示的示例针对“执行之前”回调。

```javascript
module.exports.fnBeforeExecuting = function(context) {
    if(context.responses[0].text === "C") {
        return false;
    }
    return true;
}
```

以下代码片段演示的示例针对“在提示时”回调。

```javascript
module.exports.fnOnPrompting = function(context) {
    // include a hint card
    var activity = context.responses.slice(-1).pop();
    activity.attachments.push({
        "contentType": "application/vnd.microsoft.card.hero",
        "content": {
            "buttons": [
                {
                    "type": "imBack",
                    "title": "1",
                    "value": "1"
                },
                {
                    "type": "imBack",
                    "title": "2",
                    "value": "2"
                }
            ]
        }
    });
}
```

以下代码片段演示的示例针对“在重新提示之前”回调。

```javascript
module.exports.fnBeforeReprompting = function(context) {
    if(context.responses[0].text === "C") {
        return false;
    }
    return true;
}
```

### <a name="feedback-state"></a>反馈状态

使用此状态向用户提供响应。 这方面的典型用例包括在尝试完成任务后提供最终结果，或者在失败的情况下向用户提供响应，等等。 

每个反馈状态都需要唯一名称和某些可能的响应值，并且可以选择性地包括自适应卡定义。 详细了解[自适应卡定义](conversation-designer-adaptive-cards.md)。

每个反馈状态也允许使用“在响应时”回调函数来编写自定义脚本，以便根据需要修改活动有效负载，然后再将其发送给用户。 


```javascript
module.exports.fnOnResponding = function(context) {
    // include a hint card
    var activity = context.responses.slice(-1).pop();
    activity.attachments.push({
        "contentType": "application/vnd.microsoft.card.hero",
        "content": {
            "buttons": [
                {
                    "type": "imBack",
                    "title": "1",
                    "value": "1"
                },
                {
                    "type": "imBack",
                    "title": "2",
                    "value": "2"
                }
            ]
        }
    });

}
```

### <a name="module-state"></a>模块状态

模块状态用于添加引用，以便执行子对话。 使用此状态可以将对话串在一起。 

每个模块状态都需要一个唯一的名称和一个指向要执行的特定对话的指针。 要执行的对话的可能选项必须位于“共享对话”下或特定的“任务”下。

## <a name="multiple-dialogues-under-a-task"></a>一个任务下的多个对话

一个任务可以有多个对话。 若要向某个任务添加对话，请直接选择该任务，单击左侧树面板中的“添加”按钮，然后单击“添加对话”。 这样就会将新对话添加到所选任务下。 

由于对话是可以组合的，因此可以将根对话流绑定到任务，以便调用其下的其他对话。 这样便可以封装子任务并启用重用功能， 同时还可以使用模块状态在一个聊天流中链接这些对话。

## <a name="next-step"></a>后续步骤
> [!div class="nextstepaction"]
> [响应模板](conversation-designer-response-templates.md)
