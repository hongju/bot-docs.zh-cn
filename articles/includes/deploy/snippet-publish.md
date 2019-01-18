---
ms.openlocfilehash: 88732d2d5490962d7a899d936767e7dd148e94c5
ms.sourcegitcommit: bdb981c0b11ee99d128e30ae0462705b2dae8572
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/17/2019
ms.locfileid: "54360823"
---
将本地机器人发布到 Azure。 此步骤可能需要一定的时间。

```cmd
az bot publish --name <bot-resource-name> --proj-name "<project-file-name>" --resource-group <resource-group-name> --code-dir <directory-path> --verbose --version v4
```

| 选项 | 说明 |
|:---|:---|
| --name | Azure 中机器人的资源名称。 |
| --proj-name | 对于 C#，请使用需要发布的启动项目文件名称（不带 .csproj）。 例如：`EnterpriseBot`。 对于 Node.js，请使用机器人的主入口点。 例如，`index.js`。 |
| --resource-group | 资源组的名称。 |
| --code-dir | 一个目录，可以从其上传机器人代码。 |

此步骤完成后如果出现“部署成功!” 消息，则表明机器人已部署在 Azure 中。