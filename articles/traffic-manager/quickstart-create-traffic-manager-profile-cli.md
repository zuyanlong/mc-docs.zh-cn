---
title: 快速入门 - 为应用程序的 HA 创建配置文件 - Azure CLI - Azure 流量管理器
description: 本快速入门文章介绍如何创建流量管理器配置文件，以生成高度可用的 Web 应用程序。
services: traffic-manager
mnager: kumud
Customer intent: As an IT admin, I want to direct user traffic to ensure high availability of web applications.
ms.service: traffic-manager
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
origin.date: 08/26/2020
author: rockboyfor
ms.date: 09/28/2020
ms.testscope: yes
ms.testdate: 09/28/2020
ms.author: v-yeche
ms.custom: devx-track-azurecli
ms.openlocfilehash: fa006fef0fd7db1916b7d51c14a63794e8a011d8
ms.sourcegitcommit: 71953ae66ddfc07c5d3b4eb55ff8639281f39b40
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/27/2020
ms.locfileid: "91395442"
---
<!--Verify sucessfully-->

# <a name="quickstart-create-a-traffic-manager-profile-for-a-highly-available-web-application-using-azure-cli"></a>快速入门：使用 Azure CLI 创建流量管理器配置文件以实现 Web 应用程序的高可用性

本快速入门介绍如何创建流量管理器配置文件，以便实现 Web 应用程序的高度可用性。

