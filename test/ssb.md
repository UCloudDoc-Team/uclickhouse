# Star Schema Benchmark

## 下载并编译工具

```shell
[root@xxxxx test]# git clone https://github.com/electrum/ssb-dbgen.git
[root@xxxxx test]# cd ssb-dbgen
[root@xxxxx ssb-dbgen]# make
```

## 生成数据

**生成6亿数据**

```shell
[root@xxxxx ssb-dbgen]# ./dbgen -s 100 -T c
[root@xxxxx ssb-dbgen]# ./dbgen -s 100 -T l
[root@xxxxx ssb-dbgen]# ./dbgen -s 100 -T p
[root@xxxxx ssb-dbgen]# ./dbgen -s 100 -T s
[root@xxxxx ssb-dbgen]# ./dbgen -s 100 -T d
```

| 表名           | 行数             | 大小  | 描述                 |
| -------------- | ---------------- | ----- | -------------------- |
| lineorder      | 6亿（600037902） | 67.1G | 商品订单表           |
| customer       | 300万（3000000） | 317M  | 客户表               |
| part           | 140万（1400000） | 135M  | 零部件表             |
| supplier       | 20万（200000）   | 19M   | 供应商表             |
| date           | 2556             | 272K  | 日期表               |
| lineorder_flat | 6亿（600037902） | 228G  | 打平后的宽表（单表） |

**生成30亿数据**

```shell
[root@xxxxx ssb-dbgen]# ./dbgen -s 500 -T c
[root@xxxxx ssb-dbgen]# ./dbgen -s 500 -T l
[root@xxxxx ssb-dbgen]# ./dbgen -s 500 -T p
[root@xxxxx ssb-dbgen]# ./dbgen -s 500 -T s
[root@xxxxx ssb-dbgen]# ./dbgen -s 500 -T d
```

| 表名           | 行数               | 大小 | 描述                 |
| -------------- | ------------------ | ---- | -------------------- |
| lineorder      | 30亿（3000028242） | 347G | 商品订单表           |
| customer       | 1500万（15000000） | 1.6G | 客户表               |
| part           | 180万（1800000）   | 173M | 零部件表             |
| supplier       | 100万（1000000）   | 94M  | 供应商表             |
| date           | 2556               | 272K | 日期表               |
| lineorder_flat | 30亿（3000346799） | 1.2T | 打平后的宽表（单表） |

**生成60亿数据**

```shell
[root@xxxxx ssb-dbgen]# ./dbgen -s 1000 -T c
[root@xxxxx ssb-dbgen]# ./dbgen -s 1000 -T l
[root@xxxxx ssb-dbgen]# ./dbgen -s 1000 -T p
[root@xxxxx ssb-dbgen]# ./dbgen -s 1000 -T s
[root@xxxxx ssb-dbgen]# ./dbgen -s 1000 -T d
```

| 表名           | 行数               | 大小 | 描述                 |
| -------------- | ------------------ | ---- | -------------------- |
| lineorder      | 60亿（5999989709） | 688G | 商品订单表           |
| customer       | 3000万(30000000)   | 3.2G | 客户表               |
| part           | 200万（2000000）   | 193M | 零部件表             |
| supplier       | 200万（2000000）   | 188M | 供应商表             |
| date           | 2556               | 272K | 日期表               |
| lineorder_flat | 60亿（5999989709)  | 2.3T | 打平后的宽表（单表） |

**特别说明：**

​	如果数据量生成较大的话，**dbgen**命令支持分割文件，指定 -C 参数，即线程数。线程越大生成数据越快，建议数据量较大时并且在机器条件允许的情况下指定较大的核数。例如以下命令生成千亿级别的数据 -C 参数指定为32线程，以上数据示例也可以按需指定 -C 参数，-C 指定的数字即为分割文件的个数。

```shell
[root@xxxxx ssb-dbgen]# ./dbgen -C 32 -s 17500 -T c
[root@xxxxx ssb-dbgen]# ./dbgen -C 32 -s 17500 -T l
[root@xxxxx ssb-dbgen]# ./dbgen -C 32 -s 17500 -T p
[root@xxxxx ssb-dbgen]# ./dbgen -C 32 -s 17500 -T s
[root@xxxxx ssb-dbgen]# ./dbgen -C 32 -s 17500 -T d
```

