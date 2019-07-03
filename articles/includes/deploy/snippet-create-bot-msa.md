---
ms.openlocfilehash: 14d9632ad578014a36b5f13e6dee883e2a6e1722
ms.sourcegitcommit: a47183f5d1c2b2454c4a06c0f292d7c075612cdd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/19/2019
ms.locfileid: "67252650"
---
1. 转到[**应用程序注册门户**](https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade)。
1. 单击“添加应用”以注册应用程序，创建**应用程序 ID**，然后单击“生成新密码”。   如果已经有应用程序和密码，但却忘记了该密码，则需在“应用程序机密”部分生成新密码。
1. 保存刚刚生成的应用程序 ID 和新密码，以便可以在 `az bot create` 命令中使用这些信息。  

```cmd
az bot create --kind webapp --name <bot-resource-name> --location <geographic-location> --version v4 --lang <language> --verbose --resource-group <resource-group-name> --appid "<application-id>" --password "<application-password>" --verbose
```

| 选项 | 说明 |
|:---|:---|
| --name | 用于在 Azure 中部署机器人的唯一名称。 此名称可与本地机器人的名称相同。 切勿在此名称中包含空格或下划线。 |
| --location | 用于创建机器人服务资源的地理位置。 例如 `eastus`、`westus`、`westus2` 等。 |
| --lang | 用来创建机器人的语言：`Csharp` 或 `Node`；默认为 `Csharp`。 |
| --resource-group | 要在其中创建机器人的资源组的名称。 可以使用 `az configure --defaults group=<name>` 配置默认组。 |
| --appid | 可以与机器人配合使用的 Microsoft 帐户 ID (MSA ID)。 |
| --password | 机器人的 Microsoft 帐户 (MSA) 密码。 |
