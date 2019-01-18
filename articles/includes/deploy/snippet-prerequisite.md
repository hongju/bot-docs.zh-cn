---
ms.openlocfilehash: 266cdc2bbeeb140e4b601c1bf0fb3fa8eb085dda
ms.sourcegitcommit: bdb981c0b11ee99d128e30ae0462705b2dae8572
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/17/2019
ms.locfileid: "54360850"
---
- 如果没有 Azure 订阅，请在开始之前创建一个[免费帐户](https://azure.microsoft.com/free/)。
- 安装最新版本的 [Azure CLI 工具](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)。
- 安装最新版本的 [MSBot](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/MSBot) 工具。
- 安装最新发布的 [Bot Framework Emulator](https://aka.ms/Emulator-wiki-getting-started) 版本。
- 安装并配置 [ngrok](https://github.com/Microsoft/BotFramework-Emulator/wiki/Tunneling-%28ngrok%29)。
- 了解 [.bot](~/v4sdk/bot-file-basics.md) 文件。

使用 msbot 4.3.2 及更高版本时，需要 Azure CLI 2.0.54 或更高版本。 如果安装了 botservice 扩展，请使用此命令将其删除。

```cmd
az extension remove --name botservice
```