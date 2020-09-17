---
keyword: data partition
---

# Partition data by using foreign tables

This topic describes how to partition data by using foreign tables.

You can use the LOCATION parameter to specify an Object Storage Service \(OSS\) folder that maps a foreign table. After you specify an OSS folder by using the LOCATION parameter, MaxCompute reads all data in the folder, including all files in the sub-folders. If the OSS folder contains a large amount of data or data in the OSS folder increases over time, a full-text scan will cause extra I/O operations and processing time. You can use one of the following methods to address this issue:

-   Store data in multiple OSS folders that map different foreign tables: You can specify multiple OSS folders to store data and use the LOCATION parameter to specify each of the folders for a foreign table. Then, MaxCompute can read data in each folder by using the corresponding foreign table.
-   Store data in an OSS folder that maps a foreign partitioned table: MaxCompute allows you to use foreign partitioned tables to better manage data.

## Standard organization method and path structure of partitioned data in OSS

MaxCompute does not have the permission to manage data stored in external storage such as OSS. If MaxCompute needs to access OSS data by using a foreign partitioned table, make sure that the storage path of the OSS data is in the following format:

```
partitionKey1=value1\partitionKey2=value2\...
```

1.  Create a foreign partitioned table.

    Assume that your daily log files are stored in OSS, and you want to use MaxCompute to process the data on a daily basis. If the log files are in the CSV format and can be accessed by using the built-in extractor or a custom extractor, you can execute the following SQL statement to create a foreign partitioned table:

    ```
    CREATE EXTERNAL TABLE log_table_external (
        click STRING,
        ip STRING,
        url STRING
      )
      PARTITIONED BY (
        year STRING,
        month STRING,
        day STRING
      )
      STORED BY 'com.aliyun.odps.CsvStorageHandler'
      WITH SERDEPROPERTIES (
     'odps.properties.rolearn'='acs:ram::xxxxx:role/aliyunodpsdefaultrole'
    ) 
      LOCATION 'oss://oss-cn-hangzhou-zmf.aliyuncs.com/oss-odps-test/log_data/';
    ```

    You can use the PARTITIONED BY clause to specify the foreign table as a foreign partitioned table. In this example, the partitioned table has three levels of partitions with the partition keys `year`, `month`, and `day`, respectively.

    For the partitions to take effect, you must specify an OSS folder in the same format as the folder specified by LOCATION in the preceding example. The following example shows a valid folder:

    ```
    osscmd ls oss://oss-odps-test/log_data/
    2017-01-14 08:03:35 128MB Standard oss://oss-odps-test/log_data/year=2016/month=06/day=01/logfile
    2017-01-14 08:04:12 127MB Standard oss://oss-odps-test/log_data/year=2016/month=06/day=01/logfile.1
    2017-01-14 08:05:02 118MB Standard oss://oss-odps-test/log_data/year=2016/month=06/day=02/logfile
    2017-01-14 08:06:45 123MB Standard oss://oss-odps-test/log_data/year=2016/month=07/day=10/logfile
    2017-01-14 08:07:11 115MB Standard oss://oss-odps-test/log_data/year=2016/month=08/day=08/logfile
    ...
    ```

    **Note:** In this example, data is uploaded to OSS by using OSS tools such as osscmd. Therefore, the storage path of the data is the path to which the data is uploaded.

2.  Execute `ALTER TABLE ADD PARTITION` statements to add partitions to the foreign partitioned table.

    ```
    ALTER TABLE log_table_external ADD PARTITION (year = '2016', month = '06', day = '01')
    ALTER TABLE log_table_external ADD PARTITION (year = '2016', month = '06', day = '02')
    ALTER TABLE log_table_external ADD PARTITION (year = '2016', month = '07', day = '10')
    ALTER TABLE log_table_external ADD PARTITION (year = '2016', month = '08', day = '08')
    ...
    ```

    **Note:** The preceding operations are the same as those on an internal partitioned table in MaxCompute. For more information about partitions, see [Partition](/intl.en-US/Product Introduction/Definitions/Partition.md). If you have prepared data and added partitions to a foreign table of MaxCompute, you can execute SQL statements to process the OSS data that maps these partitions.

3.  Analyze data.
    -   Execute the following SQL statement to obtain the number of distinctive IP addresses in the logs that were recorded on June 1, 2016:

        ```
        SELECT count(distinct(ip)) FROM log_table_external WHERE year = '2016' AND month = '06' AND day = '01';
        ```

        MaxCompute accesses only the files, logfile and logfile.1 in this example, in the log\_data/year=2016/month=06/day=01 sub-folder but not all files in the log\_data folder. This prevents unnecessary I/O operations.

    -   Execute the following SQL statement to obtain the number of distinctive IP addresses in the logs that were recorded in the second half of 2016:

        ```
        SELECT count(distinct(ip)) FROM log_table_external WHERE year = '2016' AND month > '06';
        ```


## Custom path of partitioned data in OSS

If you have historical data stored in OSS paths that are not in the `partitionKey1=value1\partitionKey2=value2\...` format, you can add partitions that map these OSS paths to a foreign table of MaxCompute. This way, MaxCompute can access data in these OSS paths.

The following example shows an OSS path that contains only partition values but not partition keys:

```
osscmd ls oss://oss-odps-test/log_data_customized/
2017-01-14 08:03:35 128MB Standard oss://oss-odps-test/log_data_customized/2016/06/01/logfile
2017-01-14 08:04:12 127MB Standard oss://oss-odps-test/log_data_customized/2016/06/01/logfile.1
2017-01-14 08:05:02 118MB Standard oss://oss-odps-test/log_data_customized/2016/06/02/logfile
2017-01-14 08:06:45 123MB Standard oss://oss-odps-test/log_data_customized/2016/07/10/logfile
2017-01-14 08:07:11 115MB Standard oss://oss-odps-test/log_data_customized/2016/08/08/logfile
...
```

You can execute the following SQL statement to map each sub-folder to a specific partition:

```
ALTER TABLE log_table_external ADD PARTITION (year = '2016', month = '06', day = '01')
LOCATION 'oss://oss-cn-hangzhou-zmf.aliyuncs.com/oss-odps-test/log_data_customized/2016/06/01/';
```

In the preceding SQL statement, the LOCATION parameter specifies an OSS path. This path maps the partition that is added to the foreign table by using the `ADD PARTITION` clause. This way, MaxCompute can access data in the OSS path even if the path is not in the `partitionKey1=value1\partitionKey2=value2\...` format.

