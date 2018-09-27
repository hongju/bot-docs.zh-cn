---
title: 认知服务 | Microsoft Docs
description: 了解如何使用 Microsoft 认知服务向机器人添加人工智能，使其更有用且更具吸引力。
keywords: 语言理解, 知识提取, 语音识别, web 搜索
author: RobStand
ms.author: rstand
manager: rstand
ms.topic: article
ms.prod: bot-framework
ms.date: 12/17/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 05841f6d708b8d22d29fb44b217e97b06053dbb4
ms.sourcegitcommit: 3bf3dbb1a440b3d83e58499c6a2ac116fe04b2f6
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/23/2018
ms.locfileid: "46707993"
---
# <a name="cognitive-services"></a>认知服务

借助 Microsoft 认知服务，你能够利用计算机视觉、语音、自然语言处理、知识提取和 Web 搜索领域专家开发的不断增长的强大 AI 算法集合。 这些服务简化各种基于 AI 的任务，为你提供一种快速方法，只需几行代码即可向机器人添加最先进的智能技术。 API 集成到大部分现代语言和平台中。 API 同时还在不断改进、学习及变得更智能，因此，体验始终是最新的。 

智能机器人能够以人类的思维方式作出响应。 它们从不同来源发现信息并提取知识，从而提供有用的答案，尤为可贵的是，它们会在获得更多经验的同时进行学习，以便不断改进自己的功能。 

## <a name="language-understanding"></a>语言理解

用户与机器人之间的交互大部分时候是自由形式的，机器人需要自然地根据上下文理解语言。 认知服务语言 API 提供强大的语言模型，可用于确定用户需求、标识给定句子中的概念和实体，以及最终让机器人以相应操作进行响应。 这五个 API 支持多种文本分析功能，例如拼写检查、情绪检测、语言建模以及从文本中提取准确而丰富的见解。 

认知服务提供这些 API 用于语言理解：

- <a href="https://www.microsoft.com/cognitive-services/en-us/language-understanding-intelligent-service-luis" target="_blank">语言理解智能服务 (LUIS)</a> 能够使用预构建的或自定义训练的语言模型处理自然语言。 有关更多详细信息，可在[机器人语言理解](v4sdk/bot-builder-concept-luis.md)中找到

- <a href="https://www.microsoft.com/cognitive-services/en-us/text-analytics-api" target="_blank">文本分析 API</a> 可从文本中检测情绪、关键短语、主题和语言。

- <a href="https://www.microsoft.com/cognitive-services/en-us/bing-spell-check-api" target="_blank">必应拼写检查 API</a> 提供强大的拼写检查功能，并能够识别名称、品牌名称和俚语之间的差异。

- 借助 <a href="https://docs.microsoft.com/en-us/azure/machine-learning/studio/text-analytics-module-tutorial" target ="_blank">Azure 机器学习工作室中的文本分析模型</a>，你可生成和操作文本分析模型，例如词形还原和文本预处理。 这些模型可以帮助解决文档分类或情绪分析等问题。

详细了解如何使用 Microsoft 认知服务进行[语言理解][language]。

## <a name="knowledge-extraction"></a>知识提取

认知服务提供四个知识 API，你可像使用个性化常见问题解答服务一样，借此标识非结构化文本中的命名实体或短语、添加个性化建议、基于用户查询的自然解释提供自动完成建议，以及搜索学术论文和其他研究成果。

- <a href="https://www.microsoft.com/cognitive-services/en-us/entity-linking-intelligence-service" target="_blank">实体链接智能服务</a>使用文本中提到的相关实体批注非结构化文本。 根据上下文，相同的单词或短语可能会指代不同的内容。 此服务可理解所提供文本的上下文，并会标识文本中的每个实体。    

- <a href="https://www.microsoft.com/cognitive-services/en-us/knowledge-exploration-service" target="_blank">知识探索服务</a>提供用户查询的自然语言解释并返回带批注的解释，实现可预测用户输入内容的丰富搜索和自动完成体验。 即时查询完成建议和预测查询优化基于你自己的数据和特定于应用的语法，让用户能够执行快速查询。    

- <a href="https://www.microsoft.com/cognitive-services/en-us/academic-knowledge-api" target="_blank">学术知识 API</a> 返回来自 <a href="https://www.microsoft.com/en-us/research/project/microsoft-academic-graph/" target="_blank">Microsoft Academic Graph</a> 的学术研究论文、作者、期刊、会议、主题和大学信息。 学术知识 API 作为知识探索服务的特定于域的示例生成，使用类似图形的对话提供知识库，且对话具有跨数以亿计的研究相关实体进行搜索的功能。 只要搜索主题、教授、大学或会议，API 就会提供相关出版物和相关实体。 语法还支持自然查询，例如“Papers by Michael Jordan about machine learning after 2010”。

- <a href="https://qnamaker.ai" target="_blank">QnA Maker</a> 是免费的、易于使用的 REST API 和基于 Web 的服务，可训练 AI 以自然的聊天方式回应用户的问题。 凭借经过优化的机器学习逻辑和集成行业领先语言处理功能的能力，QnA Maker 能够将问答对等半结构化数据提取为清楚、有用的答案。

详细了解如何使用 Microsoft 认知服务进行[知识提取][knowledge]。

## <a name="speech-recognition-and-conversion"></a>语音识别和转换

使用语音 API 向机器人添加高级语音技能，利用行业领先的算法进行语音到文本和文本到语音转换，以及说话人辨识。 语音 API 使用能够以高准确度覆盖各种方案的内置语言模型和声学模型。 

