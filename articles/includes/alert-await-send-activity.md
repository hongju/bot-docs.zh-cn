---
ms.openlocfilehash: 91b2729aa10c8ac1985e62845126296e8141ef77
ms.sourcegitcommit: fa6e775dcf95a4253ad854796f5906f33af05a42
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/16/2019
ms.locfileid: "68230648"
---
> [!IMPORTANT]
> 线程处理主机器人轮次完成后处理上下文对象释放。  确保 `await` 任何活动调用，以便主线程等待生成的活动，再完成处理并释放轮次上下文。 否则，如果某个响应（包括其处理程序）在已占用大量时间的情况下尝试对上下文对象执行操作，则会出现“上下文已释放”错误。 