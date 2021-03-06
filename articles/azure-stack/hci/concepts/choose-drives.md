---
title: 为 Azure Stack HCI 选择驱动器
description: 如何为 Azure Stack HCI 选择驱动器。
author: WenJason
ms.author: v-jay
ms.topic: conceptual
ms.service: azure-stack
ms.subservice: azure-stack-hci
origin.date: 09/01/2020
ms.date: 10/12/2020
ms.openlocfilehash: ea9cc5d44ae80739791f9ed9b8667b4082fc7ff2
ms.sourcegitcommit: bc10b8dd34a2de4a38abc0db167664690987488d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/29/2020
ms.locfileid: "91437518"
---
# <a name="choose-drives-for-azure-stack-hci"></a>为 Azure Stack HCI 选择驱动器

> 适用于：Azure Stack HCI 版本 20H2；Windows Server 2019

本主题介绍如何根据你对 Azure Stack HCI 的性能和容量要求来选择驱动器。

## <a name="drive-types"></a>驱动器类型

Azure Stack HCI 目前使用四种类型的驱动器：

| 驱动器类型 | 描述 |
|----------------------|--------------------------|
|![PMem](media/choose-drives/pmem-100px.png)|PMem 是指永久性内存，一种新的低延迟、高性能存储类型。|
|![NVMe](media/choose-drives/NVMe-100-px.png)|**NVMe**（非易失性快速内存）是指直接位于 PCIe 总线上的固态硬盘。 常见外形规格为 2.5 英寸 U.2、PCIe 附加卡 (AIC) 和 M.2。 NVMe 提供较高的 IOPS 和 IO 吞吐量，延迟也比目前支持的除 PMem 外的任何其他驱动器类型低。|
|![SSD](media/choose-drives/SSD-100-px.png)|**SSD** 是指通过传统 SATA 或 SAS 连接的固态硬盘。|
|![HDD](media/choose-drives/HDD-100-px.png)|**HDD** 是指旋转式的磁性硬盘，可提供巨量存储容量。|

## <a name="built-in-cache"></a>内置缓存

Azure Stack HCI 提供内置的服务器端缓存。 此缓存是一个大型的持久性实时读取和写入缓存。 在包含多种类型的驱动器的部署中，该缓存自动配置为使用“最快”类型的所有驱动器。 剩余的驱动器用于提供容量。

有关详细信息，请参阅[了解 Azure Stack HCI 中的缓存](cache.md)。

## <a name="option-1--maximizing-performance"></a>选项 1 - 最大化性能

若要在随机读取和写入所有数据时实现可预测且一致的次毫秒级延迟，或要实现极高的 IOPS 或 IO 吞吐量，应改用“全闪存”。

有多种方法可以实现此目的：

![关系图显示了部署选项，包括“将所有 NVMe 用于容量”、“将 NVMe 用于缓存且将 SSD 用于容量”，以及“将所有 SSD 用于容量”。](media/choose-drives/All-Flash-Deployment-Possibilities.png)