## 云数据仓库UClickhouse测试

### 单副本建库建表DDL

#### 多表测试DDL

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
) ENGINE = MergeTree PARTITION BY toYear(LO_ORDERDATE) % 100 ORDER BY (LO_ORDERDATE, LO_ORDERKEY);
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

### 双副本建库建表DDL

#### 多表测试DDL

```sql
#创建数据库
CREATE DATABASE IF NOT EXISTS ssb_test on cluster ck_cluster;
#创建 customer 本地表
CREATE TABLE IF NOT EXISTS ssb_test.customer_local on cluster ck_cluster
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
ENGINE = ReplicatedMergeTree(
             '/clickhouse/ssb_test/tables/{layer}-{shard}/customer',
             '{replica}'
) ORDER BY (C_CUSTKEY);
#创建 customer 分布式表
CREATE TABLE IF NOT EXISTS ssb_test.customer on cluster ck_cluster AS ssb_test.customer_local  ENGINE = Distributed(
       ck_cluster,
       ssb_test,
       customer_local,
       rand()
);
#创建 lineorder 本地表
CREATE TABLE IF NOT EXISTS ssb_test.lineorder_local on cluster ck_cluster
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
) ENGINE = ReplicatedMergeTree(
             '/clickhouse/ssb_test/tables/{layer}-{shard}/lineorder',
             '{replica}'
        ) PARTITION BY toYear(LO_ORDERDATE) ORDER BY (LO_ORDERDATE, LO_ORDERKEY);
#创建 lineorder 分布式表
CREATE TABLE IF NOT EXISTS ssb_test.lineorder on cluster ck_cluster AS ssb_test.lineorder_local  ENGINE = Distributed(
       ck_cluster,
       ssb_test,
       lineorder_local,
       rand()
);
#创建 part 本地表
CREATE TABLE IF NOT EXISTS ssb_test.part_local on cluster ck_cluster
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
) ENGINE = ReplicatedMergeTree(
             '/clickhouse/ssb_test/tables/{layer}-{shard}/part',
             '{replica}') ORDER BY P_PARTKEY;
#创建 part 分布式表
CREATE TABLE IF NOT EXISTS ssb_test.part on cluster ck_cluster AS ssb_test.part_local  ENGINE = Distributed(
      ck_cluster,
      ssb_test,
      part_local,
      rand()
);
#创建 supplier 本地表
CREATE TABLE IF NOT EXISTS ssb_test.supplier_local on cluster ck_cluster
(
    S_SUPPKEY UInt32,
    S_NAME    String,
    S_ADDRESS String,
    S_CITY    LowCardinality(String),
    S_NATION  LowCardinality(String),
    S_REGION  LowCardinality(String),
    S_PHONE   String
) ENGINE = ReplicatedMergeTree(
             '/clickhouse/ssb_test/tables/{layer}-{shard}/supplier',
             '{replica}'
) ORDER BY S_SUPPKEY;
#创建 supplier 分布式表
CREATE TABLE IF NOT EXISTS ssb_test.supplier on cluster ck_cluster AS ssb_test.supplier_local  ENGINE = Distributed(
       ck_cluster,
       ssb_test,
       supplier_local,
       rand()
);
#创建 dates 本地表
CREATE TABLE IF NOT EXISTS ssb_test.dates_local on cluster ck_cluster
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
)  ENGINE = ReplicatedMergeTree(
             '/clickhouse/ssb_test/tables/{layer}-{shard}/dates',
             '{replica}'
) ORDER BY D_DATEKEY;
#创建 dates 分布式表
CREATE TABLE IF NOT EXISTS ssb_test.dates on cluster ck_cluster AS ssb_test.dates_local  ENGINE = Distributed(
      ck_cluster,
      ssb_test,
      dates_local,
      rand()
);
```

## 写入数据

