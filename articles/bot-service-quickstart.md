---
title: 使用机器人服务创建机器人 | Microsoft Docs
description: 了解如何使用机器人服务（一种集成式专用机器人开发环境）创建机器人。
keywords: 快速入门, 创建机器人, 机器人服务, web 应用机器人
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: abs
ms.date: 05/31/2019
ms.openlocfilehash: 9ef15b09c8517db9e8b1f13f72172f09f9fb23eb
ms.sourcegitcommit: dbbfcf45a8d0ba66bd4fb5620d093abfa3b2f725
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/28/2019
ms.locfileid: "67464623"
---
# <a name="create-a-bot-with-azure-bot-service"></a>使用 Azure 机器人服务创建机器人

::: moniker range="azure-bot-service-3.0"

> [!NOTE]
> **从 2019 年 6 月 10 日开始，你将不能在 Azure 门户中创建 V3 SDK 机器人。我们鼓励客户今后创建 [V4 SDK](https://docs.microsoft.com/azure/bot-service/bot-service-quickstart?view=azure-bot-service-4.0) 机器人。[此处](https://docs.microsoft.com/azure/bot-service/bot-service-resources-bot-framework-faq?view=azure-bot-service-3.0#bot-framework-sdk-version-3-lifetime-support)** 提供了有关 V3 SDK 的长期支持的详细信息。 

机器人服务提供了用于创建机器人的核心组件，包括用于开发机器人的 Bot Framework SDK 以及用于将机器人连接到通道的 Bot Framework。 机器人服务提供了五个模板，你可以在创建支持 .NET 和 Node.js 的机器人时选择这些模板。 本主题介绍如何使用机器人服务创建使用 Bot Framework SDK 的新机器人。

## <a name="log-in-to-azure"></a>登录 Azure
登录到 [Azure 门户](http://portal.azure.com)。

## <a name="create-a-new-bot-service"></a>创建新的机器人服务

1. 在 Azure 门户左上角单击“创建新资源”链接，然后选择“AI + 计算机学习”>“Web 应用机器人”。   

2. 此时会打开一个包含有关“Web 应用机器人”信息的新边栏选项卡  。  

3. 在“机器人服务”边栏选项卡中，提供有关机器人的请求信息，如下图中的表所示  。  <br/>
   ![“创建 Web 应用机器人”边栏选项卡](./media/azure-bot-quickstarts/sdk-create-bot-service-blade.png)

   | 设置 | 建议的值 | 说明 |
   | ---- | ---- | ---- |
   | **机器人名称** | 机器人的显示名称 | 通道和目录中显示的机器人的显示名称。 此名称可随时更改。 |
   | **订阅** | 订阅 | 选择要使用的 Azure 订阅。 |
   | **资源组** | myResourceGroup | 可创建新的[资源组](/azure/azure-resource-manager/resource-group-overview#resource-groups)或从现有资源组中选择。 |
   | **位置** | 默认位置 | 选择资源组的地理位置。 选择的位置可以是列出的任何位置，但通常情况下，最好选择最靠近客户的位置。 创建机器人后无法更改位置。 |
   | **定价层** | F0 | 选择定价层。 可随时更新定价层。 有关详细信息，请参阅[机器人服务定价](https://azure.microsoft.com/pricing/details/bot-service/)。 |
   | **应用名称** | 唯一的名称 | 机器人的唯一 URL 名称。 例如，如果将机器人命名为 *myawesomebot*，则机器人的 URL 为 `http://myawesomebot.azurewebsites.net`。 名称只能使用字母数字和下划线字符。 此字段的限制为 35 个字符。 创建机器人后无法更改应用名称。 |
   | **机器人模板** | 基本 | 选择 **C#** 或 **Node.js**，再选择适用于本快速入门的“基本”模板，然后单击“选择”。   “基本”模板创建回显机器人。 [详细了解](bot-service-concept-templates.md)模板。 |
   | **应用服务计划/位置** | 你的应用服务计划  | 选择[应用服务计划](https://azure.microsoft.com/pricing/details/app-service/plans/)位置。 选择的位置可以是列出的任何位置，但通常情况下，最好选择最靠近客户的位置。 （不适用于 Functions 机器人。） |
   | **Application Insights** | 启用 | 决定要启用还是关闭 [Application Insights](/bot-framework/bot-service-manage-analytics)   。 如果选择“启用”，还必须指定区域位置  。 选择的位置可以是列出的任何位置，但通常情况下，最好选择机器人服务所在的位置。 |
   | **Microsoft 应用 ID 和密码** | 自动创建应用 ID 和密码 | 如果需要手动输入 Microsoft 应用 ID 和密码，请使用此选项。 否则，在机器人创建过程中系统会为你创建新的 Microsoft 应用 ID 和密码。 为机器人服务手动创建应用注册时，请确保支持的帐户类型设置为“任何组织目录中的帐户”或“任何组织目录中的帐户和个人 Microsoft 帐户（例如，Skype、Outlook.com、Xbox 等）”  |

   > [!NOTE]
   > 
   > 如果创建 **Functions 机器人**，则看不到“应用服务计划/位置”字段，  只会看到“托管计划”字段。  在这种情况下，请选择一个[托管计划](bot-service-overview-readme.md#hosting-plans)。

4. 单击“创建”创建服务并将机器人部署到云。  此过程可能需要数分钟。

查看“通知”，确认已部署机器人  。 通知将从“正在进行部署...”更改为“部署已成功”   。 单击“转到资源”按钮，打开机器人的资源边栏选项卡  。

 > [!TIP] 
 > 出于性能考虑，运行 Node.js 模板的 **Functions 机器人**使用了 *Azure Functions Pack* 工具进行打包。 *Azure Functions Pack* 工具使用所有 Node.js 模块并将其组合到一个 *.js 文件中。
 > 有关详细信息，请参阅 [Azure Functions Pack](https://github.com/Azure/azure-functions-pack)。
 
## <a name="test-the-bot"></a>测试机器人
创建机器人后，即可在[网上聊天](bot-service-manage-test-webchat.md)中测试它。 输入一条信息，机器人就会响应。

## <a name="next-steps"></a>后续步骤

本主题介绍了如何使用机器人服务来创建**基本的** Web 应用机器人/Functions 机器人，并在 Azure 中使用内置的网上聊天控件验证了机器人的功能。 现在需了解如何管理机器人并开始使用其源代码。

> [!div class="nextstepaction"]
> [管理机器人](bot-service-manage-overview.md)

::: moniker-end

::: moniker range="azure-bot-service-4.0"

[!INCLUDE [applies-to-v4](includes/applies-to.md)]

Azure 机器人服务提供了用于创建机器人的核心组件，包括用于开发机器人的 Bot Framework SDK 以及用于将机器人连接到通道的机器人服务。 在本主题中，还可以选择 .NET 或 Node.js 模板，以便使用 Bot Framework SDK v4 创建机器人。

[!INCLUDE [Azure vs local development](~/includes/snippet-quickstart-paths.md)]

## <a name="prerequisites"></a>先决条件

- [Azure](http://portal.azure.com) 帐户

### <a name="create-a-new-bot-service"></a>创建新的机器人服务

1. 登录到 [Azure 门户](http://portal.azure.com/)。
1. 在 Azure 门户左上角单击“创建新资源”链接，然后选择“AI + 机器学习”>“Web 应用机器人”。    

![创建机器人](~/media/azure-bot-quickstarts/abs-create-blade.png)

2. 此时会打开一个新的边栏选项卡，其中包含有关 **Web 应用机器人**的信息。   

3. 在“机器人服务”边栏选项卡中，提供有关机器人的请求信息，如图片下方的表中所示  。  <br/>
 ![“创建 Web 应用机器人”边栏选项卡](~/media/azure-bot-quickstarts/sdk-create-bot-service-blade.png)

 | 设置 | 建议的值 | 说明 |
 | ---- | ---- | ---- |
 | **机器人名称** | 机器人的显示名称 | 通道和目录中显示的机器人的显示名称。 此名称可随时更改。 |
 | **订阅** | 订阅 | 选择要使用的 Azure 订阅。 |
 | **资源组** | myResourceGroup | 可创建新的[资源组](/azure/azure-resource-manager/resource-group-overview#resource-groups)或从现有资源组中选择。 |
 | **位置** | 默认位置 | 选择资源组的地理位置。 选择的位置可以是列出的任何位置，但通常情况下，最好选择最靠近客户的位置。 创建机器人后无法更改位置。 |
 | **定价层** | F0 | 选择定价层。 可随时更新定价层。 有关详细信息，请参阅[机器人服务定价](https://azure.microsoft.com/pricing/details/bot-service/)。 |
 | **应用名称** | 唯一的名称 | 机器人的唯一 URL 名称。 例如，如果将机器人命名为 *myawesomebot*，则机器人的 URL 为 `http://myawesomebot.azurewebsites.net`。 名称只能使用字母数字和下划线字符。 此字段的限制为 35 个字符。 创建机器人后无法更改应用名称。 |
 | **机器人模板** | 回显机器人 | 选择“SDK v4”。  为本快速入门选择 C# 或 Node.js，然后单击“选择”。   
 | **应用服务计划/位置** | 你的应用服务计划  | 选择[应用服务计划](https://azure.microsoft.com/pricing/details/app-service/plans/)位置。 选择的位置可以是列出的任何位置，但通常情况下，最好选择机器人服务所在的位置。 |
 | **Application Insights** | 启用 | 决定要启用还是关闭 [Application Insights](/bot-framework/bot-service-manage-analytics)   。 如果选择“启用”，还必须指定区域位置  。 选择的位置可以是列出的任何位置，但通常情况下，最好选择机器人服务所在的位置。 |
 | **Microsoft 应用 ID 和密码** | 自动创建应用 ID 和密码 | 如果需要手动输入 Microsoft 应用 ID 和密码，请使用此选项。 否则，在机器人创建过程中系统会为你创建新的 Microsoft 应用 ID 和密码。 为机器人服务手动创建应用注册时，请确保支持的帐户类型设置为“任何组织目录中的帐户”或“任何组织目录中的帐户和个人 Microsoft 帐户（例如，Skype、Outlook.com、Xbox 等）” |

4. 单击“创建”创建服务并将机器人部署到云。  此过程可能需要数分钟。

查看“通知”，确认已部署机器人  。 通知将从“正在进行部署...”更改为“部署已成功”   。 单击“转到资源”按钮，打开机器人的资源边栏选项卡  。

创建机器人后，即可通过网上聊天测试它。 

## <a name="test-the-bot"></a>测试机器人
在“机器人管理”部分中，单击“通过网上聊天执行测试”   。 Azure 机器人服务将加载网上聊天控件，并连接到机器人。 

![Azure 网上聊天测试](./media/azure-bot-quickstarts/azure-webchat-test.png)

输入一条信息，机器人就会响应。

## <a name="download-code"></a>下载代码
可以下载代码以在本地进行处理。 
1. 在“机器人管理”部分中，单击“生成”   。 
1. 单击右窗格中的“下载机器人源代码”  链接。 
1. 按照提示下载代码，然后解压缩该文件夹。
    1. [!INCLUDE [download keys snippet](~/includes/snippet-abs-key-download.md)]

## <a name="next-steps"></a>后续步骤
下载代码后，可以继续在本地计算机上开发机器人。 测试了机器人并已准备好将机器人代码上传到 Azure 门户后，请按照[设置持续部署](./bot-service-build-continuous-deployment.md)主题下列出的说明执行操作，在进行更改后自动更新代码。

::: moniker-end
