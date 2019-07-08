---
title: 在应用中嵌入机器人 | Microsoft Docs
description: 了解如何设计嵌入到另一个应用中的机器人。
author: matvelloso
ms.author: mateusv
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 08/15/2018
ms.openlocfilehash: 5a0ded9af5f624398df764f16e6dd2db0105255c
ms.sourcegitcommit: a295a90eac461f8b96770dd902ba44919acf33fc
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/26/2019
ms.locfileid: "67405875"
---
# <a name="embed-a-bot-in-an-app"></a>在应用中嵌入机器人

虽然机器人通常存在于应用外部，但它们也可以与应用集成。 例如，可以在应用中嵌入[知识机器人](~/bot-service-design-pattern-knowledge-base.md)，以帮助用户查找在复杂应用结构中可能难以找到的信息。 也可以在技术支持应用中嵌入机器人，作为传入用户请求的第一响应方。 机器人可以独立解决简单问题，将更复杂的问题[移交](~/bot-service-design-pattern-handoff-human.md)给人工处理。 

## <a name="integrating-bot-with-app"></a>将机器人与应用集成

机器人与应用集成的方式因应用类型而异。 

### <a name="native-mobile-app"></a>本机移动应用

在本机代码中创建的应用可使用 [Direct Line API][directLineAPI] 通过 REST 或 WebSocket 与 Bot Framework 进行通信。

### <a name="web-based-mobile-app"></a>基于 Web 的移动应用

使用 Web 语言和框架（如 <a href="https://cordova.apache.org/" target="_blank">Cordova</a>）生成的移动应用可通过[嵌入在网站中的机器人](~/bot-service-design-pattern-embed-web-site.md)将使用的且仅封装在本机应用的 shell 中的组件与 Bot Framework 进行通信。

### <a name="iot-app"></a>IoT 应用

IoT 应用可使用 [Direct Line API][directLineAPI] 与 Bot Framework 通信。 某些情况下，它还可能使用 <a href="https://www.microsoft.com/cognitive-services/" target="_blank">Microsoft 认知服务</a>启用图像识别和语音等功能。

### <a name="other-types-of-apps-and-games"></a>其他类型的应用和游戏

其他类型的应用和游戏可使用 [Direct Line API][directLineAPI] 与 Bot Framework 通信。 

## <a name="creating-a-cross-platform-mobile-app-that-runs-a-bot"></a>创建运行机器人的跨平台移动应用

创建运行机器人的移动应用的本次示例使用 <a href="https://www.xamarin.com/" target="_blank">Xamarin</a>，这是一种用于生成跨平台移动应用程序的常用工具。 

首先，创建一个简单的 Web 视图组件，将其用于托管 <a href="https://github.com/Microsoft/BotFramework-WebChat" target="_blank">Web 聊天控件</a>。 然后，使用 Azure 门户添加网上聊天通道。 

接下来，在 Xamarin 应用中将注册的网上聊天 URL 指定为 Web 视图控件的源：

```cs
public class WebPage : ContentPage
{
    public WebPage()
    {
        var browser = new WebView();
        browser.Source = "https://webchat.botframework.com/embed/<YOUR SECRET KEY HERE>";
        this.Content = browser;
    }
}
```

使用此过程，可创建跨平台移动应用程序，该应用程序使用网上聊天控件呈现嵌入式 Web 视图。

![反向通道](~/media/bot-service-design-pattern-embed-app/xamarin-apps.png)

<!-- TODO: No sample bot available
## Sample code

For a complete sample that shows how to create a cross-platform mobile app that runs a bot (as described in this article), see the <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/capability-BotInApps" target="_blank">Bot in Apps sample</a> in GitHub.
-->

## <a name="additional-resources"></a>其他资源

- [Direct Line API][directLineAPI]
- <a href="https://www.microsoft.com/cognitive-services/" target="_blank">Microsoft 认知服务</a>

[directLineAPI]: https://docs.botframework.com/restapi/directline3/#navtitle
