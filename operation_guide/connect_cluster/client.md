# ClickHouse命令行工具连接

UClickhouse支持通过多种方式连接实例，包括第三方客户端工具。您可以综合考虑维护人员、场景需要、需要执行的操作等因素，选择合适的连接方式。以下为使用ClickHouse命令行工具连接集群的方式步骤及操作示例。

### 操作步骤

  1. 登录UCloud账号进入到[用户控制台](https://passport.ucloud.cn/#login)，在全部产品下搜索或者数据仓库下选择“数据仓库 UDW Clickhouse”，进入到[数据仓库 UClickhouse控制台](https://console.ucloud.cn/udw/clickhouse)

  2. 在**集群列表**页面，点击**详情**可查看集群节点列表，列表中已列出节点地址。

  3. 根据操作系统类型及集群内核版本下载对应版本clickhouse-client，官方下载地址：[下载Clickhouse-client](https://packages.clickhouse.com/rpm/lts/)。安装请参见[[快速上手](/uclickhouse/gettingstart)](/uclickhouse/)

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

  1. 登录UCloud账号进入到[用户控制台](https://passport.ucloud.cn/#login)，在全部产品下搜索或者数据仓库下选择“数据仓库 UDW Clickhouse”，进入到[数据仓库 UClickhouse控制台](https://console.ucloud.cn/udw/clickhouse)

  2. 在**集群列表**页面，点击**详情**可查看集群节点列表，列表中已列出节点地址。

  3. 连接云数据仓库UClickhouse，在同一地域下的云主机上执行以下命令。

     ```
     echo "select * from ck_test.lineorder"  |  curl  'http://用户名:密码@任一节点IP地址:8123/' --data-binary @-
     ```

## JDBC连接数据库

### 操作步骤

1. 使用maven管理项目下载依赖包：

```xml
<dependencies>
    <dependency>
        <groupId>ru.yandex.clickhouse</groupId>
        <artifactId>clickhouse-jdbc</artifactId>
        <version>0.1.40</version>
    </dependency>
</dependencies>
```

2. 代码示例：

```java
import java.sql.*;

public class JDBC_Clickhouse {

    private static Connection connection = null;

    static {
            try {
                Class.forName("ru.yandex.clickhouse.ClickHouseDriver");   //加载驱动包
                String url = "jdbc:clickhouse://node_ip:8123/system";   //访问url路径，node_ip为访问节点ip
                String user = "admin";   //登陆集群的账号
                String password = "password";  //登陆集群的密码
                connection = DriverManager.getConnection(url, user, password);
            } catch (Exception e) {
                e.printStackTrace();
            }

    }
    public static void main(String[] args) throws SQLException{
        Statement statement = connection.createStatement();
        ResultSet resultSet = statement.executeQuery("select * from system.clusters");
        ResultSetMetaData metaData = resultSet.getMetaData();
        int columnCount = metaData.getColumnCount();
        while (resultSet.next()) {
            for (int i = 1; i <= columnCount; ++i) {
                System.out.println(metaData.getColumnName(i) + ":" + resultSet.getString(i));
            }
        }
    }
}
```

