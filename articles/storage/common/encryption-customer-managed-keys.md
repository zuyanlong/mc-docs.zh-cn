---
title: 在 Azure Key Vault 中使用客户托管密钥管理帐户加密
titleSuffix: Azure Storage
description: 可以使用自己的加密密钥来保护存储帐户中的数据。 指定客户托管密钥时，该密钥用于保护和控制对数据加密密钥的访问。 使用客户托管密钥可以更灵活地管理访问控制。
services: storage
author: WenJason
ms.service: storage
origin.date: 07/20/2020
ms.date: 08/24/2020
ms.topic: conceptual
ms.author: v-jay
ms.reviewer: ozgun
ms.subservice: common
ms.openlocfilehash: 0a3cf7ba9b7b0783915ef1c392c1403852ea9c5c
ms.sourcegitcommit: ecd6bf9cfec695c4e8d47befade8c462b1917cf0
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/23/2020
ms.locfileid: "88753513"
---
# <a name="use-customer-managed-keys-with-azure-key-vault-to-manage-azure-storage-encryption"></a>在 Azure Key Vault 中使用客户托管密钥管理 Azure 存储加密

可以使用自己的加密密钥来保护存储帐户中的数据。 指定客户托管密钥时，该密钥用于保护和控制对数据加密密钥的访问。 使用客户托管密钥可以更灵活地管理访问控制。

必须使用 Azure Key Vault 来存储客户管理的密钥。 可以创建自己的密钥并将其存储在 Key Vault 中，或者使用 Azure Key Vault API 来生成密钥。 存储帐户和 Key Vault 必须在同一个区域和同一个 Azure Active Directory (Azure AD) 租户中，但可以在不同的订阅中。 有关 Azure Key Vault 的详细信息，请参阅[什么是 Azure Key Vault？](../../key-vault/general/overview.md)。

## <a name="about-customer-managed-keys"></a>关于客户托管密钥

下图显示了 Azure 存储如何使用 Azure Active Directory 和 Azure Key Vault 通过客户托管密钥发出请求：

![Azure 存储中客户管理的密钥的工作原理示意图](media/encryption-customer-managed-keys/encryption-customer-managed-keys-diagram.png)

以下列表解释了示意图中带编号的步骤：

1. Azure Key Vault 管理员向与存储帐户关联的托管标识授予对加密密钥的权限。
2. Azure 存储管理员使用存储帐户的客户管理密钥配置加密。
3. Azure 存储使用与存储帐户关联的托管标识，对通过 Azure Active Directory 访问 Azure Key Vault 的活动进行身份验证。
4. Azure 存储使用 Azure Key Vault 中的客户密钥包装帐户加密密钥。
5. 对于读/写操作，Azure 存储将向 Azure Key Vault 发送解包帐户加密密钥的请求，以执行加密和解密操作。

## <a name="enable-customer-managed-keys-for-a-storage-account"></a>为存储帐户启用客户管理的密钥

在配置客户托管密钥时，Azure 存储会在关联的密钥保管库中使用客户托管密钥来包装帐户的根数据加密密钥。 启用客户托管密钥不影响性能，并且会立即生效。

在启用或禁用客户托管密钥时，或者在修改密钥或密钥版本时，对根加密密钥的保护会变化，但你不需要重新加密 Azure 存储帐户中的数据。

客户托管密钥只能在现有存储帐户上启用。 必须使用访问策略对密钥保管库进行配置，这些策略将权限授予与存储帐户关联的托管标识。 托管标识仅在存储帐户创建后可用。