```shell
[root@xxxxx ssb-dbgen]# clickhouse-client --host=<云数据仓库UClickhouse节点地址> --port=9000 --user=admin            --password=<建立集群时设置的管理密码> --database=ssb_test --query "INSERT INTO customer FORMAT CSV" < customer.tbl
[root@xxxxx ssb-dbgen]# clickhouse-client --host=<云数据仓库UClickhouse节点地址> --port=9000 --user=admin            --password=<建立集群时设置的管理密码> --database=ssb_test --query "INSERT INTO supplier FORMAT CSV" < supplier.tbl
[root@xxxxx ssb-dbgen]# clickhouse-client --host=<云数据仓库UClickhouse节点地址> --port=9000 --user=admin            --password=<建立集群时设置的管理密码> --database=ssb_test --query "INSERT INTO part FORMAT CSV" < part.tbl
[root@xxxxx ssb-dbgen]# clickhouse-client --host=<云数据仓库UClickhouse节点地址> --port=9000 --user=admin            --password=<建立集群时设置的管理密码> --database=ssb_test --query "INSERT INTO lineorder FORMAT CSV" < lineorder.tbl
[root@xxxxx ssb-dbgen]# clickhouse-client --host=<云数据仓库UClickhouse节点地址> --port=9000 --user=admin            --password=<建立集群时设置的管理密码> --database=ssb_test --query "INSERT INTO dates FORMAT CSV" < date.tbl
```

当以上五张表数据导入完成后，才能执行单表的创建及数据填充：

**双副本单表测试DDL**

```sql
CREATE TABLE IF NOT EXISTS ssb_test.lineorder_flat on cluster ck_cluster
    ENGINE = MergeTree
    PARTITION BY toYear(LO_ORDERDATE)
    ORDER BY (LO_ORDERDATE, LO_ORDERKEY) AS
SELECT
    l.LO_ORDERKEY AS LO_ORDERKEY,
    l.LO_LINENUMBER AS LO_LINENUMBER,
    l.LO_CUSTKEY AS LO_CUSTKEY,
    l.LO_PARTKEY AS LO_PARTKEY,
    l.LO_SUPPKEY AS LO_SUPPKEY,
    l.LO_ORDERDATE AS LO_ORDERDATE,
    l.LO_ORDERPRIORITY AS LO_ORDERPRIORITY,
    l.LO_SHIPPRIORITY AS LO_SHIPPRIORITY,
    l.LO_QUANTITY AS LO_QUANTITY,
    l.LO_EXTENDEDPRICE AS LO_EXTENDEDPRICE,
    l.LO_ORDTOTALPRICE AS LO_ORDTOTALPRICE,
    l.LO_DISCOUNT AS LO_DISCOUNT,
    l.LO_REVENUE AS LO_REVENUE,
    l.LO_SUPPLYCOST AS LO_SUPPLYCOST,
    l.LO_TAX AS LO_TAX,
    l.LO_COMMITDATE AS LO_COMMITDATE,
    l.LO_SHIPMODE AS LO_SHIPMODE,
    c.C_NAME AS C_NAME,
    c.C_ADDRESS AS C_ADDRESS,
    c.C_CITY AS C_CITY,
    c.C_NATION AS C_NATION,
    c.C_REGION AS C_REGION,
    c.C_PHONE AS C_PHONE,
    c.C_MKTSEGMENT AS C_MKTSEGMENT,
    s.S_NAME AS S_NAME,
    s.S_ADDRESS AS S_ADDRESS,
    s.S_CITY AS S_CITY,
    s.S_NATION AS S_NATION,
    s.S_REGION AS S_REGION,
    s.S_PHONE AS S_PHONE,
    p.P_NAME AS P_NAME,
    p.P_MFGR AS P_MFGR,
    p.P_CATEGORY AS P_CATEGORY,
    p.P_BRAND AS P_BRAND,
    p.P_COLOR AS P_COLOR,
    p.P_TYPE AS P_TYPE,
    p.P_SIZE AS P_SIZE,
    p.P_CONTAINER AS P_CONTAINER
FROM ssb_test.lineorder AS l
         INNER JOIN ssb_test.customer AS c ON c.C_CUSTKEY = l.LO_CUSTKEY
         INNER JOIN ssb_test.supplier AS s ON s.S_SUPPKEY = l.LO_SUPPKEY
         INNER JOIN ssb_test.part AS p ON p.P_PARTKEY = l.LO_PARTKEY;
```

**单副本测试DDL**

