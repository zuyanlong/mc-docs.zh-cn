---
title: 使用 Windows PowerShell 在 Azure Stack HCI 上创建 Kubernetes 群集的快速入门
description: 了解如何使用 Windows PowerShell 在 Azure Stack HCI 上创建 Kubernetes 群集
author: WenJason
ms.topic: quickstart
ms.service: azure-stack
origin.date: 09/22/2020
ms.date: 10/12/2020
ms.author: v-jay
ms.openlocfilehash: 8f866d9ce721c645bff68cbd2d6e288b14d7f739
ms.sourcegitcommit: bc10b8dd34a2de4a38abc0db167664690987488d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/29/2020
ms.locfileid: "91451227"
---
# <a name="quickstart-create-kubernetes-clusters-on-azure-stack-hci-using-windows-powershell"></a>快速入门：使用 Windows PowerShell 在 Azure Stack HCI 上创建 Kubernetes 群集

> 适用于：Azure Stack HCI

本快速入门介绍如何使用 Windows PowerShell 在 Azure Stack HCI 上创建 Kubernetes 群集。 若要改为使用 Windows Admin Center，请参阅[使用 Windows Admin Center 设置 Azure Stack HCI 上的 Azure Kubernetes 服务](setup.md)。

## <a name="before-you-begin"></a>在开始之前

在开始之前，请确保：

- 具有一个 2-4 节点的 Azure Stack HCI 群集或单节点 Azure Stack HCI。 建议使用 2-4 节点的 Azure Stack HCI 群集。 如果没有，请按照[此处](./system-requirements.md)有关如何创建的说明进行操作。
- 设置了 Azure Stack Kubernetes 主机。 如果没有，请按照[此处](./setup-powershell.md)有关如何设置的说明进行操作。

## <a name="step-1-create-a-kubernetes-cluster"></a>步骤 1：创建 Kubernetes 群集

安装 Azure Kubernetes 服务主机之后，便准备好部署 Kubernetes 群集。

以管理员身份打开 PowerShell 并运行以下命令。

   ```powershell
   New-AksHciCluster -clusterName
                    [-kubernetesVersion]
                    [-controlPlaneNodeCount]
                    [-linuxNodeCount]
                    [-windowsNodeCount]
                    [-controlPlaneVmSize]
                    [-loadBalancerVmSize]
                    [-linuxNodeVmSize]
                    [-windowsNodeVmSize]
   ```

### <a name="required-parameters"></a>必需的参数

`-clusterName`

Kubernetes 群集的字母数字名称。

### <a name="optional-parameters"></a>可选参数

`-kubernetesVersion`

要部署的 Kubernetes 的版本。 默认值为 v1.18.6。 若要获取可用版本的列表，请运行 `Get-AksHciKubernetesVersion`。

`-controlPlaneNodeCount`

控制平面中的节点数。 默认值为 1。

`-linuxNodeCount`

Kubernetes 群集中的 Linux 节点数。 默认值为 1。

`-windowsNodeCount`

Kubernetes 群集中的 Windows 节点数。 默认为 0。

`-controlPlaneVmSize`

控制平面 VM 的大小。 默认值为 Standard_A2_v2。 若要获取可用 VM 大小的列表，请运行 `Get-AksHciVmSize`。

`-loadBalancerVmSize`

负载均衡器 VM 的大小。 默认值为 Standard_A2_V2。 若要获取可用 VM 大小的列表，请运行 `Get-AksHciVmSize`。

`-linuxNodeVmSize`

Linux 节点 VM 的大小。 默认值为 Standard_K8S3_v1。 若要获取可用 VM 大小的列表，请运行 `Get-AksHciVmSize`。

`-windowsNodeVmSize`

Windows 节点 VM 的大小。 默认值为 Standard_K8S3_v1。 若要获取可用 VM 大小的列表，请运行 `Get-AksHciVmSize`。

### <a name="check-your-deployed-clusters"></a>检查部署的群集

若要获取已部署 Azure Kubernetes 服务主机和 Kubernetes 群集的列表，请运行以下命令。

```powershell
Get-AksHciCluster
```

## <a name="step-2-scale-a-kubernetes-cluster"></a>步骤 2：缩放 Kubernetes 群集

如果需要纵向扩展或缩减群集，可以更改控制平面节点、Linux 工作器节点或 Windows 工作器节点的数量。

若要缩放控制平面节点，请运行以下命令。

```powershell
Set-AksHciClusterNodeCount -clusterName
                           -controlPlaneNodeCount
```

若要缩放工作器节点，请运行以下命令。

```powershell
Set-AksHciClusterNodeCount -clusterName
                           -linuxNodeCount
                           -windowsNodeCount
```

控制平面节点和工作器节点必须单独缩放。

## <a name="step-3-upgrade-kubernetes-version"></a>步骤 3：升级 Kubernetes 版本

若要查看正在运行的当前 Kubernetes 版本，请运行以下命令。

```powershell
Get-AksHciKubernetesVersion
```

若要升级到下一个 Kubernetes 版本，请运行以下命令。

```powershell
Update-AksHciCluster -clusterName
```

如果要使用 Windows 节点，所需的最低版本为 v1.1.8.6。

## <a name="step-4-access-your-clusters-using-kubectl"></a>步骤 4：使用 kubectl 访问群集

若要使用 kubectl 访问 Azure Kubernetes 服务主机或 Kubernetes 群集，请运行以下命令。 这将使用指定群集的 kubeconfig 文件作为 kubectl 的默认 kubeconfig 文件。

```powershell
Set-AksHciKubeConfig -clusterName
```

## <a name="delete-a-kubernetes-cluster"></a>删除 Kubernetes 群集

如果需要删除 Kubernetes 群集，请运行以下命令。

```powershell
Remove-AksHciCluster -clusterName
```

## <a name="get-logs"></a>获取日志

若要从所有 Pod 获取日志，请运行以下命令。 此命令将在路径 `C:\wssd\akshcilogs` 中创建名为 `akshcilogs` 的输出压缩文件夹。

```powershell
Get-AksHciLogs
```

本快速入门介绍了如何使用 PowerShell 创建、缩放和升级 Kubernetes 群集。

## <a name="next-steps"></a>后续步骤

- [在 Kubernetes 群集上部署 Linux 应用程序](./deploy-linux-application.md)。
- [在 Kubernetes 群集上部署 Windows 应用程序](./deploy-windows-application.md)。
