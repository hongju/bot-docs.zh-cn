---
title: 机器人服务的工作方式 | Microsoft Docs
description: 了解机器人服务的特性和功能。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: bdc86e5e64971e503157fe69a8b962e1d9b88542
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998860"
---
# <a name="how-bot-service-works"></a>机器人服务的工作方式

机器人服务提供了用于创建机器人的核心组件，包括用于开发机器人的 Bot Builder SDK 和用于将机器人连接到通道的 Bot Framework。 机器人服务提供了五个模板，你可以在创建支持 .NET 和 Node.js 的机器人时选择这些模板。

> [!IMPORTANT]
> 你必须拥有 Microsoft Azure 订阅才能使用机器人服务。 如果尚无订阅，可注册<a href="https://azure.microsoft.com/en-us/free/" target="_blank">免费帐户</a>。

## <a name="hosting-plans"></a>托管计划
机器人服务为机器人提供两个托管计划。 你可以选择满足需求的托管计划。

### <a name="app-service-plan"></a>应用服务计划

使用应用服务计划的机器人是一个标准的 Azure Web 应用，你可以将其设置为以可预测成本和缩放来分配预定义的容量。 借助使用此托管计划的机器人，你可以：

* 使用高级浏览器内代码编辑器联机编辑机器人源代码。
* 使用 Visual Studio 下载、调试和重新发布 C# 机器人。
* 为 Visual Studio Online 和 Github 轻松设置持续部署。
* 使用为 Bot Builder SDK 准备的示例代码。

### <a name="consumption-plan"></a>消耗量计划
使用消耗计划的机器人是在 <a href="http://go.microsoft.com/fwlink/?linkID=747839" target="_blank">Azure Functions</a> 上运行的无服务器机器人，并使用按运行付费的 Azure Functions 定价。 使用此托管计划的机器人可以扩展以处理巨大的流量峰值。 可以使用基本的浏览器内代码编辑器联机编辑机器人源代码。 有关消耗计划机器人的运行时环境的更多信息，请参阅 <a target='_blank' href='/azure/azure-functions/functions-scale'>Azure Functions 消耗计划和应用服务计划</a>。

## <a name="templates"></a>模板

机器人服务使你可以使用五个模板之一快速轻松地在 C# 或 Node.js 中创建机器人。

[!INCLUDE [Bot Service templates](~/includes/snippet-abs-templates.md)]

[详细了解](bot-service-concept-templates.md)不同模板以及它们如何帮助你生成机器人。

## <a name="develop-and-deploy"></a>开发和部署

默认情况下，机器人服务可让你使用联机代码编辑器直接在浏览器中开发机器人，而无需任何工具链。 

可以使用 Bot Builder SDK 和 IDE（例如 Visual Studio 2017）在本地开发和调试机器人。 可以使用 Visual Studio 2017 或 Azure CLI 将机器人直接发布到 Azure。 还可以使用所选的源代码管理系统（例如 VSTS 或 GitHub）[设置持续部署](bot-service-continuous-deployment.md)。 配置持续部署后，你可以在本地计算机上的 IDE 中进行开发和调试，并且你提交给源代码管理的任何代码更改都会自动部署到 Azure。  

> [!TIP]
> 启用持续部署后，请确保仅通过持续部署修改代码，而不是通过其他机制来修改代码以避免冲突。

## <a name="manage-your-bot"></a>管理机器人 

在使用机器人服务创建机器人的过程中，可以为机器人指定名称、设置其托管计划、选择定价层，以及配置其他一些设置。 创建机器人后，你可以更改其设置，将其配置为在一个或多个通道上运行，并通过网上聊天对其进行测试。 

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [使用机器人服务创建机器人](bot-service-quickstart.md)