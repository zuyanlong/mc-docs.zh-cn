---
title: Azure Cosmos DB 查询语言中的 ARRAY_CONTAINS
description: 了解 Azure Cosmos DB 中的 Array Contains SQL 系统函数如何返回一个布尔值指示数组是否包含指定值
author: rockboyfor
ms.service: cosmos-db
ms.topic: conceptual
origin.date: 09/13/2019
ms.date: 04/27/2020
ms.author: v-yeche
ms.custom: query-reference
ms.openlocfilehash: d78e93344911c66441994a699c5e173b7a20d232
ms.sourcegitcommit: f9c242ce5df12e1cd85471adae52530c4de4c7d7
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/24/2020
ms.locfileid: "82134572"
---
# <a name="array_contains-azure-cosmos-db"></a>ARRAY_CONTAINS (Azure Cosmos DB)
返回一个布尔，它指示数组是否包含指定的值。 可以通过在命令中使用布尔表达式来检查对象的部分匹配或完全匹配。 

## <a name="syntax"></a>语法

```sql
ARRAY_CONTAINS (<arr_expr>, <expr> [, bool_expr])  
```  

## <a name="arguments"></a>参数

*arr_expr*  
   是要搜索的数组表达式。  

*expr*  
   是要找到的表达式。  

*bool_expr*  
   是布尔表达式。 如果它的计算值为“true”，并且指定的搜索值是对象，则该命令将检查部分匹配（搜索对象是其中一个对象的子集）。 如果它的计算值为“false”，则该命令将检查数组中所有对象的完全匹配。 如果未指定，默认值为“false”。 

## <a name="return-types"></a>返回类型

  返回一个布尔值。  

## <a name="examples"></a>示例

  以下示例演示了如何使用 `ARRAY_CONTAINS` 检查数组中的成员身份。  

```sql
SELECT   
           ARRAY_CONTAINS(["apples", "strawberries", "bananas"], "apples") AS b1,  
           ARRAY_CONTAINS(["apples", "strawberries", "bananas"], "mangoes") AS b2  
```  

 下面是结果集。  

```json
[{"b1": true, "b2": false}]  
```  

以下示例介绍了如何使用 ARRAY_CONTAINS 检查数组内是否存在 JSON 字符串的部分匹配字符串。  

```sql
SELECT  
    ARRAY_CONTAINS([{"name": "apples", "fresh": true}, {"name": "strawberries", "fresh": true}], {"name": "apples"}, true) AS b1, 
    ARRAY_CONTAINS([{"name": "apples", "fresh": true}, {"name": "strawberries", "fresh": true}], {"name": "apples"}) AS b2,
    ARRAY_CONTAINS([{"name": "apples", "fresh": true}, {"name": "strawberries", "fresh": true}], {"name": "mangoes"}, true) AS b3 
```  

 下面是结果集。  

```json
[{
  "b1": true,
  "b2": false,
  "b3": false
}]
```

## <a name="remarks"></a>备注

此系统函数将从[范围索引](index-policy.md#includeexclude-strategy)中获益。

## <a name="next-steps"></a>后续步骤

- [数组函数 Azure Cosmos DB](sql-query-array-functions.md)
- [系统函数 Azure Cosmos DB](sql-query-system-functions.md)
- [Azure Cosmos DB 简介](introduction.md)

<!-- Update_Description: update meta properties, wording update, update link -->