# 创建视图（Create View）

**基本语法：**

```sql
CREATE VIEW [IF NOT EXISTS] [db.]view_name [ON CLUSTER ck_cluster] AS SELECT ...
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
        <td><code>view_name</code></td>
        <td>视图名。</td>
    </tr>
    <tr>
        <td><code>[ON CLUSTER ck_cluster]</code></td>
        <td>
            在每一个节点上都创建一个视图，固定为<code>ON CLUSTER ck_cluster</code>。
        </td>
    </tr>
    <tr>
        <td><code>SELECT ...</code></td>
        <td><code>SELECT</code>子句。当数据写入视图中<code>SELECT</code>子句所指定的源表时，插入的数据会通过
            <code>SELECT</code>子句查询进行转换并将最终结果插入到视图中。
            <div>
                <code>SELECT</code>查询可以包含
                <code>DISTINCT</code>、<code>GROUP BY</code>、<code>ORDER BY</code>
                和<code>LIMIT</code>等，但是相应的转换是在每个插入数据块上独立执行的。
            </div>
        </td>
    </tr>
    </tbody>
</table>

**示例：**

1. 创建SELECT子句指定的源表。

```sql
create table lineorder ON CLUSTER ck_cluster 
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
) ENGINE = MergeTree PARTITION BY toYear(LO_ORDERDATE) ORDER BY (LO_ORDERDATE, LO_ORDERKEY);
```

2. 创建基于源表的视图。

```sql
CREATE VIEW lineorder_view ON CLUSTER ck_cluster AS SELECT * FROM lineorder;
```

3. 写入数据至源表。

  使用clickhouse-client导入，参考[快速上手](/uclickhouse/gettingstart)

4. 查询视图。

```sql
SELECT * FROM lineorder_view;
```

## 官网参考文档

创建视图的更多信息，请参见[Create View](https://clickhouse.com/docs/zh/sql-reference/statements/create/view/#normal)。