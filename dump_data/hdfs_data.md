# 使用hdfs引擎同步数据

1.使用hdfs表引擎创建外部表

```
CREATE TABLE hdfs_engine_table (name String, value UInt32) ENGINE=HDFS('hdfs://hdfs1:9000/other_storage', 'TSV')
```

2.插入数据

```
INSERT INTO hdfs_engine_table VALUES ('one', 1), ('two', 2), ('three', 3)
```

3.
```
SELECT * FROM hdfs_engine_table
```