```sql
CREATE TABLE IF NOT EXISTS ssb_test.lineorder_flat  
    ENGINE = MergeTree
    PARTITION BY toYear(LO_ORDERDATE)
    ORDER BY (LO_ORDERDATE, LO_ORDERKEY) AS
SELECT
    l.LO_ORDERKEY AS LO_ORDERKEY,
    l.LO_LINENUMBER AS LO_LINENUMBER,
    l.LO_CUSTKEY AS LO_CUSTKEY,
    l.LO_PARTKEY AS LO_PARTKEY,
    l.LO_SUPPKEY AS LO_SUPPKEY,
    l.LO_ORDERDATE AS LO_ORDERDATE,
    l.LO_ORDERPRIORITY AS LO_ORDERPRIORITY,
    l.LO_SHIPPRIORITY AS LO_SHIPPRIORITY,
    l.LO_QUANTITY AS LO_QUANTITY,
    l.LO_EXTENDEDPRICE AS LO_EXTENDEDPRICE,
    l.LO_ORDTOTALPRICE AS LO_ORDTOTALPRICE,
    l.LO_DISCOUNT AS LO_DISCOUNT,
    l.LO_REVENUE AS LO_REVENUE,
    l.LO_SUPPLYCOST AS LO_SUPPLYCOST,
    l.LO_TAX AS LO_TAX,
    l.LO_COMMITDATE AS LO_COMMITDATE,
    l.LO_SHIPMODE AS LO_SHIPMODE,
    c.C_NAME AS C_NAME,
    c.C_ADDRESS AS C_ADDRESS,
    c.C_CITY AS C_CITY,
    c.C_NATION AS C_NATION,
    c.C_REGION AS C_REGION,
    c.C_PHONE AS C_PHONE,
    c.C_MKTSEGMENT AS C_MKTSEGMENT,
    s.S_NAME AS S_NAME,
    s.S_ADDRESS AS S_ADDRESS,
    s.S_CITY AS S_CITY,
    s.S_NATION AS S_NATION,
    s.S_REGION AS S_REGION,
    s.S_PHONE AS S_PHONE,
    p.P_NAME AS P_NAME,
    p.P_MFGR AS P_MFGR,
    p.P_CATEGORY AS P_CATEGORY,
    p.P_BRAND AS P_BRAND,
    p.P_COLOR AS P_COLOR,
    p.P_TYPE AS P_TYPE,
    p.P_SIZE AS P_SIZE,
    p.P_CONTAINER AS P_CONTAINER
FROM ssb_test.lineorder AS l
         INNER JOIN ssb_test.customer AS c ON c.C_CUSTKEY = l.LO_CUSTKEY
         INNER JOIN ssb_test.supplier AS s ON s.S_SUPPKEY = l.LO_SUPPKEY
         INNER JOIN ssb_test.part AS p ON p.P_PARTKEY = l.LO_PARTKEY;
```



## 运行查询

### 多表测试DML

- #### Q1.1

  ```sql
  SELECT SUM(LO_REVENUE) AS REVENUE
  FROM lineorder
  GLOBAL INNER JOIN dates ON LO_ORDERDATE = toDate(replaceRegexpAll(toString(D_DATEKEY), '(\\d{4})(\\d{2})(\\d{2})', '\\1-\\2-\\3'))
  WHERE (D_YEAR = 1993) AND ((LO_DISCOUNT >= 1) AND (LO_DISCOUNT <= 3)) AND (LO_QUANTITY < 25)
  ```

- #### Q1.2

  ```sql
  SELECT SUM(LO_REVENUE) AS REVENUE
  FROM lineorder
  GLOBAL INNER JOIN dates ON LO_ORDERDATE = toDate(replaceRegexpAll(toString(D_DATEKEY), '(\\d{4})(\\d{2})(\\d{2})', '\\1-\\2-\\3'))
  WHERE (D_YEARMONTHNUM = 199401) AND ((LO_DISCOUNT >= 4) AND (LO_DISCOUNT <= 6)) AND ((LO_QUANTITY >= 26) AND (LO_QUANTITY <= 35))
  ```

