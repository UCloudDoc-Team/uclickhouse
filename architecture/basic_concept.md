# 基本概念

## 地域（Region）

购买云数据仓库 UClickHouse的服务器所处地理位置，您可按需要选择云数据仓库 UClickhouse集群所在的Region。

## 可用区（Zone）

同一地域下，电力、网络隔离的物理区域，可用区之间内网互通，可用区内网络延时更小。

### 云数据仓库 UClickhouse集群

云数据仓库 UClickhouse 是由多个Clickhouse Server节点组成的分布式数据库，云数据仓库 UClickhouse 节点因购买规格不同可能包含一个或两个副本（Replica），一个云数据仓库 UClickHouse 集群可能包含一个或多个分片（Shard）。

## 副本（Replica）

为了保证数据的安全性和集群服务的高可用性，云数据仓库 UClickHouse提供了副本机制，可将单台节点服务器上的数据冗余存储在两台或多台服务器上。

- 双副本：每个节点包含两个副本，当某个副本服务不可用的时候，同一分片的另一个副本还可以继续提供服务。
- 单副本：每个节点只有一个副本，该副本服务不可用时，会导致整个集群不可用。

<blockquote>
说明：
  <ol>
  <li>双副本模式具备高可用特性，每2个副本节点组成一个分片，分片内节点不可用时另一个节点可以继续提供服务能力（需要结合副本表引擎使用）。</li>
    <li>单副本模式不承诺SLA，建议仅在测试/开发环境使用，生产环境推荐使用双副本模式。</li>
  </ol>
</blockquote>

## 分片（Shard）

在超大规模数据的场景下，单机器的存储及计算资源会成为数据处理分析的瓶颈，云数据仓库 UClickHouse 将海量数据分散存储到多台服务器上（节点），每台服务器（节点）只存储和处理海量数据的一部分。单副本时，一个分片（Shard）对应一个节点，双副本时，一个分片（Shard）对应两个节点。每台服务器（节点）即被称为一个分片（Shard）。

## 计算规格

云数据仓库 UClickhouse集群的资源配比，您可按需选择两种规格任一进行创建。