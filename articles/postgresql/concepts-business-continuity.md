---
title: 业务连续性 - Azure Database for PostgreSQL（单一服务器）
description: 本文介绍使用 Azure Database for PostgreSQL 时的业务连续性（时间点还原、数据中心服务中断、异地还原、副本）。
author: WenJason
ms.author: v-jay
ms.service: postgresql
ms.topic: conceptual
origin.date: 08/07/2020
ms.date: 09/28/2020
ms.openlocfilehash: f77dad968b1b15d48ccef7ede86a8af06af21eec
ms.sourcegitcommit: ba01e2d1882c85ebeffef344ef57afaa604b53a0
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/14/2020
ms.locfileid: "92041735"
---
# <a name="overview-of-business-continuity-with-azure-database-for-postgresql---single-server"></a>有关使用 Azure Database for PostgreSQL - 单一服务器确保业务连续性的概述

本概述介绍了 Azure Database for PostgreSQL 针对业务连续性和灾难恢复所提供的功能。 了解在发生破坏性事件后用于进行恢复的选项，破坏性事件可能导致数据丢失或者数据库和应用程序无法使用。 了解对一些情况的处理方式，包括用户或应用程序错误影响数据完整性、Azure 区域发生服务中断，或者应用程序需要维护。

## <a name="features-that-you-can-use-to-provide-business-continuity"></a>可用来提供业务连续性的功能

Azure Database for PostgreSQL 提供了业务连续性功能，这包括自动备份和允许用户启动异地还原的功能。 每种功能在估计恢复时间 (ERT) 和可能丢失的数据方面都有不同的特性。 估计的恢复时间 (ERT) 是指从发出还原/故障转移请求开始，到数据库恢复完全正常运行所需的估计持续时间。 了解这些选项后，便可从中进行选择，可以针对不同方案将其搭配使用。 制定业务连续性计划时，需了解应用程序在破坏性事件发生后完全恢复前的最大可接受时间，即恢复时间目标 (RTO)。 此外，还需要了解从破坏性事件恢复时，应用程序可忍受丢失的最近数据更新（时间间隔）最大数量，即恢复点目标 (RPO)。

下表比较了各种可用功能的 ERT 和 RPO：

| **功能** | **基本** | **常规用途** | **内存优化** |
| :------------: | :-------: | :-----------------: | :------------------: |
| 从备份执行时间点还原 | 保留期内的任何还原点 | 保留期内的任何还原点 | 保留期内的任何还原点 |
| 从异地复制的备份执行异地还原 | 不支持 | ERT < 12 小时<br/>RPO < 1 小时 | ERT < 12 小时<br/>RPO < 1 小时 |

还可以考虑使用[只读副本](concepts-read-replicas.md)。

## <a name="recover-a-server-after-a-user-or-application-error"></a>在发生用户或应用程序错误之后恢复服务器

可以使用服务的备份在发生各种破坏性事件后对服务器进行恢复。 用户可能会不小心删除某些数据、无意中删除重要的表，甚至删除整个数据库。 应用程序可能因为自身缺陷，意外以错误数据覆盖正确数据，等等。

可以执行**时间点还原**来创建服务器在已知良好的时间点的副本。 此时间点必须在为服务器配置的备份保留期内。 在将数据还原到新服务器后，可以将原始服务器替换为新还原的服务器，或者将所需的数据从还原的服务器复制到原始服务器。

> [!IMPORTANT]
> 已删除的服务器**无法**还原。 如果删除服务器，则属于该服务器的所有数据库也会被删除且不可恢复。 使用 [Azure 资源锁](../azure-resource-manager/management/lock-resources.md)帮助防止意外删除服务器。

## <a name="recover-from-an-azure-data-center-outage"></a>从 Azure 数据中心服务中断进行恢复

Azure 数据中心会罕见地发生中断。 发生中断时，可能仅导致业务中断持续几分钟，也可能持续数小时。

一个选项是等待数据中心中断结束时，服务器重新联机。 这适用于可以承受服务器脱机一段时间的应用程序，例如开发环境。 当数据中心发生服务中断时，你不知道中断可能会持续多长时间，因此该选项仅在一段时间不需要服务器时才有效。

## <a name="geo-restore"></a>异地还原

异地还原功能使用异地冗余备份来还原服务器。 备份托管在服务器的[配对区域](../best-practices-availability-paired-regions.md)中。 可以使用这些备份还原到任何其他区域。 异地还原使用备份中的数据创建新的服务器。 从[备份和还原概念文章](concepts-backup.md)详细了解异地还原。

> [!IMPORTANT]
> 只有当为服务器预配了异地冗余备份存储时，异地还原才是可行的。 如果要从现有服务器的本地冗余切换到异地冗余备份，必须使用现有服务器的 pg_dump 进行转储，然后将其还原到配置了异地冗余的新建服务器中。

## <a name="cross-region-read-replicas"></a>跨区域只读副本
可以使用跨区域只读副本来增强业务连续性和灾难恢复规划。 只读副本使用 PostgreSQL 的物理复制技术进行异步更新。 从[只读副本概念文章](concepts-read-replicas.md)详细了解有关只读副本、可用区域以及如何进行故障转移的信息。 

## <a name="faq"></a>常见问题解答
### <a name="where-does-azure-database-for-postgresql-store-customer-data"></a>Azure Database for PostgreSQL 将客户数据存储在何处？
默认情况下，Azure Database for PostgreSQL 不会将客户数据移出部署的区域。 但是，客户可以选择启用[地域冗余备份](concepts-backup.md#backup-redundancy-options)或创建[跨区域读取副本](concepts-read-replicas.md#cross-region-replication)，以便在另一个区域存储数据。


## <a name="next-steps"></a>后续步骤
- 详细了解 [Azure Database for PostgreSQL 中的自动备份](concepts-backup.md)。 
- 了解如何使用 [Azure 门户](howto-restore-server-portal.md)或 [Azure CLI](howto-restore-server-cli.md) 进行还原。
- 了解 [Azure Database for PostgreSQL 中的只读副本](concepts-read-replicas.md)。