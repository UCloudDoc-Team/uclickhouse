# 多表性能测试语句

## 基于Star Schema Benchmark工具

- #### Q1.1

  ```sql
  SELECT SUM(LO_REVENUE) AS REVENUE
  FROM lineorder
  GLOBAL INNER JOIN dates ON LO_ORDERDATE = toDate(replaceRegexpAll(toString(D_DATEKEY), '(\\d{4})(\\d{2})(\\d{2})', '\\1-\\2-\\3'))
  WHERE (D_YEAR = 1993) AND ((LO_DISCOUNT >= 1) AND (LO_DISCOUNT <= 3)) AND (LO_QUANTITY < 25)
  ```

- #### Q1.2

  ```sql
  SELECT SUM(LO_REVENUE) AS REVENUE
  FROM lineorder
  GLOBAL INNER JOIN dates ON LO_ORDERDATE = toDate(replaceRegexpAll(toString(D_DATEKEY), '(\\d{4})(\\d{2})(\\d{2})', '\\1-\\2-\\3'))
  WHERE (D_YEARMONTHNUM = 199401) AND ((LO_DISCOUNT >= 4) AND (LO_DISCOUNT <= 6)) AND ((LO_QUANTITY >= 26) AND (LO_QUANTITY <= 35))
  ```

- #### Q1.3

  ```sql
  SELECT SUM(LO_REVENUE) AS REVENUE
  FROM lineorder
  GLOBAL INNER JOIN dates ON LO_ORDERDATE = toDate(replaceRegexpAll(toString(D_DATEKEY), '(\\d{4})(\\d{2})(\\d{2})', '\\1-\\2-\\3'))
  WHERE (D_WEEKNUMINYEAR = 6) AND (D_YEAR = 1994) AND ((LO_DISCOUNT >= 5) AND (LO_DISCOUNT <= 7)) AND ((LO_QUANTITY >= 26) AND (LO_QUANTITY <= 35))
  ```

- #### Q2.1

  ```sql
  SELECT SUM(LO_REVENUE) AS LO_REVENUE, D_YEAR, P_BRAND
  FROM lineorder
  GLOBAL JOIN part ON LO_PARTKEY = P_PARTKEY
  GLOBAL JOIN supplier ON LO_SUPPKEY = S_SUPPKEY
  GLOBAL INNER JOIN dates ON LO_ORDERDATE = toDate(replaceRegexpAll(toString(D_DATEKEY),'(\\d{4})(\\d{2})(\\d{2})', '\\1-\\2-\\3'))
  WHERE P_CATEGORY = 'MFGR#12' AND S_REGION = 'AMERICA'
  GROUP BY D_YEAR, P_BRAND
  ORDER BY D_YEAR, P_BRAND
  ```

- #### Q2.2

  ```sql
  SELECT SUM(LO_REVENUE) AS LO_REVENUE, D_YEAR, P_BRAND
  FROM lineorder
  GLOBAL JOIN part ON LO_PARTKEY = P_PARTKEY
  GLOBAL JOIN supplier ON LO_SUPPKEY = S_SUPPKEY
  GLOBAL JOIN dates ON LO_ORDERDATE = toDate(replaceRegexpAll(toString(D_DATEKEY),'(\\d{4})(\\d{2})(\\d{2})', '\\1-\\2-\\3'))
  WHERE P_BRAND BETWEEN 'MFGR#2221' AND 'MFGR#2228' AND S_REGION = 'ASIA'
  GROUP BY D_YEAR, P_BRAND
  ORDER BY D_YEAR, P_BRAND
  ```

