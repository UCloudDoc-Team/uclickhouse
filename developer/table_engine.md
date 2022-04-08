# 表引擎

表引擎（即表的类型）决定了：

- 数据的存储方式和位置，写到哪里以及从哪里读取数据
- 支持哪些查询以及如何支持
- 并发数据访问
- 索引的使用（如果存在）
- 是否可以执行多线程请求
- 数据复制参数

云数据仓库UClickHouse支持的表引擎分为MergeTree、Log、Integrations和Special四个系列大类。

参见下表：

<table>
    <thead>
        <tr>
            <th>系列</th>
            <th>描述</th>
            <th>表引擎</th>
            <th>特点</th>
        </tr>
    </thead>
    <tbody>
    <tr>
        <td rowspan="7">MergeTree</td>
        <td rowspan="7">
            MergeTree系列引擎适用于高负载任务，支持大数据量的快速写入并进行后续的数据处理，通用程度高且功能强大。
            该系列引擎的共同特点是支持数据副本、分区、数据采样等特性。
        </td>
        <td>MergeTree</td>
        <td>用于插入极大量的数据到一张表中，数据以数据片段的形式一个接着一个的快速写入，数据片段按照一定的规则进行合并。</td>
    </tr>
    <tr>
        <td>ReplacingMergeTree</td>
        <td>用于解决MergeTree表引擎相同主键无法去重的问题，可以删除主键值相同的重复项。</td>
    </tr>
    <tr>
        <td>CollapsingMergeTree</td>
        <td>在建表语句中新增标记列<code>Sign</code>
            ，用于消除ReplacingMergeTree表引擎的如下功能限制。
            <ul>
                <li>在分布式场景下，相同主键的数据可能被分布到不同节点上，不同分片间可能无法去重。</li>
                <li>在没有彻底optimize之前，可能无法达到主键去重的效果，比如部分数据已经被去重，而另外一部分数据仍旧有主键重复。</li>
                <li>optimize是后台动作，无法预测具体执行时间点。</li>
                <li>手动执行optimize在海量数据场景下需要消耗大量时间，无法满足业务即时查询的需求。</li>
            </ul>
        </td>
    </tr>
    <tr>
        <td>VersionedCollapsingMergeTree</td>
        <td>在建表语句中新增<code>Version</code>
            列，用于解决CollapsingMergeTree表引擎乱序写入导致无法正常折叠（删除）的问题。
        </td>
    </tr>
    <tr>
        <td>SummingMergeTree</td>
        <td>用于对主键列进行预先聚合，将所有相同主键的行合并为一行，从而大幅度降低存储空间占用，提升聚合计算性能。</td>
    </tr>
    <tr>
        <td>AggregatingMergeTree</td>
        <td>预先聚合引擎的一种，用于提升聚合计算的性能，可以指定各种聚合函数。</td>
    </tr>
    <tr>
        <td>GraphiteMergeTree</td>
        <td>用于存储Graphite数据并进行汇总，可以减少存储空间，提高Graphite数据的查询效率。</td>
    </tr>
    <tr>
        <td rowspan="3">Log</td>
        <td rowspan="3">
            Log系列引擎适用于快速写入小表（1百万行左右的表）并读取全部数据的场景。
            该系列引擎的共同特点如下。
            <ul>
                <li>数据被追加写入磁盘中。</li>
                <li>不支持
                    <code>delete</code>、<code>update</code>。
                </li>
                <li>不支持索引。</li>
                <li>不支持原子性写。</li>
                <li>
                    <code>insert</code>会阻塞
                    <code>select</code>操作。
                </li>
            </ul>
        </td>
        <td>TinyLog</td>
        <td>不支持并发读取数据文件，格式简单，查询性能较差，适用于暂存中间数据。</td>
    </tr>
    <tr>
        <td>StripeLog</td>
        <td>支持并发读取数据文件，将所有列存储在同一个大文件中，减少了文件数，查询性能比TinyLog好。</td>
    </tr>
    <tr>
        <td>Log</td>
        <td>支持并发读取数据文件，每个列会单独存储在一个独立文件中，查询性能比TinyLog好。</td>
    </tr>
    <tr>
        <td rowspan="5">Integrations</td>
        <td rowspan="5">
            Integrations系列引擎适用于将外部数据导入到
            云数据仓库UClickHouse中，或者在云数据仓库UClickHouse中直接使用外部数据源。
        </td>
        <td>Kafka</td>
        <td>将Kafka Topic中的数据直接导入到云数据仓库UClickHouse。
        </td>
    </tr>
    <tr>
        <td>MySQL</td>
        <td>
            将MySQL作为存储引擎，直接在云数据仓库UClickHouse中对MySQL表进行
            <code>select</code>等操作。
        </td>
    </tr>
    <tr>
        <td>JDBC</td>
        <td>通过指定JDBC连接串读取数据源。</td>
    </tr>
    <tr>
        <td>ODBC</td>
        <td>通过指定ODBC连接串读取数据源。</td>
    </tr>
    <tr>
        <td>HDFS</td>
        <td>直接读取HDFS上特定格式的数据文件。</td>
    </tr>
    <tr>
        <td rowspan="12">Special</td>
        <td rowspan="12">Special系列引擎适用于特定的功能场景。</td>
        <td>Distributed</td>
        <td>本身不存储数据，可以在多个服务器上进行分布式查询。</td>
    </tr>
    <tr>
        <td>MaterializedView</td>
        <td>用于创建物化视图。</td>
    </tr>
    <tr>
        <td>Dictionary</td>
        <td>
            将字典数据展示为一个云数据仓库UClickHouse表。
        </td>
    </tr>
    <tr>
        <td>Merge</td>
        <td>本身不存储数据，可以同时从任意多个其他表中读取数据。</td>
    </tr>
    <tr>
        <td>File</td>
        <td>直接将本地文件作为数据存储。</td>
    </tr>
    <tr>
        <td>NULL</td>
        <td>写入数据被丢弃，读取数据为空。</td>
    </tr>
    <tr>
        <td>Set</td>
        <td>数据总是保存在RAM中。</td>
    </tr>
    <tr>
        <td>Join</td>
        <td>数据总是保存在内存中。</td>
    </tr>
    <tr>
        <td>URL</td>
        <td>用于管理远程HTTP、HTTPS服务器上的数据。</td>
    </tr>
    <tr>
        <td>View</td>
        <td>
            本身不存储数据，仅存储指定的<code>SELECT</code>查询。
        </td>
    </tr>
    <tr>
        <td>Memory</td>
        <td>
            数据存储在内存中，重启后会导致数据丢失。查询性能极好，适合于对于数据持久性没有要求的1亿以下的小表。
            在云数据仓库UClickHouse中，通常用来做临时表。
        </td>
    </tr>
    <tr>
        <td>Buffer</td>
        <td>为目标表设置一个内存Buffer，当Buffer达到了一定条件之后会写入到磁盘。</td>
    </tr>
    </tbody>
