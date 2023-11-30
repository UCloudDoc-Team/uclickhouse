# 恢复管理

UClickhouse 支持将备份数据恢复至一个新实例。数据恢复时，会将源实例备份数据全量恢复到新实例。新实例配置必须与源实例配置一致。

## 恢复至新实例

在【集群列表】页面，点击【详情】->【备份与恢复管理】，可看到备份模块，备份模块下包括所有备份数据记录，“已完成”状态的实例可“恢复至新实例”

![img](/images/guide/backup_restore/restore_button.png)

点击【恢复至新实例】后，展示恢复新实例弹窗，按需选择配置后，需支付购买新实例

> 注意：恢复创建的新实例配置必须与源实例一致。

![img](/images/guide/backup_restore/restore_detail.png)

## 恢复列表

在【集群列表】页面，点击【详情】->【备份与恢复管理】->【恢复管理】，恢复列表包含任务ID、目标实例ID、备份ID、恢复开始时间、恢复结束时间、实例状态等信息

![img](/images/guide/backup_restore/restore_list.png)
