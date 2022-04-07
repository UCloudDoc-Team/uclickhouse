# 创建物化视图（Materialized View）

**基本语法：**

```
CREATE MATERIALIZED VIEW [IF NOT EXISTS] [db.]Materialized_name [TO[db.]name] [ON CLUSTER cluster] 
ENGINE = engine_name()
ORDER BY expr 
[POPULATE] 
AS SELECT ...
```

**参数说明：**

<table>
    <thead>
        <tr>
            <th>参数</th>
            <th>说明</th>
        </tr>
    </thead>
    <tbody>
    <tr>
        <td><code>db</code></td>
        <td>数据库的名称，默认为当前选择的数据库，本文以ck_cluster为例。</td>
    </tr>
    <tr>
        <td><code>Materialized_name</code></td>
        <td>物化视图名。</td>
    </tr>
    <tr>
        <td><code>TO[db.]name</code></td>
        <td>将物化视图的数据写入到新表中。
           如果需要将物化视图的数据写入新表，不能使用<code>POPULATE</code>关键字。
        </td>
    </tr>
    <tr>
        <td><code>[ON CLUSTER cluster]</code></td>
        <td>在每一个节点上都创建一个物化视图，固定为
            <code>ON CLUSTER ck_cluster</code>。
        </td>
    </tr>
    <tr>
        <td><code>ENGINE = engine_name()</code></td>
        <td>
            表引擎类型，具体请参见<span><a href="/uclickhouse/developer/table_engine">表引擎</a></span>。
        </td>
    </tr>
    <tr>
        <td><code>[POPULATE]</code></td>
        <td><code>POPULATE</code>关键字。如果创建物化视图时指定了
            <code>POPULATE</code>关键字，则在创建时将
            <code>SELECT</code>子句所指定的源表数据插入到物化视图中。不指定
            <code>POPULATE</code>关键字时，物化视图只会包含在物化视图创建后新写入源表的数据。
           一般不推荐使用<code>POPULATE</code>关键字，因为在物化视图创建期间写入源表的数据将不会写入物化视图中。
        </td>
    </tr>
    <tr>
        <td><code>SELECT ...</code></td>
        <td>
            <code>SELECT</code>
            子句。当数据写入物化视图中
            <code>SELECT</code>子句所指定的源表时，插入的数据会通过
            <code>SELECT</code>子句查询进行转换并将最终结果插入到物化视图中。
             <code>SELECT</code>
                查询可以包含
                <code>DISTINCT</code>、
                <code>GROUP BY</code>、
                <code>ORDER BY</code>和
                <code>LIMIT</code>等，但是相应的转换是在每个插入数据块上独立执行的。
        </td>
    </tr>
    </tbody>
</table>

示例：

1. 创建SELECT子句指定的源表。

   ```sql
   CREATE TABLE ck_test.lineorder_local on cluster ck_cluster
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
   ) PARTITION BY toYear(LO_ORDERDATE) ORDER BY (LO_ORDERDATE, LO_ORDERKEY)
   ```

2. 写入数据至源表。

   参考[快速上手](/uclickhouse/gettingstart)

3. 创建基于源表的物化视图。

   ```sql
   CREATE MATERIALIZED VIEW lineorder_view ON CLUSTER ck_cluster
   ENGINE = MergeTree()
   ORDER BY (id) AS SELECT * FROM lineorder_local;
   ```

4. 查询物化视图，验证未指定POPULATE关键字时，是否能查询到物化视图创建前写入源表的数据。

   ```sql
   SELECT * FROM lineorder_view;
   ```

   查询数据为空，说明未指定`POPULATE`关键字时，查询不到物化视图创建前写入源表的数据。

5. 写入数据至源表。

   参考[快速上手](/uclickhouse/gettingstart)

6. 查询物化视图。

   ```sql
   SELECT * FROM lineorder_view;
   ```

## 官方文档

创建物化视图的更多信息，请参见[Create Materialized View](https://clickhouse.com/docs/zh/sql-reference/statements/create/view/#materialized)。