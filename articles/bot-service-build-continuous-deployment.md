---
title: 为机器人服务配置持续部署 | Microsoft Docs
description: 了解如何从源代码管理为机器人服务设置持续部署。
keywords: 持续部署, 发布, 部署, azure 门户
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/06/2018
ms.openlocfilehash: ffbc3ef83c8fe1cd6f04697a3fff9e229df9956f
ms.sourcegitcommit: 080b9633925ffe381f2c3cf11c8f8ca4b37e2046
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/07/2018
ms.locfileid: "53068710"
---
# <a name="set-up-continuous-deployment"></a>设置连续部署
如果你的代码已签入 **GitHub** 或 **Azure DevOps（以前称为 Visual Studio Team Services）**，请使用持续部署自动将代码更改从源存储库部署到 Azure。 在本主题中，我们将介绍如何为 **GitHub** 和 **Azure DevOps** 设置持续部署。

> [!NOTE]
> 本文所述的方案假设你已将机器人部署到 Azure，现在想要为该机器人启用持续部署。 另外，假设在设置持续部署后，Azure 门户中的联机代码编辑器是只读的。

## <a name="continuous-deployment-using-github"></a>使用 GitHub 进行持续部署

若要使用 GitHub 存储库（包含要部署到 Azure 的源代码）设置持续部署，请执行以下操作：

1. 在 [Azure 门户](https://portal.azure.com)中，转到机器人的“所有应用服务设置”边栏选项卡，并单击“部署选项(经典)”。 

1. 单击“选择源”并选择“GitHub”。

   ![选择“GitHub”](~/media/azure-bot-build/continuous-deployment-setup-github.png)

1. 单击“授权”，然后单击“授权”按钮并按照提示授予 Azure 访问 GitHub 帐户的权限。

1. 单击“选择项目”并选择一个项目。

1. 单击“选择分支”并选择一个分支。

1. 单击“确定”完成设置过程。

现在，已使用 GitHub 设置完成持续部署。 每当提交到源代码存储库时，你的更改将自动部署到 Azure 机器人服务。

## <a name="continuous-deployment-using-azure-devops"></a>使用 Azure DevOps 进行持续部署

1. 在 [Azure 门户](https://portal.azure.com)中，转到机器人的“所有应用服务设置”边栏选项卡，并单击“部署选项(经典)”。 
2. 单击“选择源”并选择“Visual Studio Team Services”。 请记住，Visual Studio Team Services 现在是 Azure DevOps Services。

   ![选择“Visual Studio Team Services”](~/media/azure-bot-build/continuous-deployment-setup-vs.png)

3. 单击“选择帐户”并选择一个帐户。

> [!NOTE]
> 如果你的帐户未列出，则需要[将帐户链接到 Azure 订阅](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/connect-organization-to-azure-ad?view=vsts&tabs=new-nav)。 请注意，仅支持 VSTS Git 项目。

4. 单击“选择项目”并选择一个项目。
5. 单击“选择分支”并选择一个分支。
6. 单击“确定”完成设置过程。

   ![Visual Studio 配置](~/media/azure-bot-build/continuous-deployment-setup-vs-configuration.png)

现在，已使用 Azure DevOps 设置完成持续部署。 每次提交时，所做的更改都将自动部署到 Azure。

## <a name="disable-continuous-deployment"></a>禁用持续部署

机器人配置为持续部署时，不可以使用联机代码编辑器对机器人进行更改。 如果要使用连接代码编辑器，可以暂时禁用持续部署。

要禁用持续部署，请执行以下操作：
1. 在 [Azure 门户](https://portal.azure.com)中，转到机器人的“所有应用服务设置”边栏选项卡，并单击“部署选项(经典)”。 
2. 单击“断开连接”以禁用持续部署。 若要重新启用持续部署，请重复上述相应部分中的步骤。

## <a name="additional-information"></a>其他信息
- Visual Studio Team Services 现已命名为 [Azure DevOps Services](https://docs.microsoft.com/en-us/azure/devops/?view=vsts)。
