# 开发指南

## SQL语法

### 创建库

CREATE DATABASE基本语法：
```
CREATE DATABASE [IF NOT EXISTS] db_name [ON CLUSTER cluster];
```

ON CLUSTER 关键字用于指定集群名称（UClickHouse唯一指定集群名称为 ch_cluster）

```
CREATE DATABASE db_test ON CLUSTER ch_cluster;
```

### 创建表

基本建表语句：

```
CREATE TABLE [IF NOT EXISTS] [db.]table_name ON CLUSTER cluster
(
    name1 [type1] [DEFAULT|MATERIALIZED|ALIAS expr1],
    name2 [type2] [DEFAULT|MATERIALIZED|ALIAS expr2],
    ...
    INDEX index_name1 expr1 TYPE type1(...) GRANULARITY value1,
    INDEX index_name2 expr2 TYPE type2(...) GRANULARITY value2
) ENGINE = engine_name()
[PARTITION BY expr]
[ORDER BY expr]
[PRIMARY KEY expr]
[SAMPLE BY expr]
[SETTINGS name=value, ...];
```


创建复制表

双副本集群，要求必须使用Replicated* 系列表引擎，数据在副本间自动同步，达到高可用。
{shard}、{replica}为宏定义，无需具体制定修改。

```
CREATE TABLE city_local on cluster ch_cluster (
  `fdate` Int64,
  `city_code` Int32,
  `city_name` String,
  `total_cnt` Int64
) ENGINE = ReplicatedMergeTree('/clickhouse/tables/{shard}/city_local', '{replica}')
PARTITION BY fdate
ORDER BY (fdate, city_code, city_name)
SETTINGS index_granularity = 8192;
```

创建分布式表

```
CREATE TABLE city_all on cluster ch_cluster (
  `fdate` Int64,
  `city_code` Int32,
  `city_name` String,
  `total_cnt` Int64
) ENGINE = Distributed(ch_cluster, default, city_local, rand())
```

创建视图

```
CREATE [MATERIALIZED] VIEW [IF NOT EXISTS] [db.]table_name [TO[db.]name] ON CLUSTER ch_cluster [ENGINE = engine] [POPULATE] AS SELECT ...
```

创建临时表

```
CREATE TEMPORARY TABLE [IF NOT EXISTS] table_name ON CLUSTER ch_cluster
(
    name1 [type1] [DEFAULT|MATERIALIZED|ALIAS expr1],
    name2 [type2] [DEFAULT|MATERIALIZED|ALIAS expr2],
    ...
)
```

### INSERT INTO

```
INSERT INTO [db.]table [(c1, c2, c3)] VALUES (v11, v12, v13), (v21, v22, v23), ...
```

对于存在于表结构中但不存在于插入列表中的列，它们将会按照如下方式填充数据：
* 如果存在DEFAULT表达式，根据DEFAULT表达式计算被填充的值。 
* 如果没有定义DEFAULT表达式，则填充零或空字符串。

### SELECT

```
SELECT [DISTINCT] expr_list
    [FROM [db.]table | (subquery) | table_function] [FINAL]
    [SAMPLE sample_coeff]
    [ARRAY JOIN ...]
    [GLOBAL] ANY|ALL INNER|LEFT JOIN (subquery)|table USING columns_list
    [PREWHERE expr]
    [WHERE expr]
    [GROUP BY expr_list] [WITH TOTALS]
    [HAVING expr]
    [ORDER BY expr_list]
    [LIMIT [n, ]m]
    [UNION ALL ...]
    [INTO OUTFILE filename]
    [FORMAT format]
    [LIMIT n BY columns]
```
所有的子句都是可选的，除了SELECT之后的表达式列表（expr_list）。 下面将选择部分子句进行说明。ClickHouse官网中文文档有更详细说明，请参考[查询语法](https://clickhouse.tech/docs/zh/sql-reference/statements/select/?spm=a2c4g.11186623.2.9.513a4e127y4bnI)。

## 数据库引擎

### 延时引擎Lazy

在距最近一次访问间隔expiration_time_in_seconds时间段内，将表保存在内存中，仅适用于&Log引擎表
由于针对这类表的访问间隔较长，对保存大量的*Log引擎表进行了优化
创建数据库
```
CREATE DATABASE testlazy ENGINE = Lazy(expiration_time_in_seconds);
```

### Atomic

支持非阻塞的 drop 和 rename 表查询以及原子交换表t1和t2查询。默认情况下使用 Atomic 数据库引擎。
创建数据库
```
CREATE DATABASE test ENGINE = Atomic
```

### MySQL

MySQL引擎用于将远程的myqsl服务器中的表映射到clickhouse中，并允许对表进行 insert 和 select 查询，方便在clickhouse与mysql之间进行数据交换。
MySQL数据库引擎会将对其的查询转换为mysql预防并发送到mysql服务器中，因此，可以执行 show tables 或者 show create table 之类的操作。
无法进行的操作（rename、create table、alter）
创建数据库
```
CREATE DATABASE[IF NOT EXISTS] db_name[ON CLUSTER cluster] ENGINE = MySQL('host:port',['database'|database],'user','password')
```

MySQL数据库引擎参数:
* host：port  --  链接的mysql地址
* database  --  链接的mysql数据库
* user  --  链接的mysql用户
* password  --  链接的mysql用户密码


## 表引擎

表引擎（即表的类型）决定了：
* 数据的存储方式和位置，写到哪里以及从哪里读取数据
* 支持哪些查询以及如何支持
* 并发数据访问
* 索引的使用（如果存在）
* 是否可以执行多线程请求
* 数据复制参数



