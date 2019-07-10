# JOIN {#concept_cxf_rkb_wdb .concept}

您可以使用JOIN操作连接两张表的查询结果。MaxCompute的JOIN操作分为左连接、右连接、全连接、内连接。支持多路连接，但不支持CROSS JOIN笛卡尔积，即无ON条件的连接。

命令格式如下。

``` {#codeblock_q3k_9c2_7fj}
join_table:
        table_reference join table_factor [join_condition]
        | table_reference {left outer|right outer|full outer|inner} join table_reference join_condition
    table_reference:
        table_factor
        | join_table
    table_factor:
        tbl_name [alias]
        | table_subquery alias
        | ( table_references )
    join_condition:
        on equality_expression ( and equality_expression )
```

**说明：** `equality_expression`是一个等式表达式。

-   `left outer join`：左连接。返回左表中的所有记录，即使右表中没有与之匹配的记录。

    ``` {#codeblock_9p3_d99_ay3}
    select a.shop_name as ashop, b.shop_name as bshop from shop a
            left outer join sale_detail b on a.shop_name=b.shop_name;
        -- 由于表shop及sale_detail中都有shop_name列，因此需要在select子句中使用别名进行区分。
    ```

    **说明：** 如果右边值不唯一，建议不要连续使用过多`left join`，以免在`join`的过程中产生数据膨胀，导致作业停滞。

-   `right outer join`：右连接。返回右表中的所有记录，即使左表中没有与之匹配的记录。

    ``` {#codeblock_t3k_bzx_035}
    select a.shop_name as ashop, b.shop_name as bshop from shop a
            right outer join sale_detail b on a.shop_name=b.shop_name;
    ```

-   `full outer join`：全连接。返回左右表中的所有记录。

    ``` {#codeblock_2d2_eh6_wmv}
    select a.shop_name as ashop, b.shop_name as bshop from shop a
            full outer join sale_detail b on a.shop_name=b.shop_name;
    ```


以下示例中，表A是`test_table_a`，表B是`test_table_b`，使用了`join`查询和`where`过滤查询。查询分区大于20180101的数据中`origin`和`id`一致的记录数，使用`left join`可以保留左表中`test_table_a`的全表记录。正常情况下，如果您使用`left join` ，左表会保留全表记录；如果您使用`right join`，右表会保留全表记录。

错误示例如下。

``` {#codeblock_790_4jv_pbr}
SELECT s.id
        ,s.name
        ,s.origin
        ,d.value
FROM    test_table_a s
LEFT JOIN   test_table_b d
ON      s.origin = d.id
WHERE   s.ds > "20180101" AND d.ds>"20180101";
```

如果`join`在`where`条件之前， 会先做`join`操作，然后对`join`的结果做`where`条件过滤。您会发现获取的结果是两个表的交集，而不是全表。

修正后的SQL如下，通过这种方式可以显示全表。

``` {#codeblock_2l3_ff6_b2x}
SELECT s.id
        ,s.name
        ,s.origin
        ,d.value
FROM  (select * from  test_table_a where ds > "20180101" ) s
LEFT JOIN (select * from  test_table_b where ds > "20180101") d
ON s.origin = d.id;
```

表中存在至少一个匹配时，`inner join`返回行。关键字`inner`可省略。

``` {#codeblock_lnu_7c9_9gb}
select a.shop_name from shop a inner join sale_detail b on a.shop_name=b.shop_name;
select a.shop_name from shop a join sale_detail b on a.shop_name=b.shop_name;
```

连接条件，只允许`and`连接的等值条件。只有在[MAPJOIN](cn.zh-CN/开发/SQL及函数/SELECT语句/MAPJOIN HINT.md#)中，可以使用不等值连接或者使用`or`连接多个条件。

``` {#codeblock_wrb_438_kas}
select a.* from shop a full outer join sale_detail b on a.shop_name=b.shop_name
        full outer join sale_detail c on a.shop_name=c.shop_name;
    -- 支持多路join链接示例。
select a.* from shop a join sale_detail b on a.shop_name != b.shop_name;
    -- 不支持不等值Join链接条件，报错返回。
```

IMPLICIT JOIN：MaxCompute支持以下`join`方式。

``` {#codeblock_e2m_si6_0e6}
SELECT * FROM table1, table2 WHERE table1.id = table2.id;
--执行的效果相当于以下语句。
SELECT * FROM table1 JOIN table2 ON table1.id = table2.id;
```

