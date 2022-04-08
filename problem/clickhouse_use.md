# Clickhouse 常见问题

## 内存超限问题

![image](images/problem1.png)

ClickHouse服务端对所有查询线程都配有memory tracker，同一个查询下的所有线程tracker会汇报给一个memory tracker for query，再上层还是memory tracker for total。

- `Memory limit (for query)` ：查询时内存占用过多（实例总内存的70%），说明您此时的集群内存配置已经满足不了当前查询所需，您可以通过升级配置提高内存。

- `Memory limit (for total)`：总内存使用超限（实例总内存的90%），您可以尝试降低查询并发，如果仍然不行则可能是后台异步任务占用了比较大的内存（常常是写入后主键合并任务），您可以通过升级配置提高内存。

## Clickhouse超时问题

**distributed_ddl_task_timeout超时问题**

分布式DDL查询（带有 on cluster clause）的执行等待时间，系统默认是180s。您可以在DMS上执行以下命令来设置全局参数，设置后需要重启集群。

```
set global on cluster default distributed_ddl_task_timeout = 1800;
```

由于分布式DDL是基于zookeeper构建任务队列异步执行，执行等待超时并不代表查询失败，只表示之前发送还在排队等待执行，用户不需要重复发送任务。

**max_execution_time超时问题**

一般查询的执行超时时间，超时限制触发之后查询会自动取消。用户可以进行查询级别更改，例如`select * from system.numbers settings max_execution_time = 3600`，也可以在DMS上执行以下命令来设置全局参数。

```
set global on cluster default max_execution_time = 3600;
```

**socket_timeout超时问题**

HTTP协议在监听socket返回结果时的等待时间。该参数不是Clickhouse系统内的参数，它属于jdbc在HTTP协议上的参数，但它是会影响到前面的**max_execution_time**参数设置效果，因为它决定了客户端在等待结果返回上的时间限制。所以一般用户在调整**max_execution_time**参数的时候也需要配套调整**socket_timeout**参数，略微高于**max_execution_time**即可。用户设置参数时需要在jdbc链接串上添加**socket_timeout**这个property，单位是毫秒，例如：'jdbc:clickhouse://127.0.0.1:8123/default?socket_timeout=3600000'。

## 如何处理导入数据时报too many parts？

ClickHouse每次写入都会生成一个data part，如果每次写入一条或者少量的数据，那会造成ClickHouse内部有大量的data part（会给merge和查询造成很大的负担）。为了防止出现大量的data part，ClickHouse内部做了很多限制，这就是too many parts报错的内在原因。出现该错误，第一要做的事情就是增加写入的批量大小。如果确实没有办法调整批量大小，可以适当的修改参数：`max_partitions_per_insert_block`。

## 如何处理insert into select xxx内存超限问题？

- 常见原因1：内存使用率比较高。

  解决方案：调整参数max_insert_threads，减少可能的内存使用量。

- 常见原因2：当前是通过`insert into select`把数据从一个ClickHouse集群导入到另外一个集群。

  解决方案：通过导入文件的方式来迁移数据。

## 如何查询Clickhouse耗费的CPU、Memory资源情况?

system.query_log系统表里可以自助查看CPU、Memory高峰期的查询日志，里面包含每个查询的CPU消耗和Memory消耗统计。详细请参考[官方文档](https://clickhouse.tech/docs/en/operations/system-tables/query_log/)。

## 为什么建库建表之后查询库或表不存在？

- 单副本时集群规模只有一个节点不存在此问题。

- 双副本存在多个节点，您需要在每一个节点上执行DDL语句，或者在集群的任一节点执行DDL语句，但必须加上 `ON CLUSTER ck_cluster`关键字，请参考**开发指南**

## 如何进行DDL增加列、删除列、修改列操作？

本地表的修改直接执行即可。如果要对分布式表进行修改，需分如下情况进行。

- 如果没有数据写入，您可以先修改本地表，然后修改分布式表。
- 如果数据正在写入，您需要区分不同的类型进行操作。

<table>
    <thead>
    <tr>
        <th>类型</th>
        <th>操作步骤</th>
    </tr>
    </thead>
    <tbody>
    <tr>
        <td>增加Nullable的列</td>
        <td rowspan="2">
            <ol>
                <li>修改本地表。</li>
                <li>修改分布式表。</li>
            </ol>
        </td>
    </tr>
    <tr>
        <td>修改列的数据类型（类型可以相互转换）</td>
    </tr>
    <tr>
        <td>删除Nullable列</td>
        <td>
            <ol>
                <li>修改分布式表。</li>
                <li>修改本地表。</li>
            </ol>
        </td>
    </tr>
    <tr>
        <td>增加非Nullable的列</td>
        <td rowspan="3">
            <ol>
                <li>停止数据的写入。</li>
                <li>执行SYSTEM FLUSH DISTRIBUTED分布式表。</li>
                <li>修改本地表。</li>
                <li>修改分布式表。</li>
                <li>重新进行数据的写入。</li>
            </ol>
        </td>
    </tr>
    <tr>
        <td>删除非Nullable的列</td>
    </tr>
    <tr>
        <td>修改列的名称</td>
    </tr>
    </tbody>
</table>

## 为什么DDL执行慢，经常卡住？

常见原因：DDL全局的执行是串行执行，复杂查询会导致死锁。

您可以采取如下解决方案。

- 等待运行结束。
- 在控制台尝试终止查询。

## 如何处理分布式DDL报错：longer than distributed_ddl_task_timeout (=xxx) seconds？

可以通过使用`set global on cluster default distributed_ddl_task_timeout=xxx`命令修改默认超时时间，xxx为自定义超时时间，单位为秒。

## 如何处理语法报错：set global on cluster default？

- 常见原因1：ClickHouse客户端会进行语法解析，而`set global on cluster default`是服务端增加的语法。在客户端尚未更新到与服务端对齐的版本时，该语法会被客户端拦截。

  解决方案：
  - 使用JDBC Driver等不会在客户端解析语法的工具，比如DataGrip、DBeaver。
  - 编写JDBC程序来执行该语句。

- 常见原因2：`set global on cluster default key = value; `中value是字符串，但是漏写了引号。

  解决方案：在字符串类型的value两侧加上引号。

## 如何查看每张表所占的磁盘空间？

执行以下语句：

```
SELECT table, formatReadableSize(sum(bytes)) as size, min(min_date) as min_date, max(max_date) as max_date FROM system.parts WHERE active GROUP BY table; 
```

## 常用的系统表有哪些？

<table>
    <thead>
    <tr>
        <th>名称</th>
        <th>作用</th>
    </tr>
    </thead>
    <tbody>
    <tr>
        <td>system.processes</td>
        <td>查询正在执行的SQL。</td>
    </tr>
    <tr>
        <td>system.query_log</td>
        <td>查询历史执行过的SQL。</td>
    </tr>
    <tr>
        <td>system.merges</td>
        <td>查询集群上的merge信息。</td>
    </tr>
    <tr>
        <td>system.mutations</td>
        <td>查询集群上的mutation信息。</td>
    </tr>
    </tbody>
</table>

## 如何修改查询最大内存使用？

可以在执行语句的settings里增加

```
settings max_memory_usage = XXX;
```

例如修改为20G：

```
SET max_memory_usage = 20000000000; #20G
```

