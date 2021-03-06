---
title: “超出作业大小”错误
description: 说明如何在作业大小或模板过大时排查错误。
ms.topic: troubleshooting
origin.date: 09/25/2020
author: rockboyfor
ms.date: 10/05/2020
ms.testscope: no
ms.testdate: ''
ms.author: v-yeche
ms.openlocfilehash: 670e821752db34c3ebf272349ac09ac72b3e7130
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937825"
---
<!--Verified successfully on charactors only-->
# <a name="resolve-errors-for-job-size-exceeded"></a>解决“超出作业大小”错误

本文介绍如何解决“JobSizeExceededException”和“DeploymentSizeExceededException”错误。

## <a name="symptom"></a>症状

部署模板时收到一条错误，指出部署已超出限制。

## <a name="cause"></a>原因

如果模板的大小超过 4 MB，则会收到此错误。 为使用 [copy ](copy-resources.md) 创建许多实例的资源定义扩展模板后，4-MB 限制适用于模板的最终状态。 最终状态还包括变量和参数的已解析值。

部署作业还包括有关请求的元数据。 对于大型模板，与模板组合的元数据可能会超出作业允许的大小。

模板的其他限制如下：

* 256 个参数
* 256 个变量
* 800 个资源（包括副本计数）
* 64 个输出值
* 模板表达式中不超过 24,576 个字符

## <a name="solution-1---simplify-template"></a>解决方案 1 - 简化模板

第一种选择是简化模板。 如果模板部署了大量不同的资源类型，则可以进行这种选择。 考虑将模板划分为[链接模板](linked-templates.md)。 将资源类型划分为逻辑组，并为每个组添加链接模板。 例如，如果需要部署大量网络资源，则可以将这些资源移到链接模板中。

可以将其他资源设置为依赖于链接模板，并[从链接模板的输出获取值](linked-templates.md#get-values-from-linked-template)。

## <a name="solution-2---use-serial-copy"></a>解决方案 2 - 使用串行复制

第二种选择是将复制循环[从并行处理更改为串行处理](copy-resources.md#serial-or-parallel)。 仅当你怀疑错误是由于通过复制部署了大量资源时，才进行此选择。 此更改可能会显著增加部署时间，因为不会并行部署资源。

<!-- Update_Description: new article about error job size exceeded -->
<!--NEW.date: 10/05/2020-->