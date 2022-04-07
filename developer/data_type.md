# 数据类型

ClickHouse完整数据类型请参考[官方数据类型](https://clickhouse.tech/docs/zh/sql_reference/data_types/)。

下表仅列出常用的。

<table>
  <tr>
  		<th>类别</th>
      <th>数据类型</th>
      <th>取值/取值范围</th>
  </tr>
  <tr>
  	<td rowspan='4'>整数类型</td>
    <td>Int8</td>
    <td>取值范围：-128 - 127。</td>
  </tr>
  <tr>
    <td>Int16</td>
    <td>取值范围 ：-32768 - 32767。</td>
  </tr>
  <tr>
    <td>Int32</td>
    <td>取值范围： -2147483648 - 2147483647。</td>
  </tr>
  <tr>
    <td>Int64</td>
    <td>取值范围 ：-9223372036854775808 - 9223372036854775807。</td>
  </tr>
  <tr>
  	<td rowspan='2'>浮点类型</td>
    <td>Float32</td>
    <td>单精度浮点数在机内占4个字节，用32位二进制描述</td>
  </tr>
  <tr>
    <td>Float64</td>
    <td>双精度浮点数在机内占8个字节，用64位二进制描述。</td>
  </tr>
   <tr>
    <td>Decimal类型</td>
    <td>Decimal</td>
    <td>有符号的定点数，可在加、减和乘法运算过程中保持精度。支持几种写法：
      <ul>
        <li>Decimal(P, S)</li>
        <li>Decimal32(S)</li>
        <li>Decimal64(S)</li>
        <li>Decimal128(S)</li>
      </ul>
    </td>
  </tr>
  <tr>
  	<td rowspan='2'>字符串类型</td>
    <td>String</td>
    <td>字符串可以是任意长度的。它可以包含任意的字节集，包含空字节。因此，字符串类型可以代替其他 DBMSs 中的VARCHAR、BLOB、CLOB 等类型。</td>
  </tr>
  <tr>
    <td>FixedString</td>
    <td>当数据的长度恰好为N个字节时，FixedString类型是高效的。 在其他情况下，这可能会降低效率。可以有效存储在FixedString类型的列中的值的示例：
     <ul>
        <li>二进制表示的IP地址（IPv6使用FixedString（16））</li>
        <li>语言代码（ru_RU, en_US … ）</li>
        <li>货币代码（USD, RUB … ）</li>
        <li>二进制表示的哈希值（MD5使用FixedString（16），SHA256使用FixedString（32））</li>
      </ul>
    </td>
  </tr>
  <tr>
    <td rowspan="3">时间日期类型</td>
    <td>Date</td>
    <td>用两个字节存储，表示从 1970-01-01（无符号）到当前的日期值。日期中没有存储时区信息。</td>
  </tr>
  <tr>
    <td>DateTime</td>
    <td>用四个字节（无符号的）存储 Unix 时间戳。允许存储与日期类型相同的范围内的值。最小值为 0000-00-00 00:00:00。时间戳类型值精确到秒（没有闰秒）。时区使用启动客户端或服务器时的系统时区。
    </td>
   </tr>
  <tr>
    <td>Datetime64</td>
    <td>此类型允许以日期（date）加时间（time）的形式来存储一个时刻的时间值。</td>
  </tr>
  <tr>
     <td>布尔型</td>
     <td>Boolean</td>
     <td>ClickHouse没有单独的类型来存储布尔值。可以使用UInt8 类型，取值限制为0或 1。</td>
   </tr>
  <tr>
     <td >数组类型</td>
     <td>Array</td>
     <td>Array(T)，由 T 类型元素组成的数组。T 可以是任意类型，包含数组类型。但不推荐使用多维数组，ClickHouse对多维数组的支持有限。例如，不能在MergeTree表中存储多维数组。
    </td>
   </tr>
  <tr>
     <td>元组类型</td>
     <td>Tuple</td>
     <td>Tuple(T1, T2, ...)，元组，其中每个元素都有单独的类型，不能在表中存储元组（除了内存表）。它们可以用于临时列分组。在查询中，IN表达式和带特定参数的lambda 函数可以来对临时列进行分组。
      </td>
   </tr>
  <tr>
     <td>Domain数据类型</td>
     <td>Domain</td>
     <td>Domain类型是特定实现的类型： 
        <p >IPv4是与UInt32类型保持二进制兼容的Domain类型，用于存储IPv4地址的值。它提供了更为紧凑的二进制存储的同时支持识别可读性更加友好的输入输出格式。</p>
        <p >IPv6是与FixedString（16）类型保持二进制兼容的Domain类型，用于存储IPv6地址的值。它提供了更为紧凑的二进制存储的同时支持识别可读性更加友好的输入输出格式。
       </p>
     </td>
   </tr>
  <tr>
    <td rowspan="2">枚举类型</td>
    <td>Enum8</td>
    <td>取值范围：-128 - 127。</td>
   </tr>
  <tr>
    <td>Enum16</td>
    <td>取值范围 ：-32768 - 32767。</td>
   </tr>
  <tr>
    <td>可为空</td>
    <td>Nullable</td>
    <td>除非在 ClickHouse 服务器配置中另有说明，否则 NULL 是任何 Nullable 类型的默认值。Nullable 类型字段不能包含在表索引中。     </td>
  </tr>
  <tr>
     <td>嵌套类型</td>
     <td>nested</td>
     <td>嵌套的数据结构就像单元格内的表格。嵌套数据结构的参数（列名和类型）的指定方式与CREATE TABLE查询中的指定方式相同。每个表行都可以对应于嵌套数据结构中的任意数量的行。
    </td>
  </tr>
</table>

