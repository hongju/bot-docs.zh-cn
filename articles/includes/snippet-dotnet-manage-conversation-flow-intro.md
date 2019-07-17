---
ms.openlocfilehash: 7eed8c8328456bad43f3bdfe0029df09efa062d6
ms.sourcegitcommit: fa6e775dcf95a4253ad854796f5906f33af05a42
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/16/2019
ms.locfileid: "68230543"
---
这个图显示了传统应用程序的屏幕流程与机器人的对话流程的对比。 

![机器人](~/media/designing-bots/core/dialogs-screens.png)

在传统应用程序中，一切都从主屏幕  开始。
**主屏幕**调用“新建订单”屏幕  。
“新建订单”屏幕  在关闭或调用其他屏幕之前一直处于控制状态。 如果“新建订单”屏幕  关闭，则用户将返回**主屏幕**。

在机器人中，一切都从根对话框  开始。 **根对话**调用“新建订单”对话  。 此时，“新建订单”对话  控制聊天并保持控制状态，直到它关闭或调用其他对话。 如果“新建订单”对话  关闭，聊天的控制权会返回给**根对话**。