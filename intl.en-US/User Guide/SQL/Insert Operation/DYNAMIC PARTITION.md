# DYNAMIC PARTITION {#concept_b1p_qdb_wdb .concept}

To ‘insert overwrite’ into a partition table, specify the partition value in the statement. It can also be realized in a more flexible way, to specify a partition column in a partition table but not give the value.

## OverView {#section_jkb_4yy_bgb .section}

Correspondingly, the columns in Select clause are used to specify these partition values.

Statement format:

``` {#codeblock_0cq_ziy_agi}
insert overwrite table tablename partition (partcol1, partcol2 ...) select_statement from from_statement;
```

**Note:** 

-   In the ‘select\_statement’ field, the following field provides a dynamic partition value for the target table. If the target table has only one-level dynamic partition, the last field value of select\_statement is the dynamic partition value of the target table.
-   Currently, a single worker can only output up to 512 dynamic partitions in a distributed environment, otherwise it leads to abnormality.
-   Currently, any dynamic partition SQL cannot generate more than 2,000 dynamic partitions; otherwise it causes abnormality.
-   The value of dynamic partition cannot be NULL, and also does not support special or Chinese characters, otherwise an exception is thrown. The exception is as follows:

    ``` {#codeblock_iix_8g5_shh}
    FAILED: ODPS-0123031:Partition exception - invalid dynamic partition value:
                  province=xxx
    ```

-   If the destination table has multi-level partitions, it is allowed to specify parts of partitions to be static partitions through ‘Insert’ statement, but the static partitions must be advanced partitions.

## Examples {#section_k34_vyy_bgb .section}

A simple example to explain dynamic partition is as follows:

``` {#codeblock_u0l_6eg_pym}
create table total_revenues (revenue bigint) partitioned by (region string);
    insert overwrite table total_revenues partition(region)
        select total_price as revenue, region
            from sale_detail;
```

As mentioned in the preceding example, user is unable to know which partitions are generated before running SQL. Only after the Select statement running ends, user can confirm which partitions have been generated using ‘region’ as the value. This is why the partition is called as the **Dynamic Partition**.

Other Examples:

``` {#codeblock_sbr_am3_bc3}
create table sale_detail_dypart like sale_detail; --Create target table.
```

-   --Example 1:

    ``` {#codeblock_3lb_kf0_rjy}
    insert overwrite table sale_detail_dypart partition (sale_date, region)
    select shop_name,customer_id,total_price,sale_date,region from sale_detail;
        -- Return successfully.
    ```

    -   In ‘sales\_detail’ table, the value of the sale\_date determines the sales\_date partition value of the target table, and the value of the region determines the region partition value of the target table.
    -   In a dynamic partition, the correspondence between the select\_statement field and the dynamic partition of the target table is determined by the order of the fields. In this example, if the Select statement is written as the following:

        ``` {#codeblock_oi5_pcl_32e}
        select shop_name,customer_id,total_price,region,sale_date from
                  sale_detail;
        ```

        the region value determines the sale\_date partition value of the target table, and the value of sale\_date determines the region partition value of the target table.

-   --Example 2:

    ``` {#codeblock_1py_niy_qgw}
    insert overwrite table sale_detail_dypart partition (sale_date='2013', region)
            select shop_name,customer_id,total_price,region from sale_detail;
        -- Return successfully; multiple partitions; specify a secondary partition.
    ```

-   --Example 3:

    ``` {#codeblock_q3g_bwq_t3e}
    insert overwrite table sale_detail_dypart partition (sale_date='2013', region)
            select shop_name,customer_id,total_price from sale_detail;
        -- Return failure information. When inserting a dynamic partition, the dynamic partition column must appear in Select list.
    ```

-   --Example 4:

    ``` {#codeblock_g1e_3ao_dzt}
    insert overwrite table sales partition (region='china', sale_date)
    select shop_name,customer_id,total_price,sale_date from sale_detail;
        -- Return failure information. User cannot specify the lowsubpartition only, but needs to insert advanced partition dynamically.
    ```


When the old version of MaxCompute performs dynamic partitioning, if the partition column type is not exactly the same as the column type in the corresponding select list, an error is reported. MaxCompute 2.0 supports implicit conversion, as shown in the following :

``` {#codeblock_36p_l2d_vzy}
create table parttable(a int, b double) partitioned by (p string);
insert into parttable partition(p) select key, value, current_timestmap() from src;
select * from parttable;
```

The result is as follows:

|a|b|c|
|:-|:-|:-|
|0|NULL|2017-01-23 22:30:47.130406621|
|0|NULL|2017-01-23 22:30:47.130406621|

