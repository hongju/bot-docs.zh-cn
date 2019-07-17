---
ms.openlocfilehash: 0b991c438c0006d1fb4bafa90982f73f4a18be77
ms.sourcegitcommit: fa6e775dcf95a4253ad854796f5906f33af05a42
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/16/2019
ms.locfileid: "68230661"
---
Bot Builder Framework 使机器人能够在特定聊天的上下文中存储和检索与用户、聊天或特定用户关联的状态数据。 状态数据可用于多种目的，例如确定先前聊天中断的位置或仅按名称来问候返回的用户。 如果存储用户的首选项，则可以在下次聊天时使用该信息自定义聊天。 例如，可以提醒用户注意有关他们感兴趣的主题的新闻文章，或者在约会可用时提醒用户。 

出于测试和原型设计的目的，可以使用 Bot Builder Framework 的内存中数据存储。 对于生产机器人，可以实施自己的存储适配器或使用 Azure 扩展之一。 Azure 扩展允许你将机器人的状态数据存储在表存储、CosmosDB 或 SQL 中。 本文展示如何使用内存中存储适配器来存储机器人的状态数据。 

> [!IMPORTANT]
> 建议不要将 Bot Framework State Service API 用于生产环境，该 API 可能会在将来的版本中弃用。 建议更新机器人代码以使用内存中存储适配器进行测试，或者将 **Azure 扩展**之一用于生产机器人。
