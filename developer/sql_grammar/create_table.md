# 建表（Create Table）

- ### 创建本地表

**语法：**

```sql
CREATE TABLE [IF NOT EXISTS] [db.]local_table_name ON CLUSTER ck_cluster
(
    name1 [type1] [DEFAULT|MATERIALIZED|ALIAS expr1],
    name2 [type2] [DEFAULT|MATERIALIZED|ALIAS expr2],
    ...
    INDEX index_name1 expr1 TYPE type1(...) GRANULARITY value1,
    INDEX index_name2 expr2 TYPE type2(...) GRANULARITY value2
) ENGINE = engine_name()
[PARTITION BY expr]
ORDER BY expr
[PRIMARY KEY expr]
[SAMPLE BY expr]
[SETTINGS name=value, ...];
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
        <td>数据库的名称，默认为当前选择的数据库，本文以ck_test为例。</td>
    </tr>
    <tr>
        <td><code>local_table_name</code></td>
        <td>本地表名。</td>
    </tr>
    <tr>
        <td><code>ON CLUSTER ck_test</code></td>
        <td>在每一个节点上都创建一个本地表，固定为
            <code>ON CLUSTER ck_test</code>。
        </td>
    </tr>
    <tr>
        <td><code>name1,name2</code></td>
        <td>列名。</td>
    </tr>
    <tr>
        <td><code>type1,type2</code></td>
        <td>列类型。云数据仓库UClickhouse支持的数据类型，
            请参见<a title="" href="/uclickhouse/developer/data_type">数据类型</a></span>。
        </td>
    </tr>
    <tr>
        <td>
            <code>ENGINE = engine_name()</code>
        </td>
        <td>表引擎类型。
            <div>
                双副本版集群建表时，需要使用MergeTree系列引擎中支持数据复制的Replicated*引擎，否则副本之间不进行数据复制，导致数据查询结果不一致。使用该引擎建表时，参数填写方式如下。
                <ul>
                    <li>
                          ReplicatedMergeTree('/clickhouse/tables/{database}/{table}/{shard}','{replica}')，参数中的字符不允许修改。
                    </li>
                    <li>ReplicatedMergeTree()，等同于ReplicatedMergeTree('/clickhouse/tables/{database}/{table}/{shard}',
                        '{replica}')。
                    </li>
                </ul>
            </div>
          <div>
             云数据库ClickHouse支持的表引擎类型，请参见
            <a href="/uclickhouse/developer/table_engine">表引擎</a>。
        </div>
    </td>
</tr>
<tr>
    <td><code>ORDER BY expr</code></td>
    <td>排序键，必填项，可以是一组列的元组或任意表达式。</td>
</tr>
<tr>
    <td><code>[DEFAULT|MATERIALIZED|ALIAS expr]</code></td>
    <td>默认表达式。
        <ul>
            <li>DEFAULT：普通的默认表达式。在字段缺省的情况下，自动生成默认值并填充。</li>
            <li>MATERIALIZED：物化表达式。</li>
            <li>ALIAS：别名表达式。</li>
        </ul>
    </td>
</tr>
<tr>
    <td><code>GRANULARITY</code></td>
    <td>索引粒度参数。</td>
</tr>
<tr>
    <td>
        <code>[PARTITION BY expr]</code>
    </td>
    <td>分区键。一般按照日期分区，也可以使用其他字段或字段表达式。</td>
</tr>
<tr>
    <td><code>[PRIMARY KEY expr]</code></td>
    <td>主键，默认情况下主键和排序键相同。因此，多数情况下，不需要再专门使用
        <code>PRIMARY KEY</code>子句指定主键。
    </td>
</tr>
<tr>
    <td><code>[SAMPLE BY expr]</code></td>
    <td>抽样表达式。如果要使用抽样表达式，主键中必须包含这个表达式。</td>
</tr>
<tr>
    <td><code>[SETTINGS name=value, ...]</code></td>
    <td>影响性能的额外参数。
        <strong>提示</strong> 
        <code>SETTINGS</code>中支持的参数，请参见
        <span><a href="https://clickhouse.com/docs/zh/engines/table-engines/mergetree-family/mergetree/"
                target="_blank">SETTINGS配置项</a>
        </span>。
    </td>
</tr>
</tbody>
</table>

参数`ORDER BY`、`GRANULARITY`、`PARTITION BY`、`PRIMARY KEY`、`SAMPLE BY`和`[SETTINGS name=value, ...]`只有MergeTree系列表引擎支持。更多参数说明，请参见[CREATE TABLE](https://clickhouse.com/docs/zh/sql-reference/statements/create/table/)。

**示例：**

```sql
CREATE TABLE lineorder_local on cluster ck_cluster
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
ENGINE = MergeTree PARTITION BY toYear(LO_ORDERDATE) ORDER BY (LO_ORDERDATE, LO_ORDERKEY);
```

- ### 创建分布式表

分布式表是本地表的集合，它将多个本地表抽象为一张统一的表，对外提供写入、查询功能。当数据写入分布式表时，会被自动分发到集合中的各个本地表中。当查询分布式表时，集合中的各个本地表都会被分别查询，并且把最终结果汇总后返回。您需要先创建本地表，再创建分布式表。

**基本语法：**

```sql
CREATE TABLE [db.]distributed_table_name ON CLUSTER ck_cluster
 AS db.local_table_name ENGINE = Distributed(cluster, db, local_table_name [, sharding_key])
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
        <td><code>distributed_table_name</code></td>
        <td>分布式表名。</td>
    </tr>
    <tr>
        <td><code>ON CLUSTER ck_cluster</code></td>
        <td>在每一个节点上都创建一个表，固定为<code>ON CLUSTER ck_cluster</code>。
        </td>
    </tr>
    <tr>
        <td><code>local_table_name</code></td>
        <td>已创建的本地表名。</td>
    </tr>
    <tr>
        <td><code>sharding_key</code></td>
        <td>分片表达式，用于决定将数据写到哪个分片。
            <p>
                <code>sharding_key</code>
                可以是一个表达式，例如函数<code>rand()</code>，也可以是一列，例如
                <code>user_id</code>（Integer类型）。
            </p>
        </td>
    </tr>
    </tbody>
