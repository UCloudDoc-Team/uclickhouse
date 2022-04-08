# 删除数据

使用 DROP 或 TRUNCATE 语句来完成数据删除，DROP 删除元数据和数据，TRUNCATE 只删除数据。

**基本语法：**

```
DROP DATABASE [IF EXISTS] db [ON CLUSTER ck_cluster]DROP [TEMPORARY] TABLE [IF EXISTS] [db.]name [ON CLUSTER ck_cluster]
TRUNCATE TABLE [IF EXISTS] [db.]name [ON CLUSTER ck_cluster]
```

