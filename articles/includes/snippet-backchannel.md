---
ms.openlocfilehash: b320aadc876074a76fe209ad55a81cb70b1ddcac
ms.sourcegitcommit: a295a90eac461f8b96770dd902ba44919acf33fc
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/26/2019
ms.locfileid: "67405542"
---
<a href="https://github.com/Microsoft/BotFramework-WebChat" target="_blank">开源网上聊天控件</a>使用 [Direct Line API](https://docs.botframework.com/restapi/directline3/#navtitle) 与机器人通信，此 API 让用户可在客户端和机器人之间来回发送 `activities`。 最常见的活动类型是 `message`，但也有其他类型。 例如，活动类型 `typing` 表示用户正在键入或机器人正在编译答复。 

通过将活动类型设置为 `event`，可使用反向通道机制在客户端和机器人之间交换信息，而无需将其呈现给用户。 网上聊天控件将自动忽略 `type="event"` 的所有活动。