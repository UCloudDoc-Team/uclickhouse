# 修改表结构

使用 ALTER 语句来完成表结构修改。

**基本语法：**

```
# 对表的列操作
ALTER TABLE [db].name [ON CLUSTER cluster] ADD COLUMN [IF NOT EXISTS] name [type] [default_expr] [codec] [AFTER name_after]ALTER TABLE [db].name [ON CLUSTER cluster] DROP COLUMN [IF EXISTS] nameALTER TABLE [db].name [ON CLUSTER cluster] CLEAR COLUMN [IF EXISTS] name IN PARTITION partition_nameALTER TABLE [db].name [ON CLUSTER cluster] COMMENT COLUMN [IF EXISTS] name 'comment'ALTER TABLE [db].name [ON CLUSTER cluster] MODIFY COLUMN [IF EXISTS] name [type] [default_expr] [TTL]

# 对表的分区操作
ALTER TABLE table_name DETACH PARTITION partition_exprALTER TABLE table_name DROP PARTITION partition_exprALTER TABLE table_name CLEAR INDEX index_name IN PARTITION partition_expr

# 对表的属性操作
ALTER TABLE table-name MODIFY TTL ttl-expression
```

更多信息，请具体参考[官方文档](https://clickhouse.com/docs/en/sql-reference/statements/alter/)