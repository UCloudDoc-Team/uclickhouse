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

MySQL中的数据类型与ClickHouse类型映射关系如下表：

| MySQL                             | ClickHouse         |
|-----------------------------------|--------------------|
| UNSIGNED TINYINT                  | UInt8              |   
| TINYINT                           | Int8               | 
| UNSIGNED SMALLINT                 | UInt16             |
| SMALLINT                          | Int16              | 
| UNSIGNED INT, UNSIGNED MEDIUMINT  | UInt32             |
| INT, MEDIUMINT                    | Int32              |
| UNSIGNED BIGINT                   | UInt64             |
| BIGINT                            | Int64              |
| FLOAT                             | Float32            |
| DOUBLE                            | Float64            |
| DATE                              | Date               |
| DATETIME, TIMESTAMP               | DateTime           |
| BINARY                            | FixedString        | 



## 表引擎

表引擎（即表的类型）决定了：
* 数据的存储方式和位置，写到哪里以及从哪里读取数据
* 支持哪些查询以及如何支持
* 并发数据访问
* 索引的使用（如果存在）
* 是否可以执行多线程请求
* 数据复制参数


### MergeTree
适用于高负载任务的最通用和功能最强大的表引擎。这些表引擎的共同特点是可以快速插入数据并后续的后台数据处理。MergeTree系列引擎支持数据复制（使用Replicated*的引擎版本），分区和一些其他引擎不支持的其他功能。

* MergeTree
  clickhouse中最强大的表引擎MergeTree引擎及该系列（*MergeTree）中的其他引擎。该引擎被设计用于插入极大量的数据到一张表当中。数据可以以数据片段的形式一个接一个的快速写入，数据片段在后台按照一定的规则进行合并。相比在插入时不断修改（重写）已存储的数据，这种策略会高效很多。

* ReplacingMergeTree
  该引擎和 MergeTree的不同之处在于它会删除排序键值相同的重复项。用于在后台清除重复的数据以节省空间，但是它不保证没有重复的数据出现。

* SummingMergeTree
  当合并SummingMergeTree表的数据片段时，clickhouse会把所有具有相同主键的行合并为一行，该行包含了被合并的行中具有数值数据类型的列的汇总值。

* AggregatingMergeTree
  改变数据片段合并逻辑，clcikhouse会将一个数据片段内所有具有相同主键（排序键）的行替换成一行，这一行会存储一系列聚合函数的状态。

* CollapsingMergeTree
  在数据块合并算法中添加了折叠行的逻辑。能异步删除（折叠）这些除了特定列Sign有1和-1的值以外，其余所有字段的值都相等的成对的行。

* VersionedCollapsingMergeTree
  1.允许快速写入不断变化的对象状态2.删除后台中的旧对象状态。这显著降低存储体积。

* GraphitMergeTree
  该引擎用来对Graphite数据进行瘦身及汇总。对于想使用ch来存储Graphite数据的开发者来说很有用。

### 数据副本
只有MergeTree系列里的表可支持副本

* ReplicatedMergeTree
* ReplicatedSummingMergeTree
* ReplicatedReplacingMergeTree
* ReplicatedAggregatingMergeTree
* ReplicatedCollapsingMergeTree
* ReplicatedVersionedCollapsingMergetree
* ReplicatedGraphiteMergeTree

副本是表级别的，不是整个服务器级的。所以，服务器里可以同时有复制表和非复制表。
副本不依赖分片。每个分片有它自己的独立副本。
对于 INSERT 和 ALTER 语句操作数据的会在压缩的情况下被复制

创建复制表：
```
CREATE TABLE table_name
(
    EventDate DateTime,
    CounterID UInt32,
    UserID UInt32
) ENGINE = ReplicatedMergeTree('/clickhouse/tables/{layer}-{shard}/table_name', '{replica}')
PARTITION BY toYYYYMM(EventDate)
ORDER BY (CounterID, EventDate, intHash32(UserID))
SAMPLE BY intHash32(UserID)
```

* /clickhouse/tables/ 是公共前缀，我们推荐使用这个。
* {layer}-{shard} 是分片标识部分。在此示例中，由于 Yandex.Metrica 集群使用了两级分片，所以它是由两部分组成的。但对于大多数情况来说，你只需保留 {shard} 占位符即可，它会替换展开为分片标识。
* table_name 是该表在 ZooKeeper 中的名称。使其与 ClickHouse 中的表名相同比较好。 这里它被明确定义，跟 ClickHouse 表名不一样，它并不会被 RENAME 语句修改。
* HINT：你可以在前面添加一个数据库名称 table_name 也是 例如。 db_name.table_name
### 集成引擎

用于与其他的数据存储于处理系统集成的引擎
该类型的引擎：
Kafka
MySQL
ODBC
JDBC
HDFS

### 用于其他特定功能的引擎

该类型的引擎：
Distributed
MaterializedView
Dictionary
Merge
File
Null
Set
Join
URL
View
Memory
Buffer

其他引擎具体使用方法参考官方文档:[https://clickhouse.tech/docs/en/engines/table-engines/](https://clickhouse.tech/docs/en/engines/table-engines/)
