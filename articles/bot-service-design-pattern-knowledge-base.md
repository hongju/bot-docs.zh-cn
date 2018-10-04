---
title: 设计知识型机器人 | Microsoft Docs
description: 了解设计知识型机器人的不同方法，这种机器人可查找并返回信息来响应用户的输入或查询。
author: matvelloso
ms.author: mateusv
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: dd8869c26a87718177462db2508e41aa82810e21
ms.sourcegitcommit: f0b22c6286e44578c11c9f15d22b542c199f0024
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/27/2018
ms.locfileid: "47404073"
---
# <a name="design-knowledge-bots"></a>设计知识型机器人

可设计知识型机器人，让其几乎能够提供任何主题的相关信息。 例如，一个知识型机器人可能会回答活动相关问题，例如“本次会议中进行了哪些机器人活动？”、“下一次雷鬼表演是什么时候？”或“谁是 Tame Impala？” 另一个可能会回答与 IT 相关的问题，例如“如何更新我的操作系统？” 或“在哪里重置密码？” 但另外一个可能会回答有关联系人的问题，如“谁是何石？” 或“何石的电子邮件地址是什么？” 

无论要设计哪一种知识型机器人，其基本目标都是一样的：通过利用大量数据（例如 SQL 数据库中的关系数据、非关系存储区的 JSON 数据或文档存储区中的 PDF）来查找并返回用户已请求的信息。 

## <a name="search"></a>搜索

搜索功能是机器人中的一项重要工具。 

首先，“模糊搜索”使机器人能够返回可能与用户问题相关的信息，而无需用户提供精确的输入。 例如，如果用户向音乐知识型机器人询问关于“impala”（而不是“Tame Impala”）的信息，则机器人会答复与该输入最可能相关的信息。

![对话结构](~/media/bot-service-design-pattern-knowledge-base/fuzzySearch2.png)

搜索分数表示特定搜索的结果的置信度，使机器人能够相应地对其结果进行排序，甚至根据置信度定制其通信。 例如，如果置信度很高，机器人可能会回答“下面是与你的搜索最匹配的活动：”。

![对话结构](~/media/bot-service-design-pattern-knowledge-base/searchScore2.png)

如果置信度较低，机器人可能会回答“嗯......你在查找这些事件吗？”

![对话结构](~/media/bot-service-design-pattern-knowledge-base/searchScore1.png)

### <a name="using-search-to-guide-a-conversation"></a>通过搜索功能指导聊天

如果构建机器人的动机是实现基本的搜索引擎功能，那么可能根本不需要机器人。 聊天界面中提供的哪些内容是用户无法在 Web 浏览器的典型搜索引擎中获取的？ 

知识型机器人通常在专用于指导聊天时效果最佳。 聊天由用户和机器人之间的来回交流构成，这使机器人有机会通过基本搜索无法实现的方式提出澄清式问题、提供选项和验证结果。 例如，以下机器人引导用户完成分面和筛选数据集的聊天，直到找到用户正在查找的信息为止。

![对话结构](~/media/bot-service-design-pattern-knowledge-base/guidedConvo1.png)

![对话结构](~/media/bot-service-design-pattern-knowledge-base/guidedConvo2.png)

![对话结构](~/media/bot-service-design-pattern-knowledge-base/guidedConvo3.png)

![对话结构](~/media/bot-service-design-pattern-knowledge-base/guidedConvo4.png)

通过在每个步骤中处理用户输入的内容并提供相关选项，机器人引导用户获得其正在查找的信息。 机器人提供了该信息后，它甚至能够提供相关指导来帮助用户在未来以更有效的方式找到类似信息。 

![对话结构](~/media/bot-service-design-pattern-knowledge-base/Training.png)

### <a name="azure-search"></a>Azure 搜索

通过 <a href="https://azure.microsoft.com/en-us/services/search/" target="_blank">Azure 搜索</a>，可创建机器人可轻松搜索、分面和筛选的高效搜索索引。 请考虑使用通过 Azure 门户创建的搜索索引。

![对话结构](~/media/bot-service-design-pattern-knowledge-base/search3.PNG)

