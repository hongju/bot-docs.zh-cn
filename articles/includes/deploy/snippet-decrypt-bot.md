---
ms.openlocfilehash: eac6abae509d92ea4714bc01221f180ea575950f
ms.sourcegitcommit: a47183f5d1c2b2454c4a06c0f292d7c075612cdd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/19/2019
ms.locfileid: "67252624"
---
获取加密密钥。

1. 登录到 [Azure 门户](http://portal.azure.com/)。
1. 打开机器人的 Web 应用机器人资源。
1. 打开机器人的“应用程序设置”。 
1. 在“应用程序设置”  窗口中，向下滚动到“应用程序设置”  。
1. 找到 **botFileSecret** 并复制其值。

解密 .bot 文件。

```cmd
msbot secret --bot <name-of-bot-file> --secret "<bot-file-secret>" --clear
```

| 选项 | 说明 |
|:---|:---|
| --bot | 已下载的 .bot 文件的相对路径。 |
| --secret | 加密密钥。 |

将解密的 `.bot` 文件复制到本地机器人项目所在的目录，更新机器人以便使用这个新的 `.bot` 文件，然后删除旧的 `.bot` 文件。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

在 **appsettings.json** 中更新 **botFilePath** 属性，使之指向新的 `.bot` 文件，该文件已添加到本地目录。

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

在 **.env** 中更新 **botFilePath** 属性，使之指向新的 `.bot` 文件，该文件已添加到本地目录。

---

更新机器人以后，请删除已下载机器人的临时目录。