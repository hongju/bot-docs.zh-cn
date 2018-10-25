---
title: 使用 Bot Builder SDK for JavaScript 创建机器人 | Microsoft Docs
description: 使用 Bot Builder SDK for JavaScript 快速创建机器人。
keywords: 快速入门, bot builder sdk, 入门
author: jonathanfingold
ms.author: jonathanfingold
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 09/23/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 62451f8581d7a629f9e7ba628e70cdad7b531845
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998974"
---
```
{
    "name": "BasicBot",
    "description": "Demo",
    "services": [
        {
            "type": "endpoint",
            "name": "development",
            "id": "http://localhost:3978/api/messages",
            "appId": "",
            "appPassword": "",
            "endpoint": "http://localhost:3978/api/messages"
        },
        {
            "type": "luis",
            "name": "basic-bot-LUIS",
            "appId": "<your app id>",
            "version": "0.1",
            "authoringKey": "<your authoring key>",
            "subscriptionKey": "<your subscription key>",
            "region": "westus",
            "id": "206"
        }
    ],
    "secretKey": "",
    "version": "2.0"
}
```
