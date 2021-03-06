---
title: 缩放弹性池资源
description: 本页描述如何在 Azure SQL 数据库中缩放弹性池资源。
services: sql-database
ms.service: sql-database
ms.subservice: elastic-pools
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: WenJason
ms.author: v-jay
ms.reviewer: sstein
origin.date: 09/16/2019
ms.date: 10/12/2020
ms.openlocfilehash: ae984dd6134deff0926be0f228262f06a8ef72c8
ms.sourcegitcommit: 1810e40ba56bed24868e573180ae62b9b1e66305
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/09/2020
ms.locfileid: "91872445"
---
# <a name="scale-elastic-pool-resources-in-azure-sql-database"></a>在 Azure SQL 数据库中缩放弹性池资源
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

本文介绍如何在 Azure SQL 数据库中缩放适用于弹性池和共用数据库的计算和存储资源。

## <a name="change-compute-resources-vcores-or-dtus"></a>更改计算资源（vCore 或 DTU）

最初选择 vCore 或 eDTU 数量后，可以使用 [Azure 门户](elastic-pool-manage.md#azure-portal)、[PowerShell](https://docs.microsoft.com/powershell/module/az.sql/Get-AzSqlElasticPool)、[Azure CLI](/cli/sql/elastic-pool#az-sql-elastic-pool-update) 或 [REST API](https://docs.microsoft.com/rest/api/sql/elasticpools/update)，根据实际体验动态扩展或缩减弹性池。

### <a name="impact-of-changing-service-tier-or-rescaling-compute-size"></a>更改服务层级或重新缩放计算大小的影响

更改弹性池的服务层级或计算大小遵循适用于单一数据库的类似模式，该过程主要涉及到由服务执行的以下步骤：

1. 为弹性池创建新的计算实例  

    使用请求的服务层级和计算大小为弹性池创建新的计算实例。 更改后，对于服务层级和计算大小的某些组合，必须在新的计算实例中创建每个数据库的副本，此过程涉及到数据复制，可能会对总体延迟造成很大的影响。 无论如何，在执行此步骤期间，数据库会保持联机，并且连接会继续定向到原始计算实例中的数据库。

2. 将连接路由切换到新的计算实例

    将删除与原始计算实例中的数据库建立的现有连接。 将与新计算实例中的数据库建立任何新的连接。 更改后，对于服务层级和计算大小的某些组合，在切换期间会分离再重新附加数据库文件。  无论如何，切换操作都可能会导致服务出现短暂的中断，此时，数据库一般会出现 30 秒以下的不可用情况（通常只有几秒钟）。 如果连接断开时有长时间运行的事务正在运行，则此步骤的持续时间可能会变长，以便恢复中止的事务。 [加速的数据库恢复](../accelerated-database-recovery.md)可以降低中止长期运行的事务的影响。

> [!IMPORTANT]
> 执行工作流中的任何步骤期间都不会丢失数据。

### <a name="latency-of-changing-service-tier-or-rescaling-compute-size"></a>更改服务层级或重新缩放计算大小所造成的延迟

更改服务层、缩放单个数据库或弹性池的计算大小、将数据库移入/移出弹性池或在弹性池之间移动数据库的预计延迟的参数设置如下：

|服务层|基本单一数据库，</br>标准 (S0-S1)|基本弹性池，</br>标准 (S2-S12)， </br>常规用途单一数据库或弹性池|高级或业务关键型单一数据库或弹性池|超大规模
|:---|:---|:---|:---|:---|
|**基本单一数据库，</br>标准 (S0-S1)**|&bull; &nbsp;延迟时间较为恒定，与已用空间无关</br>&bull; &nbsp;通常小于 5 分钟|&bull; &nbsp;由于数据复制，延迟与已用数据库空间成比例</br>&bull; &nbsp;对于每 GB 的已用空间，延迟通常小于 1 分钟|&bull; &nbsp;由于数据复制，延迟与已用数据库空间成比例</br>&bull; &nbsp;对于每 GB 的已用空间，延迟通常小于 1 分钟|&bull; &nbsp;由于数据复制，延迟与已用数据库空间成比例</br>&bull; &nbsp;对于每 GB 的已用空间，延迟通常小于 1 分钟|
|**基本弹性池，</br>标准 (S2-S12)，</br>常规用途单一数据库或弹性池**|&bull; &nbsp;由于数据复制，延迟与已用数据库空间成比例</br>&bull; &nbsp;对于每 GB 的已用空间，延迟通常小于 1 分钟|&bull; &nbsp;对于单一数据库，延迟时间是恒定的，与已用空间无关</br>&bull; &nbsp;对于单一数据库，通常不超过 5 分钟</br>&bull; &nbsp;对于弹性池，与数据库数量成正比|&bull; &nbsp;由于数据复制，延迟与已用数据库空间成比例</br>&bull; &nbsp;对于每 GB 的已用空间，延迟通常小于 1 分钟|&bull; &nbsp;由于数据复制，延迟与已用数据库空间成比例</br>&bull; &nbsp;对于每 GB 的已用空间，延迟通常小于 1 分钟|
|**高级或业务关键型单一数据库或弹性池**|&bull; &nbsp;由于数据复制，延迟与已用数据库空间成比例</br>&bull; &nbsp;对于每 GB 的已用空间，延迟通常小于 1 分钟|&bull; &nbsp;由于数据复制，延迟与已用数据库空间成比例</br>&bull; &nbsp;对于每 GB 的已用空间，延迟通常小于 1 分钟|&bull; &nbsp;由于数据复制，延迟与已用数据库空间成比例</br>&bull; &nbsp;对于每 GB 的已用空间，延迟通常小于 1 分钟|&bull; &nbsp;由于数据复制，延迟与已用数据库空间成比例</br>&bull; &nbsp;对于每 GB 的已用空间，延迟通常小于 1 分钟|
|**超大规模**|空值|空值|空值|&bull; &nbsp;延迟时间较为恒定，与已用空间无关</br>&bull; &nbsp;通常小于 2 分钟|

> [!NOTE]
>
> - 如果更改服务层或者重新缩放弹性池的计算大小，则应使用池中所有数据库的已用空间之和来计算估计值。
> - 如果向/从弹性池移动数据库，则只有数据库使用的空间会影响延迟，弹性池使用的空间不会影响延迟。
> - 对于标准和常规用途弹性池，如果弹性池使用高级文件共享 ([PFS](/storage/files/storage-files-introduction)) 存储，则将数据库移入/移出弹性池或在弹性池之间移动数据库的延迟与数据库大小成正比。 若要确定池是否正在使用 PFS 存储，请在任何池数据库上下文中执行以下查询。 如果 AccountType 列中的值为 `PremiumFileStorage`，则该池使用的是 PFS 存储。

```sql
SELECT s.file_id,
       s.type_desc,
       s.name,
       FILEPROPERTYEX(s.name, 'AccountType') AS AccountType
FROM sys.database_files AS s
WHERE s.type_desc IN ('ROWS', 'LOG');
```

> [!TIP]
> 若要监视正在进行的操作，请参阅：[使用 SQL REST API 管理操作](https://docs.microsoft.com/rest/api/sql/operations/list)、[使用 CLI 管理操作](/cli/sql/db/op)、[使用 T-SQL 监视操作](https://docs.microsoft.com/sql/relational-databases/system-dynamic-management-views/sys-dm-operation-status-azure-sql-database)及以下两个 PowerShell 命令：[Get-AzSqlDatabaseActivity](https://docs.microsoft.com/powershell/module/az.sql/get-azsqldatabaseactivity) 和 [Stop-AzSqlDatabaseActivity](https://docs.microsoft.com/powershell/module/az.sql/stop-azsqldatabaseactivity)。

### <a name="additional-considerations-when-changing-service-tier-or-rescaling-compute-size"></a>更改服务层级或重新缩放计算大小时的其他注意事项

- 减少弹性池的 vCore 或 eDTU 时，该池使用的空间必须小于目标服务层和池 eDTU 所允许的最大大小。
- 当重新缩放弹性池的 eDTU 时，如果 (1) 目标池支持池的存储上限，(2) 存储上限超过了目标池附送的存储量，将产生额外存储费用。 例如，如果最大大小为 100 GB 的 100 eDTU 标准池缩小为 50 eDTU 标准池，那么将产生额外存储费用，因为目标池支持的最大大小为 100 GB，其附送的存储量仅为 50 GB。 因此，额外的存储量为 100 GB - 50 GB = 50 GB。 有关额外存储定价的信息，请参阅 [SQL 数据库定价](https://azure.cn/pricing/details/sql-database/)。 如果实际使用的空间量小于附送的存储量，只要将数据库最大大小减少到附送的量，就能避免此项额外费用。

### <a name="billing-during-rescaling"></a>重新缩放期间的计费

将根据使用最高服务层级的数据库存在的每个小时 + 在该小时适用的计算大小进行计费，无论使用方式或数据库处于活动状态是否少于一小时。 例如，如果创建了单一数据库，并在五分钟后将其删除，则将按该数据库存在一小时收费。

## <a name="change-elastic-pool-storage-size"></a>更改弹性池存储大小

> [!IMPORTANT]
> 在某些情况下，可能需要收缩数据库来回收未使用的空间。 有关详细信息，请参阅[管理 Azure SQL 数据库中的文件空间](file-space-manage.md)。

### <a name="vcore-based-purchasing-model"></a>基于 vCore 的购买模型

- 可将存储预配到最大大小限制。

  - 对于“标准”或“常规用途”服务层级中的存储，按 10 GB 增量增减大小
  - 对于“高级”或“业务关键”服务层级中的存储，按 250 GB 增量增减大小
- 可以通过增大或减小其最大大小来预配弹性池的存储空间。
- 弹性池的存储价格等于存储量乘以服务层级的存储单价。 有关额外存储价格的详细信息，请参阅 [SQL 数据库定价](https://azure.cn/pricing/details/sql-database/)。

> [!IMPORTANT]
> 在某些情况下，可能需要收缩数据库来回收未使用的空间。 有关详细信息，请参阅[管理 Azure SQL 数据库中的文件空间](file-space-manage.md)。

### <a name="dtu-based-purchasing-model"></a>基于 DTU 的购买模型

- 弹性池的 eDTU 价格附送了一定容量的存储，无需额外费用。 超出附送的量后，可花费额外的费用预配额外的存储，但不能超过存储上限，不超过 1 TB 时，以 250 GB 为增量进行预配，超出 1 TB 时，以 256 GB 为增量进行预配。 有关附送存储量和大小上限，请参阅[弹性池：存储大小和计算大小](resource-limits-dtu-elastic-pools.md#elastic-pool-storage-sizes-and-compute-sizes)。
- 可通过 [Azure 门户](elastic-pool-manage.md#azure-portal)、[PowerShell](https://docs.microsoft.com/powershell/module/az.sql/Get-AzSqlElasticPool)、[Azure CLI](/cli/sql/elastic-pool#az-sql-elastic-pool-update) 或 [REST API](https://docs.microsoft.com/rest/api/sql/elasticpools/update) 为弹性池增加大小上限，以预配额外存储。
- 弹性池的额外存储价格等于额外存储量乘以服务层级的额外存储单价。 有关额外存储价格的详细信息，请参阅 [SQL 数据库定价](https://azure.cn/pricing/details/sql-database/)。

> [!IMPORTANT]
> 在某些情况下，可能需要收缩数据库来回收未使用的空间。 有关详细信息，请参阅[管理 Azure SQL 数据库中的文件空间](file-space-manage.md)。

## <a name="next-steps"></a>后续步骤

有关总体资源限制，请参阅 [SQL 数据库基于 vCore 的资源限制 - 弹性池](resource-limits-vcore-elastic-pools.md)和 [SQL 数据库基于 DTU 的资源限制 - 弹性池](resource-limits-dtu-elastic-pools.md)。
