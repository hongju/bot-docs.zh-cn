---
ms.openlocfilehash: 86faca976bc95a5e91e17749096cd148483edc61
ms.sourcegitcommit: 980612a922b8290b2faadaca193496c4117e415a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/26/2019
ms.locfileid: "64563636"
---
## <a name="payment-process-overview"></a>支付流程概述

支付流程包括三个不同的部分：

1. 机器人发送支付请求。

2. 用户使用 Microsoft 帐户登录以提供支付、寄送和联系信息。 向机器人发送回调以指示机器人何时需要执行某些操作（更新寄送地址、更新寄送选项、完成支付）。

3. 机器人处理收到的回调，包括寄送地址更新、寄送选项更新和支付完成。 

机器人仅须执行此流程的第一步和第三步；第二步发生在机器人环境之外。 