- #### Q1.3

  ```sql
  SELECT SUM(LO_REVENUE) AS REVENUE
  FROM lineorder
  GLOBAL INNER JOIN dates ON LO_ORDERDATE = toDate(replaceRegexpAll(toString(D_DATEKEY), '(\\d{4})(\\d{2})(\\d{2})', '\\1-\\2-\\3'))
  WHERE (D_WEEKNUMINYEAR = 6) AND (D_YEAR = 1994) AND ((LO_DISCOUNT >= 5) AND (LO_DISCOUNT <= 7)) AND ((LO_QUANTITY >= 26) AND (LO_QUANTITY <= 35))
  ```

- #### Q2.1

  ```sql
  SELECT SUM(LO_REVENUE) AS LO_REVENUE, D_YEAR, P_BRAND
  FROM lineorder
  GLOBAL JOIN part ON LO_PARTKEY = P_PARTKEY
  GLOBAL JOIN supplier ON LO_SUPPKEY = S_SUPPKEY
  GLOBAL INNER JOIN dates ON LO_ORDERDATE = toDate(replaceRegexpAll(toString(D_DATEKEY),'(\\d{4})(\\d{2})(\\d{2})', '\\1-\\2-\\3'))
  WHERE P_CATEGORY = 'MFGR#12' AND S_REGION = 'AMERICA'
  GROUP BY D_YEAR, P_BRAND
  ORDER BY D_YEAR, P_BRAND
  ```

- #### Q2.2

  ```sql
  SELECT SUM(LO_REVENUE) AS LO_REVENUE, D_YEAR, P_BRAND
  FROM lineorder
  GLOBAL JOIN part ON LO_PARTKEY = P_PARTKEY
  GLOBAL JOIN supplier ON LO_SUPPKEY = S_SUPPKEY
  GLOBAL JOIN dates ON LO_ORDERDATE = toDate(replaceRegexpAll(toString(D_DATEKEY),'(\\d{4})(\\d{2})(\\d{2})', '\\1-\\2-\\3'))
  WHERE P_BRAND BETWEEN 'MFGR#2221' AND 'MFGR#2228' AND S_REGION = 'ASIA'
  GROUP BY D_YEAR, P_BRAND
  ORDER BY D_YEAR, P_BRAND
  ```

- #### Q2.3

  ```sql
  SELECT
      SUM(LO_REVENUE) AS LO_REVENUE,
      D_YEAR,
      P_BRAND
  FROM lineorder
  GLOBAL INNER JOIN part ON LO_PARTKEY = P_PARTKEY
  GLOBAL INNER JOIN supplier ON LO_SUPPKEY = S_SUPPKEY
  GLOBAL INNER JOIN dates ON LO_ORDERDATE = toDate(replaceRegexpAll(toString(D_DATEKEY), '(\\d{4})(\\d{2})(\\d{2})', '\\1-\\2-\\3'))
  WHERE (P_BRAND = 'MFGR#2239') AND (S_REGION = 'EUROPE')
  GROUP BY
      D_YEAR,
      P_BRAND
  ORDER BY
      D_YEAR ASC,
      P_BRAND ASC
  ```

- #### Q3.1

  ```sql
  SELECT C_NATION, S_NATION, D_YEAR, SUM(LO_REVENUE) AS LO_REVENUE
  FROM lineorder
  GLOBAL JOIN customer ON LO_CUSTKEY = C_CUSTKEY
  GLOBAL JOIN supplier ON LO_SUPPKEY = S_SUPPKEY
  GLOBAL JOIN dates ON LO_ORDERDATE = toDate(replaceRegexpAll(toString(D_DATEKEY),'(\\d{4})(\\d{2})(\\d{2})', '\\1-\\2-\\3'))
  WHERE C_REGION = 'ASIA' AND S_REGION = 'ASIA'AND D_YEAR >= 1992 AND D_YEAR <= 1997
  GROUP BY C_NATION, S_NATION, D_YEAR
  ORDER BY D_YEAR ASC, LO_REVENUE DESC
  ```

