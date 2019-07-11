# 更新表数据（INSERT OVERWRITE and INTO） {#concept_yzd_ndb_wdb .concept}

本文向您介绍如何使用INSERT OVERWRITE和INSERT INTO两种命令更新表数据。

## INSERT操作 {#section_o5d_vdy_gfb .section}

命令格式

``` {#codeblock_ty7_8cb_kqu}
INSERT OVERWRITE|INTO TABLE tablename [PARTITION (partcol1=val1, partcol2=val2 ...)] [(col1,col2 ...)]
select_statement
FROM from_statement;
```

**说明：** 

-   MaxCompute的`insert`语法与通常使用的MySQL或Oracle的`insert`[语法有差别](../../../../intl.zh-CN/最佳实践/SQL/与标准SQL的主要区别及解决方法.md#)，在`insert overwrite/into`后需要加入`table`关键字，而非直接使用`tablename`。
-   当`insert`的目标表是分区表时，指定分区值`[PARTITION (partcol1=val1, partcol2=val2 …)]`语法中不允许使用函数等表达式。
-   目前`insert overwrite`不支持指定插入列的功能，暂时只能用`insert into`。
-   不支持`insert into`到Hash Clustering表。
-   当遇到并发写入时，MaxCompute会根据ACID进行并发写的保障。关于ACID的具体语义，请参见[ACID语义](../../../../intl.zh-CN/产品简介/基本概念/ACID语义.md#)。

在MaxCompute SQL处理数据的过程中，`insert overwrite/into`用于将计算的结果保存目标表中。

`insert into`与`insert overwrite`的区别是：`insert into`会向表或表的分区中追加数据，而`insert overwrite`会在向表或分区中插入数据前清空表中的原有数据。

**说明：** 在反复对同一个分区进行`insert overwrite`的时候，通过`describe`查看到的数据分区Size会不同。这是因为从同一个表的同一个分区`select`出来再`insert overwrite`回相同分区时，文件切分逻辑发生变化，从而导致数据的Size发生变化。数据的总长度在`insert overwrite`前后是不变的，您不必担心存储计费会存在问题。

在使用MaxCompute处理数据的过程中，`insert overwrite/into`是最常用到的语句，它们会将计算的结果保存到一个表中，以供下一步计算使用。例如计算`sale_detail`表中不同地区的销售额，操作如下。

``` {#codeblock_1ht_2ke_5hk}
create table sale_detail_insert like sale_detail;
alter table sale_detail_insert add partition(sale_date='2013', region='china');
insert overwrite table sale_detail_insert partition (sale_date='2013', region='china')
select shop_name, customer_id,total_price from sale_detail;
```

**说明：** 在进行`insert`更新表数据时，源表与目标表的对应关系依赖于在`select`子句中列的顺序，而不是表与表之间列名的对应关系。以下为合法的SQL语句。

``` {#codeblock_jih_uu6_w2f}
insert overwrite table sale_detail_insert partition (sale_date='2013', region='china')
select customer_id, shop_name, total_price from sale_detail;
-- 在创建sale_detail_insert表时，列的顺序为：
-- shop_name string, customer_id string, total_price bigint
-- 而从sale_detail向sale_detail_insert插入数据是，sale_detail的插入顺序为：
-- customer_id, shop_name, total_price
-- 此时，会将sale_detail.customer_id的数据插入sale_detail_insert.shop_name。
-- 将sale_detail.shop_name的数据插入sale_detail_insert.customer_id。
```

向某个分区插入数据时，分区列不允许出现在`select`列表中。

``` {#codeblock_3kf_6xj_2qm}
insert overwrite table sale_detail_insert partition (sale_date='2013', region='china')
select shop_name, customer_id, total_price, sale_date, region  from sale_detail;
-- 报错返回，sale_date，region为分区列，不允许出现在静态分区的insert语句中。
```

同时，`partition`的值只能是常量，不可以出现表达式。以下为错误用法。

``` {#codeblock_x6i_w72_bg5}
insert overwrite table sale_detail_insert partition (sale_date=datepart('2016-09-18 01:10:00', 'yyyy') , region='china')
select shop_name, customer_id, total_price from sale_detail;
```

## 更新表数据到动态分区 {#section_anh_h5q_pgb .section}

动态分区使用注意事项：

-   `insert into partition`时，如果分区不存在，会自动创建分区。
-   多个`insert into partition`作业并发时，如果分区不存在，会自动创建分区，但只会成功创建一个分区。
-   如果不能控制`insert into partition`作业并发，则只能通过预创建分区避免问题。

更多动态分区详情请参考[输出到动态分区（DYNAMIC PARTITION）](intl.zh-CN/开发/SQL及函数/INSERT语句/输出到动态分区（DYNAMIC PARTITION）.md#)。

