# 查看信息

## SHOW语句

**基本语法：**

```
SHOW DATABASES [INTO OUTFILE filename] [FORMAT format]SHOW PROCESSLIST [INTO OUTFILE filename] [FORMAT format]SHOW [TEMPORARY] TABLES [{FROM | IN} <db>] [LIKE '<pattern>' | WHERE expr] [LIMIT <N>] [INTO OUTFILE <filename>] [FORMAT <format>]SHOW DICTIONARIES [FROM <db>] [LIKE '<pattern>'] [LIMIT <N>] [INTO OUTFILE <filename>] [FORMAT <format>]
```

更多信息，请参考[官方文档](https://clickhouse.tech/docs/en/query_language/show/)

## DESCRIBE 语句

**基本语法：**

```
DESC|DESCRIBE TABLE [db.]table [INTO OUTFILE filename] [FORMAT format]
```

