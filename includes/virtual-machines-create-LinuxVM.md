---
author: rockboyfor
ms.service: virtual-machines
ms.topic: include
origin.date: 10/26/2018
ms.date: 11/26/2018
ms.author: v-yeche
ms.openlocfilehash: ca73298024352ed6d824c2e271dfc8db70297a0e
ms.sourcegitcommit: 59db70ef3ed61538666fd1071dcf8d03864f10a9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/30/2018
ms.locfileid: "52675650"
---
1. 使用[从 Azure 经典 CLI 连接到 Azure](https://docs.azure.cn/zh-cn/cli/authenticate-azure-cli?view=azure-cli-latest) 中列出的步骤登录到 Azure 订阅。

2. 确保使用经典部署模式，如下所示：

    ```azurecli
    azure config mode asm
    ```

3. 从可用映像中找出要加载的 Linux 映像，如下所示：

   ```azurecli   
    azure vm image list | grep "Linux"
    ```

    在 Windows 命令提示符窗口中，使用 **find** 而不是 grep。

4. 使用 `azure vm create` 通过上一列表中的 Linux 映像创建 VM。 此步骤创建云服务和存储帐户。 还可通过 `-c` 选项将此 VM 连接到现有云服务。 使用 `-e` 选项创建 SSH 终结点以登录到 Linux 虚拟机。 以下示例在 `China North` 位置使用 `Ubuntu-14_04_4-LTS` 映像创建名为 `myVM` 的 VM，并添加用户名 `ops`：

    ```azurecli
    azure vm create myVM \
        b549f4301d0b4295b8e76ceb65df47d4__Ubuntu-14_04_4-LTS-amd64-server-20160516-en-us-30GB \
        -g ops -p P@ssw0rd! -z "Small" -e -l "China North"
    ```

    输出类似于以下示例：

    ```azurecli
    info:    Executing command vm create
    + Looking up image b549f4301d0b4295b8e76ceb65df47d4__Ubuntu-14_04_4-LTS-amd64-server-20160516-en-us-30GB
    + Looking up cloud service
    info:    cloud service myVM not found.
    + Creating cloud service
    + Retrieving storage accounts
    + Creating VM
    info:    vm create command OK
    ```

   > [!NOTE]
   > 对于 Linux 虚拟机，必须在 `vm create` 中提供 `-e` 选项。 在创建该虚拟机后无法启用 SSH。 有关 SSH 的详细信息，请参阅[如何在 Azure 中将 SSH 用于 Linux](../articles/virtual-machines/linux/mac-create-ssh-keys.md?toc=%2fvirtual-machines%2flinux%2ftoc.json)。

5. 可以通过使用 `azure vm show` 命令验证 VM 的属性。 以下示例列出名为 `myVM`的 VM 的信息：

    ```azurecli   
    azure vm show myVM
    ```

6. 使用 `azure vm start` 命令启动 VM，如下所示：

    ```azurecli
    azure vm start myVM
    ```

## <a name="next-steps"></a>后续步骤
有关上述所有 Azure 经典CLI 虚拟机命令的详细信息，请参阅[将 Azure 经典 CLI 与经典部署 API 配合使用](https://docs.azure.cn/zh-cn/cli/get-started-with-az-cli2?view=azure-cli-latest)。

<!-- Update_Description: update meta properties, wording update -->