# clickhouse间数据导入

当两个UClickHouse集群需要同步数据且在同一可用区下

下载clickhouse-client客户端

## 重建表结构

查看源clickhouse集群database列表

```
clickhouse-client --host="<old host>" --port="<old port>" --user="<old user name>" --password="<old password>" --query="SHOW databases"  > database.list
```

查看源clickhouse集群table列表

```
clickhouse-client --host="<old host>" --port="<old port>" --user="<old user name>" --password="<old password>" --query="SHOW tables from <database_name>"  > table.list
```

导出源clickhouse集群建表DDL

```
clickhouse-client --host="<old host>" --port="<old port>" --user="<old user name>" --password="<old password>" --query="SHOW CREATE TABLE <database_name>.<table_name>"  > table.sql
```

将建表DDL导入到目标clickhouse集群

```
clickhouse-client --host="<new host>" --port="<new port>" --user="<new user name>" --password="<new password>"  < table.sql
```

## 数据迁移

### remote数据迁移

```
insert into <new_database>.<new_table> select * from remote('old_endpoint', <old_database>.<old_table>, '<username>', '<password>');
```

各参数含义：

| 参数                 | 描述                      |
|----------------------|---------------------------|
| new_database         | 目标clickhouse集群库名    |
| new_table            | 目标clickhouse集群表名    |
| old_endpoint         | 源clickhouse的endpoint    |
| old_database         | 源clickhouse集群的库名    |
| old_table  | UInt32  | 源clickhouse集群的表名    |
| username             | 源clickhouse集群用户名    |
| password             | 源clickhouse集群的密码    |


### 导出文件迁移

1.将源数据导出成csv格式文件

```
clickhouse-client --host="<old host>" --port="<oldport>" --user="<old user name>" --password="<old password>"  --query="select * from <database_name>.<table_name> FORMAT CSV"  > table.csv
```

2.将csv文件导入目标集群

```
clickhouse-client --host="<new host>" --port="<new port>" --user="<new user name>" --password="<new password>"  --query="insert into <database_name>.<table_name> FORMAT CSV"  < table.csv
```
