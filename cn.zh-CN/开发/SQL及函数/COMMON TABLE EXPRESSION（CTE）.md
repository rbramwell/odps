# COMMON TABLE EXPRESSION（CTE） {#concept_lh3_xkb_wdb .concept}

MaxCompute支持SQL标准的CTE，提高SQL语句的可读性与执行效率。

命令格式如下所示。

``` {#codeblock_ut9_2gz_nm5}
WITH 
     cte_name AS
    (
        cte_query
    )
    [,cte_name2  AS 
     (
     cte_query2
     )
    ,……]
```

-   `cte_name`：CTE的名称，不能与当前`WITH`子句中的其他CTE的名称相同。查询中任何使用到`cte_name`标识符的地方，均指CTE。
-   `cte_query`：一个`SELECT`语句。它产生的结果集用于填充CTE。

示例如下。

``` {#codeblock_tbl_fsj_s23}
INSERT OVERWRITE TABLE srcp PARTITION (p='abc')
SELECT * FROM (
    SELECT a.key, b.value
    FROM (
        SELECT * FROM src WHERE key IS NOT NULL    ) a
    JOIN (
        SELECT * FROM src2 WHERE value > 0    ) b
    ON a.key = b.key
) c
UNION ALL
SELECT * FROM (
    SELECT a.key, b.value
    FROM (
        SELECT * FROM src WHERE key IS NOT NULL    ) a
    LEFT OUTER JOIN (
        SELECT * FROM src3 WHERE value > 0    ) b
    ON a.key = b.key AND b.key IS NOT NULL
)d;
```

顶层的`UNION`两侧各为一个`JOIN`，`JOIN`的左表是相同的查询。通过写子查询的方式，只能重复这段代码。

使用CTE的方式重写以上语句。

``` {#codeblock_ue2_onf_2i1}
with 
  a as (select * from src where key is not null),
  b as (select  * from src2 where value>0),
  c as (select * from src3 where value>0),
  d as (select a.key,b.value from a join b on a.key=b.key),
  e as (select a.key,c.value from a left outer join c on a.key=c.key and c.key is not null)
insert overwrite table srcp partition (p='abc')
select * from d union all select * from e;
```

重写后，`a`对应的子查询只需写一次，便可在后面进行重用。您可以在CTE的`WITH`子句中指定多个子查询，像使用变量一样在整个语句中反复重用。除重用外，不必反复嵌套。

