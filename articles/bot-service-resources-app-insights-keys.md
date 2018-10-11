---
title: Application Insights 密钥 | Microsoft Docs
description: 了解如何获取 Application Insights 密钥以向机器人添加遥测。
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 07fb6e9630996a61932da99b0575d43f4604141e
ms.sourcegitcommit: 3cb288cf2f09eaede317e1bc8d6255becf1aec61
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/27/2018
ms.locfileid: "47389426"
---
# <a name="application-insights-keys"></a>Application Insights 密钥

Azure **Application Insights** 在 Microsoft Azure 资源中显示有关应用程序的数据。 若要向机器人添加遥测，需要 Azure 订阅以及为机器人创建的 Application Insights 资源。 从此资源可以获得用于配置机器人的三个密钥：

1. 检测密钥
2. 应用程序 ID
3. API 密钥

本主题将演示如何创建这些 Application Insights 密钥。

> [!NOTE]
> 在机器人创建或注册过程中，可以选择启用或禁用 **Application Insights**。 如果已启用它，则机器人已拥有它所需的所有必需的 Application Insights 密钥。 但是，如果已将其禁用，则可以按照本主题中的说明操作，以帮助你手动创建这些密钥。

## <a name="instrumentation-key"></a>检测密钥

若要获取检测密钥，请执行以下操作：
1. 在 [Azure 门户](http://portal.azure.com)中的“监视”部分下，创建一个新的 **Application Insights** 资源（或使用现有资源）。
![Application Insights 列表的门户屏幕截图](~/media/portal-app-insights-add-new.png)

2. 从 Application Insights 资源列表中，单击刚刚创建的 Application Insight 资源。

3. 单击“概览”。

4. 展开“概要”块并找到“检测密钥”。 
![检测密钥的门户屏幕截图](~/media/portal-app-insights-instrumentation-key.png)

5. 复制**检测密钥**并将其粘贴到机器人设置的“Application Insights 检测密钥”字段中。

## <a name="application-id"></a>应用程序 ID

若要获取应用程序 ID，请执行以下操作：
1. 在 Application Insights 资源中，单击“API 访问权限”。

2. 复制**应用程序 ID** 并将其粘贴到机器人设置的“Application Insights 应用程序 ID”字段中。 
![应用程序 ID 的门户屏幕截图](~/media/portal-app-insights-appid.png)

## <a name="api-key"></a>API 密钥

若要获取 API 密钥，请执行以下操作：
1. 在 Application Insights 资源中，单击“API 访问权限”。

2. 单击“创建 API 密钥”。

3. 输入简短说明，选中“读取遥测”选项，然后单击“生成密钥”按钮。
![应用程序 ID 和 API 密钥的门户屏幕截图](~/media/portal-app-insights-appid-apikey.png)

   > [!WARNING]
   > 复制此 **API 密钥**并保存，因为此密钥永远不会再次向你显示。 如果丢失此密钥，则必须创建一个新密钥。

4. 将 API 密钥复制到机器人设置的“Application Insights API 密钥”字段。

## <a name="additional-resources"></a>其他资源
有关如何将这些字段连接到机器人设置的详细信息，请参阅[启用分析](~/bot-service-manage-analytics.md#enable-analytics)。