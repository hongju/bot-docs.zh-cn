---
ms.openlocfilehash: f8aad539a2d1e415833609f66cd5b398c88206f1
ms.sourcegitcommit: a47183f5d1c2b2454c4a06c0f292d7c075612cdd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/19/2019
ms.locfileid: "67252670"
---
打开命令提示符以登录到 Azure 门户。

```cmd
az login
```

此时会打开一个用于登录的浏览器窗口。

### <a name="set-the-subscription"></a>设置订阅

设置要使用的默认订阅。

```cmd
az account set --subscription "<azure-subscription>"
```

如果你不确定要使用哪个订阅来部署机器人，可以使用 `az account list` 命令查看帐户的 `subscriptions` 列表。

导航到 bot 文件夹。

```cmd
cd <local-bot-folder>
```