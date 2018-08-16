---
title: 从示例机器人创建会话设计器机器人 | Microsoft Docs
description: 使用其中的一个示例机器人启动一个新机器人。
author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ROBOTS: NoIndex, NoFollow
ms.openlocfilehash: 0ebf0d1b90b03789d8a77710631c83f0914099c5
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/18/2018
ms.locfileid: "39298029"
---
# <a name="create-a-conversation-designer-bot-from-a-sample-bots"></a>从示例机器人创建会话设计器机器人

会话设计器是一种强大的框架，可以提供视觉机器人生成器、用于定义机器人的声明性模型，以及简化机器人创建过程的运行时。 简化机器人创建的一种方法是从现有机器人启动机器人。

可以从许多示例机器人中的一个机器人或以前导出的会话设计器机器人创建会话设计器机器人。 每个示例机器人都是功能完备的机器人。 这为你提供了一个坚实的起点，可以从这里开始构建自己的机器人或自定义示例机器人以适合你的方案。

## <a name="the-sample-bots"></a>示例机器人

会话设计器附带一组示例机器人。 可以根据其中一个示例机器人开始构建机器人。 这些示例机器人中的大多数涵盖了功能完备的机器人的小型方案。 如果想要看到所有这些示例机器人一起工作，请创建完整的 Contoso 咖啡馆机器人。 

下表列出了你可以创建的机器人类型，其中还包含有关每个机器人及其涵盖的方案的简要说明。 

