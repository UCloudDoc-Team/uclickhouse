# Star Schema Benchmark

## 下载并编译工具

```shell
[root@xxxxx test]# git clone https://github.com/electrum/ssb-dbgen.git
[root@xxxxx test]# cd ssb-dbgen
[root@xxxxx ssb-dbgen]# make
```

## 生成数据

**生成6亿数据**

```shell
[root@xxxxx ssb-dbgen]# ./dbgen -s 100 -T c
[root@xxxxx ssb-dbgen]# ./dbgen -s 100 -T l
[root@xxxxx ssb-dbgen]# ./dbgen -s 100 -T p
[root@xxxxx ssb-dbgen]# ./dbgen -s 100 -T s
[root@xxxxx ssb-dbgen]# ./dbgen -s 100 -T d
```

| 表名           | 行数             | 大小  | 描述                 |
| -------------- | ---------------- | ----- | -------------------- |
| lineorder      | 6亿（600037902） | 67.1G | 商品订单表           |
| customer       | 300万（3000000） | 317M  | 客户表               |
| part           | 140万（1400000） | 135M  | 零部件表             |
| supplier       | 20万（200000）   | 19M   | 供应商表             |
| date           | 2556             | 272K  | 日期表               |
| lineorder_flat | 6亿（600037902） | 228G  | 打平后的宽表（单表） |

**生成30亿数据**

```shell
[root@xxxxx ssb-dbgen]# ./dbgen -s 500 -T c
[root@xxxxx ssb-dbgen]# ./dbgen -s 500 -T l
[root@xxxxx ssb-dbgen]# ./dbgen -s 500 -T p
[root@xxxxx ssb-dbgen]# ./dbgen -s 500 -T s
[root@xxxxx ssb-dbgen]# ./dbgen -s 500 -T d
```

| 表名           | 行数               | 大小 | 描述                 |
| -------------- | ------------------ | ---- | -------------------- |
| lineorder      | 30亿（3000028242） | 347G | 商品订单表           |
| customer       | 1500万（15000000） | 1.6G | 客户表               |
| part           | 180万（1800000）   | 173M | 零部件表             |
| supplier       | 100万（1000000）   | 94M  | 供应商表             |
| date           | 2556               | 272K | 日期表               |
| lineorder_flat | 30亿（3000346799） | 1.2T | 打平后的宽表（单表） |

**生成60亿数据**

```shell
[root@xxxxx ssb-dbgen]# ./dbgen -s 1000 -T c
[root@xxxxx ssb-dbgen]# ./dbgen -s 1000 -T l
[root@xxxxx ssb-dbgen]# ./dbgen -s 1000 -T p
[root@xxxxx ssb-dbgen]# ./dbgen -s 1000 -T s
[root@xxxxx ssb-dbgen]# ./dbgen -s 1000 -T d
```

| 表名           | 行数               | 大小 | 描述                 |
| -------------- | ------------------ | ---- | -------------------- |
| lineorder      | 60亿（5999989709） | 688G | 商品订单表           |
| customer       | 3000万(30000000)   | 3.2G | 客户表               |
| part           | 200万（2000000）   | 193M | 零部件表             |
| supplier       | 200万（2000000）   | 188M | 供应商表             |
| date           | 2556               | 272K | 日期表               |
| lineorder_flat | 60亿（5999989709)  | 2.3T | 打平后的宽表（单表） |

**特别说明：**

​ 如果数据量生成较大的话，**dbgen**命令支持分割文件，指定 -C 参数，即线程数。线程越大生成数据越快，建议数据量较大时并且在机器条件允许的情况下指定较大的核数。例如以下命令生成千亿级别的数据 -C 参数指定为32线程，以上数据示例也可以按需指定 -C 参数，-C 指定的数字即为分割文件的个数。

```shell
[root@xxxxx ssb-dbgen]# ./dbgen -C 32 -s 17500 -T c
[root@xxxxx ssb-dbgen]# ./dbgen -C 32 -s 17500 -T l
[root@xxxxx ssb-dbgen]# ./dbgen -C 32 -s 17500 -T p
[root@xxxxx ssb-dbgen]# ./dbgen -C 32 -s 17500 -T s
[root@xxxxx ssb-dbgen]# ./dbgen -C 32 -s 17500 -T d
```

