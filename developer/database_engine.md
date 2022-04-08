# 数据库引擎

### 延时引擎Lazy

在距最近一次访问间隔expiration_time_in_seconds时间段内，将表保存在内存中，仅适用于&Log引擎表
由于针对这类表的访问间隔较长，对保存大量的*Log引擎表进行了优化
创建数据库

```
CREATE DATABASE testlazy ENGINE = Lazy(expiration_time_in_seconds);
```

### Atomic

支持非阻塞的 drop 和 rename 表查询以及原子交换表t1和t2查询。默认情况下使用 Atomic 数据库引擎。
创建数据库

```
CREATE DATABASE test ENGINE = Atomic
```

### MySQL

MySQL引擎用于将远程的myqsl服务器中的表映射到clickhouse中，并允许对表进行 insert 和 select 查询，方便在clickhouse与mysql之间进行数据交换。
MySQL数据库引擎会将对其的查询转换为mysql预防并发送到mysql服务器中，因此，可以执行 show tables 或者 show create table 之类的操作。
无法进行的操作（rename、create table、alter）
创建数据库

```
CREATE DATABASE[IF NOT EXISTS] db_name[ON CLUSTER cluster] ENGINE = MySQL('host:port',['database'|database],'user','password')
```

MySQL数据库引擎参数:

* host：port  --  链接的mysql地址
* database  --  链接的mysql数据库
* user  --  链接的mysql用户
* password  --  链接的mysql用户密码

MySQL中的数据类型与ClickHouse类型映射关系如下表：

| MySQL                            | ClickHouse  |
| -------------------------------- | ----------- |
| UNSIGNED TINYINT                 | UInt8       |
| TINYINT                          | Int8        |
| UNSIGNED SMALLINT                | UInt16      |
| SMALLINT                         | Int16       |
| UNSIGNED INT, UNSIGNED MEDIUMINT | UInt32      |
| INT, MEDIUMINT                   | Int32       |
| UNSIGNED BIGINT                  | UInt64      |
| BIGINT                           | Int64       |
| FLOAT                            | Float32     |
| DOUBLE                           | Float64     |
| DATE                             | Date        |
| DATETIME, TIMESTAMP              | DateTime    |
| BINARY                           | FixedString |