| 机器人名称 | 方案 | 引入的功能 |
| ---- | ---- | ---- |
| **基础机器人** | 具有一些基础功能的机器人。 例如，在将新用户添加到会话时，该机器人会显示欢迎消息，对问候消息做出响应，并在用户询问机器人不理解的内容时执行回退行为。 | - [任务](conversation-designer-tasks.md) <br/>- 任务触发器和操作 <br/>- [语言理解触发器](conversation-designer-luis.md)<br/>- [脚本触发器](conversation-designer-code-recognizer.md)<br/>- [回复操作](conversation-designer-reply.md) |
| **回显机器人** | 回显用户消息的基础机器人。  无论你发送向机器人发送什么内容，它都会回显消息说：“这就是你说的......消息......”。 | - [脚本触发器](conversation-designer-code-recognizer.md)<br/>- 解释代码中的上下文对象<br/>- 在代码中创建实体<br>- 在机器人的回复中使用实体 |
| **QnA 机器人** | 可以使用 [QnAMaker.ai](http://qnamaker.ai) 执行单轮问答体验的机器人。 要使此示例正常工作，必须设置 [QnA Maker](http://qnamaker.ai) 帐户并通过一些问题和答案对其进行定型。 然后，打开此 QnA 机器人，转到“脚本”页，将这两个占位符替换为 QnA Maker 的信息：`<INSERT YOUR KB ID>` 和 `<INSERT YOUR SUBSCRIPTION KEY>`。 输入这两条信息后，[保存](conversation-designer-save-bot.md)该机器人，然后可以[测试](conversation-designer-debug-bot.md)机器人。 | - [脚本触发器](conversation-designer-code-recognizer.md)<br/>- 从代码发出 HTTP 请求<br/>- 连接自定义 NLP/LU 服务（示例中的 QnAMaker.ai） |
| **会话机器人** | 可以参与简单会话并记住上下文（例如用户名字）的机器人。 | - [对话框操作](conversation-designer-dialogues.md)<br/>- 基本的[对话框状态](conversation-designer-dialogues.md#dialogue-states)和属性<br/>- [提示对话框](conversation-designer-dialogues.md#prompt-state)简介（定义提示和重新提示行为）<br/>- 在提示对话框中通过标记实体[创建意向](conversation-designer-luis.md#create-a-new-intent)<br/>- 在机器人的回复中使用由语言理解生成的实体<br/>- 使用[卡片](conversation-designer-adaptive-cards.md)来改善最终用户体验<br/>- 将机器人实体绑定到[输入表单](conversation-designer-adaptive-cards.md#input-form)<br/>- 创建和使用[响应模板](conversation-designer-response-templates.md)机器人资产 |
| **内存机器人** | 可以询问用户偏好设置并稍后回忆出会话信息的机器人。 当你说出类似于“设置通信首选项”这样的指令时， 该机器人会询问你是希望通过“呼叫”还是“文本”接收通信。 你还可以通过说出类似“查找咖啡馆位置”或“告诉我到华盛顿州雷德蒙市的 Contoso 咖啡馆的路线”这样的指令，询问此机器人 Contoso 咖啡馆的位置信息。 机器人找到该信息后，就会呼叫你或发短信，告诉你它所找到的信息。 <br/><br/>如果已经设置了通信首选项，机器人将只使用该首选项，否则它将提示用户。 | - [对话框操作](conversation-designer-dialogues.md)<br/>- 使用[`context.global`](conversation-designer-context-object.md#context-properties) 将数据保存到机器人内存<br/>- 从内存中调用上下文信息以在对话框中使用 |
| **预定座位机器人** | 可以与用户进行多轮次会话，以帮助用户在 Contoso 咖啡馆预订座位的机器人。 <br/><br/>要预订座位，此机器人需要你提供三条信息：(1) 参与方人数、(2) 日期/时间及 (3) 位置。 <br/><br/>你可以在一条消息中提供所有三条信息，说出类似以下的指令：“我想在星期二下午 2 点预订位于雷德蒙德市位置的 4 人桌。” 如果你遗漏这三条信息中的任何一条，机器人都会提示你。 | - [对话框操作](conversation-designer-dialogues.md)<br/>- 引用其他对话框的复杂对话框操作<br/>- 共同撰写对话框<br/>- 使用 LUIS 的所有强大功能来设置机器人的对话框，以管理转接填槽和更正体验等内容<br/>- 使用[卡片](conversation-designer-adaptive-cards.md)来改善最终用户体验<br/>- 将机器人实体绑定到[输入表单](conversation-designer-adaptive-cards.md#input-form)<br/>- 创建和使用[条件响应模板](conversation-designer-response-templates.md#conditional-response-templates)机器人资产 |
| **搜索优化机器人** | 可以使用以前会话中传递的上下文信息来帮助优化搜索的机器人。 可以说出类似“这个周末西雅图商店营业吗？”的指令， 接着问“伦顿商店呢”- 机器人会记得第二个问题你还在谈论这个周末。 也可以尝试“周五晚上 8 点哪家商店营业”，然后问“晚上 10 点呢”？ 等等 | - [对话框操作](conversation-designer-dialogues.md)<br/>- 使用[`context.global`](conversation-designer-context-object.md#context-properties) 将数据保存到机器人内存<br/>- 使用机器人内存与用户进行上下文转发类型会话<br/>- 从内存中调用上下文信息以在对话框中使用 |
| **点三明治机器人** | 演示在点三明治时以不同方法提示用户进行输入的机器人。 例如，这个机器人将帮助你点三明治。 <br/><br/>要点三明治，机器人需要四条信息：(1) 三明治尺寸、(2) 面包类型、(3) 蛋白质选择 及(4) 三明治配菜。 <br/><br/>类似于预订座位机器人，你可以在一条消息中提供所有所需的四条信息，或者可以一次提供一个，说出以下指令，“我想要点一个三明治。” 或者“我可以点一个大号三明治吗？” 然后，机器人会询问你处理订单所需的其他信息。 | - [对话框操作](conversation-designer-dialogues.md)<br/>- 设置[提示](conversation-designer-response-templates.md)以便通过不同方式收集信息 |
| **Contoso 咖啡馆机器人** | 为 Contoso 咖啡馆创建的功能完备的机器人。 这个机器人可以执行其他示例机器人所做的一切操作。 它提供了一个欢迎消息，其中包含你可以与之交互的富卡。 它支持帮助、问候和回退消息。 帮助你预订座位或查找咖啡馆位置或点三明治。 | - 会话设计器的所有功能<br/>- 使用[`context.global`](conversation-designer-context-object.md#context-properties) 将数据保存到机器人内存<br/>- 使用[卡片](conversation-designer-adaptive-cards.md)根据用户输入触发特定的机器人任务（请通过欢迎消息中的按钮查看卡片）<br/>- 构建一个可以执行多个不同[任务](conversation-designer-tasks.md)的复杂机器人。 |

## <a name="explore-sample-bots"></a>探索示例机器人

如果使用的当前示例机器人不是你想要的，你可以随时切换到另一个示例机器人。 可以在**探索示例机器人**页面中找到所有示例机器人的列表。

> [!WARNING] 
> 导入示例机器人会将当前机器人替换为示例机器人。 如果不想丢失当前机器人中的自定义，请将其[导出](conversation-designer-export-import-bot.md#export-a-conversation-designer-bot)为一个 .zip 文件。

若要切换到其他示例机器人，请执行以下操作：
1. 在当前的机器人中，单击“探索示例机器人”。 可以在左侧导航面板的底部找到此选项。 
2. 从“示例机器人”列表中选择一个机器人，然后单击“导入”。
3. 如果要将工作保存在当前机器人中，请选择“备份并导入”选项。 此选项会将当前机器人保存到本地计算机，然后导入新的“示例机器人”。 否则，请选择“导入而不备份”。

机器人现在已替换为刚刚选择的新示例机器人。 

## <a name="next-step"></a>后续步骤
> [!div class="nextstepaction"]
> [保存机器人](conversation-designer-save-bot.md)
