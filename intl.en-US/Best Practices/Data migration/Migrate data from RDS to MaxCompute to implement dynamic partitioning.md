# Migrate data from RDS to MaxCompute to implement dynamic partitioning {#concept_gxd_crn_4fb .concept}

This topic describes how to use the data synchronization feature of DataWorks to automatically create partitions and dynamically migrate data from RDS to MaxCompute.

## Preparations {#section_od5_frc_pfb .section}

1.  [Activate MaxCompute](../../../../reseller.en-US/Prepare/Activate MaxCompute.md#) and [Create a project](../../../../reseller.en-US/Prepare/Create a project.md#) in China \(Beijing\).

    **Note:** If you are using DataWorks for the first time, you need to complete the operations described in [Preparation](../../../../reseller.en-US/Prepare/Create an Alibaba Cloud account.md#). For example, you need to get your account ready, set the project role, and configure the project.

2.  Add data sources.

    Add [RDS](../../../../reseller.en-US/User Guide/Data integration/Data source configuration/Configure MySQL data source.md#) as the data source and add [ODPS](https://www.alibabacloud.com/help/zh/faq-detail/74280.htm) as the destination data source for receiving the RDS data.

    After completing the settings, click **New Source** on the data integration page. Then, added data sources are displayed, as shown in the following figure.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/24452/155928446514385_en-US.png)


## Create a partition {#section_sfm_3rc_pfb .section}

After the preparations are completed, the data in RDS needs to be synchronized to MaxCompute on a daily basis, so that the date-based partition can be automatically created. For more information about how to configure a data synchronization task, see [Data development and O&M in DataWorks](../../../../reseller.en-US/Quick Start/Instructions.md#).

1.  Create a destination table.

    In the ODPS database, create a destination table named ods\_user\_info\_d. This table corresponds to a table in RDS. Under **Data Development**, right-click **Create ODPS SQL Node**, create a node named create\_table\_ddl, and enter the table creation statements, as shown in the following figure.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/24452/155928446533545_en-US.png)

    The SQL statements are as follows:

    ```
    
    CREATE TABLE IF NOT EXISTS ods_user_info_d (
    uid STRING COMMENT 'UserID',
    gender STRING COMMENT 'gender',
    age_range STRING COMMENT 'Age',
    zodiac STRING COMMENT 'zodiac'
    )
    PARTITIONED BY (
    dt STRING
    );
    							
    ```

    You can also choose **Business Flow** \> **Table** and select **Create Table**, as shown in the following figure.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/24452/155928446633547_en-US.png)

    For more information, see [Create a table and upload data](https://www.alibabacloud.com/help/zh/doc-detail/84670.htm).

2.  Create a business flow.

    Log on to the DataWorks console and click **Data Analytics**. Right-click **Business Flow** and select **Create Business Flow** to create a workshop, as shown in the following figure.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/24452/155928446633546_en-US.png)

3.  Create and configure a synchronization task node.

    Create a synchronization node named rds\_sync under the workshop business flow, as shown in the following figure.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/24452/155928446614388_en-US.png)

4.  Select the data source and data destination, as shown in the following figure.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/24452/155928446614391_en-US.png)

