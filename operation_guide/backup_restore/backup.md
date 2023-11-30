# 备份管理

UClickhouse 提供数据备份与恢复功能，您可进入[数据仓库 UClickhouse控制台](https://console.ucloud.cn/udw/clickhouse)对集群进行周期性备份、手动备份，集群数据均被备份到 US3 存储中。

> 注意：数据备份只支持 MergeTree 系列表引擎

## 开启备份与恢复功能

备份与恢复功能默认未开启，集群首次进入“备份与恢复管理”页面时，需要先绑定US3存储空间

![img](/images/guide/backup_restore/backup_init.png)

绑定 US3 存储空间后，会进入备份列表页面。

## 备份列表

在【集群列表】页面，点击【详情】->【备份与恢复管理】

![img](/images/guide/backup_restore/backup_list.png)

列表页面展示两个模块：备份存储位置、备份列表。

备份存储位置包括US3令牌、存储空间等信息；备份列表包括备份ID、备份名称、类型、备份大小、备份开始时间、备份结束时间、状态等信息。

## 修改 US3 存储空间

> 注意：修改US3空间后，现有备份记录将被清空，US3空间中的备份文件不会联动清除，需要手动删除，可参考 [US3 工具](https://docs.ucloud.cn/ufile/tools/us3cli/command?id=rm)。

在【集群列表】页面，点击【详情】->【备份与恢复管理】，可看到备份存储位置模块

![img](/images/guide/backup_restore/backup_us3_update.png)

点击【修改】，展示“修改US3”弹框：

![img](/images/guide/backup_restore/backup_us3_update_detail.png)

## 开启自动备份策略

在【集群列表】页面，点击【详情】->【备份与恢复管理】，可看到备份模块，其中包括“自动备份策略”按钮

![img](/images/guide/backup_restore/backup_auto_policy.png)

点击【自动备份策略】，展示“自动备份策略”弹框，默认为关闭状态

![img](/images/guide/backup_restore/backup_auto_policy_close.png)

开启“自动备份策略”后，展示备份周期、备份时间、保留时长等信息。默认每天都会执行备份，可按需选择。最多可保留两年的备份数据，可按实际情况选择保留时长。

> 注意：数据备份时会消耗节点资源，建议备份时间选择在业务低峰期执行。

![img](/images/guide/backup_restore/backup_auto_policy_open.png)

## 手动备份

在【集群列表】页面，点击【详情】->【备份与恢复管理】，可看到备份模块，其中包括“手动备份”按钮

![img](/images/guide/backup_restore/backup_manual.png)

点击【手动备份】，填入备份名称后，可执行手动备份

![img](/images/guide/backup_restore/backup_manual_detail.png)

> 1. 数据备份时会消耗节点资源，建议在业务低峰期执行手动备份。
> 2. 手动备份不会自动过期。
