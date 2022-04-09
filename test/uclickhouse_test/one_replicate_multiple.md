# 多表测试

## 新建云数据仓库UClickhouse

请参阅 [快速上手](/uclickhouse/gettingstart)

## 连接云数据仓库UClickhouse

请参阅 **操作指南->[连接集群](/uclickhouse/operation_guide/connect_cluster)**

## 执行建库建表

**DDL语句如下：**

```sql
#创建数据库
CREATE DATABASE IF NOT EXISTS ssb_test ;
#创建 customer
CREATE TABLE IF NOT EXISTS ssb_test.customer  
(
    C_CUSTKEY    UInt32,
    C_NAME       String,
    C_ADDRESS    String,
    C_CITY       LowCardinality(String),
    C_NATION     LowCardinality(String),
    C_REGION     LowCardinality(String),
    C_PHONE      String,
    C_MKTSEGMENT LowCardinality(String)
)
ENGINE = MergeTree ORDER BY (C_CUSTKEY);
#创建 lineorder
CREATE TABLE IF NOT EXISTS ssb_test.lineorder
(
    LO_ORDERKEY      UInt32,
    LO_LINENUMBER    UInt8,
    LO_CUSTKEY       UInt32,
    LO_PARTKEY       UInt32,
    LO_SUPPKEY       UInt32,
    LO_ORDERDATE     Date,
    LO_ORDERPRIORITY LowCardinality(String),
    LO_SHIPPRIORITY  UInt8,
    LO_QUANTITY      UInt8,
    LO_EXTENDEDPRICE UInt32,
    LO_ORDTOTALPRICE UInt32,
    LO_DISCOUNT      UInt8,
    LO_REVENUE       UInt32,
    LO_SUPPLYCOST    UInt32,
    LO_TAX           UInt8,
    LO_COMMITDATE    Date,
    LO_SHIPMODE      LowCardinality(String)
) ENGINE = MergeTree PARTITION BY toYear(LO_ORDERDATE) ORDER BY (LO_ORDERDATE, LO_ORDERKEY);
#创建 part
CREATE TABLE IF NOT EXISTS ssb_test.part 
(
    P_PARTKEY   UInt32,
    P_NAME      String,
    P_MFGR      LowCardinality(String),
    P_CATEGORY  LowCardinality(String),
    P_BRAND     LowCardinality(String),
    P_COLOR     LowCardinality(String),
    P_TYPE      LowCardinality(String),
    P_SIZE      UInt8,
    P_CONTAINER LowCardinality(String)
) ENGINE = MergeTree ORDER BY P_PARTKEY;
#创建 supplier
CREATE TABLE IF NOT EXISTS ssb_test.supplier 
(
    S_SUPPKEY UInt32,
    S_NAME    String,
    S_ADDRESS String,
    S_CITY    LowCardinality(String),
    S_NATION  LowCardinality(String),
    S_REGION  LowCardinality(String),
    S_PHONE   String
) ENGINE = MergeTree ORDER BY S_SUPPKEY;
#创建 dates
CREATE TABLE IF NOT EXISTS ssb_test.dates
(
    D_DATEKEY          UInt32,
    D_DATE             String,
    D_DAYOFWEEK        String,
    D_MONTH            String,
    D_YEAR             UInt32,
    D_YEARMONTHNUM     UInt32,
    D_YEARMONTH        String,
    D_DAYNUMINWEEK     UInt32,
    D_DAYNUMINMONTH    UInt32,
    D_DAYNUMINYEAR     UInt32,
    D_MONTHNUMINYEAR   UInt32,
    D_WEEKNUMINYEAR    UInt32,
    D_SELLINGSEASON    String,
    D_LASTDAYINWEEKFL  UInt32,
    D_LASTDAYINMONTHFL UInt32,
    D_HOLIDAYFL        UInt32,
    D_WEEKDAYFL        UInt32
) ENGINE = MergeTree ORDER BY D_DATEKEY;
```

## 数据准备

请参阅 **性能测试工具 -> [Star Schema Benchmark](/uclickhouse/test/tool/ssb)**

## 数据导入

```shell
[root@xxxxx ssb-dbgen]# clickhouse-client --host=<云数据仓库UClickhouse节点地址> --port=9000 --user=admin            --password=<建立集群时设置的管理密码> --database=ssb_test --query "INSERT INTO customer FORMAT CSV" < customer.tbl
[root@xxxxx ssb-dbgen]# clickhouse-client --host=<云数据仓库UClickhouse节点地址> --port=9000 --user=admin            --password=<建立集群时设置的管理密码> --database=ssb_test --query "INSERT INTO supplier FORMAT CSV" < supplier.tbl
[root@xxxxx ssb-dbgen]# clickhouse-client --host=<云数据仓库UClickhouse节点地址> --port=9000 --user=admin            --password=<建立集群时设置的管理密码> --database=ssb_test --query "INSERT INTO part FORMAT CSV" < part.tbl
[root@xxxxx ssb-dbgen]# clickhouse-client --host=<云数据仓库UClickhouse节点地址> --port=9000 --user=admin            --password=<建立集群时设置的管理密码> --database=ssb_test --query "INSERT INTO lineorder FORMAT CSV" < lineorder.tbl
[root@xxxxx ssb-dbgen]# clickhouse-client --host=<云数据仓库UClickhouse节点地址> --port=9000 --user=admin            --password=<建立集群时设置的管理密码> --database=ssb_test --query "INSERT INTO dates FORMAT CSV" < date.tbl
```

## 运行多表性能测试语句

请参阅 **性能测试指南 -> [多表性能测试语句](/uclickhouse/test/multiple_query)**

