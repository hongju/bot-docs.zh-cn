---
title: 教程 - 保存用户状态数据 | Microsoft Docs
description: 了解如何将用户状态数据保存在 Bot Builder SDK 上。
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 4/23/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 86f70fd66f1bc2261339cbe0590061913b51ddbc
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/24/2018
ms.locfileid: "42904435"
---
# <a name="save-user-state-data"></a>保存用户状态数据

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

当机器人请求用户输入时，你很可能想要将一些信息保存到某种形式的存储中。 Bot Builder SDK 允许你使用内存中存储、文件存储、数据库存储（如 CosmosDB 或 SQL）存储用户输入。 

本教程将演示如何定义中间件层中的存储对象，以及如何将用户到保存到存储对象中，以便能持久保存。

## <a name="prequisite"></a>先决条件 

本教程基于 [Manage a conversation flow with waterfall](bot-builder-tutorial-waterfall.md)（使用瀑布管理聊天流）教程。

## <a name="add-storage-to-middleware-layer"></a>将存储添加到中间件层


## <a name="save-user-input-to-storage"></a>将用户输入保存到存储

要管理餐位预订对话，你需要定义具有四个步骤的瀑布式对话。 在此聊天中，你还将在 `TextPrompt` 之外使用 `DatetimePrompt` 和 `NumberPrompt`。

`reserveTable` 对话如下所示：

```javascript
// Reserve a table:
// Help the user to reserve a table

var reservationInfo = {
    dateTime: '',
    partySize: '',
    reserveName: ''
}

dialogs.add('reserveTable', [
    async function(dc, args, next){
        await dc.context.sendActivity("Welcome to the reservation service.");
        reservationInfo = {}; // Clears any previous data
        await dc.prompt('dateTimePrompt', "Please provide a reservation date and time.");
    },
    async function(dc, result){
        reservationInfo.dateTime = result[0].value;

        // Ask for next info
        await dc.prompt('partySizePrompt', "How many people are in your party?");
    },
    async function(dc, result){
        reservationInfo.partySize = result;

        // Ask for next info
        await dc.prompt('textPrompt', "Who's name will this be under?");
    },
    async function(dc, result){
        reservationInfo.reserveName = result;

        // Confirm reservation
        var msg = `Reservation confirmed. Reservation details: 
            <br/>Date/Time: ${reservationInfo.dateTime} 
            <br/>Party size: ${reservationInfo.partySize} 
            <br/>Reservation name: ${reservationInfo.reserveName}`;
        await dc.context.sendActivity(msg);
        await dc.end();
    }
]);

```

现在，你已准备好将此消息连接到机器人逻辑中。

## <a name="start-the-dialog"></a>启动对话

修改机器人的 `processActivity()` 方法，使其如下所示：

```javascript
// Listen for incoming activity 
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type === 'message') {
            // State will store all of your information 
            const state = conversationState.get(context);
            const dc = dialogs.createContext(context, state);

            if(context.activity.text.match(/hi/ig)){
                await dc.begin('greeting');
            }
            if(context.activity.text.match(/reserve table/ig)){
                await dc.begin('reserveTable');
            }
            else{
                // Continue executing the "current" dialog, if any.
                await dc.continue();
            }
        }
    });
});
```

在执行时，每当用户发送包含字符串 `reserve table` 的消息，机器人都将启动 `reserveTable` 聊天。

## <a name="next-steps"></a>后续步骤

??? 

> [!div class="nextstepaction"]
> [保存用户状态数据](bot-builder-tutorial-save-data.md)