- #### Q3.2

  ```sql
  SELECT C_CITY, S_CITY, D_YEAR, SUM(LO_REVENUE) AS LO_REVENUE
  FROM lineorder
  GLOBAL JOIN customer ON LO_CUSTKEY = C_CUSTKEY
  GLOBAL JOIN supplier ON LO_SUPPKEY = S_SUPPKEY
  GLOBAL JOIN dates ON LO_ORDERDATE = toDate(replaceRegexpAll(toString(D_DATEKEY),'(\\d{4})(\\d{2})(\\d{2})', '\\1-\\2-\\3'))
  WHERE C_NATION = 'UNITED STATES' AND S_NATION = 'UNITED STATES'
  AND D_YEAR >= 1992 AND D_YEAR <= 1997
  GROUP BY C_CITY, S_CITY, D_YEAR
  ORDER BY D_YEAR ASC, LO_REVENUE DESC
  ```

- #### Q3.3

  ```sql
  SELECT
      C_CITY,
      S_CITY,
      D_YEAR,
      SUM(LO_REVENUE) AS LO_REVENUE
  FROM lineorder
  GLOBAL INNER JOIN customer ON LO_CUSTKEY = C_CUSTKEY
  GLOBAL INNER JOIN supplier ON LO_SUPPKEY = S_SUPPKEY
  GLOBAL INNER JOIN dates ON LO_ORDERDATE = toDate(replaceRegexpAll(toString(D_DATEKEY), '(\\d{4})(\\d{2})(\\d{2})', '\\1-\\2-\\3'))
  WHERE ((C_CITY = 'UNITED KI1') OR (C_CITY = 'UNITED KI5')) AND ((S_CITY = 'UNITED KI1') OR (S_CITY = 'UNITED KI5')) AND (D_YEAR >= 1992) AND (D_YEAR <= 1997)
  GROUP BY
      C_CITY,
      S_CITY,
      D_YEAR
  ORDER BY
      D_YEAR ASC,
      LO_REVENUE DESC
  ```

- #### Q3.4

  ```sql
  SELECT
      C_CITY,
      S_CITY,
      D_YEAR,
      SUM(LO_REVENUE) AS LO_REVENUE
  FROM lineorder
  GLOBAL INNER JOIN customer ON LO_CUSTKEY = C_CUSTKEY
  GLOBAL INNER JOIN supplier ON LO_SUPPKEY = S_SUPPKEY
  GLOBAL INNER JOIN dates ON LO_ORDERDATE = toDate(replaceRegexpAll(toString(D_DATEKEY), '(\\d{4})(\\d{2})(\\d{2})', '\\1-\\2-\\3'))
  WHERE ((C_CITY = 'UNITED KI1') OR (C_CITY = 'UNITED KI5')) AND ((S_CITY = 'UNITED KI1') OR (S_CITY = 'UNITED KI5')) AND (D_YEARMONTH = 'Dec1997')
  GROUP BY
      C_CITY,
      S_CITY,
      D_YEAR
  ORDER BY
      D_YEAR ASC,
      LO_REVENUE DESC
  ```

- #### Q4.1

  ```sql
  SELECT
      D_YEAR,
      C_NATION,
      SUM(LO_REVENUE) - SUM(LO_SUPPLYCOST) AS PROFIT
  FROM lineorder
  GLOBAL INNER JOIN customer ON LO_CUSTKEY = C_CUSTKEY
  GLOBAL INNER JOIN supplier ON LO_SUPPKEY = S_SUPPKEY
  GLOBAL INNER JOIN part ON LO_PARTKEY = P_PARTKEY
  GLOBAL INNER JOIN dates ON LO_ORDERDATE = toDate(replaceRegexpAll(toString(D_DATEKEY), '(\\d{4})(\\d{2})(\\d{2})', '\\1-\\2-\\3'))
  WHERE (C_REGION = 'AMERICA') AND (S_REGION = 'AMERICA') AND ((P_MFGR = 'MFGR#1') OR (P_MFGR = 'MFGR#2'))
  GROUP BY
      D_YEAR,
      C_NATION
  ORDER BY
      D_YEAR ASC,
      C_NATION ASC
  ```

