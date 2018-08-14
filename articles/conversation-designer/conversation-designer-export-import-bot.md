---
title: 导入和导出聊天设计器机器人 | Microsoft Docs
description: 了解如何导入和导出聊天设计器机器人。
author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ROBOTS: NoIndex, NoFollow
ms.openlocfilehash: ed055cb0f75d148e6a0dd3248366851901d9a675
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/18/2018
ms.locfileid: "39298445"
---
# <a name="export-and-import-a-conversation-designer-bot"></a>导出和导入聊天设计器机器人

聊天设计器中的机器人可以作为 .zip 文件导出到计算机。 这允许你备份机器人。 稍后可以使用任一导出的 .zip 文件将机器人还原到以前的状态。 

## <a name="export-a-conversation-designer-bot"></a>导出聊天设计器机器人

通过导出，可将聊天设计器机器人的当前状态保存到本地计算机。 

若要导出聊天设计器机器人，请执行以下操作：
1. 在导航面板的右上角，单击省略号 (**...**)。
2. 单击“导出”。 服务器将收集必要的信息，并返回一个用于保存 .zip 文件的选项。
3. 将导出的 .zip 文件保存到本地计算机。

机器人现在以 .zip 文件的形式保存在计算机上。 稍后可以通过将其[导入](#import-a-conversation-designer-bot)回机器人，将机器人还原到此状态。

## <a name="import-a-conversation-designer-bot"></a>导入聊天设计器机器人

通过导入，可将聊天设计器机器人还原到以前的状态。 导入时会将当前机器人替换为导入机器人。 如果不想丢失当前机器人当前拥有的内容，请确保在执行导入操作之前[导出](#export-a-conversation-designer-bot)当前机器人。

若要导入聊天设计器机器人，请执行以下操作：
1. 在导航面板的右上角，单击省略号 (**...**)。
2. 单击“导入”。 
3. 如果要将工作保存在当前机器人中，请选择“备份并导入”选项。 此选项会将当前机器人保存到本地计算机，然后询问你要导入的 .zip 文件的位置。 否则，请选择“导入而不备份”。

现在已导入机器人。

> [!NOTE]
> 只能导入由聊天设计器导出的机器人。

## <a name="import-a-luis-app-into-a-conversation-designer-bot"></a>将 LUIS 应用导入聊天设计器机器人

如果你有一个 LUIS 应用，并且想在聊天设计器机器人中使用它，则可以将 LUIS 应用导入到聊天设计器机器人中。 从概念上讲，此过程要求导出 LUIS 应用和聊天设计器机器人，然后将机器人 **luis.model** 文件的内容与 **luis.json** 文件的内容交换。 接着，将所做更改导回到聊天设计器机器人。 从根本上讲，就是将机器人中的 LUIS 意向替换为 LUIS 应用的意向。 因此，建议在开始自定义机器人的 LUIS 意向之前执行此导入操作；否则，此导入操作将覆盖你的所有工作。

> [!NOTE]
> 如果在与机器人关联的 [LUIS](https://luis.ai) 应用中进行更改（每个聊天设计器机器人都有相应的 LUIS 应用），则无需执行这些步骤。 你只需进入聊天设计器机器人并单击 [**保存**](conversation-designer-save-bot.md) 即可。

若要将 LUIS 应用导入聊天设计器机器人，请执行以下操作：

1. 从 [LUIS.ai](https://luis.ai) 打开 LUIS 应用，然后单击“设置”。
2. 选择要使用的应用**版本**，然后单击 **{ }** 操作图标。 此操作会将 **luis.json** 文件下载到本地计算机。 
3. 从聊天设计器中[创建新的机器人](conversation-designer-create-bot.md#create-a-conversation-designer-bot)或打开现有的机器人。
4. [导出](#export-a-conversation-designer-bot)机器人。 这会将机器人以 .zip 文件的形式导出到计算机。
5. 导航到导出的 .zip 文件并将其解压缩。
6. 在文本编辑器中打开 **luis.model** 文件，并将此文件的内容替换为 **luis.json** 文件的内容。 保存文件。
7. 压缩聊天设计器机器人的文件夹。
8. 将机器人[导入](#import-a-conversation-designer-bot)回聊天设计器机器人。

机器人现在可以使用刚刚导入的新 LUIS 意向。

## <a name="next-step"></a>后续步骤
> [!div class="nextstepaction"]
> [任务](conversation-designer-tasks.md)
