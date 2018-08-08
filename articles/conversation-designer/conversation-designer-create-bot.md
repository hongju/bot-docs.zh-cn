---
title: 创建聊天设计器机器人 | Microsoft Docs
description: 了解如何使用聊天设计器创建新的机器人。
author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ROBOTS: NoIndex, NoFollow
ms.openlocfilehash: 1b33d08e56bf8ae473b28cf1cf3a26d8138ce861
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/18/2018
ms.locfileid: "39297774"
---
# <a name="create-a-new-conversation-designer-bot"></a>创建新的聊天设计器机器人
> [!IMPORTANT]
> 聊天设计器尚未可供所有客户使用。 关于聊天设计器可用性的更多详细信息将在今年晚些时候发布。

本教程将介绍创建新的聊天设计器机器人的分步说明。 

## <a name="prerequisites"></a>先决条件

- 聊天设计器需要 Azure 订阅。 可以从<a href="https://azure.microsoft.com/en-us/" target="_blank">这里</a>开始
- 使用它们创建帐户之后，请确保至少登录 [LUIS 门户](https://luis.ai)一次（如果尚未这样做）。
- 熟悉 JavaScript 编程。 自定义脚本函数以 JavaScript 编写。
- Microsoft Edge 或 Google Chrome

## <a name="create-a-conversation-designer-bot"></a>创建聊天设计器机器人

若要创建聊天设计器机器人，请执行以下步骤：
1. 转到 https://dev.botframework.com/ 并登录。 使用你提供给 Microsoft Corporation 的电子邮件地址参与此个人预览版。
2. 单击右上方导航面板中的“创建机器人”。 
3. 单击“创建”，以使用聊天设计器创建机器人。
4. 首先，从许多[示例机器人](conversation-designer-sample-bots.md)中选择一个。 单击“下一步”。 如果不确定要使用哪个**示例机器人**，只需选择你认为最接近要构建的机器人的示例。 以后可以切换到其他**示例机器人**。
5. 完成所有字段并单击“创建机器人” - 完成机器人预配大约需要 2 分钟。 

## <a name="bot-provisioning"></a>机器人预配

创建聊天设计器机器人时，将自动预配以下 Azure 功能： 

1. 具有你指定的机器人名称的 Azure 资源组
2. Azure 应用服务
3. Azure 应用服务计划 
4. Azure 存储帐户
5. Application Insights 
6. [LUIS.ai](https://luis.ai) 的认知服务订阅。 使用**机器人句柄**（加上随机生成的字符串）作为应用名称创建 LUIS 应用。
7. Microsoft 帐户单个应用。 [了解详细信息](https://apps.dev.microsoft.com/#/appList)

## <a name="welcome-message"></a>欢迎消息

预配机器人后，聊天设计器将打开机器人的“生成”页。 将显示一条欢迎消息，其中包含可帮助你入门的信息。 浏览这些选项或关闭消息并开始使用机器人。 可以通过单击左上角导航面板中的省略号 (**...**)，然后选择“欢迎...”选项来返回到欢迎消息。

## <a name="next-step"></a>后续步骤
> [!div class="nextstepaction"]
> [保存机器人](conversation-designer-save-bot.md)

## <a name="additional-resources"></a>其他资源
* 了解[任务](conversation-designer-tasks.md)
* 详细了解 [Bot Builder SDK for Node](../nodejs/index.md) 