- #### Q4.2

  ```sql
  SELECT
      D_YEAR,
      S_NATION,
      P_CATEGORY,
      SUM(LO_REVENUE) - SUM(LO_SUPPLYCOST) AS PROFIT
  FROM lineorder
  GLOBAL INNER JOIN customer ON LO_CUSTKEY = C_CUSTKEY
  GLOBAL INNER JOIN supplier ON LO_SUPPKEY = S_SUPPKEY
  GLOBAL INNER JOIN part ON LO_PARTKEY = P_PARTKEY
  GLOBAL INNER JOIN dates ON LO_ORDERDATE = toDate(replaceRegexpAll(toString(D_DATEKEY), '(\\d{4})(\\d{2})(\\d{2})', '\\1-\\2-\\3'))
  WHERE (C_REGION = 'AMERICA') AND (S_REGION = 'AMERICA') AND ((D_YEAR = 1997) OR (D_YEAR = 1998)) AND ((P_MFGR = 'MFGR#1') OR (P_MFGR = 'MFGR#2'))
  GROUP BY
      D_YEAR,
      S_NATION,
      P_CATEGORY
  ORDER BY
      D_YEAR ASC,
      S_NATION ASC,
      P_CATEGORY ASC
  ```

- #### Q4.3

  ```sql
  SELECT
      D_YEAR,
      S_CITY,
      P_BRAND,
      SUM(LO_REVENUE) - SUM(LO_SUPPLYCOST) AS PROFIT
  FROM lineorder
  GLOBAL INNER JOIN customer ON LO_CUSTKEY = C_CUSTKEY
  GLOBAL INNER JOIN supplier ON LO_SUPPKEY = S_SUPPKEY
  GLOBAL INNER JOIN part ON LO_PARTKEY = P_PARTKEY
  GLOBAL INNER JOIN dates ON LO_ORDERDATE = toDate(replaceRegexpAll(toString(D_DATEKEY), '(\\d{4})(\\d{2})(\\d{2})', '\\1-\\2-\\3'))
  WHERE (C_REGION = 'AMERICA') AND (S_NATION = 'UNITED STATES') AND ((D_YEAR = 1997) OR (D_YEAR = 1998)) AND (P_CATEGORY = 'MFGR#14')
  GROUP BY
      D_YEAR,
      S_CITY,
      P_BRAND
  ORDER BY
      D_YEAR ASC,
      S_CITY ASC,
      P_BRAND ASC
  ```

### 单表测试DML

- Q1.1

  ```sql
  SELECT sum(LO_EXTENDEDPRICE * LO_DISCOUNT) AS revenue
  FROM lineorder_flat
  WHERE toYear(LO_ORDERDATE) = 1993 AND LO_DISCOUNT BETWEEN 1 AND 3 AND LO_QUANTITY < 25;
  ```

- Q1.2

  ```sql
  SELECT sum(LO_EXTENDEDPRICE * LO_DISCOUNT) AS revenue
  FROM lineorder_flat
  WHERE toYYYYMM(LO_ORDERDATE) = 199401 AND LO_DISCOUNT BETWEEN 4 AND 6 AND LO_QUANTITY BETWEEN 26 AND 35;
  ```

- Q1.3

  ```sql
  SELECT sum(LO_EXTENDEDPRICE * LO_DISCOUNT) AS revenue
  FROM lineorder_flat
  WHERE toISOWeek(LO_ORDERDATE) = 6 AND toYear(LO_ORDERDATE) = 1994
    AND LO_DISCOUNT BETWEEN 5 AND 7 AND LO_QUANTITY BETWEEN 26 AND 35;
  ```

- Q2.1

  ```sql
  SELECT
      sum(LO_REVENUE),
      toYear(LO_ORDERDATE) AS year,
      P_BRAND
  FROM lineorder_flat
  WHERE P_CATEGORY = 'MFGR#12' AND S_REGION = 'AMERICA'
  GROUP BY
      year,
      P_BRAND
  ORDER BY
      year,
      P_BRAND;
  ```

- Q2.2

  ```sql
  SELECT
      sum(LO_REVENUE),
      toYear(LO_ORDERDATE) AS year,
      P_BRAND
  FROM lineorder_flat
  WHERE P_BRAND >= 'MFGR#2221' AND P_BRAND <= 'MFGR#2228' AND S_REGION = 'ASIA'
  GROUP BY
      year,
      P_BRAND
  ORDER BY
      year,
      P_BRAND;
  ```

