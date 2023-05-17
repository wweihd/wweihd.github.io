---
layout:     post
title:      PostgreSQL-查询某一字段相同，另一字段不同的数据
subtitle:   PostgreSQL 分组查询
date:       2023-05-17
author:     W
header-img: img/post-bg-postgres.png
catalog: true
tags:
    - 数据库
    - PostgreSQL
    - group by
    - 分组
---

## 正文

根据某一字段分组，查询该字段相同时另一字段详情

```sql
SELECT column1,column2 FROM table_name
WHERE column1 IN (SELECT column1 FROM table_name GROUP BY column1 HAVING COUNT(DISTINCT column2)>1)
GROUP BY column1,column2
ORDER BY column1
```
