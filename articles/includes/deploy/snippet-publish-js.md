---
ms.openlocfilehash: 0891b9652154f8ed086cc45ce6018aa0be1a67b8
ms.sourcegitcommit: fa6e775dcf95a4253ad854796f5906f33af05a42
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/16/2019
ms.locfileid: "68230574"
---
若要将本地 JavaScript 机器人发布回 Azure，必须首先手动创建单个压缩的文件，其中包含用于在本地生成和运行机器人的所有文件。 这包括下载到 `node_modules` 文件夹中的所有 npm 库。 创建此 zip 文件时，请确保所用根目录是 index.js 文件所在的目录。 

创建包含机器人的所有源代码的 zip 文件以后，请打开一个命令提示符窗口并运行下述 _Az cli_ 命令。 

此步骤可能需要一定的时间。

```cmd
az webapp deployment source config-zip --resource-group <resource-group-name> --name <bot-resource-name> --src <directory-path>
```

| 选项 | 说明 |
|:---|:---|
| --resource-group | Azure 中资源组的名称。 |
| --name | Azure 中机器人的资源名称。 |
| --src | 一个完整的目录路径，可以从其上传压缩的机器人代码。 例如 `c:\my-local-repository\this-app-folder\my-zipped-code.zip` |

此操作成功完成以后，机器人就已经部署在 Azure 中了。
