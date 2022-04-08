# 常规函数

## 数学函数

<table>
    <thead>
    <tr>
        <th>函数名称</th>
        <th>描述</th>
        <th>语法</th>
    </tr>
    </thead>
    <tbody><tr>
        <td>plus(a, b), a + b</td>
        <td>计算两个字段的和</td>
        <td>plus(table.field1, table.field2)</td>
    </tr>
    <tr>
        <td>minus(a, b), a - b</td>
        <td>计算两个字段的差</td>
        <td>minus(table.field1, table.field2)</td>
    </tr>
    <tr>
        <td>multiply(a, b), a * b</td>
        <td>计算两个字段的积</td>
        <td>multiply(table.field1, table.field2)</td>
    </tr>
    <tr>
        <td>divide(a, b), a / b</td>
        <td>计算两个字段的商</td>
        <td>divide(table.field1, table.field2)</td>
    </tr>
    <tr>
        <td>modulo(a, b), a % b</td>
        <td>计算两个字段的余数</td>
        <td>modulo(table.field1, table.field2)</td>
    </tr>
    <tr>
        <td>abs(a)</td>
        <td>取绝对值</td>
        <td>abs(table.field1)</td>
    </tr>
    <tr>
        <td>negate(a)</td>
        <td>取相反数</td>
        <td>negate(table.field1)</td>
    </tr>
    </tbody></table>

## 比较函数

<table>
    <thead>
    <tr>
        <th>函数名称</th>
        <th>描述</th>
        <th>语法</th>
    </tr>
    </thead>
    <tbody><tr>
        <td>=, ==</td>
        <td>判断是否相等</td>
        <td>table.field1 = value</td>
    </tr>
    <tr>
        <td>!=, &lt;&gt;</td>
        <td>判断是否不相等</td>
        <td>table.field1 != value</td>
    </tr>
    <tr>
        <td>&gt;</td>
        <td>判断是否大于</td>
        <td>table.field1 &gt; value</td>
    </tr>
    <tr>
        <td>&gt;=</td>
        <td>判断是否大于等于</td>
        <td>table.field1 &gt;= value</td>
    </tr>
    <tr>
        <td>&lt;</td>
        <td>判断是否小于</td>
        <td>table.field1 &lt; value</td>
    </tr>
    <tr>
        <td>&lt;=</td>
        <td>判断是否小于等于</td>
        <td>table.field1 &lt;= value</td>
    </tr>
    </tbody></table>

## 逻辑运算函数

<table>
    <thead>
    <tr>
        <th>函数名称</th>
        <th>描述</th>
        <th>语法</th>
    </tr>
    </thead>
    <tbody><tr>
        <td>AND</td>
        <td>两个条件都必须满足</td>
        <td>-</td>
    </tr>
    <tr>
        <td>OR</td>
        <td>两个条件满足其中之一</td>
        <td>-</td>
    </tr>
    <tr>
        <td>NOT</td>
        <td>取条件判断的相反</td>
        <td>-</td>
    </tr>
    </tbody></table>

## 类型转换函数

**注意：**转换函数可能会溢出，溢出后的数字与 C 语言中数据类型保持一致。

<table>
    <thead>
    <tr>
        <th>函数名称</th>
        <th>用途</th>
        <th>使用场景</th>
    </tr>
    </thead>
    <tbody><tr>
        <td>toInt(8|16|32|64)</td>
        <td>将字符型转化为整数型</td>
        <td>toInt8('128') 结果为-127</td>
    </tr>
    <tr>
        <td>toUInt(8|16|32|64)</td>
        <td>将字符型转化为无符号整数型</td>
        <td>toUInt8('128') 结果为128</td>
    </tr>
    <tr>
        <td>toInt(8|16|32|64)OrZero</td>
        <td>将整数字符型转化为整数型，异常时返回0</td>
        <td>toInt8OrZero('a') 结果为0</td>
    </tr>
    <tr>
        <td>toUInt(8|16|32|64)OrZero</td>
        <td>将整数字符型转化为整数型，异常时返回0</td>
        <td>toUInt8OrZero('a') 结果为0</td>
    </tr>
    <tr>
        <td>toInt(8|16|32|64)OrNull</td>
        <td>将整数字符型转化为整数型，异常时返回 NULL</td>
        <td>toInt8OrNull('a') 结果为 NULL</td>
    </tr>
    <tr>
        <td>toUInt(8|16|32|64)OrNull</td>
        <td>将整数字符型转化为整数型，异常时返回 NULL</td>
        <td>toUInt8OrNull('a') 结果为 NULL</td>
    </tr>
    </tbody></table>

浮点数类型或日期类型也有上述类似的函数。具体请参考[官方文档](https://clickhouse.tech/docs/en/query_language/functions/type_conversion_functions/)

## 日期函数

具体请参考[官方文档](https://clickhouse.tech/docs/en/query_language/functions/date_time_functions/)。

## 字符串函数

具体请参考[官方文档](https://clickhouse.tech/docs/en/query_language/functions/string_functions/)。

## UUID

具体请参考[官方文档](https://clickhouse.tech/docs/en/query_language/functions/uuid_functions/) 。

## JSON 处理函数

具体请参考[官方文档](https://clickhouse.tech/docs/en/query_language/functions/json_functions/)。