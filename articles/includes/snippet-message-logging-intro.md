---
ms.openlocfilehash: 12ead266dea859c84450e08ae12ed98d4952d698
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/11/2019
ms.locfileid: "54226012"
---
Bot Framework SDK 中的“中间件”功能让机器人能够拦截用户和机器人之间交换的所有消息。 对于每个被拦截的消息，你可以选择将消息保存到指定的数据存储（这会创建会话日志）或以某种方式检查消息等，并执行代码指定的任何操作。 

> [!NOTE]
> Bot Framework 不会自动保存会话详细信息，因为这样做可能会捕获机器人和用户不希望与外部各方共享的私人信息。 如果你的机器人保存会话详细信息，它应将该信息传达给用户并描述将对数据执行的操作。