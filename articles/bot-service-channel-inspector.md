---
title: 使用 Channel Inspector 预览机器人功能 | Microsoft Docs
description: 通过使用 Channel Inspector 了解各种 Bot Framework 功能在不同通道上的外观和工作方式。
keyword: bot, preview features, channel, display, Channel Inspector, Text formatting, markdown, XML
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: def3f811704af2e2a22612ba5755eb5dbd2d8044
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/18/2018
ms.locfileid: "39297851"
---
# <a name="preview-bot-features-with-the-channel-inspector"></a>使用 Channel Inspector 预览机器人功能

借助 Bot Framework，用户可创建具有各种功能的机器人，如文本、按钮、图像、以轮播或列表格式显示的多个资讯卡等。 但是，最终由每个通道控制其消息传递客户端呈现功能的方式。

即使多个通道均支持某个功能，每个通道也可能以略有不同的方式呈现该功能。 如果消息包含某通道本身不支持的功能，则该通道可能会尝试将消息内容向下呈现为文本或静态图像，这可能会显著影响消息在客户端上的外观。 某些情况下，通道可能根本不支持特定功能。 例如，GroupMe 客户端无法显示键入指示器。

## <a name="the-channel-inspector"></a>Channel Inspector

通过创建 [Channel Inspector][inspector]，用户可以预览各种 Bot Framework 功能在不同通道上的外观和工作方式。 了解各种通道如何呈现功能后，便能据此设计机器人，以便在其通信通道上提供卓越的用户体验。 Channel Inspector 还提供了一种了解和直观探索 Bot Framework 功能的好方法。

> [!NOTE]
> 资讯卡是机器人信息交换的开发标准，可确保信息在多个通道间显示一致。 有关资讯卡的详细信息，请参阅 [.NET][netcard] 或 [Node.js][nodecard] 文档。

## <a name="text-formatting"></a>文本格式

文本格式可增强文本信息的视觉效果。 除纯文本外，机器人还可使用 markdown 或 xml 格式向支持它们的通道发送短信。 下表列出了一些最常用的 markdown 和 xml 文本格式。 各个通道支持的文本格式可能比此处列出的更少或更多。 可查看 [Channel Inspector][inspector]，了解目标通道是否支持你要使用的功能。

### <a name="markdown-text-format"></a>Markdown 文本格式

`textFormat` 设置为 markdown 时，可支持以下样式：  

