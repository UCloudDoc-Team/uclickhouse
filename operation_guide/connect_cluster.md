# 连接集群

## ClickHouse命令行工具连接

### 操作步骤

  1. 登录Ucloud账号进入到[用户控制台](https://passport.ucloud.cn/#login)，在全部产品下搜索或者数据仓库下选择“数据仓库 UDW Clickhouse”，进入到[数据仓库 UClickhouse控制台](https://console.ucloud.cn/udw/clickhouse)

  2. 在**集群列表**页面，点击**详情**可查看集群节点列表，列表中已列出节点地址。

  3. 根据操作系统类型及集群内核版本下载对应版本clickhouse-client，下载地址参见[下载Clickhouse-client](https://repo.yandex.ru/clickhouse/rpm/stable/x86_64/)。安装请参见[[快速上手](/uclickhouse/gettingstart)](/uclickhouse/)

  4. 连接云数据仓库UClickhouse，在同一地域下的云主机上执行以下命令。

     ```
     clickhouse-client --host=<host> --port=<port> --user=<user> --password=<password>; 
     ```

参数说明如下

| 参数     | 说明                                                         |
| -------- | ------------------------------------------------------------ |
| host     | 节点的ip地址。<br />要求clickhouse-client所在云主机和云数据仓库 UClickhouse在同一地域内（同一网段） |
| port     | TCP端口号，默认9000                                          |
| user     | 管理用户名，默认admin                                        |
| password | 您创建集群时设置的管理密码                                   |

## 通过HTTP操作集群

### 操作步骤

  1. 登录Ucloud账号进入到[用户控制台](https://passport.ucloud.cn/#login)，在全部产品下搜索或者数据仓库下选择“数据仓库 UDW Clickhouse”，进入到[数据仓库 UClickhouse控制台](https://console.ucloud.cn/udw/clickhouse)

  2. 在**集群列表**页面，点击**详情**可查看集群节点列表，列表中已列出节点地址。

  3. 连接云数据仓库UClickhouse，在同一地域下的云主机上执行以下命令。

     ```
     echo "select * from ck_test.lineorder"  |  curl  'http://用户名:密码@任一节点IP地址:8123/' --data-binary @-
     ```