</table>

**示例：**

```
CREATE TABLE lineorder_distribute  AS lineorder on cluster ck_cluster
ENGINE = Distributed(
    ck_cluster,
    ch_test,
    lineorder_local,
    rand()
  );
```

- ### 通过复制表结构创建表

**基本语法：**

```sql
CREATE TABLE [IF NOT EXISTS] [db.]table_name2 ON CLUSTER ck_cluster AS [db.]table_name1 [ENGINE = engine_name];
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
        <td><code>table_name1</code></td>
        <td>被复制表结构的源表，本文以已创建的本地表local_table为例。</td>
    </tr>
    <tr>
        <td><code>table_name2</code></td>
        <td>新创建的表。</td>
    </tr>
    <tr>
        <td><code>ON CLUSTER ck_cluster</code></td>
        <td>在每一个节点上都创建一个表，固定为<code>ON CLUSTER ck_cluster</code>。
        </td>
    </tr>
    <tr>
        <td><code>[ENGINE = engine_name]</code></td>
        <td>表引擎类型。如果没有指定表引擎，默认与被复制表结构的表相同。
            <strong>说明</strong> <span >云数据仓库UClickHouse</span>支持的表引擎类型，请参见<span>
                <a href="/uclickhouse/developer/table_engine">表引擎</a>
            </span>。
        </td>
    </tr>
    </tbody>
</table>

**示例：**

```sql
create table lineorder ON CLUSTER ck_cluster as ck_test.lineorder_local;
```

- ### 通过SELECT语句创建表

**基本语法：**

```
CREATE TABLE [IF NOT EXISTS] [db.]table_name ON CLUSTER ck_cluster ENGINE = engine_name() AS SELECT ...
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
        <td><code>table_name</code></td>
        <td>通过SELECT语句创建的表。</td>
    </tr>
    <tr>
        <td><code>ON CLUSTER ck_cluster</code></td>
        <td>
            在每一个节点上都创建一个表，固定为<code>ON CLUSTER ck_cluster</code>。
        </td>
    </tr>
    <tr>
        <td><code>ENGINE = engine_name()</code></td>
        <td>表引擎类型。
             云数据仓库 UClickHouse</span>支持的表引擎类型，请参见<a href="/uclickhouse/developer/table_engine">表引擎</a>
        </td>
    </tr>
    <tr>
        <td><code>SELECT ...</code></td>
        <td><code>SELECT</code>子句。
        </td>
    </tr>
    </tbody>
</table>

**示例：**

```
create table lineorder ON CLUSTER ck_cluster ENGINE =MergeTree() order by Year as select * from ck_cluster.local_table;
```

### 官网参考文档

- 创建表的更多信息，请参见[CREATE TABLE](https://clickhouse.com/docs/zh/sql-reference/statements/create/table/)。
- 通过复制表结构创建表的更多信息，请参见[With a Schema Similar to Other Table](https://clickhouse.com/docs/zh/sql-reference/statements/create/table/#with-a-schema-similar-to-other-table)。
- 通过SELECT语句创建表的更多信息，请参见[From SELECT query](https://clickhouse.com/docs/zh/sql-reference/statements/create/table/#from-select-query)。