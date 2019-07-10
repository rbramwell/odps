# MAPJOIN HINT {#concept_bf5_tkb_wdb .concept}

当一个大表和一个或多个小表JOIN时，您可以使用MAPJOIN提升性能。

`MAPJOIN`的基本原理：在小数据量的场景中，SQL会将您指定的小表全部加载到执行`JOIN`操作的程序的内存中，从而加快`JOIN`的执行速度。

**说明：** 当您使用 MAPJOIN时，需要注意以下几点：

-   `LEFT OUTER JOIN`的左表必须是大表。
-   `RIGHT OUTER JOIN`的右表必须是大表。
-   `INNER JOIN`的左表或右表均可以作为大表。
-   `FULL OUTER JOIN`不能使用`MAPJOIN`。
-   `MAPJOIN`支持小表为子查询。
-   使用`MAPJOIN`时，在引用小表或是子查询时，需要引用别名。
-   在`MAPJOIN`中，可以使用不等值连接或者使用OR连接多个条件。您可以通过不写`ON`语句或通过`mapjoin on 1 = 1`的形式，实现笛卡尔乘积的计算，例如`select /* + mapjoin(a) */ a.id from shop a join table_name b on 1=1`，但这样可能带来数据量膨胀问题。
-   目前，MaxCompute在`MAPJOIN`中最多支持指定256张小表，否则报语法错误。MAPJOIN HINT中多个小表用逗号隔开，例如`/*+mapjoin(a,b,c)*/`。
-   如果使用`MAPJOIN`，则小表占用的总内存不得超过640MB。由于MaxCompute是压缩存储，因此小表在被加载到内存后，数据大小会急剧膨胀。此处的640MB限制是加载到内存后的空间大小。

示例

``` {#codeblock_2u3_zd6_hxg}
select /* + mapjoin(a) */
        a.shop_name,
        b.customer_id,
        b.total_price
from shop a join sale_detail b
on a.shop_name = b.shop_name;
```

MaxCompute SQL不支持在普通`JOIN`的`ON`条件中使用不等值表达式、`OR`逻辑等复杂的`JOIN`条件，但是在`MAPJOIN`中可以进行上述操作。示例如下。

``` {#codeblock_pzf_pv2_o8a}
select /*+ mapjoin(a) */
        a.total_price,
        b.total_price
from shop a join sale_detail b
on a.total_price < b.total_price or a.total_price + b.total_price < 500;
```

`MAPJOIN`多个小表示例请参见[用MAPJOIN缓存多张小表](https://help.aliyun.com/knowledge_detail/40270.html)。

