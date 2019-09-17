# MaxCompute表的高级功能 {#concept_226663 .concept}

本文为您介绍MaxCompute表的生命周期、避免全表扫描和小文件以及Hash Clustering表等高级功能。

## 生命周期 {#section_9ac_ndp_tb2 .section}

MaxCompute为表和分区提供数据生命周期管理。表（分区）数据从最后一次更新时间算起，在经过指定的时间后如没有变动，则此表（分区）将被MaxCompute自动回收，这个指定的时间就是生命周期。生命周期只能以表为单位进行设置。

``` {#codeblock_uoj_ofw_tuh}
create table test_lifecycle(key string) lifecycle 100;/alter table test_lifecycle set lifecycle 50;
```

MaxCompute会根据每张非分区表或者分区表中分区的LastDataModifiedTime和lifecycle的设置，进行判断是否要回收他们。

MaxCompute SQL提供touch操作用来修改分区的LastDataModifiedTime。此操作会将分区的LastDataModifiedTime值修改为当前时间，这样MaxCompute会认为表或分区的数据有变动，生命周期的计算会重新开始。

``` {#codeblock_xlm_80g_h8r}
ALTER TABLE table_name TOUCH PARTITION(partition_col='partition_col_value', ...);
```

**说明：** 

-   合理规划表的生命周期，在创建表时即设置生命周期，可有效减少存储压力。
-   对表数据的任何变动都会影响生命周期回收数据的判断时间，包括小文件合并。

## 避免全表扫描 {#section_sbw_rqp_0yj .section}

-   在表设计时如何避免全表扫描

    表设计是指建立分区表或者对扫描条件进行列设计。在设计时，需要注意以下几点：

    -   对数据表进行合理的分区。
    -   把常用查询条件设置成列名。
    -   对常用查询条件进行hash clustering。

在数据计算时如何避免全表扫描

-   您可以增加分区过滤的条件或减少扫描的分区数，实现减少数据扫描量。
-   把全局扫描表的中间结果进行存储，形成中间表。
-   如果每天都需扫描某表一整年的分区，则计算消耗是非常大的。因此，建议您拆出一张中间表，每天做一次汇总，然后再扫描这张中间表的一整年的分区，从而减少扫描的数据量。

## 避免小文件 {#section_h0x_sxj_3yu .section}

-   Reduce计算过程产生的小文件：只需要`insert overwrite`源表（或分区）即可，或者写入到新表删除源表。
-   Tunnel数据采集过程中产生的小文件建议：
    -   调用Tunnel SDK时，每当buffer达到64M时，进行一次提交。
    -   使用console时应避免频繁上传小文件，建议积累至一定的数量后再一次性上传。
    -   如果导入的是分区表，建议给分区表设置生命周期，过期不用的数据将会被自动清理。
    -   也可以同第一种方案，使用`insert overwrite`语句对源表（或分区）进行操作。
    -   使用alter合并模式时，通过console命令进行合并。
-   建议创建临时表时加上生命周期，在到期后垃圾回收机制会自动回收临时表。
-   申请过多的DataHub shard将会产生小文件问题，以下是申请DataHub shard数目时的策略。
    -   单个shard的默认吞吐量是1MB/s，可以按照这个分配实际的shard数目（可以在此基础上多加几个）。
    -   ODPS的同步逻辑是每个shard会有一个单独的task（每隔5分钟或每满64MB，此task会commit一次），默认设置5分钟是为了能在ODPS上尽快查到数据。当按照小时建partition时，一个shard每个小时将会产生12个文件。若此时数据量很少而shard很多，则ODPS里就会出现很多小文件（shard\*12/hour）。
    -   应按需分配shard，避免过度分配。

## 转化Hash Clustering表 {#section_4i5_ryt_wc2 .section}

Hash Clustering表的优势在于可以实现Bucket Pruning优化、Aggregation优化以及存储优化。在创建表时，使用clustered by指定Hash Key后，MaxCompute将对指定列进行Hash运算，按照Hash值分散到各个Bucket里。Hash Key值的选择原则为选择重复键值少的列。

转化为Hash Clustering表的方法如下。

``` {#codeblock_ieh_3q8_buh}
ALTER TABLE table_name [CLUSTERED BY (col_name [, col_name, ...]) [SORTED BY (col_name [ASC | DESC] [, col_name [ASC | DESC] ...])] INTO number_of_buckets BUCKETS]
```

`alter table`语句适用于存量表，在增加了新的聚集属性之后，新的分区将做hash cluster存储。

创建完Hash Clustering表后，您可以使用`insert overwrite`语句将源表转化为Hash Clustering表。

**说明：** Hash Clustering表存在以下限制：

-   不支持`insert into`语句，只能通过`insert overwrite`来添加数据。
-   不支持直接使用tunnel upload数据到range cluster表，因为tunnel上传的数据是无序的。

