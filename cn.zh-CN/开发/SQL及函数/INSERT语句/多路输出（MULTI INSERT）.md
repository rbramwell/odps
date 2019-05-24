# 多路输出（MULTI INSERT） {#concept_jkh_pdb_wdb .concept}

MaxCompute SQL支持在一个语句中插入不同的结果表或者分区实现多路输出。

命令格式如下：

```
FROM from_statement
INSERT OVERWRITE | INTO TABLE tablename1 [PARTITION (partcol1=val1, partcol2=val2 ...)]
select_statement1 [FROM from_statement]
[INSERT OVERWRITE | INTO TABLE tablename2 [PARTITION (partcol1=val3, partcol2=val4 ...)]
select_statement2 [FROM from_statement]]
```

**说明：** 

-   一般情况下，单个SQL中最多可以写128路输出，超过128路，则报语法错误。
-   在一个multi insert中：
    -   对于分区表，同一个目标分区不允许出现多次。
    -   对于未分区表，该表不能出现多次。
-   对于同一张分区表的不同分区，不能同时有insert overwrite和insert into操作，否则报错返回。

示例如下：

```
create table sale_detail_multi like sale_detail;
from sale_detail
insert overwrite table sale_detail_multi partition (sale_date='2010', region='china' ) 
select shop_name, customer_id, total_price where .....
insert overwrite table sale_detail_multi partition (sale_date='2011', region='china' )
select shop_name, customer_id, total_price where .....;
-- 成功返回，将sale_detail的数据插入到sales里的2010年及2011年中国大区的销售记录中。
from sale_detail
insert overwrite table sale_detail_multi partition (sale_date='2010', region='china' )
select shop_name, customer_id, total_price
insert overwrite table sale_detail_multi partition (sale_date='2010', region='china' )
select shop_name, customer_id, total_price;
-- 出错返回，同一分区出现多次。
from sale_detail
insert overwrite table sale_detail_multi partition (sale_date='2010', region='china' )
select shop_name, customer_id, total_price
insert into table sale_detail_multi partition (sale_date='2011', region='china' )
select shop_name, customer_id, total_price;
-- 出错返回，同一张表的不同分区，不能同时有insert overwrite和insert into操作。
```

