---
title: 将机器人部署到 Azure | Microsoft Docs
description: 将机器人部署到 Azure 云。
keywords: 部署机器人, azure 部署, 机器人通道注册, 发布 visual studio
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: get-started-article
ms.prod: bot-framework
ms.date: 09/19/2018
ms.openlocfilehash: 1503d5e349839de6f3403ad9b769c7eb0ae41e78
ms.sourcegitcommit: 3bf3dbb1a440b3d83e58499c6a2ac116fe04b2f6
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/23/2018
ms.locfileid: "46706963"
---
# <a name="deploy-your-bot-to-azure"></a>将机器人部署到 Azure

[!INCLUDE [pre-release-label](./includes/pre-release-label.md)]

创建机器人并在本地对其进行验证后，可以将其推送到 Azure，以便可以从任何位置访问它。 对于 C# 机器人，可以使用 Visual Studio 或 Azure CLI 将机器人发布到 Azure。 

## <a name="publish-from-visual-studio"></a>从 Visual Studio 发布
你将首先在应用服务中从 Visual Studio 将机器人部署到 Azure。 然后，你将使用机器人通道注册通过 Azure 机器人服务配置机器人。

在“解决方案资源管理器”窗口中右键单击项目的节点，然后选择“发布”。

![发布设置](media/azure-bot-quickstarts/getting-started-publish-setting.png)

2. 在“选取发布目标”对话框中，确保在左侧选择了“应用服务”，并在右侧选择了“新建”。

3. 单击“发布”按钮。

4. 在对话框的右上角，确保对话框显示 Azure 订阅的正确用户 ID。

![发布主要信息](media/azure-bot-quickstarts/getting-started-publish-main.png)

5. 输入“应用名称”、“订阅”、“资源组”和“托管计划”信息。

6. 准备就绪后，单击“创建”。 完成此过程可能需要几分钟时间。

7. 完成后，将打开 Web 浏览器，其中显示机器人的公共 URL。

8. 复制此 URL（它将类似于 https://<yourbotname>.azurewebsites.net/）。

> [!NOTE] 
> 注册机器人时，需要使用 HTTPS 版本的 URL。 Azure 通过 Azure 应用服务提供 SSL 支持。

## <a name="create-your-bot-channels-registration"></a>创建机器人通道注册
在 Azure 中部署机器人后，需要将其注册到 Azure 机器人服务。

1. 访问 https://portal.azure.com 处的 Azure 门户。

2. 使用先前在 Visual Studio 中使用的相同标识登录以发布机器人。

3. 单击“创建资源”。

4. 在“搜索市场”字段中，键入“机器人通道注册”，然后按 Enter。

5. 在返回的列表中，选择“机器人通道注册”：

![发布](media/azure-bot-quickstarts/getting-started-bot-registration.png)

6. 在打开的边栏选项卡中单击“创建”。

7. 为机器人提供名称。

8. 选择部署了机器人代码的同一订阅。

9. 选择将设置位置的现有资源组。

10. 可以选择 F0 定价层进行开发和测试。

11. 输入机器人的 URL。 请确保以 HTTPS 开头并添加 /api/messages，例如 https://yourbotname.azurewebsites.net/api/messages

12. 暂时关闭 Application Insights。

13. 单击“Microsoft 应用 ID 和密码”

14. 在新的边栏选项卡中，单击“新建”。

15. 在右侧打开的新边栏选项卡中，单击“在应用注册门户中创建应用 ID”，该门户将在新的浏览器标签页中打开。

![机器人 MSA](media/azure-bot-quickstarts/getting-started-msa.png)

16. 在新标签页中，复制应用 ID 并将其保存在某处。 

17. 单击“生成应用密码以继续”按钮。

18. 将打开一个浏览器对话框，并为你提供应用的密码，这是你获得密码的唯一机会。 复制此密码并将其保存在以后可以访问的地方。

19. 保存密码后，单击“确定”。

20. 只需关闭浏览器标签页并返回到 Azure 门户标签页。

21. 在正确的字段中粘贴“应用 ID”和“密码”，然后单击“确定”。

22. 现在单击“创建”以设置通道注册。 这可能需要几秒钟到几分钟。

## <a name="update-your-bots-application-settings"></a>更新机器人的应用程序设置
为了让机器人使用 Azure 机器人服务进行身份验证，需要在 Azure 应用服务中将两个设置添加到机器人的应用程序设置。 

1. 单击“应用程序服务”。 在“订阅”文本框中键入机器人名称。 然后单击列表中的机器人名称。

![应用服务](media/azure-bot-quickstarts/getting-started-app-service.png)

2. 在机器人选项中左侧的选项列表中，找到“设置”部分中的“应用程序设置”，然后单击它。

![机器人 ID](media/azure-bot-quickstarts/getting-started-app-settings-1.png)

3. 滚动窗口直到找到“应用程序设置”部分。

![机器人 MSA](media/azure-bot-quickstarts/getting-started-app-settings-2.png)

4. 单击“添加新设置”。

5. 键入 **MicrosoftAppId** 作为名称，并键入应用 ID 作为值。

6. 单击“添加新设置”

7. 键入 **MicrosoftAppPassword** 作为名称，并键入密码作为值。

8. 单击顶部的“保存”按钮。

## <a name="test-your-bot-in-production"></a>在生产环境中测试机器人
此时，可以使用内置的 Web Chat 客户端从 Azure 测试机器人。

1. 返回到 Azure 门户中的“资源组”

2. 打开机器人。

3. 在“机器人管理”下，选择“通过网上聊天执行测试”。

![通过网上聊天执行测试](media/azure-bot-quickstarts/getting-started-test-webchat.png)

4. 键入 `Hi` 之类的消息，然后按 Enter。 机器人将回显 `Turn 1: You sent Hi`。

## <a name="next-steps"></a>后续步骤
> [!div class="nextstepaction"]
> [设置持续部署](bot-service-build-continuous-deployment.md)
