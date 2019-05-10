---
ms.openlocfilehash: 8e5677fe59dd9edad6ac1da9d029e5f7c08bf179
ms.sourcegitcommit: 980612a922b8290b2faadaca193496c4117e415a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/26/2019
ms.locfileid: "64563960"
---
## <a name="next-steps"></a>后续步骤
通过使用 Bot Framework Emulator 测试机器人，将机器人部署到云并验证部署是否成功后，机器人发布过程的下一步将取决于是否已经在 Bot Framework 中注册了机器人。

### <a name="if-you-have-already-registered-your-bot-with-the-bot-framework"></a>如果已在 Bot Framework 中注册了机器人：

1. 返回到 <a href="https://dev.botframework.com" target="_blank">Bot Framework 门户</a>并[更新机器人的设置数据](~/bot-service-manage-settings.md)以指定机器人的消息终结点。

2. [配置机器人以在一个或多个通道上运行](~/bot-service-manage-channels.md)。

### <a name="if-you-have-not-yet-registered-your-bot-with-the-bot-framework"></a>如果尚未在 Bot Framework 中注册机器人：

1. [在 Bot Framework 中注册机器人](~/bot-service-quickstart-registration.md)。

2. 在已部署的应用程序的配置设置中更新“Microsoft 应用 Id”和“Microsoft 应用密码”值，以指定在注册过程中为机器人生成的“appID”和“password”值。 若要查找机器人的“AppID”和“AppPassword”，请参阅 [MicrosoftAppID 和 MicrosoftAppPassword](~/bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword)。

3. [配置机器人以在一个或多个通道上运行](~/bot-service-manage-channels.md)。