可随时在客户管理的密钥与 Microsoft 管理的密钥之间进行切换。 有关 Microsoft 管理的密钥的详细信息，请参阅[关于加密密钥管理](storage-service-encryption.md#about-encryption-key-management)。

要了解如何将客户管理的密钥与 Azure 密钥保管库配合使用来对 Azure 存储进行加密，请参阅以下文章之一：

- [通过 Azure 门户使用密钥保管库配置客户管理的密钥用于 Azure 存储加密](storage-encryption-keys-portal.md)
- [通过 PowerShell 使用密钥保管库配置客户管理的密钥用于 Azure 存储加密](storage-encryption-keys-powershell.md)
- [通过 Azure CLI 使用密钥保管库配置客户管理的密钥用于 Azure 存储加密](storage-encryption-keys-cli.md)

> [!IMPORTANT]
> 客户托管密钥依赖于 Azure 资源的托管标识，后者是Azure AD 的一项功能。 托管标识当前不支持跨目录方案。 在 Azure 门户中配置客户管理的密钥时，系统会在幕后自动将一个托管标识分配到你的存储帐户。 如果随后将订阅、资源组或存储帐户从一个 Azure AD 目录移到另一个目录，与存储帐户关联的托管标识不会传输到新租户，因此客户管理的密钥可能不再起作用。 有关详细信息，请参阅 [Azure 资源的常见问题解答和已知问题](../../active-directory/managed-identities-azure-resources/known-issues.md#transferring-a-subscription-between-azure-ad-directories)中的“在 Azure AD 目录之间转移订阅”。  

## <a name="store-customer-managed-keys-in-azure-key-vault"></a>将客户管理的密钥存储在 Azure 密钥保管库

若要在存储帐户上启用客户管理的密钥，必须使用 Azure 密钥保管库来存储密钥。 必须同时启用密钥保管库上的“软删除”和“不清除”属性 。

Azure 存储加密支持 2048、3072 和 4096 大小的 RSA 密钥。 有关密钥的详细信息，请参阅[关于 Azure Key Vault 密钥、机密和证书](../../key-vault/about-keys-secrets-and-certificates.md#key-vault-keys)中的“Key Vault 密钥”。

使用 Azure Key Vault 具有相关的成本。 有关详细信息，请参阅 [Key Vault 定价](https://azure.cn/pricing/details/key-vault/)。

## <a name="rotate-customer-managed-keys"></a>轮换客户管理的密钥

可以根据自己的合规性策略，在 Azure 密钥保管库中轮换客户管理的密钥。 有两个可用于轮换客户管理的密钥的选项：

- **自动轮换：** 若要配置客户管理的密钥的自动轮换，请在使用存储帐户的客户管理的密钥启用加密时省略密钥版本。 如果省略了密钥版本，Azure 存储每天都会在 Azure Key Vault 中检查是否有新版本的客户托管的密钥。 如果新的密钥版本可用，Azure 存储将自动使用最新版本的密钥。
- **手动轮换：** 若要对 Azure 存储加密使用特定的密钥版本，请在使用存储帐户的客户管理的密钥启用加密时指定该密钥版本。 如果指定密钥版本，则 Azure 存储将使用该版本进行加密，直到手动更新密钥版本。

    手动轮换密钥后，需要更新存储帐户以使用新的密钥版本 URI。 若要了解如何更新存储帐户以在 Azure 门户中使用新版本的密钥，请参阅[手动更新密钥版本](storage-encryption-keys-portal.md#manually-update-the-key-version)。

轮换客户管理的密钥不会触发存储帐户中数据的重新加密。 用户无需执行任何其他操作。

## <a name="revoke-access-to-customer-managed-keys"></a>撤消对客户管理的密钥的访问权限

可以随时撤销存储帐户对客户托管密钥的访问权限。 在撤销对客户托管密钥的访问权限之后，或者在禁用或删除密钥之后，客户端无法调用在 Blob 或其元数据中读取或写入数据的操作。 对于所有用户来说，尝试调用以下任何操作都会失败，错误代码为“403 (禁止访问)”：

- 在请求 URI 上通过 `include=metadata` 参数调用[列出 Blob](https://docs.microsoft.com/rest/api/storageservices/list-blobs)
- [获取 Blob](https://docs.microsoft.com/rest/api/storageservices/get-blob)
- [获取 Blob 属性](https://docs.microsoft.com/rest/api/storageservices/get-blob-properties)
- [获取 Blob 元数据](https://docs.microsoft.com/rest/api/storageservices/get-blob-metadata)
- [设置 Blob 元数据](https://docs.microsoft.com/rest/api/storageservices/set-blob-metadata)
- 通过 `x-ms-meta-name` 请求标头调用[快照 Blob](https://docs.microsoft.com/rest/api/storageservices/snapshot-blob)
- [复制 Blob](https://docs.microsoft.com/rest/api/storageservices/copy-blob)
- [从 URL 复制 Blob](https://docs.microsoft.com/rest/api/storageservices/copy-blob-from-url)
- [设置 Blob 层](https://docs.microsoft.com/rest/api/storageservices/set-blob-tier)
- [放置块](https://docs.microsoft.com/rest/api/storageservices/put-block)
- [从 URL 放置块](https://docs.microsoft.com/rest/api/storageservices/put-block-from-url)
- [追加块](https://docs.microsoft.com/rest/api/storageservices/append-block)
- [从 URL 追加块](https://docs.microsoft.com/rest/api/storageservices/append-block-from-url)
- [放置 Blob](https://docs.microsoft.com/rest/api/storageservices/put-blob)
- [放置页](https://docs.microsoft.com/rest/api/storageservices/put-page)
- [从 URL 放置页](https://docs.microsoft.com/rest/api/storageservices/put-page-from-url)
- [增量复制 Blob](https://docs.microsoft.com/rest/api/storageservices/incremental-copy-blob)

若要再次调用这些操作，请还原对客户托管密钥的访问权限。

此部分中未列出的所有数据操作可以在撤销客户托管密钥或者禁用或删除某个密钥后继续。

若要撤销对客户托管密钥的访问权限，请使用 [PowerShell](storage-encryption-keys-powershell.md#revoke-customer-managed-keys) 或 [Azure CLI](storage-encryption-keys-cli.md#revoke-customer-managed-keys)。

## <a name="customer-managed-keys-for-azure-managed-disks"></a>Azure 托管磁盘的客户托管密钥

客户托管密钥也可用于管理 Azure 托管磁盘的加密。 客户管理的密钥对托管磁盘的行为不同于对 Azure 存储资源的行为。 有关详细信息，请参阅适用于 Windows 的 [Azure 托管磁盘的服务器端加密](../../virtual-machines/windows/disk-encryption.md)或适用于 Linux 的 [Azure 托管磁盘的服务器端加密](../../virtual-machines/linux/disk-encryption.md)。

## <a name="next-steps"></a>后续步骤

- [通过 Azure 门户使用密钥保管库配置客户管理的密钥用于 Azure 存储加密](storage-encryption-keys-portal.md)
- [通过 PowerShell 使用密钥保管库配置客户管理的密钥用于 Azure 存储加密](storage-encryption-keys-powershell.md)
- [通过 Azure CLI 使用密钥保管库配置客户管理的密钥用于 Azure 存储加密](storage-encryption-keys-cli.md)
- [静态数据的 Azure 存储加密](storage-service-encryption.md)