- #### Q2.3

  ```sql
  SELECT
      SUM(LO_REVENUE) AS LO_REVENUE,
      D_YEAR,
      P_BRAND
  FROM lineorder
  GLOBAL INNER JOIN part ON LO_PARTKEY = P_PARTKEY
  GLOBAL INNER JOIN supplier ON LO_SUPPKEY = S_SUPPKEY
  GLOBAL INNER JOIN dates ON LO_ORDERDATE = toDate(replaceRegexpAll(toString(D_DATEKEY), '(\\d{4})(\\d{2})(\\d{2})', '\\1-\\2-\\3'))
  WHERE (P_BRAND = 'MFGR#2239') AND (S_REGION = 'EUROPE')
  GROUP BY
      D_YEAR,
      P_BRAND
  ORDER BY
      D_YEAR ASC,
      P_BRAND ASC
  ```

- #### Q3.1

  ```sql
  SELECT C_NATION, S_NATION, D_YEAR, SUM(LO_REVENUE) AS LO_REVENUE
  FROM lineorder
  GLOBAL JOIN customer ON LO_CUSTKEY = C_CUSTKEY
  GLOBAL JOIN supplier ON LO_SUPPKEY = S_SUPPKEY
  GLOBAL JOIN dates ON LO_ORDERDATE = toDate(replaceRegexpAll(toString(D_DATEKEY),'(\\d{4})(\\d{2})(\\d{2})', '\\1-\\2-\\3'))
  WHERE C_REGION = 'ASIA' AND S_REGION = 'ASIA'AND D_YEAR >= 1992 AND D_YEAR <= 1997
  GROUP BY C_NATION, S_NATION, D_YEAR
  ORDER BY D_YEAR ASC, LO_REVENUE DESC
  ```

- #### Q3.2

  ```sql
  SELECT C_CITY, S_CITY, D_YEAR, SUM(LO_REVENUE) AS LO_REVENUE
  FROM lineorder
  GLOBAL JOIN customer ON LO_CUSTKEY = C_CUSTKEY
  GLOBAL JOIN supplier ON LO_SUPPKEY = S_SUPPKEY
  GLOBAL JOIN dates ON LO_ORDERDATE = toDate(replaceRegexpAll(toString(D_DATEKEY),'(\\d{4})(\\d{2})(\\d{2})', '\\1-\\2-\\3'))
  WHERE C_NATION = 'UNITED STATES' AND S_NATION = 'UNITED STATES'
  AND D_YEAR >= 1992 AND D_YEAR <= 1997
  GROUP BY C_CITY, S_CITY, D_YEAR
  ORDER BY D_YEAR ASC, LO_REVENUE DESC
  ```

- #### Q3.3

  ```sql
  SELECT
      C_CITY,
      S_CITY,
      D_YEAR,
      SUM(LO_REVENUE) AS LO_REVENUE
  FROM lineorder
  GLOBAL INNER JOIN customer ON LO_CUSTKEY = C_CUSTKEY
  GLOBAL INNER JOIN supplier ON LO_SUPPKEY = S_SUPPKEY
  GLOBAL INNER JOIN dates ON LO_ORDERDATE = toDate(replaceRegexpAll(toString(D_DATEKEY), '(\\d{4})(\\d{2})(\\d{2})', '\\1-\\2-\\3'))
  WHERE ((C_CITY = 'UNITED KI1') OR (C_CITY = 'UNITED KI5')) AND ((S_CITY = 'UNITED KI1') OR (S_CITY = 'UNITED KI5')) AND (D_YEAR >= 1992) AND (D_YEAR <= 1997)
  GROUP BY
      C_CITY,
      S_CITY,
      D_YEAR
  ORDER BY
      D_YEAR ASC,
      LO_REVENUE DESC
  ```

- #### Q3.4

  ```sql
  SELECT
      C_CITY,
      S_CITY,
      D_YEAR,
      SUM(LO_REVENUE) AS LO_REVENUE
  FROM lineorder
  GLOBAL INNER JOIN customer ON LO_CUSTKEY = C_CUSTKEY
  GLOBAL INNER JOIN supplier ON LO_SUPPKEY = S_SUPPKEY
  GLOBAL INNER JOIN dates ON LO_ORDERDATE = toDate(replaceRegexpAll(toString(D_DATEKEY), '(\\d{4})(\\d{2})(\\d{2})', '\\1-\\2-\\3'))
  WHERE ((C_CITY = 'UNITED KI1') OR (C_CITY = 'UNITED KI5')) AND ((S_CITY = 'UNITED KI1') OR (S_CITY = 'UNITED KI5')) AND (D_YEARMONTH = 'Dec1997')
  GROUP BY
      C_CITY,
      S_CITY,
      D_YEAR
  ORDER BY
      D_YEAR ASC,
      LO_REVENUE DESC
  ```

