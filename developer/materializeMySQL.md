# MaterializeMySQL引擎

参照[官方文档](https://clickhouse.com/docs/en/engines/database-engines/materialized-mysql/)

**语法：**

创建数据库时配置数据库引擎类型为MaterializeMySQL并配置相关参数。

```
CREATE DATABASE [IF NOT EXISTS] db_name [ON CLUSTER ck_cluster]
ENGINE = MaterializeMySQL('host:port', 'database', 'user', 'password') 
[SETTINGS...]

where SETTINGS are:
[ { include_tables | exclude_tables } ]
[ skip_error_count ]
[ skip_unsupported_tables ]
[ query_with_final] 
[ order_by_only_primary_key ]
[ enable_binlog_reserved ]
[ shard_model ]
[ rate_limiter_row_count_per_second ]
```

**引擎参数：**

<table>
    <thead>
        <tr>
            <th>参数</th>
            <th>描述</th>
        </tr>
    </thead>
    <tbody>
    <tr>
        <td>host:port</td>
        <td>MySQL数据库的URL和端口号。</td>
    </tr>
    <tr>
        <td>database</td>
        <td>MySQL数据库名称。</td>
    </tr>
    <tr>
        <td>user</td>
        <td>MySQL数据库账号。</td>
    </tr>
    <tr>
        <td>password</td>
        <td>MySQL数据库账号的密码。</td>
    </tr>
    </tbody>
</table>

**引擎配置项：**

<table>
    <thead>
        <tr>
            <th>配置项</th>
            <th>说明</th>
        </tr>
    </thead>
    <tbody>
    <tr>
        <td>max_rows_in_buffer</td>
        <td>
          允许数据缓存到内存中的最大行数(对于单个表和无法查询的缓存数据)。当超过行数时，数据将被物化。默认值: 65505。
        </td>
    </tr>
    <tr>
        <td>max_bytes_in_buffer</td>
        <td>
          允许在内存中缓存数据的最大字节数(对于单个表和无法查询的缓存数据)。当超过行数时，数据将被物化。默认值: 1048576。
        </td>
    </tr>
    <tr>
        <td>max_rows_in_buffers</td>
        <td>
          允许数据缓存到内存中的最大行数(对于数据库和无法查询的缓存数据)。当超过行数时，数据将被物化。默认值: 65505。
        </td>
    </tr>
    <tr>
        <td>max_bytes_in_buffers</td>
        <td>
          允许在内存中缓存数据的最大字节数(对于数据库和无法查询的缓存数据)。当超过行数时，数据将被物化。默认值: 1048576。
        </td>
    </tr>
    <tr>
        <td>max_flush_data_time</td>
        <td>
          允许数据在内存中缓存的最大毫秒数(对于数据库和无法查询的缓存数据)。当超过这个时间时，数据将被物化。默认值: 1000。
        </td>
    </tr>
    <tr>
        <td>max_wait_time_when_mysql_unavailable</td>
        <td>
           当MySQL不可用时重试间隔(毫秒)。负值禁止重试。默认值: 1000。
        </td>
    </tr>
    <tr>
        <td>allows_query_when_mysql_lost</td>
        <td>
          当mysql丢失时，允许查询物化表。默认值: 0 (false)。
        </td>
    </tr>
    </tbody>
</table>

**示例：**

```
CREATE DATABASE IF NOT EXISTS db_name ON CLUSTER cluster
ENGINE = MaterializeMySQL('<MySQL地址>:3306', '<数据库>', '<用户>', '<密码>') 
SETTINGS 
        allows_query_when_mysql_lost=true,
        max_wait_time_when_mysql_unavailable=10000;
```

## MySQL服务器端配置

为了`MaterializeMySQL`正确的工作，有一些强制性的`MySQL`侧配置设置应该设置:

- `default_authentication_plugin = mysql_native_password`，因为`MaterializeMySQL`只能使用此方法授权。
- `gtid_mode = on`，因为要提供正确的`MaterializeMySQL`复制，基于GTID的日志记录是必须的。注意，在打开这个模式`On`时，你还应该指定`enforce_gtid_consistency = on`。

## 使用提示

**虚拟字段**

当使用MaterializeMySQL数据库引擎在云数据仓库UClickHouse集群上新建ReplacingMergeTree引擎表时，会默认在表中增加两个虚拟字段。

<table>
    <thead>
        <tr>
            <th>字段</th>
            <th>类型</th>
            <th>说明</th>
        </tr>
    </thead>
    <tbody>
    <tr>
        <td>_version</td>
        <td>UInt64</td>
        <td>事务计数器，记录数据版本信息。</td>
    </tr>
    <tr>
        <td>_sign</td>
        <td>TypeInt8</td>
        <td>删除标记，标记该行是否删除。取值范围如下：
            <ul>
                <li>1：该行未删除。</li>
                <li>
                    -1：该行已删除。
                    <strong>说明</strong> 带有<code>_sign=-1</code>的行不会从表中物理删除。
                </li>
            </ul>
        </td>
    </tr>
    </tbody>
</table>

**DDL语句转换**

MaterializeMySQL不支持直接插入、删除和更新查询，MySQL DDL语句将被转换成相应的ClickHouse DDL语句：

- MySQL INSERT查询被转换为`INSERT with _sign=1`。
- MySQL DELETE查询被转换为`INSERT with _sign=-1`。
- MySQL UPDATE查询被转换成`INSERT with _sign=1`和`INSERT with _sign=-1`。

**说明：**

- 如果ClickHouse不能解析某些DDL语句，该语句将被忽略。
- MaterializeMySQL引擎不支持级联UPDATE/DELETE查询。

**SELECT查询**

- 如果在SELECT查询中没有指定_version，则使用FINAL修饰符，返回_version的最大值对应的数据，即最新版本的数据。
- 如果在SELECT查询中没有指定_sign，则默认使用`WHERE _sign=1`，即返回未删除状态`（_sign=1)`的数据。

**索引转换**

- ClickHouse数据库表会自动将MySQL主键和索引子句转换为`ORDER BY`元组。
- ClickHouse只有一个物理顺序，由`ORDER BY`子句决定。如果需要创建新的物理顺序，请使用物化视图。

## MySQL与云数据仓库UClickHouse的字段类型对照

<table>
    <thead>
        <tr>
            <th>MySQL</th>
            <th>ClickHouse</th>
        </tr>
    </thead>
    <tbody>
    <tr>
        <td>TINY</td>
        <td>Int8</td>
    </tr>
    <tr>
        <td>SHORT</td>
        <td>Int16</td>
    </tr>
    <tr>
        <td>INT24</td>
        <td>Int32</td>
    </tr>
    <tr>
        <td>LONG</td>
        <td>UInt32</td>
    </tr>
    <tr>
        <td>LONG</td>
        <td>UInt64</td>
    </tr>
    <tr>
        <td>FLOAT</td>
        <td>Float32</td>
    </tr>
    <tr>
        <td>DOUBLE</td>
        <td>Float64</td>
    </tr>
    <tr>
        <td>DECIMAL，NEWDECIMAL</td>
        <td>Decimal</td>
    </tr>
    <tr>
        <td>DATE，NEWDATE</td>
        <td>Date</td>
    </tr>
    <tr>
        <td>DATETIME，TIMESTAMP</td>
        <td>DateTime</td>
    </tr>
    <tr>
        <td>DATETIME2，TIMESTAMP2</td>
        <td>DateTime64</td>
    </tr>
    <tr>
        <td>STRING</td>
        <td>String</td>
    </tr>
    <tr>
        <td>VARCHAR，VAR_STRING</td>
        <td>String</td>
    </tr>
    <tr>
        <td>BLOB</td>
        <td>String</td>
    </tr>
    <tr>
        <td>BIT</td>
        <td>UInt64</td>
    </tr>
    <tr>
        <td>SET</td>
        <td>UInt64</td>
    </tr>
    <tr>
        <td>ENUM</td>
        <td>Enum16</td>
    </tr>
    <tr>
        <td>JSON</td>
        <td>String</td>
    </tr>
    <tr>
        <td>YEAR</td>
        <td>String</td>
    </tr>
    <tr>
        <td>TIME</td>
        <td>String</td>
    </tr>
    <tr>
        <td>GEOMETRY</td>
        <td>String</td>
    </tr>
    </tbody>
</table>

不支持其他类型。如果MySQL表包含此类类型的列，ClickHouse抛出异常"Unhandled data type"并停止复制。

[Nullable](https://clickhouse.com/docs/zh/sql-reference/data-types/nullable/)已经支持。

## 同步云数据库UDB MySQL

**创建数据库和表**

- 在数据库UDB MySQL中创建数据库和表

  ```
  CREATE DATABASE mysqltest;
  CREATE TABLE mysqltest.testtable (id INT PRIMARY KEY, name VARCHAR(10));
  ```

- 在云数据仓库UClickhouse中创建同步数据库。

  ```
  set allow_experimental_database_materialize_mysql=1;
  CREATE DATABASE mysqltest ENGINE = MaterializeMySQL('MYSQL数据库地址', '数据库名', '用户名', '密码');
  ```

- 在云数据仓库UClickHouse中查询同步库中的表。

  ```
  SHOW TABLES FROM mysqltest
  ```

  查询结果如下：

  ![table_engine_mysql](images/table_engine_mysql.png)

**插入数据查询**

- 在数据库UDB MySQL中插入数据

  ```
  INSERT INTO mysqltest.testtable VALUES (1,'张三'),(2,'李四'),(3,'金刚');
  ```

- 在云数据仓库UClickhouse中查询

  ```
  SELECT * FROM mysqltest.testtable;
  ```

  结果如下：

  ![table_engine_mysql_query](images/table_engine_mysql_query.png)

**删除数据，添加列并查询**

- 在数据库UDB MySQL中删除数据，添加列并更新数据进行查询。

  ```
  DELETE FROM mysqltest.testtable WHERE id=1;
  ALTER TABLE mysqltest.testtable ADD COLUMN age INT;
  UPDATE mysqltest.testtable SET age=28 where id=3;
  SELECT * FROM testtable;
  ```

  结果如下：

  ![table_engine_mysql-2](images/table_engine_mysql-2.png)

- 在云数据仓库UClickhouse中查询

  ```
  SELECT * FROM mysqltest.testtable;
  ```

  结果如下：

  ![table_engine_mysql-3](images/table_engine_mysql-3.png)