1. **全使用 NVMe。** 全部使用 NVMe 可以提供无与伦比的性能，包括最容易预测的低延迟。 如果所有驱动器的型号相同，则不会配置缓存。 也可以混用高持久性和低持久性的 NVMe 型号，并将前者配置为针对后者中的写入内容提供缓存（[需要进行设置](https://docs.microsoft.com/windows-server/storage/storage-spaces/understand-the-cache#manual-configuration)）。

2. **NVMe + SSD。** 将 NVMe 与 SSD 配合使用时，NVMe 将自动为 SSD 中的写入内容提供缓存。 这样，就可以在缓存中合并写入内容，并仅在需要时才解除暂存写入内容，以减轻 SSD 的磨损。 这种方式提供类似于 NVMe 的写入特征，而读取内容将直接从同样快速的 SSD 提供。

3. **全使用 SSD。** 与全使用 NVMe 一样，如果所有驱动器的型号相同，则不会配置缓存。 如果混用高持久性和低持久性的型号，可将前者配置为针对后者中的写入内容提供缓存（[需要进行设置](https://docs.microsoft.com/windows-server/storage/storage-spaces/understand-the-cache#manual-configuration)）。

   >[!NOTE]
   > 全使用 NVMe 或全使用 SSD 且不配置缓存的优势是，可以从每个驱动器获取可用的存储容量。 不会有任何容量“耗费”在缓存上，如果部署规模较小，这可能颇具吸引力。

## <a name="option-2--balancing-performance-and-capacity"></a>选项 2 - 平衡性能和容量

在包含各种应用程序和工作负荷的环境中，有些环境会提出严格的性能要求，而有些环境则要求提供相当大的存储容量，在这种情况下，应“混合”使用 NVMe 或 SSD 来为较大的 HDD 提供缓存。

![关系图显示了部署可行性，包括将 NVMe 用于缓存且将 HDD 用于容量、将 SSD 用于缓存且将 HDD 用于容量，以及将 NVMe 用于缓存且将混合的 SSD 和 HDD 用于容量。](media/choose-drives/Hybrid-Deployment-Possibilities.png)

1. **NVMe + HDD**。 NVMe 驱动器通过缓存来加速读取和写入操作。 缓存读取可让 HDD 专注于写入。 缓存写入可以消减 IO 突发，使写入内容可以合并，并仅在需要时才将其解除暂存，这种人为的序列化方式可将 HDD IOPS 和 IO 吞吐量最大化。 此方式提供类似于 NVMe 的写入特征，而对于频繁读取或最近读取的数据，它还提供类似于 NVMe 的读取特征。

2. **SSD + HDD**。 与前面类似，SSD 将通过缓存读取和写入内容来加速这两种操作。 此方式提供类似于 SSD 的写入特征，并针对频繁读取或最近读取的数据提供类似于 SSD 的读取特征。

    还有一个额外的特殊选项：使用上述所有三种类型的驱动器。

3. **NVMe + SSD + HDD。** 使用所有三种类型的驱动器时，NVMe 将同时为 SSD 和 HDD 提供缓存。 此选项的吸引力在于，可以在同一群集中的 SSD 和 HDD 上创建并列的卷，而这些卷全部可以通过 NVMe 来加速。 前者完全如同前面所述的“全闪存”部署，后者完全如同“混合”部署。 这在概念上类似于有两个池，并且容量管理、故障和修复周期等等基本上都是独立的。

   >[!IMPORTANT]
   > 建议使用 SSD 层，以将对性能最敏感的工作负荷放在全闪存驱动器上。

## <a name="option-3--maximizing-capacity"></a>选项 3 - 最大化容量

对于需要巨量容量且不频繁写入的工作负荷（例如存档、备份目标、数据仓库或“冷”存储），应该使用几个 SSD 来提供缓存，同时结合多个较大 HDD 来提供容量。

![最大化容量的部署选项](media/choose-drives/maximizing-capacity.png)

1. **SSD + HDD**。 SSD 将缓存读取和写入内容以消减 IO 突发，并提供类似于 SSD 的写入性能，以后还可以通过优化的方式将这些内容解除暂存到 HDD。

>[!IMPORTANT]
>仅包含 HDD 的配置不受支持。 不建议使用高持久性的 SSD 来为低持久性的 SSD 提供缓存。

## <a name="sizing-considerations"></a>大小调整注意事项

### <a name="cache"></a>缓存

每台服务器必须至少有两个缓存驱动器（实现冗余的最低要求）。 容量驱动器的数目最好是缓存驱动器数目的倍数。 例如，如果你有 4 个缓存驱动器，则配置 8 个容量驱动器（1:2 比例）的性能比配置 7 个或 9 个容量驱动器更稳定。

应该根据应用程序和工作负荷的工作集（即，在任何给定时间主动读取或写入的数据）来调整缓存的大小。 除此之外，没有其他缓存大小要求。 对于使用 HDD 的部署，合理的起点是容量的 10% – 例如，如果每台服务器有 4 x 4 TB HDD = 16 TB 容量，则每台服务器的缓存为 2 x 800 GB SSD = 1.6 TB。 对于全闪存部署，尤其是使用[持久性极高](https://techcommunity.microsoft.com/t5/storage-at-microsoft/understanding-ssd-endurance-drive-writes-per-day-dwpd-terabytes/ba-p/426024)的 SSD 时，接近 5% 的容量可能是合理的起点 – 例如，如果每台服务器有 24 x 1.2 TB SSD = 28.8 TB 容量，则每台服务器的缓存为 2 x 750 GB NVMe = 1.5 TB。 以后随时可以通过添加或移除缓存驱动器进行调整。

### <a name="general"></a>常规

建议将每台服务器的总存储容量限制为大约 400 TB。 每台服务器的存储容量越高，则停机或重新启动后（例如应用软件更新时），重新同步数据所需的时间就越长。 对于 Windows Server 2019，每个存储池的大小上限目前为 4 PB (4,000 TB)。

## <a name="next-steps"></a>后续步骤

有关详细信息，请参阅：

- [了解缓存](cache.md)
- [确定硬件要求](../deploy/before-you-start.md#determine-hardware-requirements)
- [驱动器对称注意事项](drive-symmetry-considerations.md)
- [规划卷](plan-volumes.md)
- [容错和存储效率](fault-tolerance.md)
- [了解和部署永久性内存](https://docs.microsoft.com/windows-server/storage/storage-spaces/deploy-pmem)