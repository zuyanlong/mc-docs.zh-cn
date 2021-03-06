---
title: 标记资源、资源组和订阅以便对其进行逻辑组织
description: 演示如何应用标记来组织 Azure 资源进行计费和管理。
ms.topic: conceptual
origin.date: 07/27/2020
author: rockboyfor
ms.date: 10/12/2020
ms.testscope: yes
ms.testdate: 07/13/2020
ms.author: v-yeche
ms.custom: devx-track-azurecli
ms.openlocfilehash: 29bb5f6252fc32985e54e76274963ccc08be6ef6
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937272"
---
# <a name="use-tags-to-organize-your-azure-resources-and-management-hierarchy"></a>使用标记对 Azure 资源和管理层次结构进行组织

可将标记应用到 Azure 资源、资源组和订阅，以便有条理地将它们组织成分类。 每个标记均由名称和值对组成。 例如，可以对生产中的所有资源应用名称“Environment”和值“Production”。

<!--Not Available on [Resource naming and tagging decision guide](/cloud-adoption-framework/decision-guides/resource-tagging/?toc=/azure-resource-manager/management/toc.json)-->

> [!IMPORTANT]
> 标记名称对于操作不区分大小写。 系统会更新或检索具有标记名称的标记，而不考虑大小写。 但是，资源提供程序可能会保留为标记名称提供的大小写。 你将在成本报表中看到该大小写。
> 
> 标记值区分大小写。

[!INCLUDE [Handle personal data](../../../includes/gdpr-intro-sentence.md)]

## <a name="required-access"></a>所需访问权限

