# 基于US3的Clickhouse数据迁移方案

源clickhouse集群拓扑：12个分片，每个分片单副本；包括表引擎：ReplicatedMergeTree，ReplacingMergeTree，MergeTree，View，Distributed

目标clickhouse集群拓扑：12个分片，高可用集群（即每个分片双副本）

## 1. 对全量数据进行备份+业务侧开启双写

### 1.1 短时间停止源集群的写操作，对集群的数据进行全量备份

全量备份操作如下：在源集群上依次对每个数据库进行备份，数据备份命令如下（由于命令是ON CLUSTER命令，所以只需要在一个节点执行即可）：

```sql
BACKUP DATABASE ${数据库名称} ON CLUSTER ${集群名称} TO S3(
    'http://${Bucket名称}.${US3内网Endpoint}/${备份路径}',
    '${S3令牌公钥}',
    '${S3令牌私钥}'
);
-- 示例：
BACKUP DATABASE test_db ON CLUSTER ck_cluster TO S3(
    'http://xxxxx.internal.s3-cn-bj.ufileos.com/uclickhouse/uck-xxxxx/test_db',
    'TOKEN_xxxx-xxxx-xxxx-xxxxx',
    'xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx'
);
```

**说明：**

**Bucket名称**：创建的US3桶名称

**US3内网Endpoint**：参考 [https://docs.ucloud.cn/ufile/introduction/region](https://docs.ucloud.cn/ufile/introduction/region)

**备份路径**：建议备份路径中标明实例名称、数据库名称，如上述，uck-xxxxx为实例名称，test_db为数据库名称

**S3令牌公钥**：见US3产品-令牌管理功能

**S3令牌私钥**：见US3产品-令牌管理功能

查看US3的对应目录下是否有已经备份的数据，等待备份完成

```sql
SELECT *
FROM system.backups
WHERE id = '${id}' AND status = 'BACKUP_CREATED';
```

### 1.2 源集群和目标集群开启写操作（业务双写）

### 1.3 数据更新和删除操作

若备份/恢复过程中，存在业务方对全量数据进行更新或者删除操作，则需要业务方记录这段时间内（开始数据备份到数据恢复完成）发生的更新操作和删除操作。等目标集群数据恢复完成时，在目标集群上执行记录到更新和删除操作。

#### 1.3.1 在数据迁移至新实例的过程中，出现了数据更新（如修改或删除）操作，则客户需要在业务侧记录更新日志（源集群或目标集群创建一张历史数据更新日志表，用来记录从备份开始时间点到恢复完成时间点发生的更新和删除），旧数据迁移完成后批量补偿。历史数据更新日志表结构如下，若需要其他字段客户可自行添加：

```plain
-- 旧集群创建更新日志表
CREATE TABLE history_update_log (
    id UInt64,  -- 被更新数据的唯一标识
    create_time DateTime,  -- 更新发生的时间
    sql String,  -- 具体的更新语句（或更新后的字段值）
    database String, -- 发生变更的数据库
    table String -- 发生变更的表
) ENGINE = MergeTree() ORDER BY update_time;

```

#### 1.3.2 在更新或删除旧集群的历史数据时，同步往 history_update_log 中写入一条记录。
#### 1.3.3 等待目标备份恢复结束后，解析 history_update_log表，将所有更新操作批量同步到新集群（需注意按照create_time的执行顺序）。

> 注意：需要客户在业务中增加写入history_update_log表的逻辑。


## 2. 目标集群恢复数据

### 2.1 备份完成后，在目标集群上依次对每个数据库进行恢复，数据恢复命令如下（由于命令是ON CLUSTER命令，所以只需要在一个节点执行即可）：

```sql
RESTORE DATABASE ${数据库名称} ON CLUSTER ${集群名称} FROM S3(
    'http://${Bucket名称}.${US3内网Endpoint}/${备份路径}',
    '${S3令牌公钥}',
    '${S3令牌私钥}'
);

-- 示例：
RESTORE DATABASE test_db ON CLUSTER ck_cluster FROM S3(
    'http://xxxxx.internal.s3-cn-bj.ufileos.com/uclickhouse/uck-xxxxx/test_db',
    'TOKEN_xxxx-xxxx-xxxx-xxxxx',
    'xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx'
);
```

**说明：**

**Bucket名称**：创建的US3桶名称

**US3内网Endpoint**：参考 [https://docs.ucloud.cn/ufile/introduction/region](https://docs.ucloud.cn/ufile/introduction/region)

**备份路径**：建议备份路径中标明实例名称、数据库名称，如上述，uck-xxxxx为实例名称，test_db为数据库名称

**S3令牌公钥**：见US3产品-令牌管理功能

**S3令牌私钥**：见US3产品-令牌管理功能

查看恢复任务：

```sql
SELECT *
FROM system.backups
WHERE id = '${id}' AND status = 'RESTORED';
```
