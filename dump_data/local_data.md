# 从本地导入数据

## 注意事项

支持导入的常见文件格式为TabSeparated、TabSeparatedWithNames、TabSeparatedWithNamesAndTypes、CSV和CSVWithNames。更多支持的文件格式，请参见官方[文件格式及说明](https://clickhouse.com/docs/zh/interfaces/formats/#tabseparated)。

## 操作步骤

### 准备测试数据

创建csv文件lineorder.csv放到之前创建的云主机数据盘/data目录下

```
1,1,7381,155190,828,"1996-01-02","5-LOW",0,17,2116823,17366547,4,2032150,74711,2,"1996-02-12","TRUCK",
1,2,7381,67310,163,"1996-01-02","5-LOW",0,36,4598316,17366547,9,4184467,76638,6,"1996-02-28","MAIL",
1,3,7381,63700,71,"1996-01-02","5-LOW",0,8,1330960,17366547,10,1197864,99822,2,"1996-03-05","REG AIR",
1,4,7381,2132,943,"1996-01-02","5-LOW",0,28,2895564,17366547,9,2634963,62047,6,"1996-03-30","AIR",
1,5,7381,24027,1625,"1996-01-02","5-LOW",0,24,2282448,17366547,10,2054203,57061,4,"1996-03-14","FOB",
1,6,7381,15635,1368,"1996-01-02","5-LOW",0,32,4962016,17366547,7,4614674,93037,2,"1996-02-07","MAIL",
2,1,15601,106170,1066,"1996-12-01","1-URGENT",0,38,4469446,4692918,0,4469446,70570,5,"1997-01-14","RAIL",
3,1,24664,4297,1959,"1993-10-14","5-LOW",0,45,5405805,19384625,6,5081456,72077,0,"1994-01-04","AIR",
3,2,24664,19036,1667,"1993-10-14","5-LOW",0,49,4679647,19384625,10,4211682,57301,0,"1993-12-20","RAIL",
3,3,24664,128449,1409,"1993-10-14","5-LOW",0,27,3989088,19384625,6,3749742,88646,7,"1993-11-22","SHIP",
```

### 创建数据库及数据表

- 双副本模式下建库建表

```sql
CREATE DATABASE IF NOT EXISTS ck_test ON CLUSTER ck_cluster;
CREATE TABLE ck_test.lineorder ON CLUSTER ck_cluster 
(
    LO_ORDERKEY             UInt32,
    LO_LINENUMBER           UInt8,
    LO_CUSTKEY              UInt32,
    LO_PARTKEY              UInt32,
    LO_SUPPKEY              UInt32,
    LO_ORDERDATE            Date,
    LO_ORDERPRIORITY        LowCardinality(String),
    LO_SHIPPRIORITY         UInt8,
    LO_QUANTITY             UInt8,
    LO_EXTENDEDPRICE        UInt32,
    LO_ORDTOTALPRICE        UInt32,
    LO_DISCOUNT             UInt8,
    LO_REVENUE              UInt32,
    LO_SUPPLYCOST           UInt32,
    LO_TAX                  UInt8,
    LO_COMMITDATE           Date,
    LO_SHIPMODE             LowCardinality(String)
)
ENGINE = ReplicatedMergeTree(
  '/clickhouse/ck_test/tables/{layer}-{shard}/lineorder',
  '{replica}'
) ORDER BY (LO_ORDERKEY);
```

- 单副本模式下建库建表

```sql
CREATE DATABASE IF NOT EXISTS ck_test;
CREATE TABLE ck_test.lineorder 
(
    LO_ORDERKEY             UInt32,
    LO_LINENUMBER           UInt8,
    LO_CUSTKEY              UInt32,
    LO_PARTKEY              UInt32,
    LO_SUPPKEY              UInt32,
    LO_ORDERDATE            Date,
    LO_ORDERPRIORITY        LowCardinality(String),
    LO_SHIPPRIORITY         UInt8,
    LO_QUANTITY             UInt8,
    LO_EXTENDEDPRICE        UInt32,
    LO_ORDTOTALPRICE        UInt32,
    LO_DISCOUNT             UInt8,
    LO_REVENUE              UInt32,
    LO_SUPPLYCOST           UInt32,
    LO_TAX                  UInt8,
    LO_COMMITDATE           Date,
    LO_SHIPMODE             LowCardinality(String)
)
ENGINE = MergeTree ORDER BY (LO_ORDERKEY);
```

#### 创建分布式表

如果您只需要同步数据至本地表，可跳过此步骤。

```
CREATE TABLE IF NOT EXISTS ck_test.lineorder on cluster ck_cluster AS ck_test.lineorder_local  ENGINE = Distributed(
       ck_cluster,
       ck_test,
       lineorder_local,
       rand()
);
```

使用clickhouse-client非交互模式执行以下命令导入本地数据

```shell
clickhouse-client --host=<任一节点IP地址> --port=9000 --user=admin --password=<创建集群时设置的密码> --database=ck_test  --query="INSERT INTO <ClickHouse表名> FORMAT <本地文件格式>" < /data/lineorder.csv
```

或者

```shell
cat <data_file> | clickhouse-client --host=<任一节点IP地址> --port=9000 --user=admin --password=<创建集群时设置的密码> --database=ck_test  --query="INSERT INTO <ClickHouse表名> FORMAT <本地文件格式>"
```

