---
title: 为机器人服务配置持续部署 | Microsoft Docs
description: 了解如何从源代码管理为机器人服务设置持续部署。
keywords: 持续部署, 发布, 部署, azure 门户
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 03/08/2018
ms.openlocfilehash: 596d264c4df72959c71ab353e5038175fc2bed31
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/18/2018
ms.locfileid: "39297865"
---
# <a name="set-up-continuous-deployment"></a>设置连续部署

通过持续部署，可以在本地开发机器人。 将机器人签入到 GitHub 或 Visual Studio Team Services 等源代码管理中时，持续部署非常有用。 返回源存储库查看更改时，所做的更改将自动部署到 Azure。

> [!NOTE]
> 设置持续部署后，[联机代码编辑器](bot-service-build-online-code-editor.md)变为“只读”。

本主题将演示如何为 GitHub 和 Visual Studio Team Services 设置持续部署。

## <a name="continuous-deployment-using-github"></a>使用 GitHub 进行持续部署

要使用 GitHub 设置持续部署，请执行以下操作：

1. [创建](https://help.github.com/articles/fork-a-repo/)包含要部署到 Azure 的代码的 GitHub 存储库分支。
2. 在 Azure 门户中，转到机器人的“生成”边栏选项卡并单击“配置持续部署”。 
3. 单击“设置”。
   
   ![设置连续部署](~/media/azure-bot-build/continuous-deployment-setup.png)

4. 单击“选择资源”并选择“GitHub”。

   ![选择“GitHub”](~/media/azure-bot-build/continuous-deployment-setup-github.png)

5. 单击“授权”，然后单击“授权”按钮并按照提示授予 Azure 访问 GitHub 帐户的权限。

已使用 GitHub 设置完成持续部署。 每次提交时，所做的更改都将自动部署到 Azure。

## <a name="continuous-deployment-using-visual-studio"></a>使用 Visual Studio 进行持续部署

1. 在机器人的“生成”边栏选项卡中，单击“配置持续部署”。 
2. 单击“设置”。
   
   ![设置连续部署](~/media/azure-bot-build/continuous-deployment-setup.png)

3. 单击“选择源”并选择“Visual Studio Team Services”。

   ![选择“Visual Studio Team Services”](~/media/azure-bot-build/continuous-deployment-setup-vs.png)

4. 单击“选择帐户”并选择一个帐户。

> [!NOTE]
> 如果没有看到帐户，请确保它已链接到 Azure 订阅。
> 有关详细信息，请参阅[将 VSTS 帐户链接到 Azure 订阅](https://github.com/projectkudu/kudu/wiki/Setting-up-a-VSTS-account-so-it-can-deploy-to-a-Web-App#linking-your-vsts-account-to-your-azure-subscription)。

5. 单击“选择项目”并选择一个项目。

> [!NOTE]
> 仅支持 VSTS Git 项目。

6. 单击“选择分支”并选择要进行分支的分支。
7. 单击“确定”完成设置过程。

   ![Visual Studio 配置](~/media/azure-bot-build/continuous-deployment-setup-vs-configuration.png)

已使用 Visual Studio Team Services 设置完成持续部署。 每次提交时，所做的更改都将自动部署到 Azure。

## <a name="disable-continuous-deployment"></a>禁用持续部署

机器人配置为持续部署时，不可以使用联机代码编辑器对机器人进行更改。 如果要使用连接代码编辑器，可以暂时禁用持续部署。

要禁用持续部署，请执行以下操作：

1. 在机器人的“生成”边栏选项卡中，单击“配置持续部署”。 
2. 单击“断开连接”以禁用持续部署。 若要重新启用持续部署，请重复上述相应部分中的步骤。

## <a name="next-steps"></a>后续步骤
机器人已设置为持续部署后，使用联机网上聊天测试代码。

> [!div class="nextstepaction"]
> [通过网上聊天执行测试](bot-service-manage-test-webchat.md)
