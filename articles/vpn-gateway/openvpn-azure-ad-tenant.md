---
title: VPN 网关：用于 P2S VPN 连接的 Azure AD 租户：Azure AD 身份验证
description: 可以使用 P2S VPN 通过 Azure AD 身份验证连接到 VNet
services: vpn-gateway
author: WenJason
ms.service: vpn-gateway
ms.topic: how-to
origin.date: 09/03/2020
ms.date: 09/28/2020
ms.author: v-jay
ms.openlocfilehash: bcc367aad83186318617b993bf820ffbd718a236
ms.sourcegitcommit: 71953ae66ddfc07c5d3b4eb55ff8639281f39b40
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/27/2020
ms.locfileid: "91395429"
---
# <a name="create-an-azure-active-directory-tenant-for-p2s-openvpn-protocol-connections"></a>为 P2S OpenVPN 协议连接创建 Azure Active Directory 租户

连接到 VNet 时，可以使用基于证书的身份验证或 RADIUS 身份验证。 但是，在使用开放 VPN 协议时，还可以使用 Azure Active Directory 身份验证。 本文帮助设置用于 P2S Open VPN 身份验证的 Azure AD 租户。

> [!NOTE]
> Azure AD 身份验证仅支持 OpenVPN® 协议连接，并且需要 Azure VPN 客户端，后者仅适用于 Windows 10。
>


## <a name="1-verify-azure-ad-tenant"></a><a name="tenant"></a>1.验证 Azure AD 租户

验证你是否有 Azure AD 租户。 如果没有 Azure AD 租户，可以按照[创建新租户](../active-directory/fundamentals/active-directory-access-create-new-tenant.md)一文中的步骤创建一个：

* 组织名称
* 初始域名

示例：

   ![新 Azure AD 租户](./media/openvpn-create-azure-ad-tenant/newtenant.png)

## <a name="2-create-azure-ad-tenant-users"></a><a name="users"></a>2.创建 Azure AD 租户用户

Azure AD 租户需要以下帐户：全局管理员帐户和主用户帐户。 主要用户帐户用作主要嵌入帐户（服务帐户）。 创建 Azure AD 租户用户帐户时，可以根据要创建的用户类型调整目录角色。

使用[此文](../active-directory/fundamentals/add-users-azure-active-directory.md)中的步骤为 Azure AD 租户创建至少两个用户。 若要创建帐户类型，请务必更改“目录角色”：

* 全局管理员
* User

## <a name="3-enable-azure-ad-authentication-on-the-vpn-gateway"></a><a name="enable-authentication"></a>3.在 VPN 网关上启用 Azure AD 身份验证

1. 找到要用于身份验证的目录的目录 ID。 此 ID 在“Active Directory”页的“属性”部分中列出。

    ![Directory ID](./media/openvpn-create-azure-ad-tenant/directory-id.png)

2. 复制“目录 ID”。

3. 以拥有“全局管理员”角色的用户身份登录到 Azure 门户。

4. 接下来，做出管理员许可。 在浏览器的地址栏中复制并粘贴与部署位置相关的 URL：

    Azure 中国世纪互联

    ```
    https://login.chinacloudapi.cn/common/oauth2/authorize?client_id=49f817b6-84ae-4cc0-928c-73f27289b3aa&response_type=code&redirect_uri=https://portal.azure.cn&nonce=1234&prompt=admin_consent
    ```

5. 如果出现提示，请选择“全局管理员”帐户。

    ![Directory ID](./media/openvpn-create-azure-ad-tenant/pick.png)

6. 出现提示时选择“接受”。

    ![Accept](./media/openvpn-create-azure-ad-tenant/accept.jpg)

7. 在 Azure AD 下的“企业应用程序”中，将会发现已列出“Azure VPN”。

    ![Azure VPN](./media/openvpn-create-azure-ad-tenant/azurevpn.png)
    
8. 如果还没有正常运行的“点到站点”环境，请按照说明创建一个。 请参阅[创建点到站点 VPN](vpn-gateway-howto-point-to-site-resource-manager-portal.md)，以创建和配置点到站点 VPN 网关。 

    > [!IMPORTANT]
    > OpenVPN 不支持基本 SKU。

9. 通过导航到“点到站点配置”并选取“OpenVPN (SSL)”作为“隧道类型”，在 VPN 网关上启用 Azure AD 身份验证。 选择“Azure Active Directory”作为“身份验证类型”，然后在“Azure Active Directory”部分填写信息。

    ![Azure VPN](./media/openvpn-create-azure-ad-tenant/azure-ad-auth-portal.png)


   > [!NOTE]
   > 请确保在 `AadIssuerUri` 值的末尾包含尾随斜杠。 否则，连接可能会失败。

10. 通过单击“下载 VPN 客户端”链接来创建和下载配置文件。

11. 解压缩已下载的 zip 文件。

12. 浏览到解压缩后的“AzureVPN”文件夹。

13. 记下“azurevpnconfig.xml”文件的位置。 azurevpnconfig.xml 包含 VPN 连接的设置，可以直接导入到 Azure VPN 客户端应用程序中。 还可以将此文件分发给需要通过电子邮件或其他方式建立连接的所有用户。 用户需有有效的 Azure AD 凭据才能成功建立连接。

## <a name="next-steps"></a>后续步骤

若要连接到虚拟网络，必须创建并配置 VPN 客户端配置文件。 请参阅[配置 VPN 客户端以建立 P2S VPN 连接](openvpn-azure-ad-client.md)。
