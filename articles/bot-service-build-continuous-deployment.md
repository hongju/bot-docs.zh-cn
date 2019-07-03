---
title: 为机器人服务配置持续部署 | Microsoft Docs
description: 了解如何从源代码管理为机器人服务设置持续部署。
keywords: 持续部署, 发布, 部署, azure 门户
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 8d324138db60c2b34f9bebd3ff53c30a12c3cefa
ms.sourcegitcommit: a295a90eac461f8b96770dd902ba44919acf33fc
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/26/2019
ms.locfileid: "67405950"
---
# <a name="set-up-continuous-deployment"></a>设置连续部署

[!INCLUDE [applies-to](./includes/applies-to.md)]

本文展示了如何为机器人配置持续部署。 可以启用持续部署，以便自动将代码更改从源存储库部署到 Azure。 在本主题中，我们将介绍如何为 GitHub 设置持续部署。 若要了解如何通过其他源代码管理系统来设置持续部署，请参阅此页底部的其他资源部分。

## <a name="prerequisites"></a>先决条件
- 如果没有 Azure 订阅，请在开始之前创建一个[免费帐户](http://portal.azure.com)。
- **必须**[将机器人部署到 Azure](bot-builder-deploy-az-cli.md)，然后才能启用持续部署。

## <a name="prepare-your-repository"></a>准备存储库
确保存储库根路径具有项目中的正确文件。 这样即可从 Azure 应用服务 Kudu 生成服务器获取自动生成。 

|运行时 | 根目录文件 |
|:-------|:---------------------|
| ASP.NET Core | .sln 或 .csproj |
| Node.js | server.js、app.js 或具有启动脚本的 package.json |


## <a name="continuous-deployment-using-github"></a>使用 GitHub 进行持续部署
若要使用 GitHub 实现持续部署，请在 Azure 门户中导航至机器人的“应用服务”页。 

单击“部署中心” > “GitHub” > “授权”。   

![持续部署](~/media/azure-bot-build/azure-deployment.png)

在打开的浏览器窗口中，单击“授权 AzureAppService”  。 

![Azure Github 权限](~/media/azure-bot-build/azure-deployment-github.png)

授权 **AzureAppService** 以后，返回到 Azure 门户中的“部署中心”。 

1. 单击“继续”。  

1. 选择“应用服务生成服务”。 

1. 单击“继续”。 

1. 选择“组织”、“存储库”和“分库”。   

1. 单击“继续”，然后单击“完成”  以完成设置。 

此时就设置好了通过 GitHub 进行的持续部署。 每当提交到源代码存储库时，你的更改将自动部署到 Azure 机器人服务。

## <a name="disable-continuous-deployment"></a>禁用持续部署

机器人配置为持续部署时，不可以使用联机代码编辑器对机器人进行更改。 如果要使用连接代码编辑器，可以暂时禁用持续部署。

要禁用持续部署，请执行以下操作：
1. 在 [Azure 门户](https://portal.azure.com)中，转到机器人的“所有应用服务设置”边栏选项卡，并单击“部署中心”。   
1. 单击“断开连接”以禁用持续部署  。 若要重新启用持续部署，请重复上述相应部分中的步骤。

## <a name="additional-resources"></a>其他资源
- 若要从 BitBucket 和 Azure DevOps Services 启用持续部署，请参阅[使用 Azure 应用服务进行持续部署](https://docs.microsoft.com/azure/app-service/deploy-continuous-deployment)。


