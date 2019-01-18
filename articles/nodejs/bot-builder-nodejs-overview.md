---
title: Bot Framework SDK for Node.js | Microsoft Docs
description: 了解 Bot Framework SDK for Node.js 这一功能强大且易于使用的机器人构建框架。
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 2e25237b616810f5ef10442fec41834568afcb59
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/11/2019
ms.locfileid: "54224733"
---
# <a name="bot-framework-sdk-for-nodejs"></a>Bot Framework SDK for Node.js

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-overview.md)
> - [Node.js](../nodejs/bot-builder-nodejs-overview.md)
> - [REST](../rest-api/bot-framework-rest-overview.md)

Bot Framework SDK for Node.js 是一种功能强大且易于使用的框架，为 Node.js 开发人员提供了一种熟悉的方法来编写机器人。
可以使用它来构建各种聊用户界面（无论是简单的提示，还是不限格式的聊天）。

机器人的聊天逻辑作为 Web 服务托管。 Bot Framework SDK 使用 <a href="http://restify.com">restify</a>（一种用于构建 Web 服务的常用框架）来创建机器人的 Web 服务器。 SDK 也与 <a href="http://expressjs.com/">Express</a> 兼容，进行一些调整后还可使用其他 Web 应用框架。 

借助 SDK，可以利用以下 SDK 功能： 

- 用于构建对话以封装聊天逻辑的强大系统。
- 是/否、字符串、数字和枚举等简单事项的内置提示，以及对消息（包含图像与附件）和资讯卡（包含按钮）的支持。
- 对强大 AI 框架（如 <a href="http://luis.ai" target="_blank">LUIS</a>）的内置支持。
- 内置识别器和事件处理程序，引导用户完成聊天，根据需要提供帮助、导航、说明和确认。

## <a name="get-started"></a>入门

如果对编写机器人不熟悉，请按照分步说明[使用 Node.js 创建第一个机器人](bot-builder-nodejs-quickstart.md)，这些说明可帮助你设置项目、安装 SDK 和运行第一个机器人。 

如果对 Bot Framework SDK for Node.js 不熟悉，可以先掌握关键概念，了解 Bot Framework SDK 主要组件，具体请参阅[关键概念](bot-builder-nodejs-concepts.md)。

为确保机器人能够解决最常见的用户问题，请查看[设计原则](../bot-service-design-principles.md)和[探索模式](../bot-service-design-pattern-task-automation.md)获取指导。

## <a name="get-samples"></a>获取示例

[Bot Framework SDK for Node.js 示例](bot-builder-nodejs-samples.md)演示了以任务为中心的机器人，这些机器人展示了如何利用 Bot Framework SDK for Node.js 的功能。 这些示例可助你快速入门，构建具有丰富功能的强大机器人。

## <a name="next-steps"></a>后续步骤
> [!div class="nextstepaction"]
> [关键概念](bot-builder-nodejs-concepts.md)

## <a name="additional-resources"></a>其他资源

以下以任务为中心的操作指南演示了 Bot Framework SDK for Node.js 的各种功能。

* [响应消息](bot-builder-nodejs-use-default-message-handler.md)
* [处理用户操作](bot-builder-nodejs-dialog-actions.md)
* [识别用户意向](bot-builder-nodejs-recognize-intent-messages.md)
* [发送资讯卡](bot-builder-nodejs-send-rich-cards.md)
* [发送附件](bot-builder-nodejs-send-receive-attachments.md)
* [保存用户数据](bot-builder-nodejs-save-user-data.md)


如果遇到问题或者希望提供有关 Bot Framework SDK for Node.js 的建议，请参阅[客户支持](../bot-service-resources-links-help.md)，获取可用资源的列表。 


[DesignGuide]: ../bot-service-design-principles.md 
[DesignPatterns]: ../bot-service-design-pattern-task-automation.md 
[HowTo]: bot-builder-nodejs-use-default-message-handler.md 
