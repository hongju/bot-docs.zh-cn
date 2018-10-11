---
title: 创建新项目的企业机器人 | Microsoft Docs
description: 了解如何基于企业机器人模板创建新机器人
author: darrenj
ms.author: darrenj
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 09/18/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 2c6736cc8aa607da73c392b04ea894a19c86ff29
ms.sourcegitcommit: 87b5b0ca9b0d5e028ece9f7cc4948c5507062c2b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/24/2018
ms.locfileid: "47029775"
---
# <a name="enterprise-bot-template---creating-a-new-project"></a>企业机器人模板 - 创建新项目

> [!NOTE]
> 本主题适用于 SDK 的 v4 版本。 

企业机器人模板汇集了我们通过构建聊天体验确定的所有最佳做法和支持组件。 该模板在以下 Botbuilder SDK 平台中可用：

- .NET
- Node.js（即将推出）

## <a name="net"></a>.NET

企业机器人模板适用于 .NET（面向 SDK 的 **V4** 版本）。 它以 [VSIX](https://docs.microsoft.com/en-us/visualstudio/extensibility/anatomy-of-a-vsix-package) 包的形式提供。 若要下载，请单击以下链接：

- [BotBuilder SDK V4 企业机器人模板](https://aka.ms/GetEnterpriseBotTemplate)

#### <a name="prerequisites"></a>先决条件

- [Visual Studio 2017 或更高版本](https://www.visualstudio.com/downloads/)
- [Azure 帐户](https://azure.microsoft.com/en-us/free/)
- [Azure PowerShell](https://docs.microsoft.com/en-us/powershell/azure/overview?view=azurermps-6.8.1)

### <a name="install-the-template"></a>安装模板

从保存的目录直接打开 VSIX 包，企业机器人模板就会安装到 Visual Studio，下次打开时就可以使用了。

若要使用模板创建新的机器人项目，请直接打开 Visual Studio 并选择“文件” > “新建” > “项目”，然后从 Visual C# 中选择“Bot Framework”>“企业机器人模板”。 这样就会在本地创建新的机器人项目，可以根据需要对其进行编辑。 

![文件新建项目模板](media/enterprise-template/EnterpriseBot-NewProject.png)

## <a name="deploy-your-bot"></a>部署机器人

现在你已经创建了项目，下一步是创建支持 Azure 基础结构，并执行配置/部署，使机器人开箱即可正常工作。 继续[部署机器人](bot-builder-enterprise-template-deployment.md)。

> 必须运行此步骤，否则机器人初始化 (AppInsights) 和 LUIS 依赖项将不可用。
## <a name="customize-your-bot"></a>自定义机器人

验证已成功将机器人部署为开箱即用后，可以根据方案和需求自定义机器人。 继续[自定义机器人](bot-builder-enterprise-template-customize.md)。
