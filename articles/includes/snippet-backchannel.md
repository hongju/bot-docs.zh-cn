---
ms.openlocfilehash: cf0e23e349ace78958f861ea2e650a5ebcb78466
ms.sourcegitcommit: 980612a922b8290b2faadaca193496c4117e415a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/26/2019
ms.locfileid: "64563740"
---
<a href="https://github.com/Microsoft/BotFramework-WebChat" target="_blank">开源网上聊天控件</a>使用 [Direct Line API](https://docs.botframework.com/en-us/restapi/directline3/#navtitle) 与机器人通信，此 API 让用户可在客户端和机器人之间来回发送 `activities`。 最常见的活动类型是 `message`，但也有其他类型。 例如，活动类型 `typing` 表示用户正在键入或机器人正在编译答复。 

通过将活动类型设置为 `event`，可使用反向通道机制在客户端和机器人之间交换信息，而无需将其呈现给用户。 网上聊天控件将自动忽略 `type="event"` 的所有活动。