---
ms.openlocfilehash: f95c8a37b1207a26dab0a714b86412a9dba2dcb4
ms.sourcegitcommit: a47183f5d1c2b2454c4a06c0f292d7c075612cdd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/19/2019
ms.locfileid: "67252636"
---
```cmd
az bot create --kind webapp --name <bot-resource-name> --location <geographic-location> --version v4 --lang <language> --verbose --resource-group <resource-group-name>
```

| 选项 | 说明 |
|:---|:---|
| --name | 用于在 Azure 中部署机器人的唯一名称。 此名称可与本地机器人的名称相同。 切勿在此名称中包含空格或下划线。 |
| --location | 用于创建机器人服务资源的地理位置。 例如 `eastus`、`westus`、`westus2` 等。 |
| --lang | 用来创建机器人的语言：`Csharp` 或 `Node`；默认为 `Csharp`。 |
| --resource-group | 要在其中创建机器人的资源组的名称。 可以使用 `az configure --defaults group=<name>` 配置默认组。 |