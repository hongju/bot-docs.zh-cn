---
title: 为机器人服务配置持续部署 | Microsoft Docs
description: 了解如何从源代码管理为机器人服务设置持续部署。
keywords: 持续部署, 发布, 部署, azure 门户
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 09/18/2018
ms.openlocfilehash: 62cbbcc560e049776b8aa891c167b9a6eaba3264
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997794"
---
# <a name="set-up-continuous-deployment"></a>设置连续部署
如果你的代码已签入 **GitHub** 或 **Azure DevOps（以前称为 Visual Studio Team Services）**，请使用持续部署自动将代码更改从源存储库部署到 Azure。 在本主题中，我们将介绍如何为 **GitHub** 和 **Azure DevOps** 设置持续部署。

> [!NOTE]
> 设置持续部署后，Azure 门户中的联机代码编辑器将变为“只读”。

## <a name="continuous-deployment-using-github"></a>使用 GitHub 进行持续部署

要使用 GitHub 设置持续部署，请执行以下操作：

1. 使用包含要部署到 Azure 的源代码的 GitHub 存储库。 可以[创建](https://help.github.com/articles/fork-a-repo/)现有存储库的分支或创建自己的存储库，并将相关的源代码上传到 GitHub 存储库。
2. 在 [Azure 门户](https://portal.azure.com)中，转到机器人的“生成”边栏选项卡并单击“配置持续部署”。 
3. 单击“设置”。
   
   ![设置连续部署](~/media/azure-bot-build/continuous-deployment-setup.png)

4. 单击“选择源”并选择“GitHub”。

   ![选择“GitHub”](~/media/azure-bot-build/continuous-deployment-setup-github.png)

5. 单击“授权”，然后单击“授权”按钮并按照提示授予 Azure 访问 GitHub 帐户的权限。

6. 单击“选择项目”并选择一个项目。

7. 单击“选择分支”并选择一个分支。

8. 单击“确定”完成设置过程。

现在，已使用 GitHub 设置完成持续部署。 每当提交到源代码存储库时，你的更改将自动部署到 Azure 机器人服务。

## <a name="continuous-deployment-using-azure-devops"></a>使用 Azure DevOps 进行持续部署

1. 在 [Azure 门户](https://portal.azure.com)中，从机器人的“生成”边栏选项卡单击“配置持续部署”。 
2. 单击“设置”。
   
   ![设置连续部署](~/media/azure-bot-build/continuous-deployment-setup.png)

3. 单击“选择源”并选择“Visual Studio Team Services”。 请记住，Visual Studio Team Services 现在是 Azure DevOps Services。

   ![选择“Visual Studio Team Services”](~/media/azure-bot-build/continuous-deployment-setup-vs.png)

4. 单击“选择帐户”并选择一个帐户。

> [!NOTE]
> 如果没有看到帐户，请确保它已链接到 Azure 订阅。 若要将帐户链接到 Azure 订阅，请转到 Azure 门户，打开  **Azure DevOps Services 组织（以前称为 Team Services）**。 你将看到你在 Azure DevOps 中拥有的组织列表。 单击进入包含要部署的机器人源代码的那个组织，将找到“连接 AAD”按钮。 如果所选的组织未链接到 Azure 订阅，则此按钮将处于活动状态。 因此，单击此按钮以设置连接。 你可能需要等待一段时间才能生效。

5. 单击“选择项目”并选择一个项目。

> [!NOTE]
> 仅支持 VSTS Git 项目。

6. 单击“选择分支”并选择一个分支。
7. 单击“确定”完成设置过程。

   ![Visual Studio 配置](~/media/azure-bot-build/continuous-deployment-setup-vs-configuration.png)

现在，已使用 Azure DevOps 设置完成持续部署。 每次提交时，所做的更改都将自动部署到 Azure。

## <a name="disable-continuous-deployment"></a>禁用持续部署

机器人配置为持续部署时，不可以使用联机代码编辑器对机器人进行更改。 如果要使用连接代码编辑器，可以暂时禁用持续部署。

要禁用持续部署，请执行以下操作：

1. 在机器人的“生成”边栏选项卡中，单击“配置持续部署”。 
2. 单击“断开连接”以禁用持续部署。 若要重新启用持续部署，请重复上述相应部分中的步骤。

## <a name="additional-information"></a>其他信息
- [Azure DevOps](https://docs.microsoft.com/en-us/azure/devops/?view=vsts)
