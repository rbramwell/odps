# SELECT语法介绍 {#concept_i1q_lkb_wdb .concept}

本文为您介绍MaxCompute SELECT语法格式及使用SELECT语法执行嵌套查询、排序操作、分组查询等操作的注意事项。

## SELECT语法格式 {#section_dcq_y11_hfb .section}

``` {#codeblock_ylo_pw8_nk6}
SELECT [ALL | DISTINCT] select_expr, select_expr, ...
FROM table_reference
[WHERE where_condition]
[GROUP BY col_list]
[ORDER BY order_condition]
[DISTRIBUTE BY distribute_condition [SORT BY sort_condition] ]
[LIMIT number]
```

使用`select`语句时的注意事项请参见本文后续部分。

## 列表达式 {#section_xwx_cv2_ggb .section}

 `select`操作从表中读取数据，要读的列可以用列名指定，或者用`*`代表所有的列，一个简单的`select`语句，如下所示。

``` {#codeblock_wap_8wg_qjp}
select * from sale_detail;
```

只读取`sale_detail`的一列`shop_name`。

``` {#codeblock_syu_7hu_3c4}
select shop_name from sale_detail;
```

在`where`中可以指定过滤的条件。

``` {#codeblock_gdw_at7_w2q}
select * from sale_detail where shop_name like 'hang%';
```

当使用`select`语句屏显时，目前最多只能显示10000行结果。当`select`作为子句时，无此限制，`select`子句会将全部结果返回给上层查询。

使用说明：

-   `select`分区表时禁止全表扫描。

    2018-01-10 20点后创建的新项目，默认情况下执行SQL时，针对该Project里的分区表不允许全表扫描，必须有分区条件指定需要扫描的分区，由此减少SQL的不必要I/O，从而减少计算资源的浪费，同时也减少了不必要的后付费模式的计算费用（后付费模式中，数据输入量是计量计费参数之一）。

    例如表定义是`t1(c1,c2) partitioned by(ds)`，在新项目里执行如下语句会被禁止，返回Error。

    ``` {#codeblock_q75_f3w_u8r}
    select * from t1 where c1=1;
    select * from t1 where (ds=‘20180202’ or c2=3);
    select * from t1 left outer join t2 on a.id =b.id and a.ds=b.ds and b.ds=‘20180101’);  
    --join进行关联时，若分区剪裁条件放在where中，则分区剪裁生效，如果放在on条件中，从表的分区剪裁会生效，主表则进行全表扫描。
    ```

    如果您需要对分区表进行全表扫描，可以在对分区表全表扫描的SQL语句前加`set odps.sql.allow.fullscan=true;`，并和SQL语句一起提交执行。假设`sale_detail`表为分区表，则要全表扫描需同时提交以下的简单查询命令。

    ``` {#codeblock_4vf_btn_0zq}
    set odps.sql.allow.fullscan=true;
    select * from sale_detail;
    ```

-   在`table_reference`中支持使用嵌套子查询，如下所示。

    ``` {#codeblock_icv_49p_5wg}
    select * from (select region from sale_detail) t where region = 'shanghai';
    ```

    **说明：** 当使用`select`语句屏显时，目前最多只能显示10000行结果。当`select`作为子句时则无此限制，`select`子句会将全部结果返回给上层查询。


DISTINCT去重

您可以通过在字段前使用`distinct`去掉重复字段，只返回一个值；而使用`all`会返回字段中所有重复的值。不指定此选项时，默认效果和使用`all`的效果相同。

使用`distinct`只返回一行记录，如下所示。

``` {#codeblock_m34_dxs_qkj}
select distinct region from sale_detail;
select distinct region, sale_date from sale_detail;
-- distinct多列，distinct的作用域是select的列集合，不是单个列。
```

## SELECT\_EXPR正则表达式 {#section_hk3_mvl_ghb .section}

MaxCompute SQL支持使用正则表达式`select_expr`选列。

使用正则表达式`select_expr`：

-   `SELECT `abc.*` FROM t;`选出`t`表中所有列名以`abc`开头的列。
-   `SELECT `(ds)?+.+` FROM t;`选出`t`表中列名不为`ds`的所有列。
-   `SELECT `(ds|pt)?+.+` FROM t;`选出`t`表中排除`ds`和`pt`两列的其它列。
-   `SELECT `(d.*)?+.+` FROM t;`选出`t`表中排除列名以`d`开头的其它列。

**说明：** 

在排除多个列时，如果col2是col1的前缀，则需保证col1写在col2的前面（较长的col写前面）。例如，一个表有2个分区无需被`select`，一个分区名为`ds`，另一个为`dshh`，则正确表达式为`SELECT `(dshh|ds)?+.+` FROM tbl;`；错误表达式为`SELECT `(ds|dshh)?+.+` FROM tbl;`。

## WHERE子句过滤 {#section_lwx_cv2_ggb .section}

 `where`子句支持的过滤条件，如下表所示。

|过滤条件|描述|
|:---|:-|
|\> 、 < 、 =、 \>=、 <=、 <\>|关系操作符。|
|like、rlike|like和rlike的source和pattern参数均仅接受STRING类型。|
|in、not in|如果在in/not in条件后加子查询，子查询只能返回一列值，且返回值的数量不能超过1000。|

在`select`语句的`where`子句中，您可以指定分区范围，只扫描表的指定部分，避免全表扫描，如下所示。

``` {#codeblock_n9i_an1_yw9}
SELECT sale_detail.* 
FROM sale_detail
WHERE sale_detail.sale_date >= '2008'
AND sale_detail.sale_date <= '2014';
```

