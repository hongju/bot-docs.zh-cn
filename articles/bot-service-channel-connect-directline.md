---
title: 将机器人连接到 Direct Line | Microsoft Docs
description: 了解如何配置机器人与 Direct Line 的连接。
keywords: Direct Line, 机器人通道, 自定义客户端, 连接到通道, 配置
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 10/11/2018
ms.openlocfilehash: 9383e15590569458e795e9a0603df21f63609001
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997734"
---
# <a name="connect-a-bot-to-direct-line"></a>将机器人连接到 Direct Line

可使用 Direct Line 通道启用自己的客户端应用程序以与机器人进行通信。 

## <a name="add-the-direct-line-channel"></a>添加 Direct Line 通道

若要添加 Direct Line 通道，请在 [Azure 门户](https://portal.azure.com/)中打开机器人，单击“通道”边栏选项卡，然后单击“Direct Line”。

![添加 Direct Line 通道](~/media/bot-service-channel-connect-directline/directline-addchannel.png)

## <a name="add-new-site"></a>添加新站点

接下来，添加一个新站点，该站点代表要连接到机器人的客户端应用程序。 单击“添加新站点”，输入站点名称，然后单击“完成”。

![添加 Direct Line 站点](~/media/bot-service-channel-connect-directline/directline-addsite.png)

## <a name="manage-secret-keys"></a>管理密钥

创建站点后，Bot Framework 会生成密钥，客户端应用程序可使用这些密钥来对发出的与机器人进行通信的 Direct Line API 请求进行[身份验证](~/rest-api/bot-framework-rest-direct-line-3-0-authentication.md)。 若要以纯文本格式查看密钥，请单击相应密钥的“显示”按钮。

![显示 Direct Line 密钥](~/media/bot-service-channel-connect-directline/directline-showkey.png)

复制并安全存储显示的密钥。 然后使用密钥对客户端发出的与机器人进行通信的 Direct Line API 请求进行[身份验证](~/rest-api/bot-framework-rest-direct-line-3-0-authentication.md)。
或者使用 Direct Line API，[以密钥换取令牌](~/rest-api/bot-framework-rest-direct-line-3-0-authentication.md#generate-token)，客户端使用该令牌在单个聊天范围内对其后续请求进行身份验证。

![复制 Direct Line 密钥](~/media/bot-service-channel-connect-directline/directline-copykey.png)

## <a name="configure-settings"></a>配置设置

最后，配置站点设置。

- 选择客户端应用程序用于与机器人进行通信的 Direct Line 协议版本。

> [!TIP]
> 如果要在客户端应用程序和机器人之间创建新连接，请使用 Direct Line API 3.0。

完成后，单击“完成”以保存站点配置。 对于要连接到机器人的每个客户端应用程序，可从[添加新站点](#add-new-site)开始重复此过程。
