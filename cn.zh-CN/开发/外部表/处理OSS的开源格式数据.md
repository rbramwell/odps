# 处理OSS的开源格式数据 {#concept_tlm_34v_ydb .concept}

本文将为您介绍对于存储在OSS上的各种流行的开源数据格式（ORC、PARQUET、SEQUENCEFILE、RCFILE、AVRO和TEXTFILE）如何通过非结构化框架在MaxCompute进行处理。

您可以通过DataWorks配合MaxCompute对外部表进行可视化的创建、搜索、查询、配置、加工和分析。详情请参见[外部表](../../../../cn.zh-CN/使用指南/数据开发/外部表.md#)。

[访问OSS非结构化数据](cn.zh-CN/开发/外部表/访问OSS非结构化数据.md)为您介绍了如何在MaxCompute上访问存储在OSS上的文本、音频、图像等格式的数据。非结构框架会直接调用开源社区的实现来进行开源数据格式的解析，并且与MaxCompute系统无缝对接。

**说明：** 处理OSS的开源格式数据前，需要首先对OSS进行[STS模式授权](cn.zh-CN/开发/外部表/OSS的STS模式授权.md)。

## 创建External Table {#section_bxs_hsv_ydb .section}

MaxCompute非结构化数据框架通过External Table与各种数据的关联，关联OSS上开源格式数据的External Table建表的DDL语句格式如下所示。

```
DROP TABLE [IF EXISTS] <external_table>;
CREATE EXTERNAL TABLE [IF NOT EXISTS] <external_table>
(<column schemas>)
[PARTITIONED BY (partition column schemas)]
[ROW FORMAT SERDE '<serde class>'
  [WITH SERDEPROPERTIES ('odps.properties.rolearn'='${roleran}' [,'name2'='value2',...])]
]
STORED AS <file format>
LOCATION 'oss://${endpoint}/${bucket}/${userfilePath}/';
```

**说明：** 该语法格式与Hive的语法相当接近，但需注意以下问题。

-   STORED AS关键字，在该语法格式中不是[普通非结构化外表](cn.zh-CN/开发/外部表/访问OSS非结构化数据.md)用的STORED BY关键字，这是目前在读取开源兼容数据时独有的。

    STORED AS后面接的是文件格式名字，例如ORC/PARQUET/RCFILE/SEQUENCEFILE/TEXTFILE等。

-   外部表的column schemas必须与具体OSS上存储的数据的schema相符合。
-   ROW FORMAT SERDE：非必选选项，只有在使用一些特殊的格式上，比如TEXTFILE时才需要使用。
-   WITH SERDEPROPERTIES：当关联OSS权限使用[STS模式授权](cn.zh-CN/开发/外部表/OSS的STS模式授权.md)时，需要该参数指定odps.properties.rolearn属性，属性值为RAM中具体使用的Role的Arn的信息。您可以在配置STORED AS <file format\>的同时也通过<serde class\>说明file format文件格式。以ORC文件格式为例，如下所示。

    ```
    CREATE EXTERNAL TABLE [IF NOT EXISTS] <external_table>
    (<column schemas>)
    [PARTITIONED BY (partition column schemas)]
    ROW FORMAT SERDE 'org.apache.hadoop.hive.ql.io.orc.OrcSerde'
    WITH SERDEPROPERTIES ('odps.properties.rolearn'='${roleran}'
    STORED AS ORC
    LOCATION 'oss://${endpoint}/${bucket}/${userfilePath}/'
    ```

    不同file format对应的serde class如下：

    -   CFILE: com.aliyun.apsara.serde.CFileSerDe
    -   ALIORC: com.aliyun.apsara.serde.AliOrcSerDe
    -   SEQUENCEFILE: org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe
    -   TEXTFILE: org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe
    -   RCFILE: org.apache.hadoop.hive.serde2.columnar.LazyBinaryColumnarSerDe
    -   ORC: org.apache.hadoop.hive.ql.io.orc.OrcSerde
    -   ORCFILE: org.apache.hadoop.hive.ql.io.orc.OrcSerde
    -   PARQUET: org.apache.hadoop.hive.ql.io.parquet.serde.ParquetHiveSerDe
    -   AVRO: org.apache.hadoop.hive.serde2.avro.AvroSerDe
-   如果不使用STS模式授权，则无需指定odps.properties.rolearn属性，直接在Location传入明文`AccessKeyId`和`AccessKeySecret`。
-   Location如果关联OSS，需使用明文AK，写法如下所示。

    ```
    LOCATION 'oss://${accessKeyId}:${accessKeySecret}@${endpoint}/${bucket}/${userPath}/'
    ```

-   访问OSS外部表，目前不支持使用外网Endpoint。
-   目前STORE AS单个文件大小不能超过3G，如果文件过大，建议split拆分。

## 关联OSS的PARQUET数据 {#section_spx_ttv_ydb .section}

假设有一些PARQUET文件存放在一个OSS路径上，每个文件都是PARQUET格式，存放的schema为16列（4列Bigint、4列Double和8列String）的数据，建表DDL语句示例如下所示。

```
CREATE EXTERNAL TABLE tpch_lineitem_parquet
(
  l_orderkey bigint,
  l_partkey bigint,
  l_suppkey bigint,
  l_linenumber bigint,
  l_quantity double,
  l_extendedprice double,
  l_discount double,
  l_tax double,
  l_returnflag string,
  l_linestatus string,
  l_shipdate string,
  l_commitdate string,
  l_receiptdate string,
  l_shipinstruct string,
  l_shipmode string,
  l_comment string
)
STORED AS PARQUET
LOCATION 'oss://${accessKeyId}:${accessKeySecret}@oss-cn-hangzhou-zmf.aliyuncs.com/bucket/parquet_data/';
```

默认PARQUET数据不压缩，如果您需要在MaxCompute上压缩PARQUET数据，需要`set odps.sql.hive.compatible=true;`。支持压缩类型包括：SNAPPY, GZIP。

## 关联OSS的TEXT数据 {#section_gvy_15v_ydb .section}

如果数据是每行以JSON格式，存储成OSS上TEXTFILE文件，同时数据在OSS通过多个目录组织，这时可以使用MaxCompute分区表和数据关联，建分区表DDL语句示例如下所示。

```
CREATE EXTERNAL TABLE tpch_lineitem_textfile
(
  l_orderkey bigint,
  l_partkey bigint,
  l_suppkey bigint,
  l_linenumber bigint,
  l_quantity double,
  l_extendedprice double,
  l_discount double,
  l_tax double,
  l_returnflag string,
  l_linestatus string,
  l_shipdate string,
  l_commitdate string,
  l_receiptdate string,
  l_shipinstruct string,
  l_shipmode string,
  l_comment string
)
PARTITIONED BY (ds string)
ROW FORMAT serde 'org.apache.hive.hcatalog.data.JsonSerDe'
STORED AS TEXTFILE
LOCATION 'oss://${accessKeyId}:${accessKeySecret}@oss-cn-hangzhou-zmf.aliyuncs.com/bucket/text_data/';
```

如果OSS表目录下面的子目录是以Partition Name方式组织，示例如下。

```
oss://${accessKeyId}:${accessKeySecret}@oss-cn-hangzhou-zmf.aliyuncs.com/bucket/text_data/ds=20170102/'
oss://${accessKeyId}:${accessKeySecret}@oss-cn-hangzhou-zmf.aliyuncs.com/bucket/text_data/ds=20170103/'
...
```

则可以使用以下DDL语句ADD PARTITION。

```
ALTER TABLE tpch_lineitem_textfile ADD PARTITION(ds="20170102");
ALTER TABLE tpch_lineitem_textfile ADD PARTITION(ds="20170103");
```

如果OSS分区目录不是按这种方式组织，或者根本不在表目录下，示例如下。

```
oss://${accessKeyId}:${accessKeySecret}@oss-cn-hangzhou-zmf.aliyuncs.com/bucket/text_data_20170102/;
oss://${accessKeyId}:${accessKeySecret}@oss-cn-hangzhou-zmf.aliyuncs.com/bucket/text_data_20170103/;
...
```

这种情况下，可以使用以下DDL语句ADD PARTITION。

```
ALTER TABLE tpch_lineitem_textfile ADD PARTITION(ds="20170102")
LOCATION 'oss://${accessKeyId}:${accessKeySecret}@oss-cn-hangzhou-zmf.aliyuncs.com/bucket/text_data_20170102/';
ALTER TABLE tpch_lineitem_textfile ADD PARTITION(ds="20170103")
LOCATION 'oss://${accessKeyId}:${accessKeySecret}@oss-cn-hangzhou-zmf.aliyuncs.com/bucket/text_data_20170103/';
...
```

TEXT数据支持SERDEPROPERTIES（key：默认值）

```
Fields terminator：'\001'
Escape delimitor：'\\'
Collection items terminator：'\002'
Map keys terminator：'\003'
Lines terminator：'\n'
Null defination：'\\N'
```

## 关联OSS的CSV数据 {#section_jq3_sgq_5fb .section}

DDL语句格式如下所示。

```
CREATE EXTERNAL TABLE [IF NOT EXISTS] 
(<column schemas>)
[PARTITIONED BY (partition column schemas)]
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.OpenCSVSerde'
  WITH SERDEPROPERTIES
    ('separatorChar'=',', 'quoteChar'='"', 'escapeChar'='\\')
STORED AS TEXTFILE
LOCATION 'oss://${endpoint}/${bucket}/${userfilePath}/';
```

从上述语句可以看出，CSV数据DDL语句支持SERDEPROPERTIES（key：默认值）

```
separatorChar：','
quoteChar：'"'
escapeChar：'\'
```

**说明：** hive OpenCSVSerde只支持string类型。

OpenCSVSerde当前不属于Builtin Serde，DML语句执行时您需要设置set odps.sql.hive.compatible=true;

## 关联OSS的JSON数据 {#section_hsx_nhq_5fb .section}

DDL语句格式如下所示，支持SERDEPROPERTIES。

```
CREATE EXTERNAL TABLE [IF NOT EXISTS] 
(<column schemas>)
[PARTITIONED BY (partition column schemas)]
ROW FORMAT SERDE 'org.apache.hive.hcatalog.data.JsonSerDe'
STORED AS TEXTFILE
LOCATION 'oss://${endpoint}/${bucket}/${userfilePath}/';
```

## 关联OSS的ORC数据 {#section_ycz_nhq_5fb .section}

DDL语句格式如下所示。

```
CREATE EXTERNAL TABLE [IF NOT EXISTS] 
(<column schemas>)
[PARTITIONED BY (partition column schemas)]
STORED AS ORC
LOCATION 'oss://${endpoint}/${bucket}/${userfilePath}/';
```

## 关联OSS的AVRO数据 {#section_wh1_4hq_5fb .section}

DDL语句格式如下所示。

```
CREATE EXTERNAL TABLE [IF NOT EXISTS] 
(<column schemas>)
[PARTITIONED BY (partition column schemas)]
STORED AS AVRO
LOCATION 'oss://${endpoint}/${bucket}/${userfilePath}/';
```

## 关联OSS的SEQUENCEFILE数据 {#section_cqs_b3q_5fb .section}

```
CREATE EXTERNAL TABLE [IF NOT EXISTS] 
(<column schemas>)
[PARTITIONED BY (partition column schemas)]
STORED AS SEQUENCEFILE
LOCATION 'oss://${endpoint}/${bucket}/${userfilePath}/';
```

## 读取以及处理OSS的开源格式数据 {#section_ewt_bvv_ydb .section}

对比前文的两个创建外部表示例，可以看出对于不同文件类型，只要简单修改STORED AS后的格式名。在下述示例中将只集中描述对上面PARQUET数据对应的外表（tpch\_lineitem\_parquet）的处理。如果要处理不同的文件类型，只要在DDL创建外表时指定是PARQUET/ORC/TEXTFILE/RCFILE/TEXTFILE即可，处理数据的语句一样。

-   直接读取以及处理OSS的开源数据 

    创建数据外表进行关联后，直接对外表就可以进行与普通MaxCompute表一样的操作，如下所示。

    ```
    SELECT l_returnflag, l_linestatus,
    SUM(l_extendedprice*(1-l_discount)) AS sum_disc_price,
    AVG(l_quantity) AS avg_qty,
    COUNT(*) AS count_order
    FROM tpch_lineitem_parquet
    WHERE l_shipdate <= '1998-09-02'
    GROUP BY l_returnflag, l_linestatus;
    ```

    外表tpch\_lineitem\_parquet被当作一个普通的内部表一样使用，不同之处在于，MaxCompute内部的计算引擎是直接从OSS读取对应的PARQUET数据进行处理的。

    前文创建的关联textfile的外部分区表tpch\_lineitem\_textfile，因为使用了`ROW FORMAT` + `STORED AS`，需要手动设置flag `set odps.sql.hive.compatible=true;`（只使用STORED AS，odps.sql.hive.compatible默认为FALSE）再创建外部表或读取数据，否则会有报错。

    ```
    SELECT * FROM tpch_lineitem_textfile LIMIT 1;
    FAILED: ODPS-0123131:User defined function exception - Traceback:
    com.aliyun.odps.udf.UDFException: java.lang.ClassNotFoundException: com.aliyun.odps.hive.wrapper.HiveStorageHandlerWrapper
    --需要手动设置hive兼容flag
    set odps.sql.hive.compatible=true;
    SELECT * FROM tpch_lineitem_textfile LIMIT 1;
    +------------+------------+------------+--------------+------------+-----------------+------------+------------+--------------+--------------+------------+--------------+---------------+----------------+------------+-----------+
    | l_orderkey | l_partkey  | l_suppkey  | l_linenumber | l_quantity | l_extendedprice | l_discount | l_tax      | l_returnflag | l_linestatus | l_shipdate | l_commitdate | l_receiptdate | l_shipinstruct | l_shipmode | l_comment |
    +------------+------------+------------+--------------+------------+-----------------+------------+------------+--------------+--------------+------------+--------------+---------------+----------------+------------+-----------+
    | 5640000001 | 174458698  | 9458733    | 1            | 14.0       | 23071.58        | 0.08       | 0.06       | N            | O            | 1998-01-26 | 1997-11-16   | 1998-02-18    | TAKE BACK RETURN | SHIP       | cuses nag silently. quick |
    +------------+------------+------------+--------------+------------+-----------------+------------+------------+--------------+--------------+------------+--------------+---------------+----------------+------------+-----------+
    ```

    **说明：** 直接使用外表，每次读取数据都需要涉及外部OSS的I/O操作，且MaxCompute系统本身针对内部存储做的许多高性能优化都用不上，如此一来性能上就会有所损失。 因此如果是需要对数据进行反复计算以及对计算的高效性比较敏感的场景，推荐使用下文介绍的用法：先将数据导入MaxCompute内部再进行计算。

    SQL（ create、select、insert等操作）中涉及到这几个复杂数据类型，需在SQL语句前加语句`set odps.sql.type.system.odps2=true;`，执行时set语句和SQL语句一起提交执行。详情请参见[数据类型](../../../../cn.zh-CN/开发/基本概念/数据类型.md#)。

-   将OSS的开源数据导入MaxCompute再进行计算 

    首先创建一个与外部表schema一样的内部表tpch\_lineitem\_internal，然后将OSS上的开源数据导入MaxCompute内部表，以MaxCompute内部数据存储格式进行存储。

    ```
    CREATE TABLE tpch_lineitem_internal LIKE tpch_lineitem_parquet;
    INSERT OVERWRITE TABLE tpch_lineitem_internal;
    SELECT * FROM tpch_lineitem_parquet;
    ```

    接下来直接对内部表进行同样的操作。

    ```
    SELECT l_returnflag, l_linestatus,
    SUM(l_extendedprice*(1-l_discount)) AS sum_disc_price,
    AVG(l_quantity) AS avg_qty,
    COUNT(*) AS count_order
    FROM tpch_lineitem_internal
    WHERE l_shipdate <= '1998-09-02'
    GROUP BY l_returnflag, l_linestatus;
    ```

    通过此方式将数据先导入MaxCompute系统进行存储，对同样数据进行计算处理会更高效。


## 处理OSS数据常见问题 {#section_zm3_bym_1gb .section}

**作业报错**：`Inline data exceeds the maximun allowed size`。

**问题原因**：OSS Store对于每一个小文件有一个大小限制，如果超过3GB则报错。

**处理方法**：针对该问题，您可以通过调整以下两个flag值进行处理。其原理是通过flag调整执行计划，控制每个reducer写入外部表OSS的数据大小，使得OSS Store文件不超过3GB的限制。

```
set odps.sql.mapper.split.size=256; #调整每个mapper读取table数据的大小，单位是MB。
set odps.sql.reducer.instances=100; #调整执行计划的reducer数量。
```

