---
layout:     post
title:      PostgreSQL-distinct用法小记
subtitle:   PostgreSQL 查询去重处理
date:       2022-12-28
author:     W
header-img: img/post-bg-postgres.png
catalog: true
tags:
    - 数据库
    - PostgreSQL
    - distinct
    - 去重
---

## 前言

在 PostgreSQL 中，DISTINCT 关键字与 SELECT 语句一起使用，用于去除重复记录，只获取唯一的记录。

我们平时在操作数据时，有可能出现一种情况，在一个表中有多个重复的记录，当提取这样的记录时，DISTINCT 关键字就显得特别有意义，它只获取唯一一次记录，而不是获取重复记录。

## 正文

标准用法

```sql
SELECT DISTINCT column1, column2,.....columnN
FROM table_name
WHERE [condition]
```

很多时候我们在查询时，只想根据部分字段进行去重，但仍需要返回完整数据结构，PostgreSQL 提供了 `DISTINCT ON`关键词

```sql
SELECT DISTINCT ON (b.column1,b.column2,.....b.columnN) b.* FROM table_name AS b
```

