---
title: PowerShell 和 REST API
description: 了解 Azure Synapse Analytics SQL 池的最常用 PowerShell cmdlet，包括如何暂停和恢复数据库。
services: synapse-analytics
author: WenJason
manager: digimobile
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: ''
origin.date: 04/17/2018
ms.date: 05/11/2020
ms.author: v-jay
ms.reviewer: igorstan
ms.custom: seo-lt-2019
ms.openlocfilehash: 101d1e49a67c43c31957d0400390041eac25740e
ms.sourcegitcommit: f8d6fa25642171d406a1a6ad6e72159810187933
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2020
ms.locfileid: "82198574"
---
# <a name="powershell--rest-apis-for-azure-synapse-analytics-sql-pool"></a>用于 Azure Synapse Analytics SQL 池的 PowerShell 和 REST API

许多 Azure Synapse Analytics SQL 池管理任务可以使用 Azure PowerShell cmdlet 或 REST API 进行管理。  下面是如何使用 PowerShell 命令自动执行 SQL 池中的常见任务的一些示例。  如需一些好的 REST 示例，请参阅[使用 REST 管理可伸缩性](sql-data-warehouse-manage-compute-rest-api.md)一文。

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="get-started-with-azure-powershell-cmdlets"></a>Azure PowerShell cmdlet 入门

1. 打开 Windows PowerShell。
2. 在 PowerShell 提示符下，运行以下命令以登录到 Azure Resource Manager，并选择用户的订阅。

    ```powershell
    Connect-AzAccount -Environment AzureChinaCloud
    Get-AzSubscription
    Select-AzSubscription -SubscriptionName "MySubscription"
    ```

## <a name="pause-data-warehouse-example"></a>暂停数据仓库示例

暂停名为“Server01”的服务器上托管的名为“Database02”的数据库。  该服务器位于名为“ResourceGroup1”的 Azure 资源组中。

```Powershell
Suspend-AzSqlDatabase -ResourceGroupName "ResourceGroup1" -ServerName "Server01" -DatabaseName "Database02"
```

作为一种变体，此示例通过管道将检索到的对象传递给 [Suspend-AzSqlDatabase](https://docs.microsoft.com/powershell/module/az.sql/suspend-azsqldatabase?toc=/synapse-analytics/sql-data-warehouse/toc.json&bc=/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json)。  因此会暂停该数据库。 最后一个命令显示结果。

```Powershell
$database = Get-AzSqlDatabase -ResourceGroupName "ResourceGroup1" -ServerName "Server01" -DatabaseName "Database02"
$resultDatabase = $database | Suspend-AzSqlDatabase
$resultDatabase
```

## <a name="start-data-warehouse-example"></a>启动数据仓库示例

恢复“Server01”的服务器上托管的“Database02”数据库的运行。 该服务器包含在名为“ResourceGroup1”的资源组中。

```Powershell
Resume-AzSqlDatabase -ResourceGroupName "ResourceGroup1" -ServerName "Server01" -DatabaseName "Database02"
```

作为一种变体，此示例可从“ResourceGroup1”资源组包含的“Server01”服务器中检索“Database02”数据库。 它通过管道将检索到的对象传递给 [Resume-AzSqlDatabase](https://docs.microsoft.com/powershell/module/az.sql/resume-azsqldatabase?toc=/synapse-analytics/sql-data-warehouse/toc.json&bc=/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json)。

```Powershell
$database = Get-AzSqlDatabase -ResourceGroupName "ResourceGroup1" -ServerName "Server01" -DatabaseName "Database02"
$resultDatabase = $database | Resume-AzSqlDatabase
```

> [!NOTE]
> 注意，如果服务器是 foo.database.chinacloudapi.cn，请使用“foo”作为 PowerShell cmdlet 中的 -ServerName。

## <a name="other-supported-powershell-cmdlets"></a>其他支持的 PowerShell cmdlet

Azure Synapse Analytics 数据仓库支持这些 PowerShell cmdlet。

* [Get-AzSqlDatabase](https://docs.microsoft.com/powershell/module/az.sql/get-azsqldatabase?toc=/synapse-analytics/sql-data-warehouse/toc.json&bc=/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json)
* [Get-AzSqlDeletedDatabaseBackup](https://docs.microsoft.com/powershell/module/az.sql/get-azsqldeleteddatabasebackup?toc=/synapse-analytics/sql-data-warehouse/toc.json&bc=/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json)
* [Get-AzSqlDatabaseRestorePoint](https://docs.microsoft.com/powershell/module/az.sql/get-azsqldatabaserestorepoint?toc=/synapse-analytics/sql-data-warehouse/toc.json&bc=/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json)
* [New-AzSqlDatabase](https://docs.microsoft.com/powershell/module/az.sql/new-azsqldatabase?toc=/synapse-analytics/sql-data-warehouse/toc.json&bc=/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json)
* [Remove-AzSqlDatabase](https://docs.microsoft.com/powershell/module/az.sql/remove-azsqldatabase?toc=/synapse-analytics/sql-data-warehouse/toc.json&bc=/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json)
* [Restore-AzSqlDatabase](https://docs.microsoft.com/powershell/module/az.sql/restore-azsqldatabase?toc=/synapse-analytics/sql-data-warehouse/toc.json&bc=/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json)
* [Resume-AzSqlDatabase](https://docs.microsoft.com/powershell/module/az.sql/resume-azsqldatabase?toc=/synapse-analytics/sql-data-warehouse/toc.json&bc=/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json)
* [Set-AzSqlDatabase](https://docs.microsoft.com/powershell/module/az.sql/set-azsqldatabase?toc=/synapse-analytics/sql-data-warehouse/toc.json&bc=/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json)
* [Suspend-AzSqlDatabase](https://docs.microsoft.com/powershell/module/az.sql/suspend-azsqldatabase?toc=/synapse-analytics/sql-data-warehouse/toc.json&bc=/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json)

## <a name="next-steps"></a>后续步骤

有关更多的 PowerShell 示例，请参阅：

* [使用 PowerShell 创建数据仓库](create-data-warehouse-powershell.md)
* [数据库还原](sql-data-warehouse-restore-points.md)

若要了解其他可使用 PowerShell 自动执行的任务，请参阅 [Azure SQL 数据库 cmdlet](https://docs.microsoft.com/powershell/module/az.sql?toc=/synapse-analytics/sql-data-warehouse/toc.json&bc=/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json)。 Azure Synapse Analytics 数据仓库并非支持所有 Azure SQL 数据库 cmdlet。 有关可以使用 REST 自动执行的任务列表，请参阅 [Azure SQL 数据库的操作](https://docs.microsoft.com/rest/api/sql/?toc=/synapse-analytics/sql-data-warehouse/toc.json&bc=/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json)。
