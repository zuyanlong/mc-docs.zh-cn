---
title: datetime_diff() - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 datetime_diff()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 08/06/2020
ms.openlocfilehash: 932dc2dfae09db2224dd080ca9b03720d43086bc
ms.sourcegitcommit: 7ceeca89c0f0057610d998b64c000a2bb0a57285
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/06/2020
ms.locfileid: "87841774"
---
# <a name="datetime_diff"></a>datetime_diff()

计算两个 [datetime](./scalar-data-types/datetime.md) 值在日历上的差距。

**语法**

`datetime_diff(`*period*`,`*datetime_1*`,`*datetime_2*`)`

**参数**

* `period`: `string`. 
* `datetime_1`：[datetime](./scalar-data-types/datetime.md) 值。
* `datetime_2`：[datetime](./scalar-data-types/datetime.md) 值。

period 的可能值： 
- Year
- Quarter
- Month
- Week
- 日期
- Hour
- Minute
- Second
- Millisecond
- Microsecond
- Nanosecond

**返回**

一个整数，表示减法结果中 `periods` 的量 (`datetime_1` - `datetime_2`)。

**示例**

```kusto
print
year = datetime_diff('year',datetime(2017-01-01),datetime(2000-12-31)),
quarter = datetime_diff('quarter',datetime(2017-07-01),datetime(2017-03-30)),
month = datetime_diff('month',datetime(2017-01-01),datetime(2015-12-30)),
week = datetime_diff('week',datetime(2017-10-29 00:00),datetime(2017-09-30 23:59)),
day = datetime_diff('day',datetime(2017-10-29 00:00),datetime(2017-09-30 23:59)),
hour = datetime_diff('hour',datetime(2017-10-31 01:00),datetime(2017-10-30 23:59)),
minute = datetime_diff('minute',datetime(2017-10-30 23:05:01),datetime(2017-10-30 23:00:59)),
second = datetime_diff('second',datetime(2017-10-30 23:00:10.100),datetime(2017-10-30 23:00:00.900)),
millisecond = datetime_diff('millisecond',datetime(2017-10-30 23:00:00.200100),datetime(2017-10-30 23:00:00.100900)),
microsecond = datetime_diff('microsecond',datetime(2017-10-30 23:00:00.1009001),datetime(2017-10-30 23:00:00.1008009)),
nanosecond = datetime_diff('nanosecond',datetime(2017-10-30 23:00:00.0000000),datetime(2017-10-30 23:00:00.0000007))
```

|year|quarter|月份|week|day|hour|minute|第 2 个|毫秒|微秒|纳秒|
|---|---|---|---|---|---|---|---|---|---|---|
|17|2|13|5|29|2|5|10|100|100|-700|
