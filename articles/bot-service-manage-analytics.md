---
title: 机器人分析 | Microsoft Docs
description: 了解如何借助 Bot Framework 中的分析功能通过数据收集和分析来改进机器人。
keywords: 机器人分析, application insights, 流量, 延迟, 集成, AppInsights
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: abs
ms.date: 12/13/2017
ms.openlocfilehash: 27dc7786554af14a24fc8d65b2f7ee31bc4864ef
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997907"
---
# <a name="bot-analytics"></a>机器人分析
Analytics 是 [Application Insights](/azure/application-insights/app-insights-analytics) 的扩展。 Application Insights 提供服务级和检测数据，例如流量、延迟和集成。 Analytics 提供有关用户、消息和通道数据的聊天级报告。

## <a name="view-analytics-for-a-bot"></a>查看机器人的分析
要访问 Analytics，请在开发人员门户中启动机器人并单击“Analytics”。

数据太多？ [启用并配置采样](/azure/application-insights/app-insights-sampling)，从而减少遥测流量和存储，同时保证统计上的分析正确无误。 

### <a name="specify-channel"></a>指定通道
选择在下图中显示的通道。 请注意，如果未在通道上启用机器人，则不会具有来自该通道的数据。

![选择通道](~/media/analytics-channels.png)

* 选中复选框可在图表中包含通道。
* 清除复选框可从图表中删除通道。

### <a name="specify-time-period"></a>指定时间段
分析功能仅可用于过去 90 天。 启用 Application Insights 时开始数据收集。

![选择时间段](~/media/analytics-timepick.png)

单击下拉列表菜单，然后单击图形应显示的时间量。
请注意，更改总体时间范围将导致图上的时间增量（X 轴）相应地发生变化。

### <a name="grand-totals"></a>总计
指定期限内的活动用户以及发送和接收的活动总数。
短划线 `--` 表示没有活动。

### <a name="retention"></a>保留
保留期跟踪发送一条消息后返回再发送另一条消息的用户数量。
以下图表显示了连续 10 天的情况；更改时间范围不影响结果。

![保留期图表](~/media/analytics-retention.png)

请注意，最近的日期可能是两天前；一位用户前天发送了消息，然后昨天返回了。

### <a name="user"></a>用户
用户图跟踪在指定时间范围内使用每个通道访问机器人的用户数量。

![用户图](~/media/analytics-users.png)

* 百分比图表显示使用每个通道的用户百分比。
* 折线图表示在特定时间访问机器人的用户数量。
* 折线图的图例表示哪种颜色代表哪个通道，并显示指定时间段内的用户总数。

### <a name="activities"></a>活动
“活动”图跟踪在指定期限内使用某一通道发送和接收的活动数。

![活动图](~/media/analytics-activities.png)

* 百分比图表显示通过每个通道传递的活动百分比。
* 折线图指示在指定期限内发送和接收的活动数量。
* 折线图的图例指示每个通道对应的线条颜色，以及在指定时段内该通道上发送和接收的活动总数。 

## <a name="enable-analytics"></a>启用分析
启用并配置 Application Insights 后，Analytics 才可用。 Application Insights 将在启用后立即开始收集数据。 例如，如果 1 周前为运行了 6 个月的机器人启用 Application Insights，则仅收集 1 周的数据。
> [!NOTE]
> 要使用 Analytics，需具备 Azure 订阅和 Application Insights [资源](/azure/application-insights/app-insights-create-new-resource)。
要访问 Application Insights，请在 [Bot Framework 门户](https://dev.botframework.com/)中启动机器人并单击“设置”。

1. 创建 Application Insights [资源](/azure/application-insights/app-insights-create-new-resource)。
2. 在仪表板中启动机器人。 单击“设置”，再向下滚动到“Analytics”部分。
3. 输入信息，将机器人连接到 Application Insights。 所有字段都是必填字段。

![连接见解](~/media/analytics-enable.png)

### <a name="appinsights-instrumentation-key"></a>AppInsights 检测密钥
要找到该值，请打开 Application Insights，再导航到“配置” > “属性”。

### <a name="appinsights-api-key"></a>AppInsights API 密钥
提供 Azure App Insights API 密钥。 了解如何[生成新的 API 密钥](https://dev.applicationinsights.io/documentation/Authorization/API-key-and-App-ID)。 只需要读取权限。

### <a name="appinsights-application-id"></a>AppInsights 应用程序 ID
要找到该值，请打开 Application Insights，再导航到“配置” > “API 访问权限”。

要详细了解如何查找这些值，请参阅 [Application Insights 密钥](~/bot-service-resources-app-insights-keys.md)。

## <a name="additional-resources"></a>其他资源
* [Application Insights 密钥](~/bot-service-resources-app-insights-keys.md)