5.  Set the parameters, as shown in the following figure.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/24452/155928446614396_en-US.png)

    Click **Schedule**. On the displayed Basics page, the default value of Parameters is `${bizdate}` in yyyymmdd format, as shown in the following figure.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/24452/155928446614394_en-US.png)

    **Note:** By default, the value of Parameters corresponds to the value of **Partition** on the **Destination** tab page. The partition date is called business date. In most cases, users process the business data generated in the previous day. Therefore, when the data synchronization task is scheduled and executed, the partition date is automatically replaced with the date one day before the task execution date. To use the task execution date as the partition value \(partition date\), you must customize the parameter.

    -   The custom parameter can be configured in different formats. You can select a date and a format as needed. The custom parameter can be set in one of the following formats:

        -   N years later: `$[add_months(yyyymmdd,12*N)]`
        -   N years ago: `$[add_months(yyyymmdd,-12*N)]`
        -   N months ago: `$[add_months(yyyymmdd,-N)]`
        -   N weeks later: `$[yyyymmdd+7*N]`
        -   N months later: `$[add_months(yyyymmdd,N)]`
        -   N weeks ago: `$[yyyymmdd-7*N]`
        -   N days later: `$[yyyymmdd+N]`
        -   N days ago: `$[yyyymmdd-N]`
        -   N hours later: `$[hh24miss+N/24]`
        -   N hours ago: `$[hh24miss-N/24]`
        -   N minutes later: `$[hh24miss+N/24/60]`
        -   N minutes later: `$[hh24miss-N/24/60]`
        **Note:** 

        -   You need to use brackets \(\[\]\) to edit the value calculation formula of the custom parameter, for example, `key1=$[yyyy-mm-dd]`.
        -   The default calculation unit of the custom parameter is day. For example, `$[hh24miss-N/24/60]` indicates `(yyyymmddhh24miss-(N/24/60 * 1 day))`. The hour, minute, and second are in the format of hh24miss.
        -   The calculation unit of add\_months is month. For example, `$[add_months(yyyymmdd,12 N)-M/24/60]` indicates `(yyyymmddhh24miss-(12 * N * 1 month))-(M/24/60 * 1 day)`. The year, month, and day are in the format of `yyyymmdd`.
        For more information, see [Parameter configuration](../../../../reseller.en-US/User Guide/Data development/Scheduling configuration/Parameter configuration.md#).

6.  Perform the test run.

    Click **Save** to save all configurations, and click **Run**, as shown in the following figure.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/24452/155928446633548_en-US.png)

    View the running log, as shown in the following figure.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/24452/155928446614399_en-US.png)

    In the sample log shown in the preceding figure, the partition value in MaxCompute \(whose printed name is ODPS\) is dt=20181025. This indicates that the partition value is automatically replaced. Verify that the data is successfully migrated to the ODPS table, as shown in the following figure.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/24452/155928446614400_en-US.png)

    **Note:** In MaxCompute 2.0, parameter settings are required for partition table query. Full query is not supported. The SQL statements are as follows:

    ```
    --Check whether the data is successfully written to MaxCompute.
    select count(*) from ods_user_info_d where dt=business date;
    								
    ```

    For more information about the SELECT command, see [Introduction to the SELECT syntax](../../../../reseller.en-US/User Guide/SQL/Select Operation/SELECT syntax.md#).

    Now the data has been migrated to the ODPS table and a partition value has been successfully created. Then, when the task is executed at scheduled time, the data in RDS is automatically synchronized to the date-based partition in MaxCompute.


## Data patching {#section_cbp_cqd_pfb .section}

If you have many historical data that is generated before the execution date, and you want to implement automatic synchronization and partitioning, you can log on to the DataWorks console, click **Maintenance Center**, select the data synchronization node, and click **Patch Data**.

1.  Filter the historical data in RDS by date. For example, filter the historical data generated on 2018-09-13, so that the data can be automatically synchronized to the 20180825 partition in MaxCompute. You can use a WHERE clause to filter data in RDS, as shown in the following figure.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/24452/155928446614401_en-US.png)

2.  Perform data patching.

    Choose **Save** \> **Submit**. After the data is submitted, choose **Maintenance Center** \> **Task List** \> **Cycle Task**, select the rds\_sync node, and choose **Patch Data** \> **Current node**, as shown in the following figure.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/24452/155928446620998_en-US.png)

3.  On the displayed Patch Data page, select the business date, and click **OK**, as shown in the following figure.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/24452/155928446620999_en-US.png)

4.  Multiple synchronization task instances are generated at the same time and executed in sequence, as shown in the following figure.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/24452/155928446621000_en-US.png)

5.  View the running log. You can see the process of extracting data from RDS, as shown in the following figure.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/24452/155928446621001_en-US.png)

    A partition has been automatically created in MaxCompute.

