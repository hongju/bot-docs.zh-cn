---
ms.openlocfilehash: b5809b6d46cdc09035efb36c3ea58c2ca9dc6c3e
ms.sourcegitcommit: fa6e775dcf95a4253ad854796f5906f33af05a42
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/16/2019
ms.locfileid: "68230724"
---
用户通常使用“帮助”、“取消”或“重新开始”等关键字来尝试访问机器人内的某些功能。 这通常发生在对话的中间，当机器人期待不同的响应时。 通过实现**全局消息处理程序**，可以将机器人设计为优雅地处理此类请求。
处理程序将检查你指定的关键字的用户输入，例如“帮助”、“取消”或“重新开始”，并进行适当的响应。 

![用户如何说话](~/media/designing-bots/capabilities/trigger-actions.png)

> [!NOTE]
> 通过在全局消息处理程序中定义逻辑，你可以使所有对话都可以访问该逻辑。 可以配置单个对话和提示以放心地忽略关键字。
