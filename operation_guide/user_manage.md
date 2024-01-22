# 用户管理

UClickhouse 提供用户管理操作，包括创建账号、详情、修改权限、重置密码、删除功能。

在 UClickhouse 列表页，点击实例“详情” -> “用户管理”，可进入到“用户管理”列表页。

> 列表页默认包含 admin 管理员用户，该用户为创建实例时指定。
> 每个实例仅有一个管理员用户。

![img](/images/guide/user_manage/user-manage-button.png)

## 创建账号

点击“创建账号”，展示“创建账号”弹框，其中包括账号名称、类型、密码、备注。按需填写后，点击“确定”可完成账号创建。

> 新创建的账号为空账号，需要执行“修改权限”完成授权才能具有实际意义。

![img](/images/guide/user_manage/user-manage-create.png)

## 详情

点击“详情”，展示用户详细信息，包括基本信息：账号名称、类型、创建时间，以及有权限访问的数据库和被授予的操作权限。

![img](/images/guide/user_manage/user-manage-detail.png)

## 修改权限

点击“修改权限”，可对用户权限进行修改，包括数据库访问权限和操作权限。

![img](/images/guide/user_manage/user-manage-modify-privilege.png)

## 重置密码

点击“重置密码”，可对用户密码进行重置。

![img](/images/guide/user_manage/user-manage-reset-password.png)

## 删除

点击“删除”，可删除用户。

![img](/images/guide/user_manage/user-manage-delete.png)
