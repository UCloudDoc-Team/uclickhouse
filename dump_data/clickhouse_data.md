# clickhouse数据迁移

方案一：在线迁移，迁移过程中读写几乎不受影响，但迁移步骤相对方案二会多一些，适合迁移数据量较大，业务无法中断场景。

方案二：离线迁移，迁移步骤较少，但迁移过程中写入会中断，中断时长与数据量大小相关，适合迁移数据量较少，且短暂中断业务不受影响场景。

## 方案一：在线迁移

### 1.前提：
#### 1.1 源、目标集群网络可互通

#### 1.2 表必须存在时间序递增等字段，如create_time

#### 1.3 数据迁移过程中源集群读写不受影响

### 2.实施步骤

#### 2.1 迁移元数据

##### 2.1.1 导出源集群待迁移的表结构

```sql
-- 可以使用该语句找到所有的表，并拼接为SHOW CREATE TABLE xxx的格式
SELECT concat('SHOW CREATE TABLE ', database, '.', name, ';')
FROM system.tables
WHERE database NOT IN ('system', 'information_schema', 'INFORMATION_SCHEMA')
ORDER BY database, name;

-- 然后使用该语句导出表结构SQL
clickhouse-client --host="${源集群IP}" --port="9000" --user="${username}" --password="${password}" --query="SHOW CREATE TABLE <database_name>.<table_name> FORMAT CSV"  > table.sql
```

##### 2.1.2 修改源集群表结构

将导出的表结构，增加 ON CLUSTER ck_cluster 集群元语，并替换Replicated*MergeTree 表引擎。（若当前表已经为Replicated复制表，则忽略）

##### 2.1.3 目标集群创建数据库

```sql
clickhouse-client --host="${源集群IP}" --port="9000" --user="${username}" --password="${password}" --query="CREATE DATABASE IF NOT EXISTS <database_name> ON CLUSTER ck_cluster;"
```

##### 2.1.4 目的集群创建表结构

登录任一目标集群节点，执行 table.sql 建表语句：

```sql
clickhouse-client --host="${源集群IP}" --port="9000" --user="${username}" --password="${password}"
```

#### 2.2 迁移数据

![img](/images/clickhouse-data-1.png)

短时间停止源集群上游写入，并找到源集群中数据库表最新数据点位，如：

```sql
SELECT max(create_time) FROM dbname.tablename;
```

上游开启双写，即同时写入源、目标集群，下游读操作仍然从源集群读取

将源集群点位以前存量数据导入目标肌群（建议使用 nohup 后台执行），在目标集群上执行：

> max_execution_time 为最大超时时间，此设置以秒为单位，建议根据数据量设置，否则数据量过大会导致执行时间过长而中断

```sql
clickhouse-client --user xx --password xxx --max_execution_time 50000 --query="INSERT INTO dbname.tablename SELECT * FROM remote('${目标集群IP}:9000', '${dbname}', '${tablename}', '${username}', '${password}') where create_time <= '';"
```

若源集群和目标集群版本不一致（源集群版本低于目标集群），比如源集群版本为22.x版本，目标集群为24.x版本，则可以在源集群执行：

```sql
clickhouse-client --user xx --password xxx --max_execution_time 50000 --query="NSERT INTO FUNCTION remote('${目标集群节点ip}:9000', '${目标集群数据库名称}.${目标集群数据表名称}', '${目标集群的用户名}', '${目标集群的用户密码}') SELECT * FROM ${源集群数据库名称}.${源集群数据表名称} WHERE create_time <= '';"
```

存量数据迁移完成后，停止源集群上游写入，并将下游应用读流量切换到目标集群

#### 2.3 捕获UPDATE/DELETE操作

在线数据迁移过程中，可能出现UPDATE/DELETE操作，为避免数据出现不一致性，需要捕获这些更新，并同步到新集群。
若数据迁移过程中，包括删除和更新操作，则客户需要在业务侧记录历史数据的更新日志（如创建一张历史数据更新日志表），旧数据迁移完成后批量补偿。如下：

##### 2.3.1 在旧集群中，为时间位点T_cut之前的历史数据创建一个更新日志表，记录所有对历史数据的更新操作（如 id、更新时间、更新内容字段），其他字段客户可以自行添加。
```sql
-- 旧集群创建更新日志表
CREATE TABLE history_update_log (
    id UInt64,  -- 被更新数据的唯一标识
    create_time DateTime,  -- 更新发生的时间
    sql String,  -- 具体的更新语句（或更新后的字段值）
    database String, -- 发生变更的数据库
    table String -- 发生变更的表
) ENGINE = MergeTree() ORDER BY update_time;
```

##### 2.3.2 在更新旧集群的历史数据时，同步往 history_update_log 中写入一条记录。
##### 2.3.3 离线迁移完成后，解析 history_update_log表，将所有更新操作批量同步到新集群（需注意执行顺序）。
> 注意：需要客户在业务中将更新操作写入history_update_log表的逻辑。

## 方案二：离线迁移

### 1. 前提

#### 1.1 源、目标实例网络可互通

#### 1.2 数据迁移过程中源集群读写不可用，影响时长主要与数据量有关

### 2. 实施步骤

#### 2.1 迁移元数据

##### 2.1.1 导出源集群待迁移的表结构

```sql
-- 可以使用该语句找到所有的表，并拼接为SHOW CREATE TABLE xxx的格式
SELECT concat('SHOW CREATE TABLE ', database, '.', name, ';')
FROM system.tables
WHERE database NOT IN ('system', 'information_schema', 'INFORMATION_SCHEMA')
ORDER BY database, name;

-- 然后使用该语句导出表结构SQL
clickhouse-client --host="${源集群IP}" --port="9000" --user="${username}" --password="${password}" --query="SHOW CREATE TABLE <database_name>.<table_name> FORMAT CSV"  > table.sql
```

##### 2.1.2 修改源集群表结构

将导出的表结构，增加 ON CLUSTER ck_cluster 集群元语，并替换Replicated*MergeTree 表引擎。（若当前表已经为Replicated复制表，则忽略）

##### 2.1.3 目标集群创建数据库

```sql
clickhouse-client --host="${源集群IP}" --port="9000" --user="${username}" --password="${password}" --query="CREATE DATABASE IF NOT EXISTS <database_name> ON CLUSTER ck_cluster;"
```

##### 2.1.4 目的集群创建表结构

登录任一目标集群节点，执行 table.sql 建表语句：

```sql
clickhouse-client --host="${源集群IP}" --port="9000" --user="${username}" --password="${password}"
```

#### 2.2 迁移数据

![img](/images/clickhouse-data-2.png)

暂停上游写入，将源集群数据导入目标（建议使用 nohup 后台执行），在目标集群执行：

> max_execution_time 为最大超时时间，此设置以秒为单位，建议根据数据量设置，否则数据量过大会导致执行时间过长而中断

```sql
clickhouse-client --user xx --password xxx --max_execution_time 50000 --query="INSERT INTO dbname.tablename SELECT * FROM remote('${CK-old IP}:9000', '${dbname}', '${tablename}', '${username}', '${password}');"
```

若源集群和目标集群版本不一致（源集群版本低于目标集群），比如源集群版本为22.x版本，目标集群为24.x版本，则可以在源集群执行：

```sql
clickhouse-client --user xx --password xxx --max_execution_time 50000 --query="INSERT INTO FUNCTION remote('${目标集群节点ip}:9000', '${目标集群数据库名称}.${目标集群数据表名称}', '${目标集群的用户名}', '${目标集群的用户密码}') SELECT * FROM ${源集群数据库名称}.${源集群数据表名称} WHERE create_time <= '';"
```