在本快速入门中，我们将创建 Web 应用程序的两个实例。 每个实例在不同的 Azure 区域运行。 需根据[终结点优先级](traffic-manager-routing-methods.md#priority-traffic-routing-method)创建流量管理器配置文件。 此配置文件将用户流量定向到运行 Web 应用程序的主站点。 流量管理器持续监视 Web 应用程序。 如果主站点不可用，它会提供目标为备份站点的自动故障转移。

如果还没有 Azure 订阅，请现在就创建一个[试用帐户](https://www.azure.cn/pricing/1rmb-trial)。

[!INCLUDE [azure-cli-2-azurechinacloud-environment-parameter](../../includes/azure-cli-2-azurechinacloud-environment-parameter.md)]

如果选择在本地安装并使用 CLI，本教程要求运行 Azure CLI 2.0.28 版或更高版本。 要查找版本，请运行 `az --version`。 如果需要进行安装或升级，请参阅[安装 Azure CLI](https://docs.azure.cn/cli/install-azure-cli)。

## <a name="create-a-resource-group"></a>创建资源组
使用 [az group create](https://docs.azure.cn/cli/group#az-group-create) 创建资源组。 Azure 资源组是在其中部署和管理 Azure 资源的逻辑容器。

以下示例在“chinaeast”  位置创建名为“myResourceGroup”  的资源组：

```azurecli

  az group create \
    --name myResourceGroup \
    --location chinaeast

```

## <a name="create-a-traffic-manager-profile"></a>创建流量管理器配置文件

使用 [az network traffic-manager profile create](https://docs.azure.cn/cli/network/traffic-manager/profile#az-network-traffic-manager-profile-create) 创建流量管理器配置文件，以根据终结点优先级定向用户流量。

在以下示例中，请将 **<profile_name**> 替换为唯一的流量管理器配置文件名称。

```azurecli

az network traffic-manager profile create \
    --name <profile_name> \
    --resource-group myResourceGroup \
    --routing-method Priority \
    --path "/" \
    --protocol HTTP \
    --unique-dns-name <profile_name> \
    --ttl 30 \
    --port 80

```

## <a name="create-web-apps"></a>创建 Web 应用

本快速入门需要两个部署在两个不同的 Azure 区域（中国东部和中国北部）的 Web 应用程序实例。**** 每个都可以充当流量管理器的主终结点和故障转移终结点。

### <a name="create-web-app-service-plans"></a>创建 Web 应用服务计划
使用 [az appservice plan create](https://docs.azure.cn/cli/appservice/plan#az-appservice-plan-create) 为要部署在两个不同 Azure 区域中的两个 Web 应用程序实例创建 Web 应用服务计划。

在以下示例中，请将 **<appspname_chinaeast>** 和 **<appspname_chinanorth>** 替换为唯一的应用服务计划名称

```azurecli

az appservice plan create \
    --name <appspname_chinaeast> \
    --resource-group myResourceGroup \
    --location chinaeast \
    --sku S1

az appservice plan create \
    --name <appspname_chinanorth> \
    --resource-group myResourceGroup \
    --location chinanorth \
    --sku S1

```

### <a name="create-a-web-app-in-the-app-service-plan"></a>在应用服务计划中创建 Web 应用
在应用服务计划中使用 [az webapp create](https://docs.azure.cn/cli/webapp#az-webapp-create) 在“中国东部”和“中国北部”Azure 区域中创建 Web 应用程序的两个实例。****

在以下示例中，请将 **<app1name_chinaeast>** 和 **<app2name_chinanorth>** 替换为唯一的应用名称，将 **<appspname_chinaeast>** 和 **<appspname_chinanorth>** 替换为在上一部分用于创建应用服务计划的名称。

```azurecli

az webapp create \
    --name <app1name_chinaeast> \
    --plan <appspname_chinaeast> \
    --resource-group myResourceGroup

az webapp create \
    --name <app2name_chinanorth> \
    --plan <appspname_chinanorth> \
    --resource-group myResourceGroup

```

## <a name="add-traffic-manager-endpoints"></a>添加流量管理器终结点
使用 [az network traffic-manager endpoint create](https://docs.azure.cn/cli/network/traffic-manager/endpoint#az-network-traffic-manager-endpoint-create) 将两个 Web 应用作为流量管理器终结点添加到流量管理器配置文件，如下所示：

- 确定 Web 应用 ID，并将位于“中国东部”Azure 区域的 Web 应用添加为主要终结点，以路由所有用户流量。 
- 确定 Web 应用 ID，并将位于“中国北部”Azure 区域的 Web 应用添加为故障转移终结点。 

当主终结点不可用时，流量自动路由到故障转移终结点。

在下面的示例中，将 <app1name_chinaeast> 和 <app2name_chinanorth> 替换为在上一部分为每个区域创建的应用名称 。 然后，将 <profile_name> 替换为上一部分中使用的配置文件名称。 

**中国东部终结点**

```azurecli

az webapp show \
    --name <app1name_chinaeast> \
    --resource-group myResourceGroup \
    --query id

```

记下输出中显示的 ID，并在以下命令中使用该 ID 添加终结点：

```azurecli

az network traffic-manager endpoint create \
    --name <app1name_chinaeast> \
    --resource-group myResourceGroup \
    --profile-name <profile_name> \
    --type azureEndpoints \
    --target-resource-id <ID from az webapp show> \
    --priority 1 \
    --endpoint-status Enabled
```

**中国北部终结点**

```azurecli

az webapp show \
    --name <app2name_chinanorth> \
    --resource-group myResourceGroup \
    --query id

```

记下输出中显示的 ID，并在以下命令中使用该 ID 添加终结点：

```azurecli

az network traffic-manager endpoint create \
    --name <app1name_chinanorth> \
    --resource-group myResourceGroup \
    --profile-name <profile_name> \
    --type azureEndpoints \
    --target-resource-id <ID from az webapp show> \
    --priority 2 \
    --endpoint-status Enabled

```

## <a name="test-your-traffic-manager-profile"></a>测试流量管理器配置文件

在此部分，需检查流量管理器配置文件的域名。 此外还需将主终结点配置为不可用。 最后可以看到该 Web 应用仍然可用。 这是因为流量管理器将流量发送到故障转移终结点。

在下面的示例中，将 <app1name_chinaeast> 和 <app2name_chinanorth> 替换为在上一部分为每个区域创建的应用名称 。 然后，将 <profile_name> 替换为上一部分中使用的配置文件名称。

### <a name="determine-the-dns-name"></a>确定 DNS 名称

使用 [az network traffic-manager profile show](https://docs.azure.cn/cli/network/traffic-manager/profile#az-network-traffic-manager-profile-show) 确定流量管理器配置文件的 DNS 名称。

```azurecli

az network traffic-manager profile show \
    --name <profile_name> \
    --resource-group myResourceGroup \
    --query dnsConfig.fqdn

```

复制 **RelativeDnsName** 值。 流量管理器配置文件的 DNS 名称为“*http://<* relativednsname *>.trafficmanager.cn”* 。 

### <a name="view-traffic-manager-in-action"></a>查看正在运行的流量管理器
1. 在 Web 浏览器中输入流量管理器配置文件的 DNS 名称 (*http://<* relativednsname *>.trafficmanager.cn*)，以查看 Web 应用的默认网站。

    > [!NOTE]
    > 在本快速入门方案中，所有请求都路由到主终结点。 它设置为“优先级 1”。****
2. 若要查看流量管理器故障转移如何进行，请使用 [az network traffic-manager endpoint update](https://docs.azure.cn/cli/network/traffic-manager/endpoint#az-network-traffic-manager-endpoint-update) 禁用主要站点。

    ```azurecli

    az network traffic-manager endpoint update \
        --name <app1name_chinaeast> \
        --resource-group myResourceGroup \
        --profile-name <profile_name> \
        --type azureEndpoints \
        --endpoint-status Disabled

    ```

3. 复制流量管理器配置文件的 DNS 名称 (*http://<* relativednsname *>.trafficmanager.cn*)，以在新的 Web 浏览器会话中查看该网站。
4. 验证 Web 应用是否仍然可用。

## <a name="clean-up-resources"></a>清理资源

完成后，请使用 [az group delete](https://docs.azure.cn/cli/group#az-group-delete) 删除资源组、Web 应用程序和所有相关资源。

```azurecli

az group delete \
    --resource-group myResourceGroup

```

## <a name="next-steps"></a>后续步骤

在本快速入门中，我们创建了一个可为 Web 应用程序提供高可用性的流量管理器配置文件。 若要详细了解如何路由流量，请继续学习流量管理器教程。

> [!div class="nextstepaction"]
> [流量管理器教程](tutorial-traffic-manager-improve-website-response.md)

<!-- Update_Description: update meta properties, wording update, update link -->