若要将标记应用到资源，必须对 Microsoft.Resources/tags 资源类型拥有写入权限。 [标记参与者](../../role-based-access-control/built-in-roles.md#tag-contributor)角色可让你将标记应用到实体，无需访问该实体本身。 目前，标记参与者角色无法通过门户将标记应用到资源或资源组。 该角色可以通过门户将标记应用到订阅。 它支持通过 PowerShell 和 REST API 执行所有标记操作。  

[参与者](../../role-based-access-control/built-in-roles.md#contributor)角色还授予对任何实体应用标记所需的访问权限。 要将标记仅应用于一种资源类型，请使用该资源的参与者角色。 例如，要将标记应用到虚拟机，请使用[虚拟机参与者](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor)。

## <a name="powershell"></a>PowerShell

### <a name="apply-tags"></a>应用标记

Azure PowerShell 提供两个用于应用标记的命令 - [New-AzTag](https://docs.microsoft.com/powershell/module/az.resources/new-aztag) 和 [Update-AzTag](https://docs.microsoft.com/powershell/module/az.resources/update-aztag)。 必须安装 Az.Resources 模块 1.12.0 或更高版本。 可以使用 `Get-Module Az.Resources` 检查自己的版本。 可以安装该模块，或[安装 Azure PowerShell](https://docs.microsoft.com/powershell/azure/install-az-ps) 3.6.1 或更高版本。

New-AzTag 替换资源、资源组或订阅中的所有标记。 调用该命令时，请传入要标记的实体的资源 ID。

以下示例将一组标记应用到存储帐户：

```powershell
$tags = @{"Dept"="Finance"; "Status"="Normal"}
$resource = Get-AzResource -Name demoStorage -ResourceGroup demoGroup
New-AzTag -ResourceId $resource.id -Tag $tags
```

该命令完成后，会发现该资源带有两个标记。

```output
Properties :
        Name    Value
        ======  =======
        Dept    Finance
        Status  Normal
```

如果再次运行该命令（但这一次使用不同的标记），则会发现以前的标记已删除。

```powershell
$tags = @{"Team"="Compliance"; "Environment"="Production"}
New-AzTag -ResourceId $resource.id -Tag $tags
```

```output
Properties :
        Name         Value
        ===========  ==========
        Environment  Production
        Team         Compliance
```

若要将标记添加到已经有标记的资源，请使用 Update-AzTag。 将 -Operation 参数设置为 Merge 。

```powershell
$tags = @{"Dept"="Finance"; "Status"="Normal"}
Update-AzTag -ResourceId $resource.id -Tag $tags -Operation Merge
```

你会发现，已将两个新标记添加到了两个现有标记中。

```output
Properties :
        Name         Value
        ===========  ==========
        Status       Normal
        Dept         Finance
        Team         Compliance
        Environment  Production
```

每个标记名称只能有一个值。 如果为标记提供新值，则即使使用合并操作，也会替换旧值。 以下示例将 Status 标记从 Normal 更改为 Green。

```powershell
$tags = @{"Status"="Green"}
Update-AzTag -ResourceId $resource.id -Tag $tags -Operation Merge
```

```output
Properties :
        Name         Value
        ===========  ==========
        Status       Green
        Dept         Finance
        Team         Compliance
        Environment  Production
```

将 -Operation 参数设置为 Replace 时，会将现有标记替换为新的标记集 。

```powershell
$tags = @{"Project"="ECommerce"; "CostCenter"="00123"; "Team"="Web"}
Update-AzTag -ResourceId $resource.id -Tag $tags -Operation Replace
```

资源中只会保留新标记。

```output
Properties :
        Name        Value
        ==========  =========
        CostCenter  00123
        Team        Web
        Project     ECommerce
```

相同的命令也适用于资源组或订阅。 传入要标记的资源组或订阅的标识符。

若要将一组新标记添加到资源组，请使用：

```powershell
$tags = @{"Dept"="Finance"; "Status"="Normal"}
$resourceGroup = Get-AzResourceGroup -Name demoGroup
New-AzTag -ResourceId $resourceGroup.ResourceId -tag $tags
```

若要更新资源组的标记，请使用：

```powershell
$tags = @{"CostCenter"="00123"; "Environment"="Production"}
$resourceGroup = Get-AzResourceGroup -Name demoGroup
Update-AzTag -ResourceId $resourceGroup.ResourceId -Tag $tags -Operation Merge
```

若要将一组新标记添加到订阅，请使用：

```powershell
$tags = @{"CostCenter"="00123"; "Environment"="Dev"}
$subscription = (Get-AzSubscription -SubscriptionName "Example Subscription").Id
New-AzTag -ResourceId "/subscriptions/$subscription" -Tag $tags
```

若要更新订阅的标记，请使用：

```powershell
$tags = @{"Team"="Web Apps"}
$subscription = (Get-AzSubscription -SubscriptionName "Example Subscription").Id
Update-AzTag -ResourceId "/subscriptions/$subscription" -Tag $tags -Operation Merge
```

一个资源组中可能有多个同名资源。 在这种情况下，可以通过以下命令设置每个资源：

```powershell
$resource = Get-AzResource -ResourceName sqlDatabase1 -ResourceGroupName examplegroup
$resource | ForEach-Object { Update-AzTag -Tag @{ "Dept"="IT"; "Environment"="Test" } -ResourceId $_.ResourceId -Operation Merge }
```

### <a name="list-tags"></a>列出标记

若要获取资源、资源组或订阅的标记，请使用 [Get-AzTag](https://docs.microsoft.com/powershell/module/az.resources/get-aztag) 命令并传入实体的资源 ID。

若要查看资源的标记，请使用：

```powershell
$resource = Get-AzResource -Name demoStorage -ResourceGroup demoGroup
Get-AzTag -ResourceId $resource.id
```

若要查看资源组的标记，请使用：

```powershell
$resourceGroup = Get-AzResourceGroup -Name demoGroup
Get-AzTag -ResourceId $resourceGroup.ResourceId
```

若要查看订阅的标记，请使用：

```powershell
$subscription = (Get-AzSubscription -SubscriptionName "Example Subscription").Id
Get-AzTag -ResourceId "/subscriptions/$subscription"
```

### <a name="list-by-tag"></a>按标记列出

若要获取具有特定标记名称和值的资源，请使用：

```powershell
(Get-AzResource -Tag @{ "CostCenter"="00123"}).Name
```

若要获取具有特定标记名称和任意标记值的资源，请使用：

```powershell
(Get-AzResource -TagName "Dept").Name
```

若要获取具有特定标记名称和值的资源组，请使用：

```powershell
(Get-AzResourceGroup -Tag @{ "CostCenter"="00123" }).ResourceGroupName
```

### <a name="remove-tags"></a>删除标记

若要删除特定的标记，请使用 Update-AzTag 并将 -Operation 设置为 Delete  。 传入要删除的标记。

```powershell
$removeTags = @{"Project"="ECommerce"; "Team"="Web"}
Update-AzTag -ResourceId $resource.id -Tag $removeTags -Operation Delete
```

指定的标记随即被删除。

```output
Properties :
        Name        Value
        ==========  =====
        CostCenter  00123
```

若要删除所有标记，请使用 [Remove-AzTag](https://docs.microsoft.com/powershell/module/az.resources/remove-aztag) 命令。

```powershell
$subscription = (Get-AzSubscription -SubscriptionName "Example Subscription").Id
Remove-AzTag -ResourceId "/subscriptions/$subscription"
```

## <a name="azure-cli"></a>Azure CLI

### <a name="apply-tags"></a>应用标记

将标记添加到资源组或资源时，可以覆盖现有的标记，或将新标记追加到现有标记之后。

若要覆盖资源的标记，请使用：

```azurecli
az resource tag --tags 'Dept=IT' 'Environment=Test' -g examplegroup -n examplevnet --resource-type "Microsoft.Network/virtualNetworks"
```

若将标记追加到资源的现有标记之后，请使用：

```azurecli
az resource update --set tags.'Status'='Approved' -g examplegroup -n examplevnet --resource-type "Microsoft.Network/virtualNetworks"
```

若要覆盖资源组的现有标记，请使用：

```azurecli
az group update -n examplegroup --tags 'Environment=Test' 'Dept=IT'
```

若要将标记追加到资源组的现有标记之后，请使用：

```azurecli
az group update -n examplegroup --set tags.'Status'='Approved'
```

目前，Azure CLI 没有用于将标记应用于订阅的命令。 但是，可以使用 CLI 来部署将标记应用于订阅的 ARM 模板。 请参阅[将标记应用于资源组或订阅](#apply-tags-to-resource-groups-or-subscriptions)。

### <a name="list-tags"></a>列出标记

若要查看资源的现有标记，请使用：

```azurecli
az resource show -n examplevnet -g examplegroup --resource-type "Microsoft.Network/virtualNetworks" --query tags
```

若要查看资源组的现有标记，请使用：

```azurecli
az group show -n examplegroup --query tags
```

该脚本返回以下格式：

```json
{
  "Dept"        : "IT",
  "Environment" : "Test"
}
```

### <a name="list-by-tag"></a>按标记列出

若要获取具有特定标记和值的所有资源，请使用 `az resource list`：

```azurecli
az resource list --tag Dept=Finance
```

若要获取具有特定标记的资源组，请使用 `az group list`：

```azurecli
az group list --tag Dept=IT
```

### <a name="handling-spaces"></a>处理空格

如果标记名称或值包含空格，则必须执行几个额外的步骤。 

Azure CLI 中的 `--tags` 参数可以接受包含字符串数组的字符串。 下面的示例将覆盖一个资源组中的标记，该资源组中的标记包含空格和连字符： 

```azurecli
TAGS=("Cost Center=Finance-1222" "Location=China North")
az group update --name examplegroup --tags "${TAGS[@]}"
```

使用 `--tags` 参数创建或更新资源组或资源时，可以使用相同的语法。

若要使用 `--set` 参数更新标记，必须以字符串的形式传递键和值。 下面的示例将单个标记追加到资源组：

```azurecli
TAG="Cost Center='Account-56'"
az group update --name examplegroup --set tags."$TAG"
```

在此示例中，标记值使用单引号进行标记，因为该值包含连字符。

你可能还需要将标记应用于多个资源。 下面的示例在标记可能包含空格时将资源组中的所有标记应用于其资源：

```azurecli
jsontags=$(az group show --name examplegroup --query tags -o json)
tags=$(echo $jsontags | tr -d '{}"' | sed 's/: /=/g' | sed "s/\"/'/g" | sed 's/, /,/g' | sed 's/ *$//g' | sed 's/^ *//g')
origIFS=$IFS
IFS=','
read -a tagarr <<< "$tags"
resourceids=$(az resource list -g examplegroup --query [].id --output tsv)
for id in $resourceids
do
  az resource tag --tags "${tagarr[@]}" --id $id
done
IFS=$origIFS
```

## <a name="templates"></a>模板

可以在使用资源管理器模板进行部署期间标记资源、资源组和订阅。

### <a name="apply-values"></a>应用值

以下示例部署具有三个标记的存储帐户。 两个标记（`Dept` 和 `Environment`）设置为文本值。 一个标记 (`LastDeployed`) 设置为默认值为当前日期的参数。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "utcShort": {
            "type": "string",
            "defaultValue": "[utcNow('d')]"
        },
        "location": {
            "type": "string",
            "defaultValue": "[resourceGroup().location]"
        }
    },
    "resources": [
        {
            "apiVersion": "2019-04-01",
            "type": "Microsoft.Storage/storageAccounts",
            "name": "[concat('storage', uniqueString(resourceGroup().id))]",
            "location": "[parameters('location')]",
            "tags": {
                "Dept": "Finance",
                "Environment": "Production",
                "LastDeployed": "[parameters('utcShort')]"
            },
            "sku": {
                "name": "Standard_LRS"
            },
            "kind": "Storage",
            "properties": {}
        }
    ]
}
```

### <a name="apply-an-object"></a>应用对象

可以定义一个存储多个标记的对象参数，并将该对象应用于标记元素。 此方法比上一个示例更灵活，因为对象可以使用不同的属性。 对象中的每个属性会成为资源的单独标记。 以下示例有一个名为 `tagValues` 的参数，该标记应用于标记元素。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "location": {
            "type": "string",
            "defaultValue": "[resourceGroup().location]"
        },
        "tagValues": {
            "type": "object",
            "defaultValue": {
                "Dept": "Finance",
                "Environment": "Production"
            }
        }
    },
    "resources": [
        {
            "apiVersion": "2019-04-01",
            "type": "Microsoft.Storage/storageAccounts",
            "name": "[concat('storage', uniqueString(resourceGroup().id))]",
            "location": "[parameters('location')]",
            "tags": "[parameters('tagValues')]",
            "sku": {
                "name": "Standard_LRS"
            },
            "kind": "Storage",
            "properties": {}
        }
    ]
}
```

### <a name="apply-a-json-string"></a>应用 JSON 字符串

要将多个值存储在单个标记中，请应用表示这些值的 JSON 字符串。 整个 JSON 字符串存储为一个标记，该标记不能超过 256 个字符。 以下示例有一个名为 `CostCenter` 的标记，其中包含 JSON 字符串中的多个值：  

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "location": {
            "type": "string",
            "defaultValue": "[resourceGroup().location]"
        }
    },
    "resources": [
        {
            "apiVersion": "2019-04-01",
            "type": "Microsoft.Storage/storageAccounts",
            "name": "[concat('storage', uniqueString(resourceGroup().id))]",
            "location": "[parameters('location')]",
            "tags": {
                "CostCenter": "{\"Dept\":\"Finance\",\"Environment\":\"Production\"}"
            },
            "sku": {
                "name": "Standard_LRS"
            },
            "kind": "Storage",
            "properties": {}
        }
    ]
}
```

### <a name="apply-tags-from-resource-group"></a>应用资源组中的标记

若要将资源组中的标记应用于资源，请使用 [resourceGroup()](../templates/template-functions-resource.md#resourcegroup) 函数。 获取标记值时，请使用 `tags[tag-name]` 语法而不是 `tags.tag-name` 语法，因为有些字符在点表示法中无法正确解析。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "location": {
            "type": "string",
            "defaultValue": "[resourceGroup().location]"
        }
    },
    "resources": [
        {
            "apiVersion": "2019-04-01",
            "type": "Microsoft.Storage/storageAccounts",
            "name": "[concat('storage', uniqueString(resourceGroup().id))]",
            "location": "[parameters('location')]",
            "tags": {
                "Dept": "[resourceGroup().tags['Dept']]",
                "Environment": "[resourceGroup().tags['Environment']]"
            },
            "sku": {
                "name": "Standard_LRS"
            },
            "kind": "Storage",
            "properties": {}
        }
    ]
}
```

### <a name="apply-tags-to-resource-groups-or-subscriptions"></a>将标记应用到多个资源组或订阅

可以通过部署 Microsoft.Resources/tags 资源类型，将标记添加到资源组或订阅。 标记会应用到此部署的目标资源组或订阅。 每次部署模板时，都要替换以前应用的任何标记。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "tagName": {
            "type": "string",
            "defaultValue": "TeamName"
        },
        "tagValue": {
            "type": "string",
            "defaultValue": "AppTeam1"
        }
    },
    "variables": {},
    "resources": [
        {
            "type": "Microsoft.Resources/tags",
            "name": "default",
            "apiVersion": "2019-10-01",
            "dependsOn": [],
            "properties": {
                "tags": {
                    "[parameters('tagName')]": "[parameters('tagValue')]"
                }
            }
        }
    ]
}
```

若要将标记应用到资源组，请使用 PowerShell 或 Azure CLI。 部署到要标记的资源组。

```powershell
New-AzResourceGroupDeployment -ResourceGroupName exampleGroup -TemplateFile https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/tags.json
```

```azurecli
az deployment group create --resource-group exampleGroup --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/tags.json
```

若要将标记应用到订阅，请使用 PowerShell 或 Azure CLI。 部署到要标记的订阅。

```powershell
New-AzSubscriptionDeployment -name tagresourcegroup -Location chinanorth2 -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/tags.json
```

```azurecli
az deployment sub create --name tagresourcegroup --location chinanorth2 --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/tags.json
```

有关订阅部署的详细信息，请参阅[在订阅级别创建资源组和资源](../templates/deploy-to-subscription.md)。

以下模板将对象中的标记添加到资源组或订阅。

```json
"$schema": "https://schema.management.azure.com/schemas/2018-05-01/subscriptionDeploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "tags": {
            "type": "object",
            "defaultValue": {
                "TeamName": "AppTeam1",
                "Dept": "Finance",
                "Environment": "Production"
            }
        }
    },
    "variables": {},
    "resources": [
        {
            "type": "Microsoft.Resources/tags",
            "name": "default",
            "apiVersion": "2019-10-01",
            "dependsOn": [],
            "properties": {
                "tags": "[parameters('tags')]"
            }
        }
    ]
}
```

## <a name="portal"></a>门户

[!INCLUDE [resource-manager-tag-resource](../../../includes/resource-manager-tag-resources.md)]

## <a name="rest-api"></a>REST API

若要通过 Azure REST API 处理标记，请使用：

* [标记 - 在范围内创建或更新](https://docs.microsoft.com/rest/api/resources/tags/createorupdateatscope)（PUT 操作）
* [标记 - 在范围内更新](https://docs.microsoft.com/rest/api/resources/tags/updateatscope)（PATCH 操作）
* [标记 - 在范围内获取](https://docs.microsoft.com/rest/api/resources/tags/getatscope)（GET 操作）
* [标记 - 在范围内删除](https://docs.microsoft.com/rest/api/resources/tags/deleteatscope)（DELETE 操作）

## <a name="inherit-tags"></a>继承标记

应用到资源组或订阅的标记不会由资源继承。 若要将订阅或资源组中的标记应用到资源，请参阅 [Azure 策略 - 标记](tag-policies.md)。

## <a name="tags-and-billing"></a>标记和计费

可使用标记对计费数据进行分组。 例如，如果针对不同组织运行多个 VM，可以使用标记根据成本中心对使用情况进行分组。 还可使用标记根据运行时环境（例如，在生产环境中运行的 VM 的计费使用情况）对成本进行分类。

可以通过使用情况逗号分隔值 (CSV) 文件检索有关标记的信息。 可以从 [Azure 帐户门户](https://account.windowsazure.cn/Subscriptions)下载使用情况文件。

<!-- Not Available [Azure Billing REST API Reference](https://msdn.microsoft.com/library/azure/1ea5b323-54bb-423d-916f-190de96c6a3c)-->
<!-- Not Available [Azure Resource Usage and RateCard APIs](../billing/billing-usage-rate-card-overview.md) -->
<!-- Not Available on portal-->

有关 REST API 操作，请参阅 [Azure 计费 REST API 参考](https://docs.microsoft.com/rest/api/billing/)。

## <a name="limitations"></a>限制

以下限制适用于标记：

* 并非所有资源类型都支持标记。 若要确定是否可以将标记应用到资源类型，请参阅 [Azure 资源的标记支持](tag-support.md)。
* 每个资源、资源组和订阅最多可以有 50 个标记名称/值对。 如果需要应用的标记超过最大允许数量，请使用 JSON 字符串作为标记值。 JSON 字符串可以包含多个应用于单个标记名称的值。 一个资源组或订阅可以包含多个资源，这些资源每个都有 50 个标记名称/值对。
* 标记名称不能超过 512 个字符，标记值不能超过 256 个字符。 对于存储帐户，标记名称不能超过 128 个字符，标记值不能超过 256 个字符。
* 不能将标记应用到云服务等经典资源。
* 标记名称不能包含以下字符：`<`、`>`、`%`、`&`、`\`、`?`、`/`

    > [!NOTE]
    > 目前，Azure DNS 区域和流量管理器服务也不允许在标记中使用空格。
    >
    > Azure Front Door 不支持在标记名称中使用 `#`。
    >
    > Azure 自动化和 Azure CDN 仅支持资源上的 15 个标记。

## <a name="next-steps"></a>后续步骤

* 并非所有资源类型都支持标记。 若要确定是否可以将标记应用到资源类型，请参阅 [Azure 资源的标记支持](tag-support.md)。

<!--Not Available on [Resource naming and tagging decision guide](/cloud-adoption-framework/decision-guides/resource-tagging/?toc=/azure-resource-manager/management/toc.json)-->

<!-- Update_Description: update meta properties, wording update, update link -->
