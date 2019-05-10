---
ms.openlocfilehash: 53db401b00e53b964027f08c32ca7a38a4915f60
ms.sourcegitcommit: 4ff7a8772124a567f43e2c3e13aded368c4002e3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/03/2019
ms.locfileid: "65035698"
---
```cmd
az group create --name <resource-group-name> --location <geographic-location> --verbose
```

| 选项 | 说明 |
|:---|:---|
| --name | 资源组的唯一名称。 切勿在此名称中包含空格或下划线。 |
| --location | 用于创建资源组的地理位置。 例如 `eastus`、`westus`、`westus2` 等。 使用 `az account list-locations` 来获取位置列表。 |