# 插入数据（INSERT INTO）

**基本语法：**

```
INSERT INTO [db.]table [(c1, c2, c3)] VALUES (v11, v12, v13), (v21, v22, v23), ...
```

对于存在于表结构中但不存在于插入列表中的列，它们将会按照如下方式填充数据：

- 如果存在DEFAULT表达式，根据DEFAULT表达式计算被填充的值。
- 如果没有定义DEFAULT表达式，则填充零或空字符串。

## 使用SELECT的结果写入

**基本语法：**

```
INSERT INTO [db.]table [(c1, c2, c3)] SELECT ...
```

写入的列与SELECT的列的对应关系是使用位置来进行对应的，它们在SELECT表达式与INSERT中的名称可以是不同的。需要对它们进行对应的类型转换。

除了VALUES格式之外，其他格式中的数据都不允许出现诸如now()，1 + 2等表达式。VALUES格式允许您有限度的使用这些表达式，但是不建议您这么做，因为执行这些表达式很低效。

## 影响性能的注意事项

在执行INSERT时将会对写入的数据进行一些处理，比如按照主键排序、按照月份对数据进行分区等。如果在您的写入数据中包含多个月份的混合数据时，将会显著地降低INSERT的性能。为了避免这种情况，通常采用以下方式：

- 数据总是以尽量大的batch进行写入，如每次写入100,000行。
- 数据在写入ClickHouse前预先对数据进行分组。

在以下的情况下，性能不会下降：

- 数据总是被实时地写入。
- 写入的数据已经按照时间排序。