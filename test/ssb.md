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

| 表名      | 行数             | 大小  | 描述       |
| --------- | ---------------- | ----- | ---------- |
| lineorder | 6亿（600037902） | 67.1G | 商品订单表 |
| customer  | 300万（3000000） | 317M  | 客户表     |
| part      | 140万（1400000） | 135M  | 零部件表   |
| supplier  | 20万（200000）   | 19M   | 供应商表   |
| date      | 2556             | 272K  | 日期表     |

