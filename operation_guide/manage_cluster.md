# 管理集群

## 新建集群

登录Ucloud账号进入到[用户控制台](https://passport.ucloud.cn/#login)，在全部产品下搜索或者数据仓库下选择“数据仓库 UDW Clickhouse”，进入到[数据仓库 UClickhouse控制台](https://console.ucloud.cn/udw/clickhouse)下，点击**创建集群**按钮。进入创建集群页面，根据创建页提供的配置进行选择并下单。具体请参考[快速上手](/uclickhouse/gettingstart)。

## 查看集群详情

当集群创建并运行后，您可进入[数据仓库 UClickhouse控制台](https://console.ucloud.cn/udw/clickhouse)，在集群列表，点击**详情**按钮，即可打开集群详情抽屉页。

-  集群概览：可以查看基本信息、配置信息、付费信息、分片信息、节点信息。

- 可以修改集群名称，操作节点等。

  ![clickhouse-overview](images/clickhouse-overview.png)

## 查看监控信息

在集群详情抽屉页切换到**监控信息**，可以查看集群各节点的监控指标。

![clickhouse-monitor](images/clickhouse-monitor-1.png)

![clickhouse-monitor](images/clickhouse-monitor-2.png)

![clickhouse-monitor](images/clickhouse-monitor-3.png)

![clickhouse-monitor](images/clickhouse-monitor-4.png)

## 查看参数配置

在集群详情抽屉页切换到**参数配置**，可以查看参数信息，该页面各配置项来自于config.xml配置文件，可以修改参数后，点击**应用到集群**按钮，修改后的参数即可对整个集群生效。

![clickhouse-config](images/clickhouse-config.png)

## 查看集群巡检

在集群详情抽屉页切换到**集群巡检**，可以查看当前集群的库表信息统计等。



## 查看操作日志

在集群详情抽屉页切换到**操作日志**，可以查看并追溯您的操作记录。

![clickhouse-operate-log](images/clickhouse-operate-log.png)