假设你希望能够访问数据存储的所有属性，因此要将每个属性都设置为“可检索”。 同时你希望能够按姓名查找歌手，因此要将 Name 属性设置为“可搜索”。 最后，你希望能够对歌手的年代进行筛选，因此要将 Eras 属性同时标记为“可分面”和“可筛选”。 

分面确定给定属性的数据存储中存在的值，同时确定每个值的大小。 例如，以下屏幕截图显示数据存储中有 5 个不同的年代：

![对话结构](~/media/bot-service-design-pattern-knowledge-base/facet.png)

反过来，筛选仅选择特定属性的指定实例。 例如，可筛选上面的结果集，使其仅包含 Era 等于 Romantic（浪漫主义时期）的项。 

> [!NOTE]
> 要通过完整示例了解使用 Azure Document DB、Azure 搜索和 Microsoft Bot Framework 创建的知识型机器人，请参阅<a href="https://github.com/ryanvolum/AzureSearchBot" target="_blank">示例机器人</a>。
> 
> 为简单起见，上面的示例显示了使用 Azure 门户创建的搜索索引。 
> 也可以编程方式创建索引。

## <a name="qna-maker"></a>QnA Maker

一些知识型机器人可能只回答常见问题 (FAQ)。 
<a href="https://www.microsoft.com/cognitive-services/en-us/qnamaker" target="_blank">QnA Maker</a> 是一款专为此用例设计的强大工具。 QnA Maker 内置有从现有常见问题解答站点中提取问题和答案的功能，它还能让你手动配置自己的问题和答案自定义列表。 QnA Maker 能够处理自然语言，由此甚至可提供用词与预期略有不同的问题的答案。 但是，它无法理解语义性的语言。 例如，它无法确定小狗是一种类型的狗。 

使用 QnA Maker Web 界面，可配置具有 3 对问答的知识库： 

![对话结构](~/media/bot-service-design-pattern-knowledge-base/KnowledgeBaseConfig.png)

然后，可通过询问一系列问题来测试它： 

![对话结构](~/media/bot-service-design-pattern-knowledge-base/exampleQnAConvo.png)

机器人正确回答了与知识库中配置的问题直接匹配的问题。 然而，对于“我可以带茶吗？”这个问题，机器人回答错误，因为此问题在结构上与“我能带伏特卡吗？”问题最相似。 QnA Maker 给出错误答案的原因是它本质上并不理解字词的含义。 它不知道“茶”是一种非酒精饮料。 因此，它回答“不允许饮酒”。

> [!TIP]
> 创建 QnA 问答对，然后使用聊天下的“检查”按钮测试并重新训练机器人，为给出的每个错误答案选择一个替代答案。 

## <a name="luis"></a>LUIS

