---
title: 在机器人服务中创建机器人通道注册 | Microsoft Docs
description: 了解如何在机器人服务中注册现有机器人。
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 6a32bc5712937c615962e4f6edfc7ea691d3ec39
ms.sourcegitcommit: 67445b42796d90661afc643c6bb6533e9a662cbc
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/06/2018
ms.locfileid: "39574973"
---
# <a name="register-a-bot-with-bot-service"></a>在机器人服务中注册机器人

[!INCLUDE [pre-release-label](includes/pre-release-label-v3.md)]

如果已在其他位置托管了机器人，并且希望使用机器人服务将它连接到其他通道，则需要在机器人服务中注册机器人。 在本主题中，了解如何通过创建“机器人通道注册”机器人服务，在机器人服务中注册机器人。

> [!IMPORTANT] 
> 如果机器人未托管在 Azure 中，则只需注册机器人。 如果已通过 Azure 门户[创建了机器人](bot-service-quickstart.md)，那么机器人已在机器人服务中注册。

## <a name="log-in-to-azure"></a>登录 Azure
登录到 [Azure 门户](http://portal.azure.com)。

> [!TIP]
> 如果尚无订阅，可注册<a href="https://azure.microsoft.com/en-us/free/" target="_blank">免费帐户</a>。

## <a name="create-a-bot-channels-registration"></a>创建机器人通道注册
使用机器人服务功能需要“机器人通道注册”机器人服务。 注册机器人可让你将机器人连接到通道。

要创建“机器人通道注册”，请执行以下操作：

1. 单击 Azure 门户左上角的“新建”按钮，然后选择“AI + 认知服务”>“机器人通道注册”。 

2. 此时会打开一个包含有关“机器人通道注册”信息的新边栏选项卡。 单击“创建”按钮以启动创建过程。 

3. 在“机器人服务”边栏选项卡中，提供有关机器人的请求信息，如图片下方的表中所示。  <br/>
   ![“创建注册机器人”边栏选项卡](~/media/azure-bot-quickstarts/registration-create-bot-service-blade.png)


   |                    设置                     |         建议的值         |                                                                                                  Description                                                                                                  |
   |------------------------------------------------|---------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
   |           <strong>机器人名称</strong>            |     机器人的显示名称     |                                                  通道和目录中显示的机器人的显示名称。 此名称可随时更改。                                                  |
   |         <strong>订阅</strong>          |        订阅        |                                                                                选择要使用的 Azure 订阅。                                                                                 |
   |        <strong>资源组</strong>         |         myResourceGroup         |                                 可创建新的[资源组](/azure/azure-resource-manager/resource-group-overview#resource-groups)或从现有资源组中选择。                                  |
   |                    位置                    |             美国西部             |                                                        选择靠近部署机器人的位置，或选择机器人将访问的其他服务附近的位置。                                                         |
   |         <strong>定价层</strong>          |               F0                |             选择定价层。 可随时更新定价层。 有关详细信息，请参阅[机器人服务定价](https://azure.microsoft.com/en-us/pricing/details/bot-service/)。              |
   |      <strong>消息传送终结点</strong>       |               代码               |                                                                               输入机器人消息传送终结点的 URL。                                                                                |
   |     <strong>Application Insights</strong>      |               启用                | 决定要启用还是关闭 [Application Insights](bot-service-manage-analytics.md)。 如果选择“启用”，还必须指定区域位置。 |
   | <strong>Microsoft 应用 ID 和密码</strong> | 自动创建应用 ID 和密码 |              如果需要手动输入 Microsoft 应用 ID 和密码，请使用此选项。 否则，在机器人创建过程中系统会为你创建新的 Microsoft 应用 ID 和密码。               |


4. 单击“创建”创建服务并注册机器人的消息传送终结点。

通过查看“通知”确认注册已创建。 通知将从“正在进行部署...”更改为“部署已成功”。 单击“转到资源”按钮，打开机器人的资源边栏选项卡。 

## <a name="bot-channels-registration-password"></a>机器人通道注册密码

“机器人通道注册”机器人服务没有与之关联的应用服务。 因此，此机器人服务只有 MicrosoftAppID。 需要手动生成密码并自行保存。 如果要使用[模拟器](bot-service-debug-emulator.md)测试机器人，则需要此密码。

要生成 MicrosoftAppPassword，请执行以下操作：

1. 在“设置”边栏选项卡中，单击“管理”。 这是“Microsoft 应用 ID”显示的链接。 此链接会打开一个窗口，在其中可生成新密码。 <br/>
  ![管理“设置”边栏选项卡中的链接](~/media/azure-bot-quickstarts/registration-settings-manage-link.png)

2. 单击“生成新密码”。 此操作会为机器人生成新密码。 复制此密码，并将其保存到文件中。 你只有在这一次能够看到此密码。 如果未保存完整密码，在以后需要密码时，需重复此过程以创建新密码。 <br/>
  ![生成 Microsoft 应用密码](~/media/azure-bot-quickstarts/registration-generate-app-password.png)

## <a name="update-the-bot"></a>更新机器人

如果使用 Bot Builder SDK for Node.js，请设置以下环境变量：

* MICROSOFT_APP_ID
* MICROSOFT_APP_PASSWORD

如果使用 Bot Builder SDK for .NET，请在 web.config 文件中设置以下键值：

* MicrosoftAppId
* MicrosoftAppPassword

## <a name="test-the-bot"></a>测试机器人

创建机器人服务后，请[在网上聊天中测试它](bot-service-manage-test-webchat.md)。 输入一条信息，机器人就会响应。

## <a name="next-steps"></a>后续步骤

在本主题中，了解了如何在机器人服务中注册托管机器人。 下一步将了解如何管理机器人服务。

> [!div class="nextstepaction"]
> [管理机器人](bot-service-manage-overview.md)

