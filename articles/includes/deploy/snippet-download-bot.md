---
ms.openlocfilehash: 867e65b25878f810e3247eb3cace95f4d31e11db
ms.sourcegitcommit: a47183f5d1c2b2454c4a06c0f292d7c075612cdd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/19/2019
ms.locfileid: "67252615"
---
使用你的当前项目目录外部的一个临时目录。 

以下命令将在 save-path 下创建一个子目录，但指定的路径必须已存在。

```cmd
az bot download --name <bot-resource-name> --resource-group <resource-group-name> --save-path "<path>"
```

| 选项 | 说明 |
|:---|:---|
| --name | Azure 中机器人的名称。 |
| --resource-group | 机器人所在的资源组的名称。 |
| --save-path | 一个现有目录，可将机器人代码下载到其中。 |