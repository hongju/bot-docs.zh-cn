---
title: 向消息添加输入提示 | Microsoft Docs
description: 了解如何使用 Bot Framework SDK 向消息添加输入提示。
keywords: 输入提示, 接受输入, 预期输入, 忽略输入, 语音
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 08/24/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 751d5067d2e4b6b6ad21e1a4fd0ccb3818385d06
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/11/2019
ms.locfileid: "54224872"
---
# <a name="add-input-hints-to-messages"></a>向消息添加输入提示

[!INCLUDE [pre-release-label](~/includes/pre-release-label.md)]

通过为消息指定输入提示，可以指示在将消息传送到客户端后，机器人是接受、预期还是忽略用户输入。 对于许多通道，这使客户端可以相应地设置用户输入控件的状态。 例如，如果消息的输入提示指示机器人忽略用户输入，则客户端可以关闭麦克风并禁用输入框以防止用户提供输入。

确保为输入提示包含必要的库。

# <a name="ctabcs"></a>[C#](#tab/cs)

```cs
using Microsoft.Bot.Schema;
```

<!--TODO: Remove the following remark after the next release of the NuGet packages.-->

在这些示例中使用的 MessageFactory 类在以下命名空间中定义。

```cs
using Microsoft.Bot.Builder.Core.Extensions;
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

```javascript
const {InputHints, MessageFactory} = require('botbuilder');
```

---

## <a name="accepting-input"></a>接受输入

若要指示机器人已被动准备好输入但不等待用户的响应，请将消息的输入提示设置为“接受输入”。 在许多通道上，这将导致客户端的输入框启用，麦克风关闭但仍可供用户访问。 例如，如果用户按住麦克风按钮，Cortana 将打开麦克风以接受用户的输入。 以下代码可创建一条消息，指示机器人正在接受用户输入。

# <a name="ctabcs"></a>[C#](#tab/cs)

```csharp
var reply = MessageFactory.Text(
    "This is the text that will be displayed.",
    "This is the text that will be spoken.",
    InputHints.AcceptingInput);
await context.SendActivityAsync(reply).ConfigureAwait(false);
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

```javascript
const basicMessage = MessageFactory.text('This is the text that will be displayed.', 'This is the text that will be spoken.', InputHints.AcceptingInput);
await context.sendActivity(basicMessage);
```

---

## <a name="expecting-input"></a>期待输入

若要指示机器人正在等待用户的响应，请将消息的输入提示设置为“预期输入”。 在许多通道上，这将导致客户端的输入框启用，麦克风打开。 下面的代码示例可创建一条消息，指示机器人期待用户输入。

# <a name="ctabcs"></a>[C#](#tab/cs)

```csharp
var reply = MessageFactory.Text(
    "This is the text that will be displayed.",
    "This is the text that will be spoken.",
    InputHints.ExpectingInput);
await context.SendActivityAsync(reply).ConfigureAwait(false);
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

```javascript
const basicMessage = MessageFactory.text('This is the text that will be displayed.', 'This is the text that will be spoken.', InputHints.ExpectingInput);
await context.sendActivity(basicMessage);
```

---

## <a name="ignoring-input"></a>忽略输入

若要指示机器人没有准备好接收用户的输入，请将消息的输入提示设置为“忽略输入”。 在许多通道上，这将导致客户端的输入框禁用，麦克风关闭。 下面的代码示例可创建一条消息，指示机器人忽略用户输入。

# <a name="ctabcs"></a>[C#](#tab/cs)

```csharp
var reply = MessageFactory.Text(
    "This is the text that will be displayed.",
    "This is the text that will be spoken.",
    InputHints.IgnoringInput);
await context.SendActivityAsync(reply).ConfigureAwait(false);
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

```javascript
const basicMessage = MessageFactory.text('This is the text that will be displayed.', 'This is the text that will be spoken.', InputHints.IgnoringInput);
await context.sendActivity(basicMessage);
```

---

## <a name="default-values-for-input-hint"></a>输入提示的默认值

如果未设置消息的输入提示，Bot Framework SDK 将使用以下逻辑自动进行设置：

- 如果机器人发送提示，则消息的输入提示将指定机器人期待输入。</li>
- 如果机器人发送单条消息，则消息的输入提示将指定机器人正在接受输入。</li>
- 如果机器人发送了一系列连续的消息，那么系列中除最后消息之外的所有消息的输入提示将指定机器人正在忽略输入，并且系列中最后消息的输入提示将指定机器人正在接受输入。

