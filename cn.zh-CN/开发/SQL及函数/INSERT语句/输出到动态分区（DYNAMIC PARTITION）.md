# 输出到动态分区（DYNAMIC PARTITION） {#concept_b1p_qdb_wdb .concept}

在使用INSERT OVERWRITE语句插入到一张分区表时，您可以在语句中指定分区的值，也可以在分区中指定一个分区列名（但不提供具体的值），并在SELECT子句中提供对应分区列的值。

## 动态分区语法 {#section_aky_dyy_bgb .section}

命令格式如下。

``` {#codeblock_hsu_6qd_owe}
insert overwrite table tablename partition (partcol1, partcol2 ...) select_statement from from_statement;
```

**说明：** 

-   `select_statement`字段中，后面的字段将提供目标表动态分区值。如果目标表只有一级动态分区，则`select_statement`最后一个字段值即为目标表的动态分区值。
-   在使用动态分区功能的SQL中，在分布式环境下，单个进程最多只能输出512个动态分区，否则会引发运行时异常。
-   任意动态分区SQL不允许生成超过2000个动态分区，否则会引发运行时异常。
-   动态生成的分区值不允许为NULL，也不支持含特殊字符和中文，否则会引发运行时异常`FAILED: ODPS-0123031:Partition exception - invalid dynamic partition value: province=xxx`。
-   如果目标表有多级分区，在运行`insert`语句时允许指定部分分区为静态，但是静态分区必须是高级分区。
-   如果目标表为Hash Clustering Table，则不支持动态分区。

## 动态分区示例 {#section_shg_fyy_bgb .section}

动态分区的示例如下。

``` {#codeblock_kaj_pl5_21p}
create table total_revenues (revenue bigint) partitioned by (region string);
insert overwrite table total_revenues partition(region)
select total_price as revenue, region
from sale_detail;
```

按照上述写法，在SQL运行之前，您无法得知会产生哪些分区。只有在`select`运行结束后，才能通过`region`字段产生的值确定会产生哪些分区。

其它示例如下。

``` {#codeblock_3yq_i9w_vlx}
create table sale_detail_dypart like sale_detail; --创建示例目标表。
```

-   示例一

    ``` {#codeblock_8sh_b8y_hfx}
    insert overwrite table sale_detail_dypart partition (sale_date, region)
    select shop_name,customer_id,total_price,sale_date,region from sale_detail;
    -- 成功返回。
    ```

    -   此时`sale_detail`表中，`sale_date`的值决定目标表的`sale_date`分区值，`region`的值决定目标表的`region`分区值。
    -   动态分区中，`select_statement`字段和目标表动态分区的对应是按字段顺序决定的。上述示例中，`select`语句如果被写成`select shop_name,customer_id,total_price,region,sale_date from sale_detail;`，则`sale_detail`表中，`region`值决定目标表的`sale_date`分区值，`sale_date`的值决定目标表的`region`分区值。
-   示例二

    ``` {#codeblock_8wx_nx0_ztg}
    insert overwrite table sale_detail_dypart partition (sale_date='2013', region)
    select shop_name,customer_id,total_price,region from sale_detail;
    -- 成功返回，多级分区，指定一级分区。
    ```

-   示例三

    ``` {#codeblock_xpm_elw_00t}
    insert overwrite table sale_detail_dypart partition (sale_date='2013', region)
    select shop_name,customer_id,total_price from sale_detail;
    -- 失败返回，动态分区插入时，动态分区列必须在select列表中。
    ```

-   示例四

    ``` {#codeblock_ylb_fze_u3y}
    insert overwrite table sales partition (region='china', sale_date)
    select shop_name,customer_id,total_price,region from sale_detail;
    -- 失败返回，不能仅指定低级子分区，而动态插入高级分区。
    ```


旧版MaxCompute在进行动态分区时，如果分区列的类型与对应`select`列表中列的类型不严格一致，会发生报错。MaxCompute 2.0则支持[隐式类型转换](intl.zh-CN/开发/数据类型.md#)，示例如下。

``` {#codeblock_0kz_5vu_gh6}
create table parttable(a int, b double) partitioned by (p string);
insert into parttable partition(p) select key, value, current_timestmap() from src;
select * from parttable;
```

执行上述语句后返回结果如下。

|a|b|p|
|:-|:-|:-|
|0|NULL|2017-01-23 22:30:47.130406621|
|0|NULL|2017-01-23 22:30:47.130406621|

**说明：** 如果您的数据是有序的，动态分区插入会把数据随机打散，导致压缩率较低。推荐您使用[Tunnel命令](intl.zh-CN/开发/数据上传下载/Tunnel上传下载命令.md#)上传数据到动态分区，以获取较好的压缩率。

使用该命令的详细示例请参见[RDS迁移至MaxCompute实现动态分区](../../../../intl.zh-CN/最佳实践/数据迁移/RDS迁移至MaxCompute实现动态分区.md#)。

