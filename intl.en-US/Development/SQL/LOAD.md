---
keyword: [LOAD, data import]
---

# LOAD

This topic describes how to use LOAD statements to import data from external storage, such as Object Storage Service \(OSS\), into a table or a table partition.

## Precautions

MaxCompute must be authorized to access data in external storage. The methods used to authorize MaxCompute to access external data by executing LOAD statements are the same as those used to authorize MaxCompute external tables. You can use one of the following methods for authorization:

-   Specify the AccessKey ID and AccessKey secret in the data storage path. Use OSS as an example:

    ```
    'oss://<yourAccessKeyId>:<yourAccessKeySecret>@oss-cn-hangzhou-zmf.aliyuncs.com/my_bucket_id/my_location/'
    ```

    **Note:** If you use this method, the AccessKey ID and AccessKey secret are displayed in plaintext. This method entails security risks and is not recommended.

-   Use STS. For more information, see [STS authorization](/intl.en-US/Development/External table/STS authorization.md).

## Syntax

-   Use an extractor or a storage handler to import data.

    ```
    LOAD OVERWRITE|INTO TABLE table_name [PARTITION(partcol1=val1, partcol2=val2 ...)]
    FROM LOCATION external_location
    [STORED BY StorageHandler]
    [WITH SERDEPROPERTIES (Options)];
    ```

-   Import data in open-source formats.

    ```
    LOAD OVERWRITE|INTO TABLE table_name [PARTITION(partcol1=val1, partcol2=val2 ...)]
    FROM LOCATION external_location
    [ROW FORMAT SERDE '<serde class>'
      [WITH SERDEPROPERTIES ('odps.properties.rolearn'='${roleran}' [,'name2'='value2',...])]
    ]
    STORED AS <file format>;
    ```


## Description

LOAD statements can append data to or overwrite data in a table or partition.

## Parameters

-   table\_name: the name of the destination table to which you want to append data. You must create a destination table before you append data to it. The schema of the destination table must be the same as the format of the external data.
-   LOAD INTO: appends data to a table or partition.
-   LOAD OVERWRITE: clears the existing data in a table or partition and then inserts data into the table or partition.
-   SORTED BY: specifies the name of the storage handler. You can use this clause in the same way as you use it for MaxCompute external tables. For more information, see [Access OSS data by using the built-in extractor](/intl.en-US/Development/External table/Access OSS data by using the built-in extractor.md).
-   STORED AS: specifies the format of the imported data file, such as ORC, PARQUET, RCFILE, SEQUENCEFILE, and TEXTFILE. You can use this clause in the same way as you use it for MaxCompute external tables. For more information, see [Process OSS data stored in open-source formats](/intl.en-US/Development/External table/Process OSS data stored in open-source formats.md).
-   ROW FORMAT SERDE: This clause is required only when you use special formats, such as TEXTFILE. You can use this clause in the same way as you use it for MaxCompute external tables. For more information, see [Process OSS data stored in open-source formats](/intl.en-US/Development/External table/Process OSS data stored in open-source formats.md).

    Mapping between file formats and SerDe classes:

    -   `SEQUENCEFILE: org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe`
    -   `TEXTFILE: org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe`
    -   `RCFILE: org.apache.hadoop.hive.serde2.columnar.LazyBinaryColumnarSerDe`
    -   `ORC: org.apache.hadoop.hive.ql.io.orc.OrcSerde`
    -   `ORCFILE: org.apache.hadoop.hive.ql.io.orc.OrcSerde`
    -   `PARQUET: org.apache.hadoop.hive.ql.io.parquet.serde.ParquetHiveSerDe`
    -   `AVRO: org.apache.hadoop.hive.serde2.avro.AvroSerDe`
-   SERDEPROPERTIES: If you use STS to authorize MaxCompute, you must use this parameter to specify the value of `odps.properties.rolearn`, which is the Alibaba Cloud Resource Name \(ARN\) specified in the AliyunODPSDefaultRole RAM role. For more information, see [STS authorization](/intl.en-US/Development/External table/STS authorization.md).

## Examples

-   Import text files to a destination table.

    ```
    LOAD OVERWRITE TABLE tsv_load_tbl 
    FROM
    LOCATION 'oss://accessId:accesssKey@oss-cn-hangzhou-zmf.aliyuncs.com/my_bucket_id/my_location/'
    STORED BY 'com.aliyun.odps.TsvStorageHandler'
    WITH SERDEPROPERTIES ('odps.text.option.delimiter'='\t');
    ```

-   Import data in open-source formats to a destination table.

    ```
    -- Create destination table oss_load_static_part.
    CREATE TABLE oss_load_static_part (
      c_int BIGINT,
      c_double DOUBLE,
      c_string STRING,
      c_bool BOOLEAN,
      c_datetime STRING )
    PARTITIONED BY (ds STRING);
    -- Import data in open-source formats from OSS to oss_load_static_part.
    LOAD OVERWRITE TABLE oss_load_static_part PARTITION(ds='20190101')
    FROM
    LOCATION 'oss://<yourAccessKeyId>:<yourAccessKeySecret>@oss-cn-hangzhou-zmf.aliyuncs.com/my_bucket_id/my_location/'
    STORED AS PARQUET;
    ```

    **Note:** Data is loaded to a partitioned table. The schema of the data in the external storage path does not define partitioning columns. In this example, the external data only includes data in the c\_int, c\_double, c\_datetime, c\_string, and c\_bool columns.

-   Import data to a destination table in dynamic partition mode.

    If the subdirectories of an OSS table directory are organized by partition name, you can import data to a partitioned table in dynamic partition mode.

    ```
    -- The following code shows the organization of the OSS table directory. Data in the external storage path does not include data in partitioning columns. The information of partitioning columns is part of the path.
    oss://${accessKeyId}:${accessKeySecret}@oss-cn-hangzhou-zmf.aliyuncs.com/bucket/text_data/ds=20170102/'
    oss://${accessKeyId}:${accessKeySecret}@oss-cn-hangzhou-zmf.aliyuncs.com/bucket/text_data/ds=20170103/'
    -- Create destination table oss_load_dyn_part.
    CREATE TABLE oss_load_dyn_part (
      int BIGINT,
      c_double DOUBLE,
      c_string STRING,
      c_bool BOOLEAN)
    PARTITIONED BY (ds STRING);
    Import data to oss_load_dyn_part in dynamic partition mode.
    LOAD OVERWRITE TABLE oss_load_dyn_part PARTITION(ds)
    FROM
    LOCATION 'oss://accessId:accesssKey@oss-cn-hangzhou-zmf.aliyuncs.com/bucket/text_data/'
    ROW FORMAT serde 'org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe'
    WITH SERDEPROPERTIES(
      'Fields terminator'='\001',
      'Escape delimitor'='\\',
      'Collection items terminator'='\002',
      'Map keys terminator'='\003',
      'Lines terminator'='\n',
      'Null defination'='\\N')
    STORED AS TEXTFILE ;
    ```

    -   ROW FORMAT and SERDEPROPERTIES can be omitted if you use their default values.
    -   You can set SERDEPROPERTIES for data of the TEXT type. The following code lists the default values:

        ```
        Fields terminator:'\001'
        Escape delimitor:'\\'
        Collection items terminator:'\002'
        Map keys terminator:'\003'
        Lines terminator:'\n'
        Null defination:'\\N'
        ```


