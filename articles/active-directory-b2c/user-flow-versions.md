---
title: Azure Active Directory B2C 中的用户流版本 | Microsoft Docs
description: 了解 Azure Active Directory B2C 中可用的用户流版本。
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 08/25/2020
ms.author: v-junlch
ms.subservice: B2C
ms.openlocfilehash: 3ec1dfe8b3f0baac8066a1d48383e94c572d3c0b
ms.sourcegitcommit: b5ea35dcd86ff81a003ac9a7a2c6f373204d111d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/27/2020
ms.locfileid: "88946529"
---
# <a name="user-flow-versions-in-azure-active-directory-b2c"></a>Azure Active Directory B2C 中的用户流版本

Azure Active Directory B2C (Azure AD B2C) 中的用户流可帮助设置完全描述客户标识体验的常见[策略](user-flow-overview.md)。 这些体验包括注册、登录、密码重置和配置文件编辑。 下表描述了 Azure AD B2C 中可用的用户流。

> [!IMPORTANT]
> 我们更改了引用用户流版本的方式。 之前，我们提供 V1（生产就绪）版本，还提供了 V1.1 和 V2（预览版）版本。 现在，我们已将用户流合并为两个版本：
>
>- 建议的用户流是用户流的新预览版本。 它们经过全面测试，并结合了旧的 V2 和 V1.1 版本的所有功能 。 今后，将维护并更新新的建议用户流。 转到这些新的建议用户流后，就可以在新功能发布后对其进行访问。
>- 标准用户流（以前称为 V1）是正式发布的生产就绪用户流 。 如果用户流是任务关键型，并且依赖于高度稳定的版本，可以继续使用标准用户流，但请认识到我们不会维护和更新这些版本。
>
>将在 2021 年 8 月 1 日之前逐渐弃用所有旧预览版用户流（V1.1 和 V2）。 强烈建议尽早[切换到新的建议版本](#how-to-switch-to-a-new-recommended-user-flow)，以便始终可以利用最新的功能和更新。 这些更改仅适用于 Azure 公有云。其他环境将继续使用[旧用户流版本控制](user-flow-versions-legacy.md)。

## <a name="recommended-user-flows"></a>建议的用户流

建议的用户流是预览版本，它将新功能与旧的 V2 和 V1.1 功能组合在一起。 其后，我们将维护并更新建议的用户流。

| 用户流 | 描述 |
| --------- | ----------- |
| 密码重置（预览版） | 允许用户在验证电子邮件后选择新密码。 使用此用户流，可配置： <ul><li>[多重身份验证](custom-policy-multi-factor-authentication.md)</li><li>令牌兼容性设置</li><li>[年龄限制](basic-age-gating.md)</li><li>[密码复杂性要求](user-flow-password-complexity.md)</li></ul> |
| 配置文件编辑（预览版） | 允许用户配置用户特性。 使用此用户流，可配置： <ul><li>[令牌生存期](tokens-overview.md)</li><li>令牌兼容性设置</li><li>会话行为</li></ul> |
| 登录（预览版） | 允许用户登录帐户。 使用此用户流，可配置： <ul><li>[多重身份验证](custom-policy-multi-factor-authentication.md)</li><li>[令牌生存期](tokens-overview.md)</li><li>令牌兼容性设置</li><li>会话行为</li><li>[年龄限制](basic-age-gating.md)</li><li>登录页自定义</li></ul> |
| 注册（预览版） | 允许用户创建账户。 使用此用户流，可配置： <ul><li>[多重身份验证](custom-policy-multi-factor-authentication.md)</li><li>[令牌生存期](tokens-overview.md)</li><li>令牌兼容性设置</li><li>会话行为</li><li>[年龄限制](basic-age-gating.md)</li><li>[密码复杂性要求](user-flow-password-complexity.md)</li></ul> |
| 注册和登录（预览版） | 允许用户创建帐户或登录帐户。 使用此用户流，可配置： <ul><li>[多重身份验证](custom-policy-multi-factor-authentication.md)</li><li>[年龄限制](basic-age-gating.md)</li><li>[密码复杂性要求](user-flow-password-complexity.md)</li></ul> |

## <a name="standard-user-flows"></a>标准用户流

标准用户流（以前称为 V1）是正式发布的生产就绪用户流。 标准用户流今后将不会更新。

| 用户流 | 描述 |
| --------- | ----------- | ----------- |
| 密码重置 | 允许用户在验证电子邮件后选择新密码。 使用此用户流，可配置： <ul><li>[多重身份验证](custom-policy-multi-factor-authentication.md)</li><li>令牌兼容性设置</li><li>[密码复杂性要求](user-flow-password-complexity.md)</li></ul> |
| 配置文件编辑 | 允许用户配置用户特性。 使用此用户流，可配置： <ul><li>[令牌生存期](tokens-overview.md)</li><li>令牌兼容性设置</li><li>会话行为</li></ul> |
| 登录 | 允许用户登录帐户。 使用此用户流，可配置： <ul><li>[多重身份验证](custom-policy-multi-factor-authentication.md)</li><li>[令牌生存期](tokens-overview.md)</li><li>令牌兼容性设置</li><li>会话行为</li><li>阻止登录</li><li>强制执行密码重置</li><li>使我保持登录状态 (KMSI)</ul><br>无法使用此用户流自定义用户界面。 |
| 注册 | 允许用户创建账户。 使用此用户流，可配置： <ul><li>[多重身份验证](custom-policy-multi-factor-authentication.md)</li><li>[令牌生存期](tokens-overview.md)</li><li>令牌兼容性设置</li><li>会话行为</li><li>[密码复杂性要求](user-flow-password-complexity.md)</li></ul> |
| 注册和登录 | 允许用户创建帐户或登录帐户。 使用此用户流，可配置： <ul><li>[多重身份验证](custom-policy-multi-factor-authentication.md)</li><li>[令牌生存期](tokens-overview.md)</li><li>令牌兼容性设置</li><li>会话行为</li><li>[密码复杂性要求](user-flow-password-complexity.md)</li></ul>|


## <a name="how-to-switch-to-a-new-recommended-user-flow"></a>如何切换到新的建议用户流

若要从旧版本的用户流切换到新的建议预览版本，请执行以下步骤：

1. 按照[教程：在 Azure Active Directory 中创建用户流](tutorial-create-user-flows.md)中的步骤创建新的用户流策略。 创建用户流时，请选择建议的版本。

3. 用在旧版策略中配置的相同设置来配置新的用户流。

4. 将应用程序登录 URL 更新到新创建的策略。

5. 测试用户流并确认其正常工作后，请按照以下步骤操作，删除旧用户流：
   1. 在 Azure AD B2C 租户概述菜单中，选择“用户流”。
   2. 找到要删除的用户流。
   3. 在最后一列中，选择上下文菜单（“...”），然后选择“删除” 。

## <a name="frequently-asked-questions"></a>常见问题解答

### <a name="can-i-still-create-legacy-v2-and-v11-user-flows"></a>是否仍可创建旧的 V2 和 V1.1 用户流？

你将无法基于旧的 V2 和 V1.1 版创建新的用户流，但可以继续读取、更新和删除当前正在使用的任何旧 V2 和 V1.1 用户流。

### <a name="is-there-any-reason-to-continue-using-legacy-v2-and-v11-user-flows"></a>是否有任何理由继续使用旧的 V2 和 V1.1 用户流？

不完全是。 新的建议的预览版本包含与旧的 V2 和 V1.1 版本相同的功能。 没有删除任何内容，事实上它们现在包含其他功能。

### <a name="if-i-dont-switch-from-legacy-v2-and-v11-policies-how-will-it-impact-my-application"></a>如果不从旧的 V2 和 V1.1 策略进行切换，我的应用程序会受到什么影响？

如果使用旧的 V2 或 V1.1 用户流，此版本更改将不会影响你的应用程序。 但是，若要访问新功能或后续策略更改，需要切换到新的建议的版本。

### <a name="will-microsoft-still-support-my-legacy-v2-or-v11-user-flow-policy"></a>Microsoft 将继续支持旧的 V2 或 V1.1 用户流策略吗？

旧 V2 和 V1.1 版本的用户流将继续完全受支持。

