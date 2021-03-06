---
title: 使用 Azure 门户监视托管应用
description: 显示如何使用 Azure 门户监视托管应用程序的可用性和警报。
author: rockboyfor
ms.topic: conceptual
origin.date: 10/04/2018
ms.date: 01/20/2020
ms.author: v-yeche
ms.openlocfilehash: 76f26cee1fb4207ce4064922978c7abe151d820a
ms.sourcegitcommit: c1ba5a62f30ac0a3acb337fb77431de6493e6096
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/17/2020
ms.locfileid: "76170771"
---
# <a name="monitor-a-deployed-instance-of-a-managed-application"></a>监视部署的托管应用程序实例

将托管应用程序部署到 Azure 订阅后，可能需要检查应用程序的状态。 本文介绍 Azure 门户中用于检查状态的选项。 在托管应用程序中，可以监视资源的可用性。 还可以设置和查看警报。

## <a name="view-resource-health"></a>查看资源运行状况

1. 选择托管应用程序实例。

    ![创建托管应用程序](./media/monitor-managed-application-portal/select-managed-application.png)

1. 选择“资源运行状况”  。

    ![选择资源运行状况](./media/monitor-managed-application-portal/select-resource-health.png)

1. 在托管应用程序中查看资源的可用性。

    ![查看资源运行状况](./media/monitor-managed-application-portal/view-health.png)

## <a name="view-alerts"></a>查看警报

1. 选择“**警报**”。

    ![选择警报](./media/monitor-managed-application-portal/select-alerts.png)

1. 如果已配置警报规则，则会看到有关已引发警报的信息。

    ![查看警报](./media/monitor-managed-application-portal/view-alerts.png)

1. 若要添加警报规则，请选择“+ 新建警报规则”  。

    ![创建警报](./media/monitor-managed-application-portal/create-new-alert.png)

可以针对托管应用程序实例或托管应用程序中的资源创建警报。 有关创建警报的信息，请参阅 [Azure 中的警报概述](../../azure-monitor/platform/alerts-overview.md)。

## <a name="next-steps"></a>后续步骤

* 有关托管应用程序示例，请参阅 [Azure 托管应用程序的示例项目](sample-projects.md)。
* 若要部署托管应用程序，请参阅[通过 Azure 门户部署服务目录应用](deploy-service-catalog-quickstart.md)。

<!-- Update_Description: new article about monitor managed application portal -->
<!--NEW.date: 01/20/2020-->