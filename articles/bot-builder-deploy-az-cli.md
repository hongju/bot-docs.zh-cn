---
title: 使用 Azure CLI 部署机器人 | Microsoft Docs
description: 将机器人部署到 Azure 云。
keywords: 部署机器人, azure 部署, 发布机器人, az 部署机器人, visual studio 部署机器人, msbot 发布, msbot 克隆
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: get-started-article
ms.service: bot-service
ms.subservice: abs
ms.date: 01/07/2019
ms.openlocfilehash: 3ebc13cf9e2d111d716d081c36f125d28a441811
ms.sourcegitcommit: bdb981c0b11ee99d128e30ae0462705b2dae8572
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/17/2019
ms.locfileid: "54360737"
---
# <a name="deploy-your-bot-using-azure-cli"></a>使用 Azure CLI 部署机器人

[!INCLUDE [pre-release-label](./includes/pre-release-label.md)]

创建机器人并在本地对其进行测试后，可将其部署到 Azure，以便可以从任何位置访问它。 将机器人部署到 Azure 需要支付服务使用费。 [计费和成本管理](https://docs.microsoft.com/en-us/azure/billing/)一文可帮助你了解 Azure 计费方式、如何监视使用量与费用，以及如何管理帐户和订阅。

本文介绍如何使用 `az` 和 `msbot` cli 将 C# 和 JavaScript 机器人部署到 Azure。 在执行相关步骤之前最好是先阅读本文，以完全了解在部署机器人时所涉及到的工作。

## <a name="prerequisites"></a>先决条件

[!INCLUDE [prerequisite snippet](~/includes/deploy/snippet-prerequisite.md)]


## <a name="deploy-javascript-and-c-bots-using-az-cli"></a>使用 az cli 部署 JavaScript 和 C# 机器人

你已在本地创建并测试一个机器人，现在想要将它部署到 Azure。 这些步骤假定你已创建所需的 Azure 资源。

[!INCLUDE [az login snippet](~/includes/deploy/snippet-az-login.md)]

### <a name="create-a-web-app-bot"></a>创建 Web 应用机器人

如果还没有可以向其发布机器人的资源组，请创建一个：

[!INCLUDE [az create group snippet](~/includes/deploy/snippet-az-create-group.md)]

[!INCLUDE [az create web app snippet](~/includes/deploy/snippet-create-web-app.md)]

在继续之前，请先根据用于登录到 Azure 的电子邮件帐户类型阅读适用的说明。

#### <a name="msa-email-account"></a>MSA 电子邮件帐户

如果使用 [MSA](https://en.wikipedia.org/wiki/Microsoft_account) 电子邮件帐户，则需在应用程序注册门户中创建可以与 `az bot create` 命令配合使用的应用 ID 和应用密码。

[!INCLUDE [create bot msa snippet](~/includes/deploy/snippet-create-bot-msa.md)]

#### <a name="business-or-school-account"></a>企业或学校帐户

[!INCLUDE [create bot snippet](~/includes/deploy/snippet-create-bot.md)]

### <a name="download-the-bot-from-azure"></a>从 Azure 下载此机器人

接下来，下载刚创建的机器人。 
[!INCLUDE [download bot snippet](~/includes/deploy/snippet-download-bot.md)]

### <a name="decrypt-the-downloaded-bot-file-and-use-in-your-project"></a>解密已下载的 .bot 文件并在项目中使用它

.bot 文件中的敏感信息已加密。

[!INCLUDE [decrypt bot snippet](~/includes/deploy/snippet-decrypt-bot.md)]

### <a name="update-the-bot-file"></a>更新 .bot 文件

如果机器人使用 LUIS、QnA Maker 或 Dispatch 服务，则需向 .bot 文件添加对这些服务的引用， 否则可以跳过此步骤。

1. 使用新的 .bot 文件在 BotFramework Emulator 中打开机器人。 此机器人不需在本地运行。
1. 在“机器人资源管理器”面板中展开“服务”部分。
1. 若要添加对 LUIS 应用的引用，请单击“服务”右侧的加号 (+)。
   1. 选择“添加语言理解(LUIS)”。
   1. 如果提示你登录到 Azure 帐户，请照做。
   1. 此时会显示一个列表，其中包含你有权访问的 LUIS 应用程序。 请选择适用于机器人的应用程序。
1. 若要添加对 QnA Maker 知识库的引用，请单击“服务”右侧的加号 (+)。
   1. 选择“添加 QnA Maker”。
   1. 如果提示你登录到 Azure 帐户，请照做。
   1. 此时会显示一个列表，其中包含你有权访问的知识库。 请选择适用于机器人的应用程序。
1. 若要添加对 Dispatch 模型的引用，请单击“服务”右侧的加号 (+)。
   1. 选择“添加 Dispatch”。
   1. 如果提示你登录到 Azure 帐户，请照做。
   1. 此时会显示一个列表，其中包含你有权访问的 Dispatch 模型。 请选择适用于机器人的应用程序。

### <a name="test-your-bot-locally"></a>在本地测试机器人

目前，机器人的工作方式应该与使用旧 .bot 文件时的工作方式相同。 请确保它在使用新的 .bot 文件时的工作方式符合预期。

### <a name="publish-your-bot-to-azure"></a>将机器人发布到 Azure

<!-- TODO: re-encrypt your .bot file? -->

[!INCLUDE [publish snippet](~/includes/deploy/snippet-publish.md)]

<!-- TODO: If we tell them to re-encrypt, this step is not necessary. -->

[!INCLUDE [clear encryption snippet](~/includes/deploy/snippet-clear-encryption.md)]

## <a name="additional-resources"></a>其他资源

[!INCLUDE [additional resources snippet](~/includes/deploy/snippet-additional-resources.md)]

## <a name="next-steps"></a>后续步骤
> [!div class="nextstepaction"]
> [设置持续部署](bot-service-build-continuous-deployment.md)
