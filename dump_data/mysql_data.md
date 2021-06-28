# 从mysql导入数据

依据MySQL表结构在ClickHouse中进行建表操作
MySQL中数据类型与ClickHouse类型映射关系参考:[开发指南](developer.md)

MySQL建表：

```
CREATE TABLE testdb.mysql_test_table (
  id int NOT NULL,
  quarter tinyint unsigned DEFAULT NULL,
  month tinyint DEFAULT NULL,
  day_of_month smallint unsigned DEFAULT NULL,
  day_of_week smallint DEFAULT NULL,
  airline_id int DEFAULT NULL,
  carrier float DEFAULT NULL,
  origin double DEFAULT NULL,
  unique_carrier varchar(80) NOT NULL,
  flight_date date NOT NULL,
  tail_date datetime DEFAULT NULL,
  origin_airport_time timestamp,
  comment varchar(100)
) ENGINE=InnoDB
```

在对应的ClickHouse建表：

```
--建立本地表
create table default.clickhouse_test_table ON CLUSTER ch_cluster (
  id Int32,
  quarter Nullable(UInt32),
  month Nullable(Int8),
  day_of_month Nullable(UInt16),
  day_of_week Nullable(Int16),
  airline_id Nullable(Int32),
  carrier Nullable(Float32),
  origin Nullable(Float64),
  unique_carrier String,
  flight_date Date,
  tail_date Nullable(Datetime),
  origin_airport_time Nullable(Datetime),
  comment Nullable(String)
) ENGINE = ReplicatedMergeTree(
    '/clickhouse/tables/clickhouse_test_table/{shard}',
    '{replica}',
    flight_date,
    (id, unique_carrier),
    8192);

--建立分布式表
CREATE TABLE clickhouse_test_table_distributed ON CLUSTER ch_cluster
 AS clickhouse_test_table
ENGINE = Distributed(default, default, clickhouse_test_table, rand());
```

插入数据

```
insert into <table_name> select * 
from mysql('<host>:<port>', '<db_name>','<table_name>', '<username>', '<password>')
```

查询数据

```
select count(*) from <table_name>
```
