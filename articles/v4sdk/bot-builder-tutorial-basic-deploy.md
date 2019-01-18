---
title: 有关如何创建和部署基本机器人的教程 | Microsoft Docs
description: 了解如何创建基本机器人并将其部署到 Azure。
keywords: echo 机器人, 部署, azure, 教程
author: Ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 1/9/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: dbde6eba946e27aaa6b883f1e9205adc63cb22f8
ms.sourcegitcommit: bdb981c0b11ee99d128e30ae0462705b2dae8572
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/17/2019
ms.locfileid: "54360938"
---
# <a name="tutorial-create-and-deploy-a-basic-bot"></a>教程：创建和部署基本机器人

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

本教程介绍如何使用 Bot Framework SDK 来创建基本机器人并将其部署到 Azure。 如果已创建基本机器人并在本地运行它，请跳转到[部署机器人](#deploy-your-bot)部分。

本教程介绍如何执行下列操作：

> [!div class="checklist"]
> * 创建基本的 Echo 机器人
> * 在本地运行该机器人并与之交互
> * 发布机器人

如果还没有 Azure 订阅，可以在开始前创建一个[免费帐户](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

[!INCLUDE [dotnet quickstart](~/includes/quickstart-dotnet.md)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

[!INCLUDE [javascript quickstart](~/includes/quickstart-javascript.md)]

---

## <a name="deploy-your-bot"></a>部署机器人

现在，将机器人部署到 Azure。

### <a name="prerequisites"></a>先决条件

[!INCLUDE [prerequisite snippet](~/includes/deploy/snippet-prerequisite.md)]

### <a name="login-to-azure-cli-and-set-your-subscription"></a>登录到 Azure CLI 并设置订阅

你已在本地创建并测试一个机器人，现在想要将它部署到 Azure。

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

.bot 文件中的敏感信息已加密，因此需将其解密以方便使用。 

首先，导航到已下载的机器人目录。

[!INCLUDE [decrypt bot snippet](~/includes/deploy/snippet-decrypt-bot.md)]

### <a name="test-your-bot-locally"></a>在本地测试机器人

目前，机器人的工作方式应该与使用旧 `.bot` 文件时的工作方式相同。 请确保它在使用新的 `.bot` 文件时的工作方式符合预期。

现在会在模拟器中看到一个生产终结点。 如果看不到，则可能是因为你仍在使用旧的 `.bot` 文件。

### <a name="publish-your-bot-to-azure"></a>将机器人发布到 Azure

<!-- TODO: re-encrypt your .bot file? -->

[!INCLUDE [publish snippet](~/includes/deploy/snippet-publish.md)]

<!-- TODO: If we tell them to re-encrypt, this step is not necessary. -->

[!INCLUDE [clear encryption snippet](~/includes/deploy/snippet-clear-encryption.md)]

现在应该可以在 Webchat 中测试机器人。

## <a name="additional-resources"></a>其他资源

[!INCLUDE [additional resources snippet](~/includes/deploy/snippet-additional-resources.md)]

## <a name="next-steps"></a>后续步骤
> [!div class="nextstepaction"]
> [向机器人添加服务](bot-builder-tutorial-add-qna.md)

