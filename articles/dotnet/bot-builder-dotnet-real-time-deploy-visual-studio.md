---
title: 将 Skype 实时媒体机器人部署到 Azure | Microsoft Docs
description: 了解如何使用 Visual Studio 的内置发布功能将 Skype 实时音频视频机器人部署到 Azure。
author: MalarGit
ms.author: malarch
manager: ssulzer
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/17
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 22cce8ad5bef3c1c6f08a8efc28118e0209dd3af
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999434"
---
# <a name="deploy-a-real-time-media-bot-from-visual-studio-to-azure"></a>将实时媒体机器人从 Visual Studio 部署到 Azure

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

在“IaaS”Azure 虚拟机或“经典”Azure 云服务中可以托管实时媒体机器人。 本文介绍如何使用 Visual Studio 的内置发布功能从 Visual Studio 部署托管在 Azure 云服务辅助角色中的机器人。

## <a name="prerequisites"></a>先决条件

必须拥有 Microsoft Azure 订阅才能将机器人部署到 Azure。 如果尚无订阅，可注册<a href="https://azure.microsoft.com/en-us/free/" target="_blank">免费帐户</a>。 此外，本文所述的过程需要 Visual Studio。 如果还没有 Visual Studio，可下载免费版 <a href="https://www.visualstudio.com/downloads/" target="_blank">Visual Studio 2017 Community</a>。

### <a name="certificate-from-a-valid-certificate-authority"></a>有效证书颁发机构的证书
机器人需要配置受信任证书颁发机构 (CA) 的有效证书。 使用者名称 (SN) 或证书的使用者可选名称 (SAN) 的最后一个条目应为云服务的名称。 当前不支持通配符证书。 如果 CNAME 用于指向云服务，则 CNAME 应为 SN 或证书的最后一个 SAN 条目。

## <a name="configure-application-settings"></a>配置应用程序设置
要使机器人在云环境中正常运行，必须确保其应用程序设置正确。 更具体地说，就是在辅助角色的 app.config 文件中设置以下键值：
> <ul><li>MicrosoftAppId</li><li>MicrosoftAppPassword</li></ul>

> [!NOTE]
> 要查找机器人的 AppID 和 AppPassword，请参阅 [MicrosoftAppID 和 MicrosoftAppPassword](~/bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword)。

## <a name="create-worker-role-in-the-azure-portal"></a>在 Azure 门户中创建辅助角色
### <a name="step-1-create-cloud-serviceclassic"></a>步骤 1：创建云服务（经典）
登录到 <a href="https://portal.azure.com">Azure 门户</a>。 单击屏幕左侧的 +，然后选择“云服务(经典)”。 在表单中提供必填信息，然后单击“创建”。

![创建云服务](../media/real-time-media-bot-portal-service-creation.png)

> [!NOTE]
> 应在机器人注册的 URL 中提供机器人的 DNS 名称。

### <a name="step-2-upload-the-certificate-for-the-bot"></a>步骤 2：上传机器人的证书
创建机器人后，上传机器人的证书。

![上传证书](../media/real-time-media-bot-portal-certificates.png)

## <a name="modify-service-configuration-with-worker-role-details"></a>使用辅助角色详细信息修改服务配置
通过 Azure RoleEnvironment API 无法获取机器人的完全限定的域名 (FQDN)。 因此，必须向机器人提供其 FQDN。 它还需要了解用于 HTTPS 的证书。 在辅助角色的服务配置 (.cscfg) 文件中可以配置这些内容。

> [!TIP]
> 如果正在部署 BotBuilder-RealTimeMediaCalling GIT 存储库中的示例，
> - 并且服务配置中使用了 $DnsName$，请使用云服务名称或 CNAME 替换 $DnsName$。
>   ```xml
>      <Setting name="ServiceDnsName" value="$DnsName$" />
>   ```
> 
> - 在配置的以下代码行中，使用上传到机器人的证书指纹替换 $CertThumbprint$。
>   ```xml
>      <Setting name="DefaultCertificate" value="$CertThumbprint$" />
>      <Certificate name="Default" thumbprint="$CertThumbprint$" thumbprintAlgorithm="sha1" />
>   ```

## <a name="publish-the-bot-from-visual-studio"></a>从 Visual Studio 发布机器人
### <a name="step-1-launch-the-microsoft-azure-publishing-wizard-in-visual-studio"></a>步骤 1：在 Visual Studio 中启动 Microsoft Azure 发布向导

在 Visual Studio 中打开项目。 在解决方案资源管理器中，右键单击云服务项目并选择“发布”。 此操作会启动 Microsoft Azure 发布向导。 使用凭据登录到相应的订阅。

![右键单击该项目，然后选择“发布”以启动 Microsoft Azure 发布向导](../media/real-time-media-bot-publish-signin.png)

### <a name="step-2-publish-the-bot"></a>步骤 2：发布机器人

单击“下一步”。 此时会打开“设置”选项卡。指定用于部署机器人的“云服务”、“环境”、“生成配置”和“服务配置”。

![单击“下一步”，转到“设置”选项卡](../media/real-time-media-bot-publish-settings.png)

可选择“高级设置”并指定部署日志的存储帐户（可用于调试问题）。

![单击“高级设置”选项卡](../media/real-time-media-bot-publish-advanced-settings.png)

验证“摘要”选项卡中的配置，然后单击“发布”将机器人部署到 Microsoft Azure。