对于需要进一步自定义的应用程序，可使用自定义智能识别服务 (CRIS)。 这样一来，你可根据应用程序的词汇，甚或用户的说话风格进行定制，从而校准语音识别器的语言模型和声学模型。

认知服务提供三个语音 API 用于处理或合成语音：

- <a href="https://www.microsoft.com/cognitive-services/en-us/speech-api" target="_blank">必应语音 API</a> 提供语音到文本和文本到语音的转换功能。
- <a href="https://www.microsoft.com/cognitive-services/en-us/custom-recognition-intelligent-service-cris" target="_blank">自定义智能识别服务 (CRIS)</a> 允许创建自定义语音识别模型，可根据应用程序的词汇或用户的说话风格定制语音到文本的转换。
- <a href="https://www.microsoft.com/cognitive-services/en-us/speaker-recognition-api" target="_blank">说话人识别 API</a> 可通过语音识别和验证说话人。

以下资源提供有关向机器人添加语音识别的其他信息。

* [适用于应用的机器人聊天视频概览](https://channel9.msdn.com/events/Build/2017/P4114)
* [适用于 UWP 或 Xamarin 应用的 Microsoft.Bot.Client 库](https://aka.ms/BotClient)
* [机器人客户端库示例](https://aka.ms/BotClientSample)
* [支持语音的网上聊天客户端](https://aka.ms/BFWebChat)

详细了解如何使用 Microsoft 认知服务进行[语音识别和转换][speech]。

## <a name="web-search"></a>Web 搜索

借助必应搜索 API，你能够向机器人添加智能 Web 搜索功能。 只需几行代码，即可访问数十亿网页、图像、视频、新闻和其他结果类型。 可将 API 配置为按地理位置、市场或语言返回结果，以实现更好的相关性。 可使用支持的搜索参数进一步自定义搜索，例如使用“安全搜索”筛选掉成人内容，以及使用“新鲜度”根据特定日期返回结果。

认知服务提供五个必应搜索 API。

- <a href="https://www.microsoft.com/cognitive-services/en-us/bing-web-search-api" target="_blank">Web 搜索 API</a> 使用单个 API 调用提供网页、图像、视频、新闻和相关搜索结果。

- <a href="https://www.microsoft.com/cognitive-services/en-us/bing-image-search-api" target="_blank">图像搜索 API</a> 返回带有增强元数据（主色、图像种类等）的图像结果，并支持使用多个图像筛选器来自定义结果。

- <a href="https://www.microsoft.com/cognitive-services/en-us/bing-video-search-api" target="_blank">视频搜索 API</a> 检索具有丰富元数据（视频大小、品质、价格等）的视频结果和视频预览，并支持使用多个视频筛选器来自定义结果。

- <a href="https://www.microsoft.com/cognitive-services/en-us/bing-news-search-api" target="_blank">新闻搜索 API</a> 在全球范围内查找与搜索查询匹配的新闻报道或 Internet 上目前热门的新闻报道。

- <a href="https://www.microsoft.com/cognitive-services/en-us/bing-autosuggest-api" target="_blank">自动推荐 API</a> 提供即时查询完成建议，可以更快地完成搜索查询并减少键入的字符。 

详细了解如何使用 Microsoft 认知服务进行 [Web 搜索][search]。

## <a name="image-and-video-understanding"></a>图像和视频理解

视觉 API 为机器人带来高级图像和视频理解技能。 借助最先进的算法，你可以处理图像或视频，并取回可转换为操作的信息。 例如，你可以使用它们来识别物体、人脸、年龄、性别甚至感受。 

视觉 API 支持各种图像理解功能。 它们可以标识成人或露骨内容、判断和突出颜色、分类图像内容、执行光学字符识别，以及使用完整的英语句子描述图像。 视觉 API 还支持多种图像和视频处理功能，例如智能生成图像或视频缩略图，或稳定视频的输出。

认知服务提供四个 API 用于处理图像或视频：

- <a href="https://www.microsoft.com/cognitive-services/en-us/computer-vision-api" target="_blank">计算机视觉 API</a> 可提取有关图像（例如物体或人物）的丰富信息、确定图像是否包含成人或露骨内容，以及处理图像中的文本（使用 OCR）。

- <a href="https://www.microsoft.com/cognitive-services/en-us/emotion-api" target="_blank">情感 API</a> 可分析人脸，并在八种可能的人类情感中识别出他们的情感。

- <a href="https://www.microsoft.com/cognitive-services/en-us/face-api" target="_blank">人脸 API</a> 可检测人脸、将其与相似的人脸进行比较，甚至可以根据视觉相似度将人物分组。

- <a href="https://www.microsoft.com/cognitive-services/en-us/video-api" target="_blank">视频 API</a> 可分析和处理视频，以稳定视频输出、检测动作、跟踪人脸，并生成视频的动作缩略图摘要。

详细了解如何使用 Microsoft 认知服务进行[图像和视频理解][vision]。

## <a name="additional-resources"></a>其他资源

有关每个产品及其相应 API 参考的综合文档，可在<a href="https://docs.microsoft.com/azure/cognitive-services" target="_blank">认知服务文档</a>中找到。

[language]: https://docs.microsoft.com/en-us/azure/cognitive-services/luis/home
[search]: https://docs.microsoft.com/en-us/azure/cognitive-services/bing-web-search/search-the-web
[vision]: https://docs.microsoft.com/en-us/azure/cognitive-services/computer-vision/home
[knowledge]: https://docs.microsoft.com/en-us/azure/cognitive-services/kes/overview
[speech]: https://docs.microsoft.com/en-us/azure/cognitive-services/speech/home
[location]: https://docs.microsoft.com/en-us/azure/cognitive-services/
