---
title: Web Chat 自定义 | Microsoft Docs
description: 了解如何自定义 Bot Framework Web Chat。
keywords: 机器人框架, webchat, 聊天, 示例, react, 引用
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 06/07/2019
ms.openlocfilehash: 80d16cda7bb51aecdbe0406d4d2b07a6c6275182
ms.sourcegitcommit: abd5b8c89e0928e03e5928b64c160c92cce03d70
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/21/2019
ms.locfileid: "67313195"
---
# <a name="web-chat-customization"></a>Web Chat 自定义

本文将详细介绍如何自定义 Web Chat 示例使其适应机器人。

## <a name="integrate-web-chat-into-your-website"></a>将 Web Chat 集成到网站

按照[概述页面](bot-builder-webchat-overview.md)上的说明将 Web Chat 控件集成到网站。

## <a name="customizing-styles"></a>自定义样式

Web Chat 控件的最新版本提供了丰富的自定义选项：可以更改颜色、大小、元素的位置，添加自定义元素并与托管网页进行交互。 下面是几个关于如何自定义这些 Web Chat UI 元素的示例。

可以在 [`defaultStyleOptions.js` 文件](https://github.com/Microsoft/BotFramework-WebChat/blob/master/packages/component/src/Styles/defaultStyleOptions.js)上找到可以在 Web Chat 中轻松修改的所有设置的完整列表。

这些设置会生成一个样式集  ，这是一套使用 [glamor](https://github.com/threepointone/glamor) 进行增强的 CSS 规则。 可以在 [`createStyleSet.js` 文件](https://github.com/Microsoft/BotFramework-WebChat/blob/master/packages/component/src/Styles/createStyleSet.js)上找到在样式集中生成的 CSS 样式的完整列表。

## <a name="set-the-size-of-the-web-chat-container"></a>设置 Web Chat 容器的大小

现可使用 `styleSetOptions` 调整 Web Chat 容器的大小。 以下示例背景色为 `paleturquoise``body`，以显示 Web Chat 容器（具有白色背景的部分）。

```js
…
<head>
  <style>
    html, body { height: 100% }
    body {
      margin: 0;
      background-color: paleturquoise;
    }

    #webchat {
      height: 100%;
      width: 100%;
    }
  </style>
</head>
<body>
  <div id="webchat" role="main"></div>
  <script>
    (async function () {
    window.WebChat.renderWebChat({
      directLine: window.WebChat.createDirectLine({ token }),
      styleOptions: {
        rootHeight: '100%',
        rootWidth: '50%'
      }
    }, document.getElementById('webchat'));
    })()
  </script>
…
```

结果如下：

<img alt="Web Chat with root height and root width set" src="https://raw.githubusercontent.com/Microsoft/BotFramework-WebChat/master/media/rootHeightWidth.png" width="600"/>

## <a name="change-font-or-color"></a>更改字体或颜色

可以自定义它们以匹配目标网页的样式，而不是使用聊天气泡内使用的默认背景色和字体。 通过以下代码片段，可更改来自用户和机器人的消息的背景色。

<img alt="Screenshot with custom style options" src="https://raw.githubusercontent.com/Microsoft/BotFramework-WebChat/master/media/sample-custom-style-options.png" width="396" />

如果需要执行一些简单的样式设置，则可以通过 `styleOptions` 来设置。 样式选项是一组可以直接修改的预定义样式，Web Chat 将基于它来计算整个样式表。

```html
<!DOCTYPE html>
<html>
   <body>
      <div id="webchat" role="main"></div>
      <script src="https://cdn.botframework.com/botframework-webchat/latest/webchat.js"></script>
      <script>
         const styleOptions = {
            bubbleBackground: 'rgba(0, 0, 255, .1)',
            bubbleFromUserBackground: 'rgba(0, 255, 0, .1)'
         };

         window.WebChat.renderWebChat(
            {
               directLine: window.WebChat.createDirectLine({
                  secret: 'YOUR_BOT_SECRET'
               }),

               // Passing 'styleOptions' when rendering Web Chat
               styleOptions
            },
            document.getElementById('webchat')
         );
      </script>
   </body>
</html>
```

## <a name="change-the-css-manually"></a>手动更改 CSS

除了颜色，还可以修改用于呈现消息的字体：

<img alt="Screenshot with custom style set" src="https://raw.githubusercontent.com/Microsoft/BotFramework-WebChat/master/media/sample-custom-style-set.png" width="396" />

此外，还可以通过直接设置 CSS 规则手动修改样式集，进行更深入的样式设置。

> 由于 CSS 规则与 DOM 树结构紧密耦合，因此可能需要更新这些规则，以使用 Web Chat 的更新版本。

```html
<!DOCTYPE html>
<html>
   <body>
      <div id="webchat" role="main"></div>
      <script src="https://cdn.botframework.com/botframework-webchat/latest/webchat.js"></script>
      <script>
         // "styleSet" is a set of CSS rules which are generated from "styleOptions"
         const styleSet = window.WebChat.createStyleSet({
            bubbleBackground: 'rgba(0, 0, 255, .1)',
            bubbleFromUserBackground: 'rgba(0, 255, 0, .1)'
         });

         // After generated, you can modify the CSS rules
         styleSet.textContent = {
            ...styleSet.textContent,
            fontFamily: "'Comic Sans MS', 'Arial', sans-serif",
            fontWeight: 'bold'
         };

         window.WebChat.renderWebChat(
            {
               directLine: window.WebChat.createDirectLine({
                  secret: 'YOUR_BOT_SECRET'
               }),

               // Passing 'styleSet' when rendering Web Chat
               styleSet
            },
            document.getElementById('webchat')
         );
      </script>
   </body>
</html>
```

## <a name="change-the-avatar-of-the-bot-within-the-dialog-box"></a>在对话框中更改机器人的虚拟形象

最新的 Web Chat 支持虚拟形象，你可以使用 `botAvatarInitials` 和 `userAvatarInitials` 属性自定义它们。

<img alt="Screenshot with avatar initials" src="https://raw.githubusercontent.com/Microsoft/BotFramework-WebChat/master/media/sample-avatar-initials.png" width="396" />

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
                  secret: 'YOUR_BOT_SECRET'
               }),

               // Passing avatar initials when rendering Web Chat
               botAvatarInitials: 'BF',
               userAvatarInitials: 'WC'
            },
            document.getElementById('webchat')
         );
      </script>
   </body>
