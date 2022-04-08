# 聚合函数

<table>
    <thead>
    <tr>
        <th>函数名称</th>
        <th>描述</th>
        <th>语法</th>
    </tr>
    </thead>
    <tbody><tr>
        <td>count</td>
        <td>统计行数或者非 NULL 值个数</td>
        <td>count(expr)、COUNT(DISTINCT expr)、count()、count(*)</td>
    </tr>
    <tr>
        <td><a href="https://clickhouse.tech/docs/en/query_language/agg_functions/reference/#agg_function-any" target="_blank">any(x)</a></td>
        <td>返回第一个遇到的值，结果不确定</td>
        <td>any(column)</td>
    </tr>
    <tr>
        <td><a href="https://clickhouse.tech/docs/en/query_language/agg_functions/reference/#anyheavyx" target="_blank">anyHeavy(x)</a></td>
        <td>基于 heavy hitters 算法，返回经常出现的值。通常结果不确定</td>
        <td>anyHeavy(column)</td>
    </tr>
    <tr>
        <td><a href="https://clickhouse.tech/docs/en/query_language/agg_functions/reference/#anylastx" target="_blank">anyLast(x)</a></td>
        <td>返回最后一个遇到的值，结果不确定</td>
        <td>anyLast(column)</td>
    </tr>
    <tr>
        <td><a href="https://clickhouse.tech/docs/en/query_language/agg_functions/reference/#groupbitand" target="_blank">groupBitAnd</a></td>
        <td>按位与</td>
        <td>groupBitAnd(expr)</td>
    </tr>
    <tr>
        <td><a href="https://clickhouse.tech/docs/en/query_language/agg_functions/reference/#groupbitor" target="_blank">groupBitOr</a></td>
        <td>按位或</td>
        <td>groupBitOr(expr)</td>
    </tr>
    <tr>
        <td><a href="https://clickhouse.tech/docs/en/query_language/agg_functions/reference/#groupbitxor" target="_blank">groupBitXor</a></td>
        <td>按位异或</td>
        <td>groupBitXor(expr)</td>
    </tr>
    <tr>
        <td><a href="https://clickhouse.tech/docs/en/query_language/agg_functions/reference/#groupbitmap" target="_blank">groupBitmap</a></td>
        <td>求基数（cardinality）</td>
        <td>groupBitmap(expr)</td>
    </tr>
    <tr>
        <td><a href="https://clickhouse.tech/docs/en/query_language/agg_functions/reference/#agg_function-min" target="_blank">min(x)</a></td>
        <td>求最小值</td>
        <td>min(column)</td>
    </tr>
    <tr>
        <td><a href="https://clickhouse.tech/docs/en/query_language/agg_functions/reference/#agg_function-max" target="_blank">max(x)</a></td>
        <td>求最大值</td>
        <td>max(x)</td>
    </tr>
    <tr>
        <td><a href="https://clickhouse.tech/docs/en/query_language/agg_functions/reference/#agg-function-argmin" target="_blank">argMin(arg, val)</a></td>
        <td>返回 val 最小值行的 arg 的值</td>
        <td>argMin(c1, c2)</td>
    </tr>
    <tr>
        <td><a href="https://clickhouse.tech/docs/en/query_language/agg_functions/reference/#agg-function-argmax" target="_blank">argMax(arg, val)</a></td>
        <td>返回 val 最大值行的 arg 的值</td>
        <td>argMax(c1, c2)</td>
    </tr>
    <tr>
        <td><a href="https://clickhouse.tech/docs/en/query_language/agg_functions/reference/#agg_function-sum" target="_blank">sum(x)</a></td>
        <td>求和</td>
        <td>sum(x)</td>
    </tr>
    <tr>
        <td><a href="https://clickhouse.tech/docs/en/query_language/agg_functions/reference/#sumwithoverflowx" target="_blank">sumWithOverflow(x)</a></td>
        <td>求和，结果溢出则返回错误</td>
        <td>sumWithOverflow(x)</td>
    </tr>
    <tr>
        <td><a href="https://clickhouse.tech/docs/en/query_language/agg_functions/reference/#agg_functions-summap" target="_blank">sumMap(key, value)</a></td>
        <td>用于数组类型，对相同 key 的 value 求和，返回两个数组的 tuple，第一个为排序后的 key，第二个为对应 key 的 value 之和</td>
        <td>-</td>
    </tr>
    <tr>
        <td><a href="https://clickhouse.tech/docs/en/query_language/agg_functions/reference/#skewpop" target="_blank">skewPop</a></td>
        <td>求 <a href="https://en.wikipedia.org/wiki/Skewness" target="_blank">偏度</a></td>
        <td>skewPop(expr)</td>
    </tr>
    <tr>
        <td><a href="https://clickhouse.tech/docs/en/query_language/agg_functions/reference/#skewsamp" target="_blank">skewSamp</a></td>
        <td>求 <a href="https://en.wikipedia.org/wiki/Skewness" target="_blank">样本偏度</a></td>
        <td>skewSamp(expr)</td>
    </tr>
    <tr>
        <td><a href="https://clickhouse.tech/docs/en/query_language/agg_functions/reference/#kurtpop" target="_blank">kurtPop</a></td>
        <td>求 <a href="https://en.wikipedia.org/wiki/Kurtosis" target="_blank">峰度</a></td>
        <td>kurtPop(expr)</td>
    </tr>
    <tr>
        <td><a href="https://clickhouse.tech/docs/en/query_language/agg_functions/reference/#kurtsamp" target="_blank">kurtSamp</a></td>
        <td>求 <a href="https://en.wikipedia.org/wiki/Kurtosis" target="_blank">样本峰度</a></td>
        <td>kurtSamp(expr)</td>
    </tr>
    <tr>
        <td><a href="https://clickhouse.tech/docs/en/query_language/agg_functions/reference/#agg-function-timeseriesgroupsum" target="_blank">timeSeriesGroupSum(uid, timestamp,   value)</a></td>
        <td>对 uid 分组的时间序列对应时间点求和，求和前缺失的时间点线性插值</td>
        <td>-</td>
    </tr>
    <tr>
        <td><a href="https://clickhouse.tech/docs/en/query_language/agg_functions/reference/#agg-function-timeseriesgroupratesum" target="_blank">timeSeriesGroupRateSum(uid, ts,   val)</a></td>
        <td>对 uid 分组的时间序列对应时间点的变化率求和</td>
        <td>-</td>
    </tr>
    <tr>
        <td><a href="https://clickhouse.tech/docs/en/query_language/agg_functions/reference/#agg_function-avg" target="_blank">avg(x)</a></td>
        <td>求平均值</td>
        <td>-</td>
    </tr>
    <tr>
        <td><a href="https://clickhouse.tech/docs/en/query_language/agg_functions/reference/#agg_function-uniq" target="_blank">uniq</a></td>
        <td>计算不同值的近似个数</td>
        <td>uniq(x[, ...])</td>
    </tr>
    <tr>
        <td><a href="https://clickhouse.tech/docs/en/query_language/agg_functions/reference/#agg_function-uniqcombined" target="_blank">uniqCombined</a></td>
        <td>计算不同值的近似个数，相比 uniq 消耗的内存更少，精度更高，但是性能稍差</td>
        <td>uniqCombined(HLL_precision)(x[, ...])、uniqCombined(x[,  ...])</td>
    </tr>
    <tr>
        <td><a href="https://clickhouse.tech/docs/en/query_language/agg_functions/reference/#agg_function-uniqcombined64" target="_blank">uniqCombined64</a></td>
        <td>uniqCombined 的 64bit 版本，结果溢出的可能性降低</td>
        <td>-</td>
    </tr>
    <tr>
        <td><a href="https://clickhouse.tech/docs/en/query_language/agg_functions/reference/#agg_function-uniqhll12" target="_blank">uniqHLL12</a></td>
        <td>计算不同值的近似个数，不建议使用。请用 uniq、uniqCombined</td>
        <td>-</td>
    </tr>
    <tr>
        <td><a href="https://clickhouse.tech/docs/en/query_language/agg_functions/reference/#agg_function-uniqexact" target="_blank">uniqExact</a></td>
        <td>计算不同值的精确个数</td>
        <td>uniqExact(x[, ...])</td>
    </tr>
    <tr>
        <td><a href="https://clickhouse.tech/docs/en/query_language/agg_functions/reference/#agg_function-grouparray" target="_blank">groupArray(x),   groupArray(max_size)(x)</a></td>
        <td>返回 x 取值的数组，数组大小可由 max_size 指定</td>
        <td>-</td>
    </tr>
    <tr>
        <td><a href="https://clickhouse.tech/docs/en/query_language/agg_functions/reference/#grouparrayinsertatvalue-position" target="_blank">groupArrayInsertAt(value, position)</a></td>
        <td>在数组的指定位置 position 插入值 value</td>
        <td>-</td>
    </tr>
    <tr>
        <td><a href="https://clickhouse.tech/docs/en/query_language/agg_functions/reference/#agg_function-grouparraymovingsum" target="_blank">groupArrayMovingSum</a></td>
        <td>-</td>
        <td>-</td>
    </tr>
    <tr>
        <td><a href="https://clickhouse.tech/docs/en/query_language/agg_functions/reference/#agg_function-grouparraymovingavg" target="_blank">groupArrayMovingAvg</a></td>
        <td>-</td>
        <td>-</td>
    </tr>
    <tr>
        <td><a href="https://clickhouse.tech/docs/en/query_language/agg_functions/reference/#groupuniqarrayx-groupuniqarraymax-sizex" target="_blank">groupUniqArray(x),   groupUniqArray(max_size)(x)</a></td>
        <td>-</td>
        <td>-</td>
    </tr>
    <tr>
        <td><a href="https://clickhouse.tech/docs/en/query_language/agg_functions/reference/#quantile" target="_blank">quantile</a></td>
        <td>-</td>
        <td>-</td>
    </tr>
    <tr>
        <td><a href="https://clickhouse.tech/docs/en/query_language/agg_functions/reference/#quantiledeterministic" target="_blank">quantileDeterministic</a></td>
        <td>-</td>
        <td>-</td>
    </tr>
    <tr>
        <td><a href="https://clickhouse.tech/docs/en/query_language/agg_functions/reference/#quantileexact" target="_blank">quantileExact</a></td>
        <td>-</td>
        <td>-</td>
    </tr>
    <tr>
        <td><a href="https://clickhouse.tech/docs/en/query_language/agg_functions/reference/#quantileexactweighted" target="_blank">quantileExactWeighted</a></td>
        <td>-</td>
        <td>-</td>
    </tr>
    <tr>
        <td><a href="https://clickhouse.tech/docs/en/query_language/agg_functions/reference/#quantiletiming" target="_blank">quantileTiming</a></td>
        <td>-</td>
        <td>-</td>
    </tr>
    <tr>
        <td><a href="https://clickhouse.tech/docs/en/query_language/agg_functions/reference/#quantiletimingweighted" target="_blank">quantileTimingWeighted</a></td>
        <td>-</td>
        <td>-</td>
    </tr>
    <tr>
        <td><a href="https://clickhouse.tech/docs/en/query_language/agg_functions/reference/#quantiletdigest" target="_blank">quantileTDigest</a></td>
        <td>-</td>
        <td>-</td>
    </tr>
    <tr>
        <td><a href="https://clickhouse.tech/docs/en/query_language/agg_functions/reference/#quantiletdigestweighted" target="_blank">quantileTDigestWeighted</a></td>
        <td>-</td>
        <td>-</td>
    </tr>
    <tr>
        <td><a href="https://clickhouse.tech/docs/en/query_language/agg_functions/reference/#median" target="_blank">median</a></td>
        <td>-</td>
        <td>-</td>
    </tr>
    <tr>
        <td><a href="https://clickhouse.tech/docs/en/query_language/agg_functions/reference/#quantiles" target="_blank">quantiles(level1, level2, …)(x)</a></td>
        <td>-</td>
        <td>-</td>
    </tr>
    <tr>
        <td><a href="https://clickhouse.tech/docs/en/query_language/agg_functions/reference/#varsampx" target="_blank">varSamp(x)</a></td>
        <td>-</td>
        <td>-</td>
    </tr>
    <tr>
        <td><a href="https://clickhouse.tech/docs/en/query_language/agg_functions/reference/#varpopx" target="_blank">varPop(x)</a></td>
        <td>-</td>
        <td>-</td>
    </tr>
    <tr>
        <td><a href="https://clickhouse.tech/docs/en/query_language/agg_functions/reference/#stddevsampx" target="_blank">stddevSamp(x)</a></td>
        <td>-</td>
        <td>-</td>
    </tr>
    <tr>
        <td><a href="https://clickhouse.tech/docs/en/query_language/agg_functions/reference/#stddevpopx" target="_blank">stddevPop(x)</a></td>
        <td>-</td>
        <td>-</td>
    </tr>
    <tr>
        <td><a href="https://clickhouse.tech/docs/en/query_language/agg_functions/reference/#topknx" target="_blank">topK(N)(x)</a></td>
        <td>-</td>
        <td>-</td>
    </tr>
    <tr>
        <td><a href="https://clickhouse.tech/docs/en/query_language/agg_functions/reference/#topkweighted" target="_blank">topKWeighted</a></td>
        <td>-</td>
        <td>-</td>
    </tr>
    <tr>
        <td><a href="https://clickhouse.tech/docs/en/query_language/agg_functions/reference/#covarsampx-y" target="_blank">covarSamp(x, y)</a></td>
        <td>-</td>
        <td>-</td>
    </tr>
    <tr>
        <td><a href="https://clickhouse.tech/docs/en/query_language/agg_functions/reference/#covarpopx-y" target="_blank">covarPop(x, y)</a></td>
        <td>-</td>
        <td>-</td>
    </tr>
    <tr>
        <td><a href="https://clickhouse.tech/docs/en/query_language/agg_functions/reference/#corrx-y" target="_blank">corr(x, y)</a></td>
        <td>-</td>
        <td>-</td>
    </tr>
    <tr>
        <td><a href="https://clickhouse.tech/docs/en/query_language/agg_functions/reference/#categoricalinformationvalue" target="_blank">categoricalInformationValue</a></td>
        <td>-</td>
        <td>-</td>
    </tr>
    <tr>
        <td><a href="https://clickhouse.tech/docs/en/query_language/agg_functions/reference/#simplelinearregression" target="_blank">simpleLinearRegression</a></td>
        <td>-</td>
        <td>-</td>
    </tr>
    <tr>
        <td><a href="https://clickhouse.tech/docs/en/query_language/agg_functions/reference/#agg_functions-stochasticlinearregression" target="_blank">stochasticLinearRegression</a></td>
        <td>-</td>
        <td>-</td>
    </tr>
    <tr>
        <td><a href="https://clickhouse.tech/docs/en/query_language/agg_functions/reference/#agg_functions-stochasticlogisticregression" target="_blank">stochasticLogisticRegression</a></td>
        <td>-</td>
        <td>-</td>
    </tr>
    <tr>
        <td><a href="https://clickhouse.tech/docs/en/query_language/agg_functions/reference/#groupbitmapand" target="_blank">groupBitmapAnd</a></td>
        <td>-</td>
        <td>-</td>
    </tr>
    <tr>
        <td><a href="https://clickhouse.tech/docs/en/query_language/agg_functions/reference/#groupbitmapor" target="_blank">groupBitmapOr</a></td>
        <td>-</td>
        <td>-</td>
    </tr>
    <tr>
        <td><a href="https://clickhouse.tech/docs/en/query_language/agg_functions/reference/#groupbitmapxor" target="_blank">groupBitmapXor</a></td>
        <td>-</td>
        <td>-</td>
    </tr>
    </tbody></table>

