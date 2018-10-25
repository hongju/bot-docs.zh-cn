---
title: 识别消息内容中的意向 | Microsoft Docs
description: 了解如何通过使用正则表达式或检查消息内容来识别用户的意向。
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 82541c4ac0848c3e995ab3ad1ed874436072fe63
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998075"
---
# <a name="recognize-user-intent-from-message-content"></a>识别消息内容中的用户意向

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

机器人接收来自用户的消息后，可使用识别器检查该消息并确定意向。 意向提供了从消息到要调用的对话的映射。 本文介绍如何使用正则表达式或通过检查消息内容识别意向。 例如，机器人可使用正则表达式检查消息是否包含“帮助”一词，并调用帮助对话。 机器人还可以检查用户消息的属性（例如查看用户发送了图像还是文本），并调用图像处理对话。 

> [!NOTE]
> 要了解如何使用 LUIS 识别意向，请参阅[使用 LUIS 识别意向和实体](bot-builder-nodejs-recognize-intent-luis.md) 


## <a name="use-the-built-in-regular-expression-recognizer"></a>使用内置的正则表达式识别器
通过 [RegExpRecognizer][RegExpRecognizer] 使用正则表达式检测用户的意向。 可将多个表达式传递给识别器，以支持多种语言。 

> [!TIP]
> 要详细了解如何实现机器人的本地化，请参阅[支持本地化](bot-builder-nodejs-localization.md)。

以下代码会创建名为 `CancelIntent` 的正则表达式识别器，并将其添加到机器人。 此示例中的识别器为 `en_us` 和 `ja_jp` 这两个区域设置均提供了正则表达式。 

[!code-js[Add a regular expression recognizer (JavaScript)](../includes/code/node-regex-recognizer.js#addRegexRecognizer)]

将识别器添加到机器人后，请将 [triggerAction][triggerAction] 附加到希望机器人在识别器检测到意向时调用的对话。 使用[匹配][matches]选项指定意向名称，如以下代码中所示：

[!code-js[Map the CancelIntent recognizer to a cancel dialog (JavaScript)](../includes/code/node-regex-recognizer.js#bindCancelDialogToRegexRecognizer)]

意向识别器是全局性的，这表示识别器将运行检查从用户处接收的每条消息。 如果识别器使用 `triggerAction` 检测到绑定到对话的意向，它会触发中断当前活动对话。 允许和处理中断这一设计非常灵活，可帮助理解用户真正要执行的操作。

> [!TIP] 
> 要了解 `triggerAction` 如何与对话配合使用，请参阅[管理聊天流](bot-builder-nodejs-manage-conversation-flow.md)。 要详细了解可与识别的意向相关联的各种操作，请参阅[处理用户操作](bot-builder-nodejs-dialog-actions.md)。

## <a name="register-a-custom-intent-recognizer"></a>注册自定义意向识别器
此外，还可以实现自定义识别器。 以下示例添加了一个简单的识别器，它期待用户说出“帮助”或“再见”，但你也可轻松添加能进行更复杂处理的识别器，例如用户发送图像时进行识别的识别器。 


[!code-js[Add a custom recognizer (JavaScript)](../includes/code/node-howto-recognize-intent.js#addCustomRecognizer)]

注册识别器后，可使用 `matches` 子句将识别器与操作相关联。

[!code-js[Bind intents to actions (JavaScript)](../includes/code/node-howto-recognize-intent.js#bindIntentsToActions)]

## <a name="disambiguate-between-multiple-intents"></a>区分多个意向

机器人可以注册多个识别器。 请注意，自定义识别器示例涉及到将数值分数分配给每个意向。 这是因为机器人可能有多个识别器，并且 Bot Builder SDK 提供了内置逻辑来区分多个识别器返回的意向。 分配给意向的得分通常介于 0.0 和 1.0 之间，但自定义识别器可定义大于 1.1 的意向，从而确保 Bot Builder SDK 的区分逻辑始终会选择该意向。 

默认情况下，识别器以并行方式运行，但你可设置 [IIntentRecognizerSetOptions][IntentRecognizerSetOptions] 中的 recognizeOrder，以便在机器人找到得分为 1.0 的意向时进程立即退出。

Bot Builder SDK 包括一个[示例][DisambiguationSample]，用于演示如何通过实现 [IDisambiguateRouteHandler][IDisambiguateRouteHandler] 在机器人中提供自定义区分逻辑。

## <a name="next-steps"></a>后续步骤
使用正则表达式和检查消息内容的逻辑可能会变复杂，尤其是在机器人采用开放式聊天流的情况下。 要帮助机器人处理来自用户的各种文本和语音输入，可使用意向识别服务（如 [LUIS][LUIS]）向机器人添加自然语言理解。

> [!div class="nextstepaction"]
> [使用 LUIS 识别意向和实体](bot-builder-nodejs-recognize-intent-luis.md)


[LUIS]: https://www.luis.ai/

[triggerAction]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#triggeraction

[matches]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.itriggeractionoptions.html#matches

[node-js-bot-how-to]: bot-builder-nodejs-recognize-intent-luis.md

[LUISAzureDocs]: /azure/cognitive-services/LUIS/Home

[IMessage]: http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.imessage

[IntentRecognizerSetOptions]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.iintentrecognizersetoptions.html

[LuisRecognizer]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.luisrecognizer

[LUISSample]: https://aka.ms/v3-js-luisSample

[LUISConcepts]: https://docs.botframework.com/en-us/node/builder/guides/understanding-natural-language/

[DisambiguationSample]: https://aka.ms/v3-js-onDisambiguateRoute

[IDisambiguateRouteHandler]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.idisambiguateroutehandler.html

[RegExpRecognizer]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.regexprecognizer.html

[AlarmBot]: https://aka.ms/v3-js-luisSample