</html>
```

在 `renderWebChat` 代码内部，我们添加了 `botAvatarInitials` 和 `userAvatarInitials`：

```js
botAvatarInitials: 'BF',
userAvatarInitials: 'WC'
```

`botAvatarInitials` 将文本设置为位于虚拟形象内左侧。 如果将它设置为假值，则会隐藏机器人一侧的虚拟形象。 与此相反，`userAvatarInitials` 会将虚拟形象文本设置为位于右侧。

## <a name="custom-rendering-activity-or-attachment"></a>自定义呈现活动或附件

使用 Web Chat 的最新版本，还可以呈现 Web Chat 不现成支持的活动或附件。 以 [Redux 中间件](https://redux.js.org/api/applymiddleware)为模型的自定义管道发送活动和附件呈现。 管道非常灵活，可使你轻松地执行以下任务：

-  修饰现有活动/附件
-  添加新的活动/附件
-  替换现有活动/附件（或将其删除）
-  用菊花链将中间件组合在一起

### <a name="show-github-repository-as-an-attachment"></a>将 GitHub 存储库显示为附件

如果想要显示一套 GitHub 存储库卡，可以为 GitHub 存储库创建新的 React 组件，并将其添加为附件的中间件。

<img alt="Screenshot with custom GitHub repository attachment" src="https://raw.githubusercontent.com/Microsoft/BotFramework-WebChat/master/media/sample-custom-github-repository-attachment.png" width="396" />

```jsx
import ReactWebChat from 'botframework-webchat';
import ReactDOM from 'react-dom';

// Create a new React component that accept render a GitHub repository attachment
const GitHubRepositoryAttachment = props => (
   <div
      style={{
         fontFamily: "'Calibri', 'Helvetica Neue', Arial, sans-serif",
         margin: 20,
         textAlign: 'center'
      }}
   >
      <svg
         height="64"
         viewBox="0 0 16 16"
         version="1.1"
         width="64"
         aria-hidden="true"
      >
         <path
            fillRule="evenodd"
            d="M8 0C3.58 0 0 3.58 0 8c0 3.54 2.29 6.53 5.47 7.59.4.07.55-.17.55-.38 0-.19-.01-.82-.01-1.49-2.01.37-2.53-.49-2.69-.94-.09-.23-.48-.94-.82-1.13-.28-.15-.68-.52-.01-.53.63-.01 1.08.58 1.23.82.72 1.21 1.87.87 2.33.66.07-.52.28-.87.51-1.07-1.78-.2-3.64-.89-3.64-3.95 0-.87.31-1.59.82-2.15-.08-.2-.36-1.02.08-2.12 0 0 .67-.21 2.2.82.64-.18 1.32-.27 2-.27.68 0 1.36.09 2 .27 1.53-1.04 2.2-.82 2.2-.82.44 1.1.16 1.92.08 2.12.51.56.82 1.27.82 2.15 0 3.07-1.87 3.75-3.65 3.95.29.25.54.73.54 1.48 0 1.07-.01 1.93-.01 2.2 0 .21.15.46.55.38A8.013 8.013 0 0 0 16 8c0-4.42-3.58-8-8-8z"
         />
      </svg>
      <p>
         <a
            href={`https://github.com/${encodeURI(props.owner)}/${encodeURI(
               props.repo
            )}`}
            target="_blank"
         >
            {props.owner}/<br />
            {props.repo}
         </a>
      </p>
   </div>
);

