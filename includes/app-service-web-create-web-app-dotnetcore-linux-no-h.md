---
title: include 文件
description: include 文件
services: app-service
author: cephalin
ms.service: app-service
ms.topic: include
origin.date: 04/22/2020
ms.date: 08/13/2020
ms.author: v-tawe
ms.custom: include file
ms.openlocfilehash: d3a07bdf52d65a10d80006c40ee573cc001a05b7
ms.sourcegitcommit: 9d9795f8a5b50cd5ccc19d3a2773817836446912
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/14/2020
ms.locfileid: "92170579"
---
在 `myAppServicePlan` 应用服务计划中创建一个 [Web 应用](../articles/app-service/overview.md#app-service-on-linux)。 

在 Azure CLI 中，可以使用 [`az webapp create`](/cli/webapp?view=azure-cli-latest) 命令。 在以下示例中，将 `<app-name>` 替换为全局唯一的应用名称（有效字符是 `a-z`、`0-9` 和 `-`）。 运行时设置为 `DOTNETCORE|3.1`。 若要查看所有受支持的运行时，请运行 [`az webapp list-runtimes --linux`](/cli/webapp?view=azure-cli-latest)。

```azurecli
# Bash
az webapp create --resource-group myResourceGroup --plan myAppServicePlan --name <app-name> --runtime "DOTNETCORE|3.1" --deployment-local-git
# PowerShell
az --% webapp create --resource-group myResourceGroup --plan myAppServicePlan --name <app-name> --runtime "DOTNETCORE|3.1" --deployment-local-git
```

创建 Web 应用后，Azure CLI 会显示类似于以下示例的输出：

<pre>
Local git is configured with url of 'https://<username>@<app-name>.scm.chinacloudsites.cn/<app-name>.git'
{
  "availabilityState": "Normal",
  "clientAffinityEnabled": true,
  "clientCertEnabled": false,
  "cloningInfo": null,
  "containerSize": 0,
  "dailyMemoryTimeQuota": 0,
  "defaultHostName": "<app-name>.chinacloudsites.cn",
  "deploymentLocalGitUrl": "https://<username>@<app-name>.scm.chinacloudsites.cn/<app-name>.git",
  "enabled": true,
  < JSON data removed for brevity. >
}
</pre>

已在 Linux 容器中创建了空的 Web 应用并启用了 Git 部署。

> [!NOTE]
> Git 远程的 URL 将显示在 `deploymentLocalGitUrl` 属性中，其格式为 `https://<username>@<app-name>.scm.chinacloudsites.cn/<app-name>.git`。 保存此 URL，后续将会用到。
>
