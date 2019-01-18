---
ms.openlocfilehash: 53db401b00e53b964027f08c32ca7a38a4915f60
ms.sourcegitcommit: bdb981c0b11ee99d128e30ae0462705b2dae8572
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/17/2019
ms.locfileid: "54360846"
---
```cmd
az group create --name <resource-group-name> --location <geographic-location> --verbose
```

| 选项 | 说明 |
|:---|:---|
| --name | 资源组的唯一名称。 切勿在此名称中包含空格或下划线。 |
| --location | 用于创建资源组的地理位置。 例如 `eastus`、`westus`、`westus2` 等。 使用 `az account list-locations` 来获取位置列表。 |