**说明：** 您可以通过`explain select`语句查看分区裁剪是否生效。普通的UDF或`join`的分区条件写法都有可能导致分区裁剪不生效，详情请参见[分区剪裁合理性评估](../../../../intl.zh-CN/最佳实践/SQL/分区剪裁合理性评估.md#)。

UDF支持分区裁剪，支持的方式是将UDF语句先当做一个小作业执行，再将执行的结果替换到原来UDF出现的位置 。实现的方式有以下两种：

-   在编写UDF的时候，UDF类上加入Annotation。

    ``` {#codeblock_468_wra_ffo}
    @com.aliyun.odps.udf.annotation.UdfProperty(isDeterministic=true)
    ```

    **说明：** `com.aliyun.odps.udf.annotation.UdfProperty`定义在odps-sdk-udf.jar文件中。您需要把引用的odps-sdk-udf版本提高到0.30.x或以上。

-   在SQL语句前设置Flag：`set odps.sql.udf.ppr.deterministic = true;`，此时SQL中所有的UDF均被视为`deterministic`。该操作执行的原理是做执行结果回填，但是结果回填存在限制，即最多回填1000个Partition。因此，如果UDF类加入Annotation，则可能会导致出现超过1000个回填结果的报错。此时您如果需要忽视此错误，可以通过设置Flag：`set odps.sql.udf.ppr.to.subquery = false;`全局关闭此功能。关闭后，UDF分区裁剪也会失效。

MaxCompute SQL的`where`子句支持`between…and`条件查询。

``` {#codeblock_kt1_83l_f33}
SELECT sale_detail.* 
FROM sale_detail 
WHERE sale_detail.sale_date BETWEEN '2008' AND '2014';
```

## GROUP BY分组查询 {#section_akw_f5f_ggb .section}

通常，`group by`和[聚合函数](intl.zh-CN/开发/SQL及函数/内建函数/聚合函数.md#)会配合使用。在`select`中包含聚合函数时有以下规则：

1.  用`group by`的Key可以是输入表的列名。
2.  也可以是由输入表的列构成的表达式，不允许是`select`语句的输出列的别名。
3.  规则1的优先级高于规则2。当规则1和规则2发生冲突时，即`group by`的Key既是输入表的列或表达式，又是`select`的输出列时，以规则1为准。

示例

``` {#codeblock_ech_0xe_yon}
select region from sale_detail group by region;
-- 直接使用输入表列名作为group by的列，可以运行。
select sum(total_price) from sale_detail group by region;
-- 以region值分组，返回每一组的销售额总量，可以运行。
select region, sum(total_price) from sale_detail group by region;
-- 以region值分组，返回每一组的region值(组内唯一)及销售额总量，可以运行。
select region as r from sale_detail group by r;
-- 使用select列的别名运行，报错返回。
select 2 + total_price as r from sale_detail group by 2 + total_price;
-- 必须使用列的完整表达式。
select region, total_price from sale_detail group by region;
-- 报错返回，select的所有列中，没有使用聚合函数的列，必须出现在group by中。
select region, total_price from sale_detail group by region, total_price;
-- 可以运行。
```

通常，在SQL解析中，`group by`操作先于`select`操作，因此`group by`只能接受输入表的列或表达式为Key。

**说明：** 

-   关于聚合函数的详情请参见[聚合函数](intl.zh-CN/开发/SQL及函数/内建函数/聚合函数.md)。
-   `order by`用于对所有数据按照某几列进行全局排序。如果您希望按照降序对记录进行排序，可以使用`desc`关键字。由于是全局排序，`order by`必须与`limit`共同使用。在使用`order by`排序时，NULL会被认为比任何值都小，这个行为与MySQL一致，但是与Oracle不一致。

    与`group by`不同，`order by`后面必须加`select`列的别名。当`select`某列时，如果没有指定列的别名，则列名会被作为列的别名。

    ``` {#codeblock_b3y_ovq_4e7}
    select * from sale_detail order by region;
    -- 报错返回，order by没有与limit共同使用。
    select * from sale_detail order by region limit 100;
    select region as r from sale_detail order by region limit 100;
    -- 报错返回，order by后面必须加列的别名。
    select region as r from sale_detail order by r limit 100;
    ```


## LIMIT NUMBER限制输出行数 {#section_dwy_n5f_ggb .section}

 `limit number`的`number`是常数，限制输出行数。当使用无`limit`的`select`语句直接从屏幕输出查看结果时，最多只输出10000行。每个项目空间的这个屏显最大限制可能不同，您可以通过`setproject`命令控制。

## ORDER BY/SORT BY/DISTRIBUTE BY {#section_elb_wx2_ggb .section}

-   `distribute by`用于对数据按照某几列的值做Hash分片，必须使用`select`的输出列别名。

    ``` {#codeblock_sl8_mn3_mcb}
    select region from sale_detail distribute by region;
    -- 列名即是别名，可以运行。
    select region as r from sale_detail distribute by region;
    -- 报错返回，后面必须加列的别名。
    select region as r from sale_detail distribute by r;
    ```

-   `sort by`用于局部排序，语句前必须加`distribute by`。实际上`sort by`是对`distribute by`的结果进行局部排序。必须使用`select`的输出列别名。

    ``` {#codeblock_c1y_39g_lwo}
    select region from sale_detail distribute by region sort by region;
    select region as r from sale_detail sort by region;
    -- 没有distribute by，报错退出。
    ```

-   `order by`不和`distribute by/sort by`共用，同时`group by`也不和`distribute by/sort by`共用，必须使用`select`的输出列别名。

**说明：** 

-   `order by/sort by/distribute by`的Key必须是`select`语句的输出列，即列的别名。列的别名可以为中文。
-   在MaxCompute SQL解析中，`order by/sort by/distribute by`是后于`select`操作的，因此它们只能接受`select`语句的输出列为Key。

