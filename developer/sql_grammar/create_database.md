# 建库（Create Database） 

**基本语法：**

```sql
CREATE DATABASE [IF NOT EXISTS] db_name [ON CLUSTER ck_cluster];
```

当数据库已经存在时，避免执行报错，可指定 IF NOT EXISTS 关键字，指定后重复执行语句不会报错且也不会重建数据库。

ON CLUSTER：用于指定集群名称，例云数据仓库UClickhouse 默认集群名称为ck_cluster。

**示例：**

```sql
CREATE DATABASE ck_test ON CLUSTER ck_cluster;
```