- Q2.3

  ```sql
  SELECT
      sum(LO_REVENUE),
      toYear(LO_ORDERDATE) AS year,
      P_BRAND
  FROM lineorder_flat
  WHERE P_BRAND = 'MFGR#2239' AND S_REGION = 'EUROPE'
  GROUP BY
      year,
      P_BRAND
  ORDER BY
      year,
      P_BRAND;
  ```

- Q3.1

  ```sql
  SELECT
      C_NATION,
      S_NATION,
      toYear(LO_ORDERDATE) AS year,
      sum(LO_REVENUE) AS revenue
  FROM lineorder_flat
  WHERE C_REGION = 'ASIA' AND S_REGION = 'ASIA' AND year >= 1992 AND year <= 1997
  GROUP BY
      C_NATION,
      S_NATION,
      year
  ORDER BY
      year ASC,
      revenue DESC;
  ```

- Q3.2

  ```sql
  SELECT
      C_CITY,
      S_CITY,
      toYear(LO_ORDERDATE) AS year,
      sum(LO_REVENUE) AS revenue
  FROM lineorder_flat
  WHERE C_NATION = 'UNITED STATES' AND S_NATION = 'UNITED STATES' AND year >= 1992 AND year <= 1997
  GROUP BY
      C_CITY,
      S_CITY,
      year
  ORDER BY
      year ASC,
      revenue DESC;
  ```

- Q3.3

  ```sql
  SELECT
      C_CITY,
      S_CITY,
      toYear(LO_ORDERDATE) AS year,
      sum(LO_REVENUE) AS revenue
  FROM lineorder_flat
  WHERE (C_CITY = 'UNITED KI1' OR C_CITY = 'UNITED KI5') AND (S_CITY = 'UNITED KI1' OR S_CITY = 'UNITED KI5') AND year >= 1992 AND year <= 1997
  GROUP BY
      C_CITY,
      S_CITY,
      year
  ORDER BY
      year ASC,
      revenue DESC;
  ```

- Q3.4

  ```sql
  SELECT
      C_CITY,
      S_CITY,
      toYear(LO_ORDERDATE) AS year,
      sum(LO_REVENUE) AS revenue
  FROM lineorder_flat
  WHERE (C_CITY = 'UNITED KI1' OR C_CITY = 'UNITED KI5') AND (S_CITY = 'UNITED KI1' OR S_CITY = 'UNITED KI5') AND toYYYYMM(LO_ORDERDATE) = 199712
  GROUP BY
      C_CITY,
      S_CITY,
      year
  ORDER BY
      year ASC,
      revenue DESC;
  ```

- Q4.1

  ```sql
  SELECT
      toYear(LO_ORDERDATE) AS year,
      C_NATION,
      sum(LO_REVENUE - LO_SUPPLYCOST) AS profit
  FROM lineorder_flat
  WHERE C_REGION = 'AMERICA' AND S_REGION = 'AMERICA' AND (P_MFGR = 'MFGR#1' OR P_MFGR = 'MFGR#2')
  GROUP BY
      year,
      C_NATION
  ORDER BY
      year ASC,
      C_NATION ASC;
  ```

- Q4.2

  ```sql
  SELECT
      toYear(LO_ORDERDATE) AS year,
      S_NATION,
      P_CATEGORY,
      sum(LO_REVENUE - LO_SUPPLYCOST) AS profit
  FROM lineorder_flat
  WHERE C_REGION = 'AMERICA' AND S_REGION = 'AMERICA' AND (year = 1997 OR year = 1998) AND (P_MFGR = 'MFGR#1' OR P_MFGR = 'MFGR#2')
  GROUP BY
      year,
      S_NATION,
      P_CATEGORY
  ORDER BY
      year ASC,
      S_NATION ASC,
      P_CATEGORY ASC;
  ```

- Q4.3

  ```sql
  SELECT
      toYear(LO_ORDERDATE) AS year,
      S_CITY,
      P_BRAND,
      sum(LO_REVENUE - LO_SUPPLYCOST) AS profit
  FROM lineorder_flat
  WHERE S_NATION = 'UNITED STATES' AND (year = 1997 OR year = 1998) AND P_CATEGORY = 'MFGR#14'
  GROUP BY
      year,
      S_CITY,
      P_BRAND
  ORDER BY
      year ASC,
      S_CITY ASC,
      P_BRAND ASC;
  ```

  