// Creating a new middleware pipeline that will render <GitHubRepositoryAttachment> for specific type of attachment
const attachmentMiddleware = () => next => card => {
   switch (card.attachment.contentType) {
      case 'application/vnd.microsoft.botframework.samples.github-repository':
         return (
            <GitHubRepositoryAttachment
               owner={card.attachment.content.owner}
               repo={card.attachment.content.repo}
            />
         );

      default:
         return next(card);
   }
};

ReactDOM.render(
   <ReactWebChat
      // Prepending the new middleware pipeline
      attachmentMiddleware={attachmentMiddleware}
      directLine={window.WebChat.createDirectLine({ token })}
   />,
   document.getElementById('webchat')
);
```

可在 [customization-card-components 示例](https://github.com/microsoft/BotFramework-WebChat/tree/master/samples/10.a.customization-card-components)中找到完整示例。

在此示例中，我们将添加名为 `GitHubRepositoryAttachment` 的新 React 组件：

```jsx
const GitHubRepositoryAttachment = props => (
   <div
      style={{
         fontFamily: "'Calibri', 'Helvetica Neue', Arial, sans-serif",
         margin: 20,
         textAlign: 'center'
      }}
   >
      <svg
         height="64"
         viewBox="0 0 16 16"
         version="1.1"
         width="64"
         aria-hidden="true"
      >
         <path
            fillRule="evenodd"
            d="M8 0C3.58 0 0 3.58 0 8c0 3.54 2.29 6.53 5.47 7.59.4.07.55-.17.55-.38 0-.19-.01-.82-.01-1.49-2.01.37-2.53-.49-2.69-.94-.09-.23-.48-.94-.82-1.13-.28-.15-.68-.52-.01-.53.63-.01 1.08.58 1.23.82.72 1.21 1.87.87 2.33.66.07-.52.28-.87.51-1.07-1.78-.2-3.64-.89-3.64-3.95 0-.87.31-1.59.82-2.15-.08-.2-.36-1.02.08-2.12 0 0 .67-.21 2.2.82.64-.18 1.32-.27 2-.27.68 0 1.36.09 2 .27 1.53-1.04 2.2-.82 2.2-.82.44 1.1.16 1.92.08 2.12.51.56.82 1.27.82 2.15 0 3.07-1.87 3.75-3.65 3.95.29.25.54.73.54 1.48 0 1.07-.01 1.93-.01 2.2 0 .21.15.46.55.38A8.013 8.013 0 0 0 16 8c0-4.42-3.58-8-8-8z"
         />
      </svg>
      <p>
         <a
            href={`https://github.com/${encodeURI(props.owner)}/${encodeURI(
               props.repo
            )}`}
            target="_blank"
         >
            {props.owner}/<br />
            {props.repo}
         </a>
      </p>
   </div>
);
```

然后，在机器人发送内容类型附件 `application/vnd.microsoft.botframework.samples.github-repository` 时，创建可呈现新 React 组件的中间件。 否则，它将通过调用 `next(card)` 继续存在于中间件上。

```jsx
const attachmentMiddleware = () => next => card => {
   switch (card.attachment.contentType) {
      case 'application/vnd.microsoft.botframework.samples.github-repository':
         return (
            <GitHubRepositoryAttachment
               owner={card.attachment.content.owner}
               repo={card.attachment.content.repo}
            />
         );

      default:
         return next(card);
   }
};
```

发送自机器人的活动如下所示：

```json
{
   "type": "message",
   "from": {
      "role": "bot"
   },
   "attachmentLayout": "carousel",
   "attachments": [
      {
         "contentType": "application/vnd.microsoft.botframework.samples.github-repository",
         "content": {
            "owner": "Microsoft",
            "repo": "BotFramework-WebChat"
         }
      },
      {
         "contentType": "application/vnd.microsoft.botframework.samples.github-repository",
         "content": {
            "owner": "Microsoft",
            "repo": "BotFramework-Emulator"
         }
      },
      {
         "contentType": "application/vnd.microsoft.botframework.samples.github-repository",
         "content": {
            "owner": "Microsoft",
            "repo": "BotFramework-DirectLineJS"
         }
      }
   ]
}
```