</table>

关于表引擎的更多详细介绍，请参考官方文档[表引擎介绍](https://clickhouse.yandex/docs/zh/operations/table_engines/?spm=a2c4g.11186623.0.0.7b79500bvC1shW)。

## MergeTree

MergeTree表引擎主要用于海量数据分析，支持数据分区、存储有序、主键索引、稀疏索引和数据TTL等。MergeTree表引擎支持云数据仓库UClickHouse的所有SQL语法，但是部分功能与标准SQL存在差异。在MergeTree表引擎中，其主要作用是加速查询，即便在Compaction完成后，主键相同的数据行也仍旧共同存在。

该类型的引擎：

- MergeTree
- ReplacingMergeTree
- SummingMergeTree
- AggregatingMergeTree
- CollapsingMergeTree
- VersionedCollapsingMergeTree
- GraphiteMergeTree

关于表引擎的更多详细介绍，请参考官方文档[MergeTree](https://clickhouse.com/docs/zh/engines/table-engines/mergetree-family/mergetree/?spm=a2c4g.11186623.0.0.7b79500bvC1shW#mergetree)。

**示例：**

  1. 创建表lineorder，指定MergeTree，根据LO_ORDERDATE分区，按照LO_ORDERDATE, LO_ORDERKEY排序。

     ```
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
     ENGINE = MergeTree() 
     PRIMARY KEY (LO_ORDERDATE, LO_ORDERKEY)
     PARTITION BY toYear(LO_ORDERDATE) 
     ORDER BY (LO_ORDERDATE, LO_ORDERKEY);
     ```

2. 写入主键重复的数据

   参照**[快速上手](/uclickhouse/gettingstart) -> 准备数据并导入**

   准备数据并保存为lineorder.csv文件:

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

   导入数据：

   ```shell
   clickhouse-client --host=<节点IP> --port=9000 --user=admin --password=<创建集群时设置的密码> --database=<数据库名> --query="INSERT INTO lineorder_local FORMAT CSV" < /data/lineorder.csv
   ```

3. 查询数据。

   ```sql
   select * from lineorder_local;
   ```

   查询结果如下：

   ![table_engine_mergetree](images/table_engine_mergetree.png)

4. 由于MergeTree系列表引擎采用类似LSM Tree的结构，很多存储层处理逻辑直到Compaction期间才会发生，因此需执行optimize语句强制后台Compaction。

   ```sql
   optimize table lineorder_local final;
   ```

5. 再次查询数据。

   ```sql
   select * from lineorder_local;
   ```

   查询结果如下，主键重复的数据仍存在。

   ![table_engine_mergetree-2](images/table_engine_mergetree-2.png)

## ReplacingMergeTree

为了解决MergeTree表引擎相同主键无法去重的问题，云数据仓库UClickHouse提供了ReplacingMergeTree表引擎，用于删除主键值相同的重复项。

虽然ReplacingMergeTree表引擎提供了主键去重的能力，但是仍然存在很多限制，因此ReplacingMergeTree表引擎更多被用于确保数据最终被去重，而无法保证查询过程中主键不重复，主要限制如下。

- 在分布式场景下，相同主键的数据可能被分布到不同节点上，不同分片间可能无法去重。
- 在没有彻底optimize之前，可能无法达到主键去重的效果，比如部分数据已经被去重，而另外一部分数据仍旧有主键重复。
- optimize是后台动作，无法预测具体执行时间点。
- 手动执行optimize在海量数据场景下需要消耗大量时间，无法满足业务即时查询的需求。

关于ReplacingMergeTree表引擎的更多信息，请参考官方文档[ReplacingMergeTree](https://clickhouse.com/docs/zh/engines/table-engines/mergetree-family/replacingmergetree/#replacingmergetree)

**示例：**

1. 创建表lineorder_local_replacing。

   ```
   CREATE TABLE lineorder_local_replacing on cluster ck_cluster
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
   ENGINE = ReplacingMergeTree() 
   PRIMARY KEY (LO_ORDERDATE, LO_ORDERKEY)
   PARTITION BY toYear(LO_ORDERDATE) 
   ORDER BY (LO_ORDERDATE, LO_ORDERKEY);
   ```

2. 写入主键重复的数据

   参照**[快速上手](/uclickhouse/gettingstart) -> 准备数据并导入**

   准备数据并保存为lineorder.csv文件:

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

   导入数据：

   ```shell
   clickhouse-client --host=<节点IP> --port=9000 --user=admin --password=<创建集群时设置的密码> --database=<数据库名> --query="INSERT INTO lineorder_local_replacing FORMAT CSV" < /data/lineorder.csv
   ```

3. 查询数据。

   ```sql
   select * from lineorder_local_replacing;
   ```

   查询结果如下,发现重复数据已消除：

   ![table_engine_replacingMergetree](images/table_engine_replacingMergetree.png)

## CollapsingMergeTree

CollapsingMergeTree表引擎用于消除ReplacingMergeTree表引擎的功能限制。该表引擎要求在建表语句中指定一个标记列Sign，按照Sign的值将行分为两类：`Sign=1`的行称为状态行，用于新增状态。`Sign=-1`的行称为取消行，用于删除状态。

**说明：**

CollapsingMergeTree表引擎虽然解决了主键相同数据即时删除的问题，但是状态持续变化且多线程并行写入情况下，状态行与取消行位置可能乱序，导致无法正常折叠（删除）。

后台Compaction时会将主键相同、`Sign`相反的行进行折叠（删除），而尚未进行Compaction的数据，状态行与取消行同时存在。因此为了能够达到主键折叠（删除）的目的，需要业务层进行如下操作。

- 记录原始状态行的值，或者在执行删除状态操作前先查询数据库获取原始状态行的值。

  具体原因：执行删除状态操作时需要写入取消行，而取消行中需要包含与原始状态行主键一样的数据（Sign列除外）。

- 在进行count()、sum(col)等聚合计算时，可能会存在数据冗余的情况。为了获得正确结果，业务层需要改写SQL，将count()、sum(col)分别改写为sum(Sign)、sum(col * Sign)。

  具体原因如下。

  - 后台Compaction时机无法预测，在发起查询时，状态行和取消行可能尚未进行折叠（删除）。
  - 云数据库ClickHouse无法保证主键相同的行落在同一个节点上，不在同一节点上的数据无法进行折叠（删除）。

关于CollapsingMergeTree表引擎的更多信息，请参考官方文档[CollapsingMergeTree](https://clickhouse.com/docs/zh/engines/table-engines/mergetree-family/collapsingmergetree/#table_engine-collapsingmergetree)。

**示例：**

1. 创建表lineorder_local_collapsing。

   ```
   CREATE TABLE lineorder_local_collapsing
   (
       LO_ORDERKEY UInt64,
       LO_LINENUMBER UInt8,
       LO_CUSTKEY UInt8,
       Sign Int8
   )
   ENGINE = CollapsingMergeTree(Sign)
   ORDER BY LO_ORDERKEY;
   ```

2. 插入状态行Sign=1。

   ```
   INSERT INTO lineorder_local_collapsing VALUES (1, 5, 146, 1);
   ```

   说明：如果先插入取消行，再插入状态行，可能会导致位置乱序，即使强制后台Compaction，也无法进行主键折叠（删除）。

3. 插入取消行`Sign=-1`，除`Sign`列外其他值与插入的状态行一致。同时，插入一行相同主键数据的新状态行。

   ```
   INSERT INTO lineorder_local_collapsing VALUES (1, 5, 146, -1), (2, 6, 185, 1);
   ```

4. 查询数据。

   ```
   SELECT * FROM lineorder_local_collapsing;
   ```

   查询结果如下：

   ![table_engine_collapsingMergetree](images/table_engine_collapsingMergetree.png)

5. 如果您需要对指定列进行聚合计算，以`sum(col)`为例，为了获得正确结果，需改写SQL语句如下。

   ```sql
   SELECT
       LO_ORDERKEY,
       sum(LO_LINENUMBER * Sign) AS LO_LINENUMBERS,
       sum(LO_CUSTKEY * Sign) AS LO_CUSTKEYS
   FROM lineorder_local_collapsing
   GROUP BY LO_ORDERKEY
   HAVING sum(Sign) > 0;
   ```

   执行后，结果如下：

   ![table_engine_collaspingMergeTree-2](images/table_engine_collaspingMergeTree-2.png)

6. 由于MergeTree系列表引擎采用类似LSM Tree的结构，很多存储层处理逻辑直到Compaction期间才会发生，因此需执行optimize语句强制后台Compaction。

   ```
   optimize table lineorder_local_collapsing final;
   ```

7. 再次查询数据。

   ```sql
   SELECT * FROM lineorder_local_collapsing;
   ```

   查询结果如下：

   ![table_engine_collaspingMergetree-3](images/table_engine_collaspingMergetree-3.png)

## VersionedCollapsingMergeTree

为了解决CollapsingMergeTree表引擎乱序写入导致无法正常折叠（删除）问题，云数据库ClickHouse提供了VersionedCollapsingMergeTree表引擎，在建表语句中新增一列`Version`，用于在乱序情况下记录状态行与取消行的对应关系。后台Compaction时会将主键相同、`Version`相同、`Sign`相反的行折叠（删除）。与CollapsingMergeTree表引擎类似，在进行`count()`、`sum(col)`等聚合计算时，业务层需要改写SQL，将`count()`、`sum(col)`分别改写为`sum(Sign)`、`sum(col * Sign)`。

**说明：**

VersionedCollapsingMergeTree表引擎的更多信息，请参考官方文档[VersionedCollapsingMergeTree](https://clickhouse.com/docs/zh/engines/table-engines/mergetree-family/versionedcollapsingmergetree/#versionedcollapsingmergetree)。

**示例：**

1. 创建lineorder_local_versioned

   ```sql
   CREATE TABLE lineorder_local_versioned
   (
       LO_ORDERKEY UInt64,
       LO_LINENUMBER UInt8,
       LO_CUSTKEY UInt8,
       Sign Int8,
       Version UInt8
   )
   ENGINE = VersionedCollapsingMergeTree(Sign, Version)
   ORDER BY LO_ORDERKEY;
   ```

2. 插入取消行Sign=-1。

   ```sql
   INSERT INTO lineorder_local_versioned VALUES (1, 3, 222, -1, 1);
   ```

3. 插入状态行`Sign=1`、`Version=1`，其他列值与插入的取消行一致。同时，插入一行相同主键数据的新状态行。

   ```sql
   INSERT INTO lineorder_local_versioned VALUES (1, 3, 222, 1, 1),(2, 6, 345, 1, 2);
   ```

4. 查询数据。

   ```sql
   SELECT * FROM lineorder_local_versioned;
   ```

   查询结果如下：

   ![table_engine_versioned](images/table_engine_versioned.png)

5. 如果您需要对指定列进行聚合计算，以`sum(col)`为例，为了获得正确结果，需改写SQL语句如下。

   ```sql
   SELECT
       LO_ORDERKEY,
       sum(LO_LINENUMBER * Sign) AS LO_LINENUMBERS,
       sum(LO_CUSTKEY * Sign) AS LO_CUSTKEYS
   FROM lineorder_local_versioned
   GROUP BY LO_ORDERKEY
   HAVING sum(Sign) > 0;
   ```

   执行后，查询结果如下：

   ![table_engine_versioned-2](images/table_engine_versioned-2.png)

6. 由于MergeTree系列表引擎采用类似LSM Tree的结构，很多存储层处理逻辑直到Compaction期间才会发生，因此需执行optimize语句强制后台Compaction。

   ```
   optimize table lineorder_local_versioned final;
   ```

7. 再次查询数据。

   ```sql
   SELECT * FROM lineorder_local_versioned;
   ```

   查询结果如下：

   ![table_engine_versioned-3](images/table_engine_versioned-3.png)

## SummingMergeTree

SummingMergeTree表引擎用于对主键列进行预先聚合，将所有相同主键的行合并为一行，从而大幅度降低存储空间占用，提升聚合计算性能。

使用SummingMergeTree表引擎时，需要注意如下几点。

- 云数据仓库UClickHouse只在后台Compaction时才会对主键列进行预先聚合，而Compaction的执行时机无法预测，所以可能存在部分数据已经被预先聚合、部分数据尚未被聚合的情况。因此，在执行聚合计算时，仍需要使用`GROUP BY`子句。
- 在预先聚合时，云数据仓库UClickHouse会对主键列之外的其他所有列进行预聚合。如果这些列是可聚合的（比如数值类型），则直接sum求和。如果不可聚合（比如String类型），则会随机选择一个值。
- 通常建议将SummingMergeTree表引擎与MergeTree表引擎组合使用，MergeTree表引擎存储完整的数据，SummingMergeTree表引擎用于存储预先聚合的结果。

**说明：**

SummingMergeTree表引擎的更多信息，请参考官方文档[SummingMergeTree](https://clickhouse.com/docs/zh/engines/table-engines/mergetree-family/summingmergetree/#summingmergetree)。

**示例：**

1. 创建表lineorder_local_summing。

   ```
   CREATE TABLE lineorder_local_summing
   (
       key UInt32,
       value UInt32
   )
   ENGINE = SummingMergeTree()
   ORDER BY key;
   ```

2. 写入数据。

   ```
   INSERT INTO lineorder_local_summing Values(1,1),(1,2),(2,1);
   ```

3. 查询数据。

   ```
   select * from lineorder_local_summing;
   ```

   查询结果如下：

   ![table_engine_summing](images/table_engine_summing.png)

## AggregatingMergeTree

AggregatingMergeTree表引擎也是预先聚合引擎的一种，用于提升聚合计算的性能。与SummingMergeTree表引擎的区别在于，SummingMergeTree表引擎用于对非主键列进行sum聚合，而AggregatingMergeTree表引擎可以指定各种聚合函数。AggregatingMergeTree的语法比较复杂，需要结合物化视图或云数据库ClickHouse的特殊数据类型AggregateFunction一起使用。

**说明：**

AggregatingMergeTree表引擎的更多信息，请参见官方文档[AggregatingMergeTree](https://clickhouse.com/docs/zh/engines/table-engines/mergetree-family/aggregatingmergetree/)。

**示例：**

- **结合物化视图**

1. 创建lineorder。

   ```
   CREATE TABLE lineorder
   (
       LO_ORDERKEY UInt64,
       LO_LINENUMBER UInt8,
       LO_ORDERDATE Date,
       SIGN Int8
   )
   ENGINE = CollapsingMergeTree(SIGN)
   ORDER BY LO_ORDERKEY;
   ```

2. 对lineorder建立物化视图lineorder_view，并使用`sumState`和`uniqState`函数对明细表进行预先聚合。

   ```
   CREATE MATERIALIZED VIEW lineorder_view
   ENGINE = AggregatingMergeTree() PARTITION BY toYYYYMM(LO_ORDERDATE) ORDER BY (LO_LINENUMBER, LO_ORDERDATE)
   AS SELECT
       LO_LINENUMBER,
       LO_ORDERDATE,
       sumState(SIGN)    AS SIGNS,
       uniqState(LO_ORDERKEY) AS ORDERKEYS
   FROM lineorder
   GROUP BY LO_LINENUMBER, LO_ORDERDATE;
   ```

3. 写入数据至lineorder中。

   ```
   INSERT INTO lineorder VALUES(0, 0, '2019-11-11', 1);
   INSERT INTO lineorder VALUES(1, 1, '2020-11-12', 1);
   ```

4. 使用聚合函数`sumMerge`和`uniqMerge`对物化视图进行聚合，并查询聚合数据。

   ```
   SELECT
       LO_ORDERDATE,
       sumMerge(SIGNS) AS SIGNS,
       uniqMerge(ORDERKEYS) AS ORDERKEYS
   FROM lineorder_view
   GROUP BY LO_ORDERDATE
   ORDER BY LO_ORDERDATE
   ```

   **说明：**函数`sum`和`uniq`不能再使用，否则会出现SQL报错：Illegal type AggregateFunction(sum, Int8) of argument for aggregate function sum...

   查询结果如下：

   ![table_engine_aggregatefunction](images/table_engine_aggregatefunction.png)

- **结合特殊数据类型AggregateFunction使用**

1. 创建lineorder。

   ```
   CREATE TABLE lineorder
   (   LO_ORDERKEY UInt8,
       LO_ORDERDATE Date,
       LO_LINENUMBER UInt64
   ) ENGINE = MergeTree() 
   PARTITION BY toYYYYMM(LO_ORDERDATE) 
   ORDER BY (LO_ORDERKEY, LO_ORDERDATE);
   ```

2. 写入数据至明细表lineorder中。

   ```
   INSERT INTO lineorder VALUES(0, '2019-11-11', 1);
   INSERT INTO lineorder VALUES(1, '2020-11-12', 1);
   ```

3. 创建聚合表lineorder_agg，其中`LO_LINENUMBER`列的类型为AggregateFunction

   ```
   CREATE TABLE lineorder_agg
   (   LO_ORDERKEY UInt8,
       LO_ORDERDATE Date,
       LO_LINENUMBER AggregateFunction(uniq, UInt64)
   ) ENGINE = AggregatingMergeTree() 
   PARTITION BY toYYYYMM(LO_ORDERDATE) 
   ORDER BY (LO_ORDERKEY, LO_ORDERDATE);
   ```

4. 使用聚合函数`uniqState`将明细表的数据插入至聚合表中。

   ```
   INSERT INTO lineorder_agg
   select LO_ORDERKEY, LO_ORDERDATE, uniqState(LO_LINENUMBER)
   from lineorder
   group by LO_ORDERKEY, LO_ORDERDATE;
   ```

   **说明** 不能使用`INSERT INTO lineorder_aggVALUES(1, '2019-11-12', 1);`语句向聚合表插入数据，否则会出现SQL报错：Cannot convert UInt64 to AggregateFunction(uniq, UInt64)...

5. 使用聚合函数`uniqMerge`对聚合表进行聚合，并查询聚合数据。

   ```
   SELECT uniqMerge(LO_LINENUMBER) AS LINENUMBER 
   FROM lineorder_agg 
   GROUP BY LO_ORDERKEY, LO_ORDERDATE;
   ```

   查询结果如下：

   ![table_engine_agg](images/table_engine_agg.png)

## 日志

具有最小功能的轻量级引擎。当您需要快速写入许多小表（最多约100万行）并在以后整体读取它们时，该类型的引擎是最有效的。

该类型的引擎：

- TinyLog
- StripeLog
- Log

## 集成引擎

用于与其他的数据存储与处理系统集成的引擎。

该类型的引擎：

- Kafka
- MySQL
- ODBC
- JDBC
- HDFS

## 用于其他特定功能的引擎

该类型的引擎：

- Distributed
- MaterializedView
- Dictionary
- Merge
- File
- Null
- Set
- Join
- URL
- View
- Memory
- Buffer

## 虚拟列

- 虚拟列是表引擎组成的一部分，它在对应的表引擎的源代码中定义。
- 虚拟列不能在 CREATE TABLE 中指定，并且不会包含在 SHOW CREATE TABLE 和 DESCRIBE TABLE 的查询结果中。虚拟列是只读的，所以不能向虚拟列中写入数据。
- 查询虚拟列中的数据时，必须在 SELECT 查询中包含虚拟列的名称。`SELECT *` 不会返回虚拟列的数据。
- 若创建的表中有一列与虚拟列的名字相同，那么虚拟列将不能再被访问。为了避免这种列名的冲突，虚拟列的名字一般都以下划线开头。