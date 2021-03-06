---
title: 快速入门：在 Azure Active Directory (Azure AD) 租户中配置应用程序的属性
description: 本快速入门使用 Azure 门户配置已注册到 Azure Active Directory (Azure AD) 租户的应用程序。
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.topic: quickstart
ms.workload: identity
ms.date: 10/12/2020
ms.author: v-junlch
ms.openlocfilehash: ec883cd7d97b703c2f9fa9a06e458cbf8105b3cb
ms.sourcegitcommit: 4d06a5e0f48472f5eadd731e43afb1e9fbba5787
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/14/2020
ms.locfileid: "92041594"
---
# <a name="quickstart-configure-properties-for-an-application-in-your-azure-active-directory-azure-ad-tenant"></a>快速入门：在 Azure Active Directory (Azure AD) 租户中配置应用程序的属性

在前面的快速入门中，你已将一个应用程序添加到 Azure Active Directory (Azure AD) 租户。 添加应用程序时，需让你的 Azure AD 租户知道它是该应用的标识提供者。 现在，你将配置应用程序的一些属性。
 
## <a name="prerequisites"></a>先决条件

要在 Azure AD 租户中配置应用程序的属性，你需要：

- 具有活动订阅的 Azure 帐户。 [创建帐户](https://www.azure.cn/pricing/1rmb-trial)。
- 以下角色之一：全局管理员、云应用程序管理员、应用程序管理员或服务主体的所有者。
- 可选：完成[查看应用](view-applications-portal.md)。

>[!IMPORTANT]
>使用非生产环境测试本快速入门中的步骤。

## <a name="configure-app-properties"></a>配置应用属性

将应用程序添加到 Azure AD 租户后，“概述”页便会显示。 如果要配置已添加的应用程序，请查看第一个快速入门。 它将引导你查看已添加到租户的应用程序。 

编辑应用程序属性：

1. 在 Azure AD 门户中，选择“企业应用程序”。 然后找到并选择要配置的应用程序。
2. 在“管理”部分中，选择“属性”以打开“属性”窗格进行编辑  。

    ![“属性”屏幕的屏幕截图，其中显示可编辑的应用属性。](./media/add-application-portal/edit-properties.png)

3. 请花点时间了解可用于配置的选项：
    - “是否启用以供用户登录?”决定分配到应用程序的用户能否登录。
    - “是否需要进行用户分配?”决定了未分配到应用程序的用户能否登录。
    - “是否对用户可见?”决定分配到应用的用户能否在[我的应用](https://account.activedirectory.windowsazure.cn/r#/applications)和 Microsoft 365 应用启动器中看到该应用。 （请参阅 Microsoft 365 网站左上角的华夫饼菜单。）
    
    > [!TIP]
    > 在导航的“用户和组”部分分配用户。

    这三个选项可以彼此独立地切换，并且产生的行为并不总是明显的。 下面是一个表，可在以下方面提供帮助：
    
    | 启用以供用户登录? | 需要进行用户分配? | 对用户可见? | 已分配或未分配到应用的用户的行为。 |
    |---|---|---|---|
    | 是 | 是 | 是 | 已分配的用户可以看到应用并登录。<br>未分配的用户看不到应用且无法登录。 |
    | 是 | 是 | 否  | 已分配的用户看不到应用，但可以登录。<br>未分配的用户看不到应用且无法登录。 |
    | 是 | 否  | 是 | 已分配的用户可以看到应用并登录。<br>未分配的用户看不到应用，但可以登录。 |
    | 是 | 否  | 否  | 已分配的用户看不到应用，但可以登录。<br>未分配的用户看不到应用，但可以登录。 |
    | 否  | 是 | 是 | 已分配的用户看不到应用且无法登录。<br>未分配的用户看不到应用且无法登录。 |
    | 否  | 是 | 否  | 已分配的用户看不到应用且无法登录。<br>未分配的用户看不到应用且无法登录。 |
    | 否  | 否  | 是 | 已分配的用户看不到应用且无法登录。<br>未分配的用户看不到应用且无法登录。 |
    | 否  | 否  | 否  | 已分配的用户看不到应用且无法登录。<br>未分配的用户看不到应用且无法登录。 |

4. 完成后，选择“保存”。

## <a name="use-a-custom-logo"></a>使用自定义徽标

若要使用自定义徽标，请执行以下操作：

1. 创建一个 215 x 215 像素的徽标，将其保存为 .png 格式。
2. 在 Azure AD 门户中，选择“企业应用程序”。 然后找到并选择要配置的应用程序。
3. 在“管理”部分中，选择“属性”以打开“属性”窗格进行编辑  。 
4. 选择图标以上传徽标。
5. 完成后，选择“保存”。

    ![显示如何更改徽标的“属性”屏幕的屏幕截图。](./media/add-application-portal/change-logo.png)

   > [!NOTE]
   > 此“属性”窗格中显示的缩略图不会立即更新。 可以关闭并重新打开“属性”窗格以查看更新的图标。

## <a name="clean-up-resources"></a>清理资源

如果你不打算继续学习快速入门系列，请考虑删除应用以清理测试租户。 本系列中的最后一个快速入门介绍如何删除应用，请参阅[删除应用](delete-application-portal.md)。


