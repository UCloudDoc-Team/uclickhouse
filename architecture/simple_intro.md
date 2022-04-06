# 什么是云数据仓库 UClickHouse

云数据仓库UDW ClickHouse为开源列式数据库管理系统ClickHouse提供了一整套安全、稳定、可靠的托管服务，针对不同规模数据、硬件进行优化，并提供额外便捷的工具支持，使您在使用ClickHouse服务时不再需要为集群的配置、选型、故障、运维、开发工具等操心，您可以在Ucloud上便捷的购买UClickhouse，拥有一套私有的Clickhouse集群。

## 产品架构

![image](images/image1.png)

UClickhouse设计为Clickhouse集群+Zookeeper集群，具体特点如下。

- Zookeeper集群与Clickhouse集群部署分离，分别部署在不同的云主机上。

- Zookeeper设计为三节点集群，Clickhouse设计为单集群，多分片，双副本结构。

- 每个副本（节点）单独部署在云主机上，每台云主机只提供一个分片的副本服务。

## 产品特性

主要特性如下。

- 天然支持面向列的数据压缩，数据压缩比高。
- 多核并行处理，大型查询自然并行化，占用当前服务器上可用的所有必要资源。
- 向量化计算引擎，数据不仅由列存储，还由向量（列的一部分）处理，从而实现较高的CPU效率。
- 数据复制和数据完整性支持，使用异步多主复制，写入任何可用的副本后，所有剩余的副本都会在后台检索其副本。
- SQL 支持，支持基于SQL的声明式查询语言，在许多情况下与ANSI SQL标准相同。
- 自适应 Join 算法。

ClickHouse官网地址，请参见https://clickhouse.com。

ClickHouse参考文档链接，请参见https://clickhouse.com/docs/。

## 增强特性

UClickhouse面对企业级用户做了以下增强。

- 可视化监控：内存、CPU、磁盘监控等。

- 可扩展性：支持多规格配置选择、弹性垂直扩容。