| Style | Markdown | 示例 |
| ---- | ---- | ---- |
| 粗体 | \*\*text\*\* | **text** |
| 斜体 | \*text\* | *text* |
| 标头 (1-5) | # H1 | # H1 |
| 删除线 | \~\~text\~\~ | ~~text~~ |
| 水平线 | ---- | ---- |
| 未排序列表 | \* 文本 |  <ul><li>text</li></ul> |
| 已排序列表 | 1. 文本 | 1. 文本 |
| 预设格式的文本 | \`text\` | `text`  |
| 块引用 | \> 文本 | <blockquote>text</blockquote> |
| 超链接 | \[bing](http://www.bing.com) | [bing](http://www.bing.com) |
| 图像链接| !\[duck](http://aka.ms/Fo983c) | ![鸭子](http://aka.ms/Fo983c) |

> [!NOTE]
> Microsoft Bot Framework 网上聊天通道不支持 Markdown 中的 HTML 标记。 如果需要在 Markdown 中使用 HTML 标记，可在支持它们的 [Direct Line](~/bot-service-channel-connect-directline.md) 通道中呈现它们。 或者，可通过将 `textFormat` 设置为 xml 并将机器人连接到 Skype 通道来使用以下 HTML 标记。

### <a name="xml-text-format"></a>XML 文本格式

`textFormat` 设置为 xml 时，可支持以下样式：

|      Style      |                       XML                       |                   示例                   |
|-----------------|-------------------------------------------------|---------------------------------------------|
|      粗体       |                 \<b\>文本\</b\>                 |            <strong>text</strong>            |
|     斜体      |                 \<i\>文本\</i\>                 |                <em>text</em>                |
|    下划线    |                 \<u\>文本\</u\>                 |                 <u>text</u>                 |
|  删除线  |                 \<s\>文本\</s\>                 |                 <s>text</s>                 |
|    超链接    |   \<a href="http://www.bing.com"\>bing\</a\>    |   <a href="http://www.bing.com">bing</a>    |
|    段落    |                 \<p\>文本\</p\>                 |                 <p>text</p>                 |
|   换行符    |                     \<br/\>                     |             第 1 行 <br/>第 2 行              |
| 水平线 |                     \<hr/\>                     |                    <hr/>                    |
|  标头 (1-4)   |                \<h1\>文本\</h1\>                |             <h1>标题 1</h1>              |
|      代码       |           \<code\>代码块\</code\>           |           <code>code block</code>           |
|      图像      | \<img src="<http://aka.ms/Fo983c>" alt="Duck"\> | <img src="http://aka.ms/Fo983c" alt="Duck"> |

> [!NOTE]
> 仅 Skype 通道支持将 `textFormat` 设置为 xml。

## <a name="preview-features-across-various-channels"></a>预览不同通道上的功能

若要查看通道如何呈现特定功能，请转到 [Channel Inspector][inspector]，从“通道”列表选择通道，然后从“功能”列表中选择功能。 例如，要查看 Skype 如何呈现 Hero 卡，请将“通道”设置为“Skype”，并将“功能”设置为“HeroCard”。

![显示 Skype 通道和 Hero 卡的 Channel Inspector](~/media/bot-service-channel-inspector.png)

Channel Inspector 按指定通道将呈现出的所选功能外观显示该功能预览。 “注释”部分传达了有关消息限制和/或显示更改的重要信息。 例如，某些类型的资讯卡仅支持一个图像，某些功能在某些通道上可能会向下呈现。

> [!NOTE]
> Channel Inspector 目前支持以下通道：
> * Cortana
> * 电子邮件
> * Facebook
> * GroupMe
> * Kik
> * Skype
> * Skype for Business
> * Slack
> * SMS
> * Microsoft Teams
> * Telegram
> * 微信
> * 网上聊天
>
> 将来可能会添加其他通道。

## <a name="features-that-can-be-previewed"></a>可预览的功能

Channel Inspector 目前允许预览以下功能。

|功能 | Description|
| --- | ----|
| 自适应卡片 | 一种可包含文本、语音、图像、按钮和输入域的任意组合的卡。 |
| 按钮| 用户可单击的按钮。 按钮显示在聊天画布上，并显示其所属的消息。 |
| 轮播| 可滚动的紧凑型水平式卡片列表。 对于垂直布局，请使用“列表”。|
| ChannelData| 一种传递元数据以访问除卡、文本和附件之外的通道特定功能的方法。|
| Codesample| 用于显示计算机代码的多行预设格式文本块。|
| DirectMessage| 发送给组聊天的单个成员的消息。
| 文档| 发送和接收非媒体附件。 |
| 表情符号| 显示支持的表情符号。
| GroupChat| 机器人发送消息以参与组聊天。 |
| HeroCard| 一种格式化附件，通常包含单个大图像、点击操作和按钮（可选），以及描述性文本内容。 |
| 映像| 显示图像附件。 |
| 列表布局| 垂直式卡片列表。 对于水平式可滚动布局，请使用“轮播”。|
| 位置| 与机器人共享用户的物理位置。 |
| Markdown| 呈现 Markdown 格式的文本。|
| 成员| 在群聊期间与机器人共享聊天中的成员列表。 |
| 收据卡| 向用户显示收据。 |
| 登录卡| 请求用户输入身份验证凭据。|
| 建议的操作 | 以按钮形式显示的操作，用户可点击按钮提供输入。 |
| 缩略图卡| 一种格式化附件，包含单个小图像（缩略图）、点击操作和按钮（可选），以及描述性文本内容。 |
| Typing| 显示键入指示器。 此功能帮助告知用户机器人仍在运行，但在后台执行某些操作。|
| 视频| 显示视频附件和播放控件。|

## <a name="additional-resources"></a>其他资源

* [Channel Inspector][inspector]
* [Node.js][nodecard] 和 [.NET][netcard] 中的资讯卡
* [Node.js][nodemedia] 和 [.NET][netmedia] 中的媒体附件
* [Node.js][nodebutton] 和 [.NET][netbutton] 中的建议操作

[inspector]: https://docs.botframework.com/en-us/channel-inspector/channels/Skype/

[syntax]: https://daringfireball.net/projects/markdown/syntax

[netcard]: ~/dotnet/bot-builder-dotnet-add-rich-card-attachments.md
[nodecard]: ~/nodejs/bot-builder-nodejs-send-rich-cards.md

[netmedia]: ~/dotnet/bot-builder-dotnet-add-media-attachments.md
[nodemedia]: ~/nodejs/bot-builder-nodejs-send-receive-attachments.md

[netbutton]: ~/dotnet/bot-builder-dotnet-add-suggested-actions.md
[nodebutton]: ~/nodejs/bot-builder-nodejs-send-suggested-actions.md