6.  View the results.

    You can check whether the data is written successfully, whether a partition is created, and whether the data is synchronized to the partition table, as shown in the following figure.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/24452/155928446621002_en-US.png)

    Query the partition information, as shown in the following figure.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/24452/155928446733549_en-US.png)

    **Note:** In MaxCompute 2.0, parameter settings are required for partition table query. The partition column needs to be updated to the business date. If the task execution date is 20180717, the business date is 20180716. The SQL statements are as follows:

    ```
    --Check whether the data is successfully written to MaxCompute.
    select count(*) from ods_user_info_d where dt=business date;
    							
    ```


## Create a partition by non-date field using hash {#section_rr4_tgt_pfb .section}

If a huge volume of data needs to be processed, or if a full amount of data is partitioned according to a non-date field \(such as province\) at the first time, a data partition cannot be created automatically during data integration. Therefore, you can use the hash algorithm to save the same values in an RDS field to the corresponding partition in MaxCompute.

The procedure is as follows:

1.  Synchronize the full amount of data to a temporary table in MaxCompute. Create an SQL script node and choose **Run** \> **Save** \> **Submit**.

    The SQL statements are as follows:

    ```
    drop table if exists ods_user_t;
    CREATE TABLE ods_user_t ( 
            dt STRING,
        uid STRING,
            gender STRING,
            age_range STRING,
            zodiac STRING);
    insert overwrite table ods_user_t select dt,uid,gender,age_range,zodiac from ods_user_info_d;--Save the data in the ODPS table to the temporary table.
    						
    ```

2.  Create a synchronization task node named mysql\_to\_odps to synchronize the full amount of RDS data to MaxCompute without setting the partition, as shown in the following figure.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/24452/155928446733591_en-US.png)

3.  Use SQL statements to dynamically create a partition for the destination table. The SQL statements are as follows:

    ```
    drop table if exists ods_user_d;
    //Create a partition table (the destination table) in ODPS.
        CREATE TABLE ods_user_d (
        uid STRING,
            gender STRING,
            age_range STRING,
            zodiac STRING
    )
    PARTITIONED BY (
        dt STRING
    );
    //Execute the dynamic partitioning SQL statements to automatically create a partition according to the dt field in the temporary table. The partition value is automatically created for a data record. Data records that share the same dt value have the same partition field value.
    //For example, some data records share the value 20180913 in the dt field. As a result, a partition is automatically created in the MaxCompute partition table with a partition value of 20181025.
    //The dynamic partitioning SQL statements are as follows:
    //A date_time field is added in the select field, indicating that a partition is automatically created according to this field.
    insert overwrite table ods_user_d partition(dt)select dt,uid,gender,age_range,zodiac from ods_user_t;
    //After the import is completed, you can delete the temporary table to reduce excessive storage costs.
    drop table if exists ods_user_t;
    ```

    In MaxCompute, you can synchronize data using SQL statements. For more information about the SQL statements. see [How to use partition tables in Alibaba Cloud MaxCompute.](https://yq.aliyun.com/articles/81775?spm=a2c4e.11153940.blogcont182972.20.72856a07pHyeLb)

4.  Configure the three nodes to form a workflow and execute these nodes in sequence, as shown in the following figure.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/24452/155928446721007_en-US.png)

5.  View the execution process. The last node represents the process of dynamic partitioning, as shown in the following figure.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/24452/155928446721008_en-US.png)

    View data. Dynamic partitioning is completed automatically. Data records with the same date are synchronized to the same partition, as shown in the following figure.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/24452/155928446733594_en-US.png)

    Query the partition information, as shown in the following figure.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/24452/155928446721013_en-US.png)

    You can follow the preceding steps to name a partition using the province field.


The data synchronization feature of DataWorks supports automatic data operations, including data synchronization, data migration, and data synchronization task scheduling. For more information about scheduling configuration, see [Time attributes](https://www.alibabacloud.com/help/zh/doc-detail/74452.htm) in Scheduling Configuration.

