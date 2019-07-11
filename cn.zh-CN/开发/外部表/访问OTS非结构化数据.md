# 访问OTS非结构化数据 {#concept_dgw_n2b_wdb .concept}

本文将进一步为您介绍如何将来自Table Store（OTS）的数据纳入MaxCompute上的计算生态，实现多种数据源之间的无缝连接。

您可以通过DataWorks配合MaxCompute对外部表进行可视化的创建、搜索、查询、配置、加工和分析。详情请参见[外部表](../../../../cn.zh-CN/使用指南/数据开发/外部表.md#)。

表格存储（Table Store）是构建在阿里云飞天分布式系统之上的NoSQL数据存储服务，提供海量结构化数据的存储和实时访问。您可以通过[Table Store文档](https://help.aliyun.com/document_detail/27280.html)对其进行了解。

MaxCompute与Table Store是两个独立的大数据计算和存储服务，所以两者之间的网络必须保证连通性。MaxCompute公共云服务访问Table Store存储时，推荐您使用Table Store私网地址，即Host名以ots-internal.aliyuncs.com作为结尾的地址，例如tablestore://odps-ots-dev.cn-shanghai.ots-internal.aliyuncs.com。

Table Store与MaxCompute都有其自身的类型系统。在MaxCompute处理Table Store数据时，两者之间的类型对应关系如下所示。

|MaxCompute Type|Table Store Type|
|:--------------|:---------------|
|STRING|STRING|
|BIGINT|INTEGER|
|DOUBLE|DOUBLE|
|BOOLEAN|BOOLEAN|
|BINARY|BINARY|

## STS模式授权 {#section_spx_rrb_wdb .section}

MaxCompute计算服务访问Table Store数据需要有一个安全的授权通道。因此，MaxCompute结合了阿里云的访问控制服务（RAM）和令牌服务（STS）实现对数据的安全访问。

您可以通过以下两种方式授予权限：

-   当MaxCompute和Table Store的Owner是同一个账号时，登录阿里云账号后，[单击此处完成一键授权](https://ram.console.aliyun.com/?spm=5176.100239.blogcont281191.24.uJg9dR#/role/authorize?request=%7B%22Requests%22:%20%7B%22request1%22:%20%7B%22RoleName%22:%20%22AliyunODPSDefaultRole%22,%20%22TemplateId%22:%20%22DefaultRole%22%7D%7D,%20%22ReturnUrl%22:%20%22https:%2F%2Fram.console.aliyun.com%2F%22,%20%22Service%22:%20%22ODPS%22%7D)。
-   自定义授权
    1.  首先在RAM控制台中授予MaxCompute访问Table Store的权限。

        登录[RAM控制台](https://ram.console.aliyun.com/#/overview)（若MaxCompute和Table Store不是同一个账号，此处需由Table Store账号登录进行授权），创建角色，例如角色AliyunODPSDefaultRole、AliyunODPSRoleForOtherUser。

    2.  修改策略内容设置。

        ``` {#codeblock_u25_d7q_90u}
        --当MaxCompute和Table Store的Owner是同一个账号时，进行如下设置。
        {
        "Statement": [
        {
         "Action": "sts:AssumeRole",
         "Effect": "Allow",
         "Principal": {
           "Service": [
             "odps.aliyuncs.com"
           ]
         }
        }
        ],
        "Version": "1"
        }
        --当MaxCompute和Table Store的Owner不是同一个账号时，进行如下设置。
        {
        "Statement": [
        {
         "Action": "sts:AssumeRole",
         "Effect": "Allow",
         "Principal": {
           "Service": [
             "MaxCompute的Owner云账号的UID@odps.aliyuncs.com"
           ]
         }
        }
        ],
        "Version": "1"
        }
        ```

        **说明：** 您可以单击右上角的头像，进入账号管理页面查看云账号的UID。

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12076/156283555049672_zh-CN.jpg)

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12076/156283555049673_zh-CN.jpg)

    3.  编辑该角色的授权策略AliyunODPSRolePolicy。

        ``` {#codeblock_ssl_1xi_ecr}
        {
        "Version": "1",
        "Statement": [
        {
         "Action": [
           "ots:ListTable",
           "ots:DescribeTable",
           "ots:GetRow",
           "ots:PutRow",
           "ots:UpdateRow",
           "ots:DeleteRow",
           "ots:GetRange",
           "ots:BatchGetRow",
           "ots:BatchWriteRow",
           "ots:ComputeSplitPointsBySize"
         ],
         "Resource": "*",
         "Effect": "Allow"
        }
        ]
        }
        --还可自定义其它权限。
        ```

    4.  将权限AliyunODPSRolePolicy授权给该角色。

## 创建外部表 {#section_dh2_ksb_wdb .section}

MaxCompute通过创建外部表，把对Table Store表数据的描述引入到MaxCompute的meta系统内部后，即可实现对Table Store数据的处理。本节通过下述示例为您说明MaxCompute对接Table Store的一些概念和实现。

创建外部表语句示例如下。

``` {#codeblock_fyj_ux1_g6s}
DROP TABLE IF EXISTS ots_table_external;
CREATE EXTERNAL TABLE IF NOT EXISTS ots_table_external
(
odps_orderkey bigint,
odps_orderdate string,
odps_custkey bigint,
odps_orderstatus string,
odps_totalprice double
)
STORED BY 'com.aliyun.odps.TableStoreStorageHandler' -- (1)
WITH SERDEPROPERTIES ( -- (2)
'tablestore.columns.mapping'=':o_orderkey,:o_orderdate,o_custkey, o_orderstatus,o_totalprice', -- ①
'tablestore.table.name'='ots_tpch_orders' -- ②
'odps.properties.rolearn'='acs:ram::xxxxx:role/aliyunodpsdefaultrole' --③
)
LOCATION 'tablestore://odps-ots-dev.cn-shanghai.ots-internal.aliyuncs.com'; -- （3）
```

示例说明：

-   `com.aliyun.odps.TableStoreStorageHandler`是MaxCompute内置的处理Table Store数据的StorageHandler，定义了MaxCompute和Table Store的交互，相关逻辑由MaxCompute实现。
-   `SERDEPROPERITES`是提供参数选项的接口，在使用TableStoreStorageHandler时，有三个必须指定的选项，分别是tablestore.columns.mapping、tablestore.table.name、odps.properties.rolearn。
    1.  tablestore.columns.mapping：用于描述MaxCompute将访问的Table Store表的列，包括主键和属性列。
        -   以冒号（：）开头用于表示Table Store主键，例如示例中的`:o_orderkey`和`:o_orderdate`，其它的均为属性列。
        -   Table Store支持1~4个主键，主键类型为STRING、INTEGER和BINARY，其中第一个主键为分区键。
        -   在指定映射时，您必须提供指定Table Store表的所有主键，无需提供全部的属性列，只需提供需要通过MaxCompute访问的属性列。提供的属性列必须是Table Store表的列，否则即使外表可以创建成功，查询时也会报错。
    2.  tablestore.table.name：需要访问的Table Store表名。如果指定的Table Store表名错误（不存在），则会报错，MaxCompute不会主动创建Table Store表。
    3.  odps.properties.rolearn中的信息是RAM中AliyunODPSDefaultRole的ARN信息。您可以通过RAM控制台中的**RAM角色管理**进行获取。
-   LOCATION Clause：用来指定Table Store的Instance名、Endpoint等具体信息。这里的Table Store数据的安全访问建立在前文介绍的RAM/STS授权的前提上。

您可以执行如下语句，查看创建好的外部表结构信息。

``` {#codeblock_92h_1zw_e76}
desc extended <table_name>;
```

在返回的信息里，除了包含和内部表一样的基础信息，Extended Info还包含外部表StorageHandler 、Location等信息。

## 查询外部表 {#section_dcl_ssb_wdb .section}

创建完外部表后，Table Store的数据便引入到了MaxCompute生态中，即可通过正常的MaxCompute SQL语法访问Table Store数据，如下所示。

``` {#codeblock_cfq_9kw_i5s}
SELECT odps_orderkey, odps_orderdate, SUM(odps_totalprice) AS sum_total
FROM ots_table_external
WHERE odps_orderkey > 5000 AND odps_orderkey < 7000 AND odps_orderdate >= '1996-05-03' AND odps_orderdate < '1997-05-01'
GROUP BY odps_orderkey, odps_orderdate
HAVING sum_total> 400000.0;
```

使用常见的MaxCompute SQL语句访问Table Store时，所有的操作细节（例如列名的选择）是在MaxCompute内部处理完成的。上述SQL示例中，使用的列名是odps\_orderkey、odps\_totalprice等，而不是原始Table Store中的主键名o\_orderkey或属性列名o\_totalprice，这是因为在创建外部表的DDL语句中，已经做了对应的mapping。您也可以在创建外部表时按需选择保留原始的Table Store主键/列名。

如果您需要对一份数据做多次计算，相比每次从Table Store去远程读数据，更高效的方法是先一次性把需要的数据导入到MaxCompute内部成为一个MaxCompute（内部）表，示例如下。

``` {#codeblock_imd_k12_vpz}
CREATE TABLE internal_orders AS
SELECT odps_orderkey, odps_orderdate, odps_custkey, odps_totalprice
FROM ots_table_external
WHERE odps_orderkey > 5000 ;
```

现在internal\_orders就是一个MaxCompute表了，也拥有所有MaxCompute内部表的特性，包括高效的压缩列存储数据格式、完整的内部宏数据以及统计信息等。同时因为表存储在MaxCompute内部，所以访问速度会比访问外部的Table Store更快。这种方法非常适用于需要进行多次计算的热点数据。

## MaxCompute导出数据到Table Store {#section_gy2_xsb_wdb .section}

**说明：** MaxCompute不会主动创建外部的Table Store表，所以在对Table Store表进行数据输出之前，必须保证该表已经在Table Store上完成创建（否则将报错）。

根据上面的操作，您已创建了外部表ots\_table\_external来打通MaxCompute与Table Store数据表ots\_tpch\_orders的链路，同时还有一份存储在MaxCompute内部表internal\_orders的数据。现在，如果您需要对internal\_orders中的数据进行处理后再写回Table Store，则可通过对外部表执行`insert overwrite table`操作实现，示例如下。

``` {#codeblock_3af_wet_rl2}
INSERT OVERWRITE TABLE ots_table_external
SELECT odps_orderkey, odps_orderdate, odps_custkey, CONCAT(odps_custkey, 'SHIPPED'), CEIL(odps_totalprice)
FROM internal_orders;
```

**说明：** 

如果ODPS表内数据本身有一定的顺序，例如已经按照Primary Key做过一次排序，则在写入到OTS表时，会导致压力集中在一个OTS分区上面，无法充分利用分布式写入的特点。因此，当出现这种情况时，建议您通过`distribute by rand()`先将数据打散，示例如下。

``` {#codeblock_vd6_6v8_vf7}
INSERT OVERWRITE TABLE ots_table_external
SELECT odps_orderkey, odps_orderdate, odps_custkey, CONCAT(odps_custkey, 'SHIPPED'), CEIL(odps_totalprice)
FROM (SELECT * FROM internal_orders DISTRIBUTE BY rand()) t;
```

对于Table Store这种KV数据的NoSQL存储介质，MaxCompute的输出将只影响相对应主键所在的行，例如示例中只影响所有odps\_orderkey + odps\_orderdate这两个主键值能对应行上的数据。而且在这些Table Store行上面，也只会更新在创建外部表（ots\_table\_external）时指定的属性列，而不会修改未在外部表中出现的数据列。

**说明：** 

-   将MaxCompute中的数据写入OTS时一次不能超过4MB，否则需要用户剔除掉超大数据再写入。此时可能会产生报错。

    ``` {#codeblock_vv0_woo_aim}
    ODPS-0010000:System internal error - Output to TableStore failed with exception:
    TableStore BatchWrite request id XXXXX failed with error code OTSParameterInvalid and message:The total data size of BatchWriteRow request exceeds the limit
    ```

-   将数据批量写入或分行写入，都算一次操作。详细描述请参考[BatchWriteRow](https://help.aliyun.com/document_detail/27311.html)。因此如果批量写入数据量太大，也可以分行写入。
-   将数据批量写入时请注意不要有重复行，否则可能产生如下报错。

    ``` {#codeblock_5u8_6qb_2i8}
    ErrorCode: OTSParameterInvalid, ErrorMessage: The input parameter is invalid 
    ```

    详细描述请参考[使用BatchWriteRow一次提交100条数据的时候报OTSParameterInvalid错误](https://help.aliyun.com/knowledge_detail/38586.html)。


更多详情请参见[MaxCompute访问TableStore（OTS）数据](https://yq.aliyun.com/articles/69314)。

