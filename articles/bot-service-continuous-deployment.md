---
title: 通过源代码管理或 Visual Studio 发布机器人服务 | Microsoft Docs
description: 了解如何通过 Visual Studio 一次性地或通过源代码管理连续地发布“机器人服务”机器人。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 980ee1c6e4e54ff3f74ccfa618b0aeff7b507815
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/18/2018
ms.locfileid: "39298160"
---
# <a name="publish-a-bot-to-bot-service"></a>将机器人发布到机器人服务

更新 C# 机器人源代码后，可以使用 Visual Studio 将其发布到机器人服务中正在运行的机器人。 每次将源文件签入源代码管理服务时，还可以自动发布在任何集成开发环境 (IDE) 中编写的 C# 或 Node.js 机器人源代码。


## <a name="publish-a-bot-on-app-service-plan-from-the-online-code-editor"></a>通过联机代码编辑器发布应用服务计划上的机器人

如果尚未配置持续部署，则可以在联机代码编辑器中修改源文件。 要部署已修改的源，请执行以下步骤。

4. 单击“打开控制台”图标。  
    ![控制台图标](~/media/azure-bot-service-console-icon.png)
2. 在控制台窗口中，键入 build.cmd，然后按 Enter 键。


## <a name="publish-c-bot-on-app-service-plan-from-visual-studio"></a>通过 Visual Studio 发布应用服务计划上的 C# 机器人 

要使用 `.PublishSettings` 文件通过 Visual Studio 设置发布，请执行以下步骤：

1. 在 Azure 门户中，单击“机器人服务”，再单击“生成”选项卡，然后单击“下载 zip 文件”。
3. 将下载的 zip 文件的内容提取到本地文件夹。
4. 在资源管理器中，找到机器人的 Visual Studio 解决方案 (.sln) 文件并双击它。
4. 在 Visual Studio 中，单击“视图”，然后单击“解决方案资源管理器”。
5. 在“解决方案资源管理器”窗格中，右键单击你的项目，单击“发布...”“发布”窗口随即打开。 
6. 在“发布”窗口中，单击“创建新的配置文件”，单击“导入配置文件”，再单击“确定”。
7. 导航到项目文件夹，再导航到“PostDeployScripts”文件夹，选择以 `.PublishSettings` 结尾的文件，然后单击“打开”。

此项目的发布现已配置完成。 要将本地源代码发布到机器人服务，右键单击你的项目，单击“发布...”，然后单击“发布”按钮。 

## <a name="set-up-continuous-deployment"></a>设置连续部署

默认情况下，机器人服务可让你使用 Azure 编辑器直接在浏览器中开发机器人，而无需任何本地编辑器或源代码管理。 但是，Azure 编辑器不允许你管理应用程序中的文件（例如，添加文件、重命名文件或删除文件）。 如果希望能够管理应用程序中的文件，则可以设置持续部署，并使用所选的集成开发环境 (IDE) 和源代码管理（例如，Visual Studio Team、GitHub、Bitbucket）。 配置持续部署后，提交到源代码管理的任何代码更改都将自动部署到 Azure。 配置持续部署后，可以[本地调试机器人](bot-service-debug-bot.md)。

> [!NOTE]
> 如果为机器人启用持续部署，则必须签入源代码管理服务的代码更改。 如果想要再次在 Azure 编辑器中编辑代码，则必须[禁用持续部署](#disable-continuous-deployment)。

通过完成以下步骤，可以为机器人启用持续部署。

## <a name="set-up-continuous-deployment-for-a-bot-on-an-app-service-plan"></a>为应用服务计划上的机器人设置持续部署

本部分介绍如何为使用机器人服务创建的具有应用服务托管计划的机器人启用持续部署。

1. 在 Azure 门户中，找到 Azure 机器人，单击“生成”选项卡，并查找“通过源代码管理进行持续部署”部分。
2. 对于 Visual Studio Online 或 GitHub，请在这些网站上提供给你发的访问令牌。 会将你的源从 Azure 拉到源存储库。
3. 对于其他源代码管理系统，请选择“其他”并按照显示的步骤操作。 
3. 单击“启用”。  

### <a name="create-an-empty-repository-and-download-bot-source-code"></a>创建一个空存储库并下载机器人源代码

如果想要使用除 Visual Studio Online 或 Github 之外的源代码管理服务，请执行以下步骤。 Visual Studio Online 和 Github 会从 Azure 拉取机器人的源代码，因此使用这两个服务的用户可以跳过这些步骤。

3. 对于应用服务计划中的机器人，在 Azure 上查找机器人页面，单击“生成”选项卡，再查找“下载源代码”部分，然后单击“下载 zip 文件”。
1. 在 Azure 支持的某个源控制系统中创建空存储库。

    ![源代码管理系统](~/media/continuous-integration-sourcecontrolsystem.png)

3. 将下载的 zip 文件的内容解压到计划在其中同步部署源的本地文件夹。
4. 单击“配置”并按照显示的步骤操作。 

## <a name="set-up-continuous-deployment-for-a-bot-on-a-consumption-plan"></a>为消耗计划上的机器人设置持续部署 

选择机器人的部署源并连接存储库。 

1. 在 Azure 门户上的 Azure 机器人中，单击“设置”选项卡，然后单击“配置”展开“持续部署”部分。  
2. 按照步骤操作，选中确认准备就绪的复选框。 
3. 单击“配置”，选择与先前在其中创建空存储库的源代码管理系统相对应的部署源，并完成连接它的步骤。   


## <a name="disable-continuous-deployment"></a>禁用持续部署 

禁用持续部署后，源代码管理服务将继续运行，但已签入的更改不会自动发布到 Azure。 要禁用持续部署，请执行以下步骤：

1. 如果机器人具有应用服务托管计划，则在 Azure 门户中找到 Azure 机器人，单击“生成”选项卡，并查找“通过源代码管理进行持续部署”部分、“或...” 
2. 如果机器人具有消耗计划，请单击“设置”项卡，展开“持续部署”部分，然后单击“配置”。
3. 在“部署”窗格中，选择启用了持续部署的源代码管理服务，然后单击“断开连接”。  


## <a name="additional-resources"></a>其他资源

要了解如何在配置了持续部署后本地调试机器人，请参阅[调试机器人服务机器人](bot-service-debug-bot.md)。

本文着重介绍了机器人服务的特定持续部署功能。 若要了解与 Azure 应用服务相关的持续部署，请参阅<a href="https://azure.microsoft.com/en-us/documentation/articles/app-service-continuous-deployment/" target="_blank">持续部署到 Azure 应用服务</a>。
