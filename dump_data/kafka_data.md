# 从Kafka同步

登录Ucloud账号进入到[用户控制台](https://passport.ucloud.cn/#login)，在全部产品下搜索或者大数据下选择“Kafka消息队列 UKafka”，进入到[Kafka消息队列 UKafka控制台](https://console.ucloud.cn/ukafka/ukafka)下，点击**创建集群**按钮。进入创建集群页面，根据创建页提供的配置进行选择并下单。

<blockquote>
  温馨提示：
   <ol>
     <li>UKafka为付费产品，具体价格请参照创建页所展示。</li>
     <li>请确保UKafka集群和云数据仓库 UClickhouse集群在同一地域内（同一VPC）,如不同请使用 私有网络 VPC -> 网络互通</li>
   </ol>
</blockquote>

## 前置条件

已创建好UKafka产品并创建了Topic和Group。请参照[UKafka创建](https://docs.ucloud.cn/ukafka/kafkasinkerintro/quickstart)。

## 操作步骤

### 连接集群

- 在集群所在地域（同一网段）下建立一台云主机，在云主机上安装Clickhouse-client，官方旧版下载地址：[下载Clickhouse-client](https://repo.yandex.ru/clickhouse/rpm/stable/x86_64/)。官方新版下载地址：[下载Clickhouse-client](https://packages.clickhouse.com/rpm/lts/)。

  建议按实际创建的内核版本选择对应版本的Clickhouse-client，如以上创建的集群内核版本为21.8.14.5，则下载如下rpm包：

  ```
  wget https://repo.yandex.ru/clickhouse/rpm/stable/x86_64/clickhouse-client-21.8.14.5-2.noarch.rpm
  wget https://repo.yandex.ru/clickhouse/rpm/stable/x86_64/clickhouse-common-static-21.8.14.5-2.x86_64.rpm
  ```

- 执行安装

  ```shell
  rpm -ivh clickhouse-common-static-21.8.14.5-2.x86_64.rpm
  rpm -ivh clickhouse-client-21.8.14.5-2.noarch.rpm
  ```

- 通过clickhouse-client连接集群

  ```shell
  clickhouse-client --host=任一节点IP地址 --port=9000 --user=admin --password=创建集群时设置的密码
  ```

  ![](/Users/dongjunyang/works/projects/product/docs/uclickhouse/images/clickhouse-client-login.png)

  以上命令将进入交互模式。用户名默认admin、端口默认9000，节点IP可到集群详情查看。

## 创建数据库

```
CREATE DATABASE IF NOT EXISTS ck_test ON CLUSTER ck_cluster;
```

## 创建Kafka消费表

```
CREATE TABLE ck_test.lineorder_kafka ON CLUSTER ck_cluster 
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
) ENGINE = Kafka()
SETTINGS
    kafka_broker_list = 'broker 列表',
    kafka_topic_list = '主题列表',
    kafka_group_name = '消费分组名称',
    kafka_format = 'CSV',
    kafka_num_consumers = 1,
    kafka_max_block_size = 65536,
    kafka_skip_broken_messages = 0,
    kafka_auto_offset_reset = 'latest';
```

```
提示：
  Kafka消费表不能直接作为结果表使用，Kafka消费表作用只是用来消费Kafka中的数据，并没有真正存储数据。
```

参数说明：

| 名称                       | 必选 | 描述                                                         |
| -------------------------- | ---- | ------------------------------------------------------------ |
| kafka_broker_list          | 是   | 以英文逗号分隔的Kafka的borker地址列表。                      |
| kafka_topic_list           | 是   | 以英文逗号分隔的Topic名称列表。                              |
| kafka_group_name           | 是   | Kafka的消费组名称。                                          |
| kafka_format               | 是   | 云数据仓库 UClickHouse支持处理的消息体格式。<br /> 支持的消息体格式参见[Input and Output Formats](https://clickhouse.com/docs/en/interfaces/formats/?spm=a2c4g.11186623.0.0.540911c3r3HPsy#) |
| kafka_row_delimiter        | 否   | 行分隔符，用于分割不同的数据行。<br />默认为“\n”，您也可以根据数据写入的实际分割格式进行设置。 |
| kafka_num_consumers        | 否   | 单个表的消费者数量，默认值为1。<br /> 一个消费者的吞吐量不足时，需要指定更多的消费者。<br /> 消费者的总数不应超过Topic中的分区数，因为每个分区只能分配一个消费者。 |
| kafka_max_block_size       | 否   | Kafka消息的最大批次大小，单位：Byte，默认值为65536 Byte。    |
| kafka_skip_broken_messages | 否   | kafka消息解析器对于脏数据的容忍度，默认值为0。<br /> 如果`kafka_skip_broken_messages=N`，则引擎将跳过N条无法解析的Kafka消息<br />（一条消息等于一行数据）。 |
| kafka_commit_every_batch   | 否   | 执行Kafka commit的频率，取值说明如下。<br /> 完全写入一整个Block数据块的数据后才执行commit。<br /> 每写完一个Batch批次的数据就执行一次commit。 |
| kafka_ auto_offset_reset   | 否   | 从哪个offset开始读取Kafka数据，取值说明如下。<br /> earliest：从最早的offset开始读取Kafka数据。<br /> latest：从最晚的offset开始读取Kafka数据。<br /> 注意：21.8版本的云数据库ClickHouse集群不支持该参数。 |

更多参数请参照[官方文档Kafka](https://clickhouse.tech/docs/zh/engines/table-engines/integrations/kafka/?spm=a2c4g.11186623.0.0.540911c3r3HPsy)

## 创建云数据仓库 UClickhouse本地表

### 创建本地表

```
CREATE TABLE ck_test.lineorder_local ON CLUSTER ck_cluster
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

### 创建分布式表

如果您只需要同步数据至本地表，可跳过此步骤。

```
CREATE TABLE IF NOT EXISTS ck_test.lineorder on cluster ck_cluster AS ck_test.lineorder_local  ENGINE = Distributed(
       ck_cluster,
       ck_test,
       lineorder_local,
       rand()
);
```

### 创建物化视图

创建物化视图将Kafka消费表消费到的数据同步到云数据仓库 UClickHouse目的表。如果您同步的目的表是本地表，请将分布式表名更换为本地表名，再进行同步。

```
CREATE MATERIALIZED VIEW consumer ON CLUSTER ck_cluster TO lineorder AS SELECT * FROM lineorder_kafka;
```

### 在UKafka中写入消息

请[参照UKafka](https://docs.ucloud.cn/ukafka/kafkasinkerintro/quickstart)。

### 查询验证

```
clickhouse-client --host=任一节点IP地址 --port=9000 --user=admin --password=创建集群时设置的密码 --query="select count(*) from lineorder"
```