- #### Q4.1

  ```sql
  SELECT
      D_YEAR,
      C_NATION,
      SUM(LO_REVENUE) - SUM(LO_SUPPLYCOST) AS PROFIT
  FROM lineorder
  GLOBAL INNER JOIN customer ON LO_CUSTKEY = C_CUSTKEY
  GLOBAL INNER JOIN supplier ON LO_SUPPKEY = S_SUPPKEY
  GLOBAL INNER JOIN part ON LO_PARTKEY = P_PARTKEY
  GLOBAL INNER JOIN dates ON LO_ORDERDATE = toDate(replaceRegexpAll(toString(D_DATEKEY), '(\\d{4})(\\d{2})(\\d{2})', '\\1-\\2-\\3'))
  WHERE (C_REGION = 'AMERICA') AND (S_REGION = 'AMERICA') AND ((P_MFGR = 'MFGR#1') OR (P_MFGR = 'MFGR#2'))
  GROUP BY
      D_YEAR,
      C_NATION
  ORDER BY
      D_YEAR ASC,
      C_NATION ASC
  ```

- #### Q4.2

  ```sql
  SELECT
      D_YEAR,
      S_NATION,
      P_CATEGORY,
      SUM(LO_REVENUE) - SUM(LO_SUPPLYCOST) AS PROFIT
  FROM lineorder
  GLOBAL INNER JOIN customer ON LO_CUSTKEY = C_CUSTKEY
  GLOBAL INNER JOIN supplier ON LO_SUPPKEY = S_SUPPKEY
  GLOBAL INNER JOIN part ON LO_PARTKEY = P_PARTKEY
  GLOBAL INNER JOIN dates ON LO_ORDERDATE = toDate(replaceRegexpAll(toString(D_DATEKEY), '(\\d{4})(\\d{2})(\\d{2})', '\\1-\\2-\\3'))
  WHERE (C_REGION = 'AMERICA') AND (S_REGION = 'AMERICA') AND ((D_YEAR = 1997) OR (D_YEAR = 1998)) AND ((P_MFGR = 'MFGR#1') OR (P_MFGR = 'MFGR#2'))
  GROUP BY
      D_YEAR,
      S_NATION,
      P_CATEGORY
  ORDER BY
      D_YEAR ASC,
      S_NATION ASC,
      P_CATEGORY ASC
  ```

- #### Q4.3

  ```sql
  SELECT
      D_YEAR,
      S_CITY,
      P_BRAND,
      SUM(LO_REVENUE) - SUM(LO_SUPPLYCOST) AS PROFIT
  FROM lineorder
  GLOBAL INNER JOIN customer ON LO_CUSTKEY = C_CUSTKEY
  GLOBAL INNER JOIN supplier ON LO_SUPPKEY = S_SUPPKEY
  GLOBAL INNER JOIN part ON LO_PARTKEY = P_PARTKEY
  GLOBAL INNER JOIN dates ON LO_ORDERDATE = toDate(replaceRegexpAll(toString(D_DATEKEY), '(\\d{4})(\\d{2})(\\d{2})', '\\1-\\2-\\3'))
  WHERE (C_REGION = 'AMERICA') AND (S_NATION = 'UNITED STATES') AND ((D_YEAR = 1997) OR (D_YEAR = 1998)) AND (P_CATEGORY = 'MFGR#14')
  GROUP BY
      D_YEAR,
      S_CITY,
      P_BRAND
  ORDER BY
      D_YEAR ASC,
      S_CITY ASC,
      P_BRAND ASC
  ```
