# 从本地导入数据

使用 clickhouse-client客户端工具从本地文件中导入数据

下载clickhouse-client客户端:
下载版本（clickhouse‐client‐21.1.2.15‐2.noarch.rpm）
[https://repo.yandex.ru/clickhouse/rpm/stable/x86_64/](https://repo.yandex.ru/clickhouse/rpm/stable/x86_64/)

使用命令进行导入：
```
cat <data_file> | ./clickhouse-client --host=<host> --port=<port> --user=<username> --password=<password> --query="INSERT INTO <table_name> FORMAT <format>";
```



