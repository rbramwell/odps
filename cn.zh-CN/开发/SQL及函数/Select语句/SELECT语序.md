# SELECT语序 {#concept_jsv_mkb_wdb .concept}

对于按照SELECT语法格式书写的SELECT语句，它的逻辑执行顺序与标准的书写语序并不相同。

## 示例一 {#section_vc7_ven_kmd .section}

``` {#codeblock_kvw_c7v_y7r}
SELECT  key
        ,MAX(value)
FROM    src t
WHERE   value > 0
GROUP BY key
HAVING  SUM(value) > 100
ORDER BY key
LIMIT   100
;
```

以上语句的逻辑执行顺序是`FROM->WHERE->GROUY BY->HAVING->SELECT->ORDER BY->LIMIT`：

-   `ORDER BY`中只能引用`SELECT`列表中生成的列，而不是访问`FROM`的源表中的列。
-   `HAVING`可以访问的是`GROUP BY key`和聚合函数。
-   `SELECT`的时候，如果有`GROUP BY`，便只能访问`GROUP BY key`和聚合函数，而不是`FROM`中源表中的列。

为避免混淆，MaxCompute支持以执行顺序书写查询语句。

``` {#codeblock_nuj_fiy_hke}
FROM    src t
WHERE   value > 0
GROUP BY key
HAVING  SUM(value) > 100
SELECT  key
        ,MAX(value)
ORDER BY key
LIMIT   100
;
```

## 示例二 {#section_q72_wgp_x5f .section}

``` {#codeblock_bze_eo4_ima}
SELECT  shop_name
        ,total_price
        ,region
FROM    sale_detail
WHERE   total_price > 150
DISTRIBUTE BY region
SORT BY region
;
```

以上语句的逻辑执行顺序是`FROM->WHERE->SELECT->DISTRIBUTE BY->SORT BY`。

