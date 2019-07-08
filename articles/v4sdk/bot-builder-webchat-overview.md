---
title: Web Chat 概述 | Microsoft Docs
description: 了解如何配置 Bot Framework Web Chat。
keywords: 机器人框架, webchat, 聊天, 示例, react, 引用
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 06/07/2019
ms.openlocfilehash: 4dab0a8f3dab2a87184a03fe25b2a9abeaff67a2
ms.sourcegitcommit: dbbfcf45a8d0ba66bd4fb5620d093abfa3b2f725
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/28/2019
ms.locfileid: "67464653"
---
# <a name="web-chat-overview"></a>Web Chat 概述

本文包含 [Bot Framework Web Chat](https://github.com/microsoft/BotFramework-WebChat) 组件的详细信息。 Bot Framework Web Chat 组件是一个基于 web 的高度可自定义客户端，用于 Bot Framework V4 SDK。 Bot Framework SDK v4 使开发人员能够创建对话模型并生成复杂的机器人应用程序。

如果想要从 Web Chat v3 迁移到 v4，请跳转到[迁移部分](#migrating-from-web-chat-v3-to-v4)。

## <a name="how-to-use"></a>如何使用

> [!NOTE]
> 要获取 Web Chat (v3) 的之前版本，请访问 [Web Chat v3 分支](https://github.com/Microsoft/BotFramework-WebChat/tree/v3)。

首先，使用 [Azure 机器人服务](https://azure.microsoft.com/services/bot-service/)创建机器人。
机器人创建完毕后，需要在 Azure 门户中[获取机器人的 Web Chat 机密](../bot-service-channel-connect-webchat.md#step-1)。 然后，使用该机密[生成令牌](../rest-api/bot-framework-rest-direct-line-3-0-authentication.md)，并将其传递到 Web Chat。

以下是将 Web Chat 控件添加到网站的方法：

```html
<!DOCTYPE html>
<html>
   <body>
      <div id="webchat" role="main"></div>
      <script src="https://cdn.botframework.com/botframework-webchat/latest/webchat.js"></script>
      <script>
         window.WebChat.renderWebChat(
            {
               directLine: window.WebChat.createDirectLine({
                  token: 'YOUR_DIRECT_LINE_TOKEN'
               }),
               userID: 'YOUR_USER_ID',
               username: 'Web Chat User',
               locale: 'en-US',
               botAvatarInitials: 'WC',
               userAvatarInitials: 'WW'
            },
            document.getElementById('webchat')
         );
      </script>
   </body>
</html>
```

> `userID`、`username`、`locale`、`botAvatarInitials` 和 `userAvatarInitials` 是要传递到 `renderWebChat` 方法的全部可选参数。 若要了解有关 Web Chat 属性的详细信息，请查看此 `README` 中的 [Web Chat API 参考](#web-chat-api-reference)。
> ![Web Chat 屏幕截图](https://raw.githubusercontent.com/Microsoft/BotFramework-WebChat/master/media/weatherquery.png.jpg)

### <a name="integrate-with-javascript"></a>与 JavaScript 进行集成

Web Chat 用于集成使用 JavaScript 或 React 的现有网站。 与 JavaScript 集成将实现适度的样式设置和可自定义性。

可使用完整的典型 webchat 包，其中包含最常用的功能。

```html
<!DOCTYPE html>
<html>
   <body>
      <div id="webchat" role="main"></div>
      <script src="https://cdn.botframework.com/botframework-webchat/latest/webchat.js"></script>
      <script>
         window.WebChat.renderWebChat(
            {
               directLine: window.WebChat.createDirectLine({
                  token: 'YOUR_DIRECT_LINE_TOKEN'
               }),
               userID: 'YOUR_USER_ID'
            },
            document.getElementById('webchat')
         );
      </script>
   </body>
</html>
```

请参阅[完整的 Web Chat 捆绑包](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/01.a.getting-started-full-bundle)的有效示例。

### <a name="integrate-with-react"></a>与 React 集成

要获得完全可自定义性，可使用 React 重新组织 Web Chat 组件。

若要从 NPM 安装生产版本，请运行`npm install botframework-webchat`。

```jsx
import { DirectLine } from 'botframework-directlinejs';
import React from 'react';
import ReactWebChat from 'botframework-webchat';

export default class extends React.Component {
  constructor(props) {
    super(props);

    this.directLine = new DirectLine({ token: 'YOUR_DIRECT_LINE_TOKEN' });
  }

  render() {
    return (
      <ReactWebChat directLine={ this.directLine } userID='YOUR_USER_ID' />
      element
    );
  }
}
```

> 此外，还可运行 `npm install botframework-webchat@master` 以安装与 Web Chat 的 GitHub `master` 分支同步的开发版本。

请参阅[通过 React 呈现的 Web Chat](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/03.a.host-with-react/) 的有效示例。

## <a name="customize-web-chat-ui"></a>自定义 Web Chat UI

Web Chat 是可自定义的，无需创建源代码分支。 下表概述了以不同方式导入 Web Chat 时可实现的自定义类型。 此列表并未囊括所有方式。

|                               | CDN 捆绑包         | React              |
| ----------------------------- | ------------------ | ------------------ |
| 更改颜色                 | :heavy_check_mark: | :heavy_check_mark: |
| 更改大小                  | :heavy_check_mark: | :heavy_check_mark: |
| 更新/替换 CSS 样式     | :heavy_check_mark: | :heavy_check_mark: |
| 侦听事件              | :heavy_check_mark: | :heavy_check_mark: |
| 与托管网页进行交互 | :heavy_check_mark: | :heavy_check_mark: |
| 自定义呈现活动      |                    | :heavy_check_mark: |
| 自定义呈现附件     |                    | :heavy_check_mark: |
| 添加新 UI 组件         |                    | :heavy_check_mark: |
| 重新组织整个 UI        |                    | :heavy_check_mark: |

请参阅[自定义 Web Chat](https://github.com/Microsoft/BotFramework-WebChat/blob/master/SAMPLES.md)，以获取有关自定义的详细信息。

## <a name="migrating-from-web-chat-v3-to-v4"></a>从 Web Chat v3 迁移到 v4

从 v3 迁移到 v4 时，有三条可能的迁移路径。 首先，请比较初始场景：

### <a name="my-current-website-integrates-web-chat-using-an-iframe-element-obtained-from-azure-bot-services-i-want-to-upgrade-to-v4"></a>我的当前网站使用从 Azure 机器人服务获取的 `<iframe>` 元素集成 Web Chat。 我想升级到 v4。

从 2019 年 5 月开始，我们将向使用 `<iframe>` 元素集成 Web Chat 的网站推出 v4。 请参阅[嵌入文档](https://github.com/Microsoft/BotFramework-WebChat/tree/master/packages/embed)，了解如何使用 `<iframe>` 集成 Web Chat。

### <a name="my-website-is-integrated-with-web-chat-v3-and-uses-customization-options-provided-by-web-chat-no-customization-at-all-or-very-little-of-my-own-customization-that-was-not-available-with-web-chat"></a>我的网站集成了 Web Chat v3，并使用 Web Chat 提供的自定义选项，无需自定义或仅使用少量不适用于 Web Chat 的我自己的自定义。

请遵循示例 [`01.c.getting-started-migration`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/01.c.getting-started-migration) 的实现，将网页从 Web Chat v3 转换到 v4。

### <a name="my-website-is-integrated-with-a-fork-of-web-chat-v3-i-have-implemented-a-lot-of-customization-in-my-version-of-web-chat-and-i-am-concerned-v4-is-not-compatible-with-my-needs"></a>我的网站集成了 Web Chat v3 的一个分支。 我已在我的 Web Chat 版本中实现大量的自定义，我担心 v4 与我的需求不兼容。

我们团队最喜欢的 Web Chat v4 功能之一是可以添加自定义而无需创建 Web Chat 分支  。 虽然这对创建过 Web Chat 分支的 v3 用户造成了额外开销，但我们会尽力支持客户进行迁移。 请尝试以下建议：

-  查看示例 [`01.c.getting-started-migration`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/01.c.getting-started-migration) 的实现。 这是一个很好的启动并运行 Web Chat 的起点。
-  接下来，请浏览[示例列表](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples)，比较你的自定义要求与 Web Chat 支持的项。 这些示例由 Web Chat 常用的功能组成。
-  如果一个或多个功能在示例中不可用，请浏览[待解决和已解决的问题](https://github.com/Microsoft/BotFramework-WebChat/issues?utf8=%E2%9C%93&q=is%3Aissue+)、[示例标签](https://github.com/Microsoft/BotFramework-WebChat/issues?utf8=%E2%9C%93&q=is%3Aissue+is%3Aopen+label%3ASample)和[迁移支持标签](https://github.com/Microsoft/BotFramework-WebChat/issues?q=is%3Aissue+migrate+label%3A%22Migration+Support%22)，为你所寻找的功能搜索示例请求和/或自定义支持。 在待解决的问题中添加注释将有助于团队优先处理需求量高的请求，并且我们非常鼓励用户参与我们的社区。
-  如果在未解决请求列表中找不到你需要的功能，请[提出请求](https://github.com/Microsoft/BotFramework-WebChat/issues/new)。 正如以上的项，其他用户在你的待解决的问题中添加注释将有助于我们确定 Web Chat 用户最常用功能的优先顺序。
-  最后，如果你希望尽快获得你需要的功能，请[拉取请求](https://github.com/Microsoft/BotFramework-WebChat/compare)到 Web Chat。 如果你有自行实现此功能的编码经验，我们将非常感谢你的额外支持！ 自行创建该功能意味着你将能更快地在 Web Chat 上使用它，并且其他想要寻找相同或类似功能的用户可以利用你贡献的内容。
-  请确保签出此 `README` 的剩余部分以详细了解 v4。

## <a name="samples-list"></a>示例列表

| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;示例&nbsp;名称&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; | 说明                                                                                                                                                                                                                         | 链接                                                                                                                                |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| [`01.a.getting-started-full-bundle`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/01.a.getting-started-full-bundle)                                                                                       | 介绍从 CDN 嵌入的 Web Chat，并演示简单且功能齐全的 Web Chat。 其中包括 Adaptive Cards、认知服务以及 Markdown-It 依赖项。                                                            | [完整捆绑包演示](https://microsoft.github.io/BotFramework-WebChat/01.a.getting-started-full-bundle)                               |
| [`01.b.getting-started-es5-bundle`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/01.b.getting-started-es5-bundle)                                                                                         | 介绍功能齐全的 Web Chat，其中内嵌针对使用 Web Chat 的 ES5 ponyfill 的 ES5 浏览器的后向兼容性。                                                                                                                | [ES5 捆绑包演示](https://microsoft.github.io/BotFramework-WebChat/01.b.getting-started-es5-bundle)                                 |
| [`01.c.getting-started-migration`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/01.c.getting-started-migration)                                                                                           | 演示如何从 Web Chat v3 机器人迁移到 v4。                                                                                                                                                                        | [迁移演示](https://microsoft.github.io/BotFramework-WebChat/01.c.getting-started-migration)                                   |
| [`02.a.getting-started-minimal-bundle`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/02.a.getting-started-minimal-bundle)                                                                                 | 介绍仅包含基本依赖项的最小化的 CDN。 不包括 Adaptive Cards、认知服务依赖项或 Markdown-It 依赖项。                                                                      | [最小捆绑包演示](https://microsoft.github.io/BotFramework-WebChat/02.a.getting-started-minimal-bundle)                         |
| [`02.b.getting-started-minimal-markdown`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/02.b.getting-started-minimal-markdown)                                                                             | 演示如何基于最小的捆绑包添加适用于 Markdown-It 依赖项的 CDN。                                                                                                                                            | [使用 Markdown 进行最小化的演示](https://microsoft.github.io/BotFramework-WebChat/02.b.getting-started-minimal-markdown)                |
| [`03.a.host-with-react`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/03.a.host-with-react)                                                                                                               | 演示如何创建托管功能完备的 Web Chat 的 React 组件。                                                                                                                                                 | [通过 React 进行托管的演示](https://microsoft.github.io/BotFramework-WebChat/03.a.host-with-react)                                       |
| [`03.b.host-with-Angular`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/03.b.host-with-angular)                                                                                                           | 演示如何创建托管功能齐全的 Web Chat 的 Angular 组件。                                                                                                                                              | [通过 Angular 进行托管的演示](https://stackblitz.com/github/omarsourour/ng-webchat-example)                                              |
| [`04.a.display-user-bot-initials-styling`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/04.a.display-user-bot-initials-styling)                                                                           | 演示如何显示这两个 Web Chat 参与者的姓名首字母。                                                                                                                                                                | [机器人姓名首字母演示](https://github.com/Microsoft/BotFramework-WebChat/blob/master/samples/04.a.display-user-bot-initials-styling/)  |
| [`04.b.display-user-bot-images-styling`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/04.b.display-user-bot-images-styling)                                                                               | 演示如何显示这两个 Web Chat 参与者的图像和姓名首字母。                                                                                                                                                     | [用户图像演示](https://microsoft.github.io/BotFramework-WebChat/04.b.display-user-bot-images-styling)                           |
| [`05.a.branding-webchat-styling`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/05.a.branding-webchat-styling)                                                                                             | 介绍设置 Web Chat 样式以匹配你的品牌的功能。 此自定义样式设置方法不会在 Web Chat 更新时中断。                                                                                                   | [Web Chat 品牌化演示](https://microsoft.github.io/BotFramework-WebChat/05.a.branding-webchat-styling)                            |
| [`05.b.idiosyncratic-manual-styling`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/05.b.idiosyncratic-manual-styling/)                                                                                    | 演示如何手动更改样式，这是一个更复杂、更耗时的自定义 Web Chat 样式的方法。 手动样式在 Web Chat 更新时可能会中断。                                                | [特殊样式演示](https://microsoft.github.io/BotFramework-WebChat/05.b.idiosyncratic-manual-styling)                    |
| [`05.c.presentation-mode-styling`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/05.c.presentation-mode-styling)                                                                                           | 演示如何设置演示模式，该模式可显示聊天历史记录，但不显示发送框，该模式禁用 Adaptive Cards 的交互性。                                                                         | [演示模式演示](https://microsoft.github.io/BotFramework-WebChat/05.c.presentation-mode-styling)                           |
| [`05.d.hide-upload-button-styling`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/05.d.hide-upload-button-styling)                                                                                         | 演示如何通过样式隐藏文件上传按钮。                                                                                                                                                                            | [隐藏上传按钮的演示](https://microsoft.github.io/BotFramework-WebChat/05.d.hide-upload-button-styling)                         |
| [`06.a.cognitive-services-bing-speech-js`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/06.a.cognitive-services-bing-speech-js)                                                                           | 介绍使用（弃用的）认知服务必应语音 API 和 JavaScript 的语音转文本和文本转语音功能。                                                                                                      | [使用 JS 的必应语音演示](https://microsoft.github.io/BotFramework-WebChat/06.a.cognitive-services-bing-speech-js)                 |
| [`06.b.cognitive-services-bing-speech-react`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/06.b.cognitive-services-bing-speech-react)                                                                     | 介绍使用（弃用的）认知服务必应语音 API 和 React 的语音转文本和文本转语音功能。                                                                                                           | [使用 React 的必应语音的演示](https://microsoft.github.io/BotFramework-WebChat/06.b.cognitive-services-bing-speech-react)           |
| [`06.c.cognitive-services-speech-services-js`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/06.c.cognitive-services-speech-services-js)                                                                   | 介绍使用认知服务语音服务 API 的语音转文本和文本转语音功能。                                                                                                                                  | [使用 JS 的语音服务演示](https://microsoft.github.io/BotFramework-WebChat/06.c.cognitive-services-speech-services-js)         |
| [`06.d.speech-web-browser`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/06.d.speech-web-browser)                                                                                                         | 演示如何使用 Web Chat 的基于浏览器的 Web 语音 API 实现文本转语音。 （与示例中的 W3C 标准关联）                                                                                                    | [Web 语音 API 演示](https://microsoft.github.io/BotFramework-WebChat/06.d.speech-web-browser)                                     |
| [`06.e.cognitive-services-speech-services-with-lexical-result`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/06.e.cognitive-services-speech-services-with-lexical-result)                                 | 演示如何从认知服务语音服务 API 使用词法结果。                                                                                                                                                 | [词法结果演示](https://microsoft.github.io/BotFramework-WebChat/06.e.cognitive-services-speech-services-with-lexical-result) |
| [`06.f.hybrid-speech`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/06.f.hybrid-speech)                                                                                                                   | 演示如何使用两种基于浏览器的 Web 语音 API 用于语音转文本，以及如何使用认知服务语音服务 API 用于文本转语音。                                                                                        | [混合语音演示](https://microsoft.github.io/BotFramework-WebChat/06.f.hybrid-speech)                                           |
| [`07.a.customization-timestamp-grouping`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/07.a.customization-timestamp-grouping)                                                                             | 演示如何通过显示或隐藏时间戳以及按时间更改消息分组来自定义时间戳。                                                                                                             | [时间戳分组演示](https://microsoft.github.io/BotFramework-WebChat/07.a.customization-timestamp-grouping)                   |
| [`07.b.customization-send-typing-indicator`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/07.b.customization-send-typing-indicator)                                                                       | 演示如何在用户开始在发送框中键入时发送键入活动。                                                                                                                                                | [用户键入指示器演示](https://microsoft.github.io/BotFramework-WebChat/07.b.customization-send-typing-indicator)             |
| [`08.customization-user-highlighting`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/08.customization-user-highlighting)                                                                                   | 演示如何自定义基于活动的样式，无论消息来自用户还是机器人。                                                                                                                      | [用户突出显示演示](https://microsoft.github.io/BotFramework-WebChat/08.customization-user-highlighting)                       |
| [`09.customization-reaction-buttons`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/09.customization-reaction-buttons/)                                                                                    | 介绍特别针对机器人需求的为 Web Chat 创建自定义组件的功能。 本教程演示将反应表情符号（例如，:thumbsup: 和 :thumbsdown:）添加到对话活动的功能。 | [反应按钮演示](https://microsoft.github.io/BotFramework-WebChat/09.customization-reaction-buttons)                         |
| [`10.a.customization-card-components`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/10.a.customization-card-components)                                                                                   | 演示如何创建自定义活动卡片附件（此示例中是 GitHub 存储库卡）。                                                                                                                                  | [卡组件演示](https://microsoft.github.io/BotFramework-WebChat/10.a.customization-card-components)                         |
| [`10.b.customization-password-input`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/10.b.customization-password-input)                                                                                     | 演示如何创建用于密码输入的自定义活动。                                                                                                                                                                      | [密码输入演示](https://microsoft.github.io/BotFramework-WebChat/10.b.customization-password-input)                           |
| [`11.customization-redux-actions`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/11.customization-redux-actions)                                                                                           | 高级教程：演示如何通过机器人发送 redux 操作，将 redux 中间件合并到 Web Chat 应用中。 此示例演示了基于机器人与用户之间的活动手动设置样式。             | [Redux 操作演示](https://microsoft.github.io/BotFramework-WebChat/11.customization-redux-actions)                               |
| [`12.customization-minimizable-web-chat`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/12.customization-minimizable-web-chat)                                                                             | 高级教程：演示如何将 Web Chat 接口作为可最小化的显示/隐藏聊天框添加到你的网站。                                                                                                              | [可最小化的 Web Chat 演示](https://microsoft.github.io/BotFramework-WebChat/12.customization-minimizable-web-chat)                 |
| [`13.customization-speech-ui`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/13.customization-speech-ui)                                                                                                   | 高级教程：演示如何完全自定义机器人的关键组件（此示例中是语音），这将完全取代基于文本的脚本 UI，改为显示一个带有机器人响应的简单语音按钮。      | [语音 UI 演示](https://microsoft.github.io/BotFramework-WebChat/13.customization-speech-ui)                                       |
| [`14.customization-piping-to-redux`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/14.customization-piping-to-redux)                                                                                       | 高级教程：演示如何将机器人活动通过管道传递到你自己的 Redux 存储，以及如何使用机器人通过机器人活动和 Redux 来控制页面。                                                                          | [通过管道传递到 Redux 的演示](https://microsoft.github.io/BotFramework-WebChat/14.customization-piping-to-redux)                           |
| [`15.a.backchannel-piggyback-on-outgoing-activities`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/15.a.backchannel-piggyback-on-outgoing-activities)                                                     | 高级教程：演示如何将自定义数据添加到每个传出活动。                                                                                                                                                | [反向通道捎带演示](https://microsoft.github.io/BotFramework-WebChat/15.a.backchannel-piggyback-on-outgoing-activities) |
| [`15.b.incoming-activity-event`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/15.b.incoming-activity-event)                                                                                               | 高级教程：演示如何将所有传入活动转发到 JavaScript 事件以进行进一步处理。                                                                                                                | [传入活动演示](https://microsoft.github.io/BotFramework-WebChat/15.b.incoming-activity-event)                             |
| [`15.c.programmatic-post-activity`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/15.c.programmatic-post-activity)                                                                                         | 高级教程：演示如何以编程方式发送消息。                                                                                                                                                             | [以编程方式发布的演示](https://microsoft.github.io/BotFramework-WebChat/15.c.programmatic-post-activity)                       |
| [`15.d.backchannel-send-welcome-event`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/15.d.backchannel-send-welcome-event)                                                                                 | 高级教程：演示如何使用客户端功能（例如浏览器语言）发送欢迎事件。                                                                                                                        | [欢迎事件演示](https://microsoft.github.io/BotFramework-WebChat/15.d.backchannel-send-welcome-event)                          |
| [`16.customization-selectable-activity`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/16.customization-selectable-activity)                                                                               | 高级教程：演示如何将自定义单击行为添加到每个活动。                                                                                                                                                  | [可选择的活动演示](https://microsoft.github.io/BotFramework-WebChat/16.customization-selectable-activity)                   |
| [`17.chat-send-history`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/17.chat-send-history)                                                                                                               | 高级教程：演示保存用户输入并允许用户通过之前发送的消息返回的功能。                                                                                                      | [聊天发送历史记录演示](https://microsoft.github.io/BotFramework-WebChat/17.chat-send-history)                                     |
| [`18.customization-open-url`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/18.customization-open-url)                                                                                                     | 高级教程：演示如何自定义打开 URL 行为。                                                                                                                                                             | [自定义打开 URL 的演示](https://microsoft.github.io/BotFramework-WebChat/18.customization-open-url)                               |
| [`19.a.single-sign-on-for-enterprise-apps`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/19.a.single-sign-on-for-enterprise-apps)                                                                         | 演示如何通过 OAuth 将单一登录用于企业应用                                                                                                                                                              | [通过 OAuth 进行单一登录](https://microsoft.github.io/BotFramework-WebChat/19.a.single-sign-on-for-enterprise-apps)         |
 

## <a name="web-chat-api-reference"></a>Web Chat API 参考

你可能会将几个属性传递到 Web Chat React 组件 (`<ReactWebChat>`) 或 `renderWebChat()` 方法。 请随时检查以 [`packages/component/src/Composer.js`](https://github.com/Microsoft/BotFramework-WebChat/blob/master/packages/component/src/Composer.js#L378) 开头的源代码。 下面是可用属性的简短说明。

| 属性                   | 说明                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| -------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `activityMiddleware`       | 以 [Redux 中间件](https://medium.com/@jacobp100/you-arent-using-redux-middleware-enough-94ffe991e6)为模型的中间件链，可使开发人员在当前现有的活动 DOM 上添加新的 DOM 组件。 该中间件签名如下：`options => next => card => children => next(card)(children)`。                                                                                                                                                                                                                                           |
| `activityRenderer`         | “平展”版 `activityMiddleware`，类似于 Redux 中的[存储增强器](https://github.com/reduxjs/redux/blob/master/docs/Glossary.md#store-enhancer)概念。                                                                                                                                                                                                                                                                                                                                                                                                                |
| `adaptiveCardHostConfig`   | 在自定义的 Adaptive Cards 主机配置中传递。请务必通过正在使用的 Adaptive Cards 版本验证主机配置。 请参阅[自定义主机配置](https://github.com/microsoft/BotFramework-WebChat/issues/2034#issuecomment-501818238)，获取有关详细信息。                                                                                                                                                                                                                                                                                                                                    |
| `attachmentMiddleware`     | 可使开发人员在附件上添加其自定义 HTML 元素的中间件链。 该签名如下：`options => next => card => next(card)`。                                                                                                                                                                                                                                                                                                                                                                                                                  |
| `attachmentRenderer`       | "平展"版 `attachmentMiddleware`。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| `cardActionMiddleware`     | 可使开发人员修改卡操作（如 Adaptive Cards 或建议的操作）的中间件链。 该中间件签名如下：`cardActionMiddleware: () => next => ({ cardAction, getSignInUrl }) => next(cardAction)`                                                                                                                                                                                                                                                                                                                                           |
| `createDirectLine`         | 用于实例化 Direct Line 对象的工厂方法。 Azure 政府用户应使用 `createDirectLine({ domain: 'https://directline.botframework.azure.us/v3/directline', token });` 更改终结点。 参数的完整列表为：`conversationId`、`domain`、`fetch`、`pollingInterval`、`secret`、`streamUrl`、`token`、`watermark` 和 `webSocket`。                                                                                                                                                                                                                         |
| `createStore`              | 可使开发人员修改存储操作的中间件链。 该中间件签名如下：`createStore: ({}, ({ dispatch }) => next => action => next(cardAction)`                                                                                                                                                                                                                                                                                                                                                                                                |
| `directLine`               | 指定具有 DirectLine 令牌的 DirectLine 对象。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| `disabled`                 | 禁用 Web Chat 的 UI（即对于演示模式）。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| `grammars`                 | 指定语音（必应语音或认知服务语音服务）的语法列表。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| `groupTimeStamp`           | 更改时间戳分组的默认设置。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| `locale`                   | 指示 Web Chat 的默认语言。 强烈建议使用四个字母的代码（例如 `en-US`）。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| `renderMarkdown`           | 更改默认 Markdown 呈现器对象。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| `sendTypingIndicator`      | 向机器人显示来自用户的键入信号，以指示用户不空闲。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| `store`                    | 指定自定义存储（例如，用于将编程活动添加到机器人）。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| `styleOptions`             | 为 Web Chat 样式存储自定义值的对象。 要获取（经常更新的）默认样式选项的完整列表，请参阅 [defaultStyleOptions.js](https://github.com/Microsoft/BotFramework-WebChat/blob/master/packages/component/src/Styles/defaultStyleOptions.js) 文件。                                                                                                                                                                                                                                                                              |
| `styleSet`                 | 不推荐的替代样式方法。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| `userID`                   | 指定 userID。 有两种方法可以指定 `userID`：在属性中，或在生成令牌调用 (`createDirectLine()`) 时的令牌中。 如果使用这两种方法指定 userID，将使用令牌 userID 属性，并且运行时会出现 `console.warn`。 如果 `userID` 是通过属性提供的，但带有前缀 `'dl'`（例如 `'dl_1234'`），则会引发该值并生成新的 `ID`。 如果没有指定 `userID`，则将默认为随机的用户 ID。 不建议多个用户共享同一个用户 ID，因为其用户状态也将共享。 |
| `username`                 | 指定用户名。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| `webSpeechPonyFillFactory` | 指定用于文本转语音和语音转文本的 Web 语音对象。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
## <a name="browser-compatibility"></a>浏览器兼容性
Web Chat 支持新式浏览器（如 Chrome、Microsoft Edge 和 FireFox）的最近 2 个版本。
如果需要在 Internet Explorer 11 中使用 Web Chat，请参阅 [ES5 捆绑bao 演示](https://microsoft.github.io/BotFramework-WebChat/01.b.getting-started-es5-bundle)。

但是，请注意：
- Web Chat 不支持 11 版以前的 Internet Explorer 版本
- Internet Explorer 不支持非 ES5 示例中显示的自定义项。 因为 IE11 是非新式浏览器，所以不支持 ES6，并且很多使用箭头函数和新式承诺的示例可能需要手动转换为 ES5。  如果应用需要进行大量自定义，我们强烈建议你开发适用于新式浏览器（如 Google Chrome 或 Microsoft Edge）的应用。
- Web Chat 不打算支持 IE11 (ES5) 的示例。
   - 对于想要手动重写其他示例以便在 IE11 中工作的客户，建议使用填充代码和 transpiler（如 [`babel`](https://babeljs.io/docs/en/next/babel-standalone.html)）查看从 ES6+ 到 ES5 的代码转换。

## <a name="how-to-test-with-web-chats-latest-bits"></a>如何使用 Web Chat 的最新位进行测试

 此时只能通过 MyGet 包装测试未发布的功能。

如果想要测试尚未发布的功能或 bug 修补程序，需要将 Web Chat 包指向 Web Chat 每日源，而不是官方 npmjs 源。

目前，可以通过订阅我们的 MyGet 源来访问 Web Chat 的日报。 为此，需要更新你的项目中的注册表。  此更改是可逆的，我们的指导包括如何还原到订阅正式版本。

### <a name="subscribe-to-latest-bits-on-mygetorg"></a>在 `myget.org` 上订阅最新位

为此，可以添加包，然后更改项目注册表。

1. 添加项目依赖项而非 Web Chat。
1. 在项目的根目录中，创建 `.npmrc` 文件
1. 将以下行添加到你的文件：`registry=https://botbuilder.myget.org/F/botframework-webchat/npm/`
1. 将 Web Chat 添加到项目依赖项 `npm i botframework-webchat --save`
1. 请注意，在 `package-lock.json` 中，注册表现在指向 MyGet。 Web Chat 项目已启用上游源代理，这会将非 MyGet 包重定向到 `npmjs.com`。

### <a name="re-subscribe-to-official-release-on-npmjscom"></a>在 `npmjs.com` 上重新订阅正式版本
需要重置注册表才能重新订阅。

1. 删除 `.npmrc file`
1. 删除根 `package-lock.json`
1. 删除 `node_modules` 目录
1. 使用 `npm i` 重新安装包
1. 请注意，在 `package-lock.json` 中，注册表再次指向 https://npmjs.com/ 。


## <a name="contributing"></a>供稿

请参阅[参与页](https://github.com/Microsoft/BotFramework-WebChat/tree/master/.github/CONTRIBUTING.md)，详细了解如何生成项目，以及用于拉取请求的存储库指南。

本项目采用 [Microsoft 开源行为准则](https://opensource.microsoft.com/codeofconduct/)。
有关详细信息，请参阅[行为准则常见问题](https://opensource.microsoft.com/codeofconduct/faq/)；若要其他任何问题或意见，请联系 [opencode@microsoft.com](mailto:opencode@microsoft.com)。

## <a name="reporting-security-issues"></a>报告安全问题

应通过电子邮件以非公开方式向 Microsoft 安全响应中心 (MSRC) [secure@microsoft.com](mailto:secure@microsoft.com) 报告安全问题和 bug。 你应该会在 24 小时内收到答复。 如果因为某种原因，你未收到答复，请通过电子邮件跟进以确保我们收到原始邮件。 详细信息（包括 [MSRC PGP](https://technet.microsoft.com/security/dn606155) 密钥）可在[安全技术中心](https://technet.microsoft.com/security/default)中找到。

版权所有 (c) Microsoft Corporation。 保留所有权利。