# 访问数据仓库

## JDBC连接数据库

1.使用maven管理项目下载依赖包：

```
    <dependencies>
        <dependency>
            <groupId>ru.yandex.clickhouse</groupId>
            <artifactId>clickhouse-jdbc</artifactId>
            <version>0.1.40</version>
        </dependency>
    </dependencies>
```
2.代码实例：

```
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