一些知识型机器人需具备自然语言处理 (NLP) 功能，这样才能分析用户的消息以确定用户的意向。 
利用[语言理解 (LUIS)](https://www.luis.ai)，可快速有效地向机器人添加 NLP 功能。 通过 LUIS，可在必应和 Cortana 中预生成的现有模型满足你的需求时使用这些模型，也可自行创建专用模型。 

在处理大型数据集时，通过实体的每个变体训练 NLP 模型不一定行得通。 例如，在音乐播放机器人中，用户可能会发送消息“播放雷鬼音乐”、“播放鲍勃·马利”或“播放 One Love”。 虽然机器人可将上述每个消息都映射到意向“playMusic”，而无需使用每个艺术家、流派和歌曲名训练机器人，但 NLP 模型会无法标识该实体是流派、艺术家还是歌曲。 通过使用 NLP 模型来标识“音乐”类型的通用实体，机器人能够在其数据存储中搜索该实体，并从该处继续执行操作。 

## <a name="combining-search-qna-maker-andor-luis"></a>结合使用搜索功能、QnA Maker 和/或 LUIS

搜索功能、QnA Maker 和 LUIS 每一项都是功能强大的工具，但也可结合使用这三者来构建具有多种功能的知识型机器人。

### <a name="luis-and-search"></a>LUIS 和搜索

在[前面介绍的](#search)音乐节机器人示例中，机器人通过显示配置列表相关按钮来引导聊天。 然而，这个机器人还可使用 LUIS 来确定“Romit Girdhar 演奏哪种音乐？”等问题中的意向和实体，从而结合使用自然语言理解工具。 然后，机器人可按歌手姓名针对 Azure 搜索索引进行搜索。 
 
由于可能存在大量的值，因此使用每个可能的歌手姓名来训练模型是不可行的，但是可为 LUIS 提供足够多的代表性示例，使其能够正确标识当前实体。  例如，请考虑通过提供歌手示例来训练模型： 

![对话结构](~/media/bot-service-design-pattern-knowledge-base/answerGenre.png)
![对话结构](~/media/bot-service-design-pattern-knowledge-base/answerGenreOneWord.png)

当使用新的话语（例如“披头士乐队演奏哪种音乐？”）测试此模型时，LUIS 成功确定了意向“answerGenre”并标识出实体“披头士乐队”。 但是，如果提交一个更长的问题（比如“The Devil Makes Three 演奏哪种音乐？”），LUIS 会将“The Devil”标识为实体。

![对话结构](~/media/bot-service-design-pattern-knowledge-base/devilMakesThreeScore.png)

通过使用代表基础数据集的示例实体训练模型，可提高机器人语言理解的准确性。 

> [!TIP]
> 通常，模型犯错的原因最好是在其实体识别中标识出多余的字词（例如从话语“请呼叫彭德威”中标识“请彭德威”），而不是标识出太少的字词，例如在话语“请呼叫彭德威”中标识“彭”。 搜索索引将忽略不相关的字词，例如“请彭德威”短语中的“请”。 

### <a name="luis-and-qna-maker"></a>LUIS 和 QnA Maker

一些知识型机器人可能会结合使用 QnA Maker 与 LUIS 来回答基本问题，从而确定意向、提取实体和发起更详细的对话。 例如，设想一个简单的 IT 支持人员机器人。 该机器人可能会使用 QnA Maker 来回答有关 Windows 或 Outlook 的基本问题，但它可能还需要帮助处理密码重置等方案，这就需要确认意向且用户与机器人之间来回通信。 机器人可通过以下几种方式实现 LUIS 和 QnA Maker 的混用：

1. 同时调用 QnA Maker 和 LUIS，并使用返回特定阈值分数的第一条内容中的信息来答复用户。 
2. 首先调用 LUIS，如果没有意向满足特定阈值分数（即触发“None”意向），则调用 QnA Maker。 或者，为 QnA Maker 创建 LUIS 意向，从而向 LUIS 模型提供映射到“QnAIntent”的 QnA 问题示例。 
3. 首先调用 QnA Maker，如果答案均不符合特定的阈值分数，则调用 LUIS。 

Bot Builder SDK 为 LUIS 和 QnA Maker 提供内置支持。 这让用户能够使用 LUIS 和/或 QnA Maker 触发对话或自动回答问题，而无需对任一工具实现自定义调用。 有关详细信息，请参阅[机器人服务模板](bot-service-concept-templates.md)。

> [!TIP]
> 在实现 LUIS、QnA Maker 和/或 Azure 搜索的组合时，通过每个工具测试输入以确定每个模型的阈值分数。 LUIS、QnA Maker 和 Azure 搜索各自使用不同的评分标准生成分数，因此通过这些工具生成的分数无法直接对比。 此外，LUIS 和 QnA Maker 将分数标准化。 某个 LUIS 模型可能认为某一分数是“良好”，而另一模型则不这么认为。 

## <a name="sample-code"></a>代码示例

- 要通过示例了解如何使用 Bot Builder SDK for .NET 创建基本的知识型机器人，请参阅 GitHub 中的<a href="https://aka.ms/qna-with-appinsights" target="_blank">知识型机器人示例</a>。 
<!-- TODO: Do not have a current bot sample to work with this
- For a sample that shows how to create more complex knowledge bots using the Bot Builder SDK for .NET, see the <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/demo-Search" target="_blank">Search-powered Bots sample</a> in GitHub.
-->

[qnamakerTemplate]: https://docs.botframework.com/en-us/azure-bot-service/templates/qnamaker/#navtitle
