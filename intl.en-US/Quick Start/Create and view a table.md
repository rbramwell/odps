# Create and view a table {#concept_rkk_kcy_5db .concept}

This topic describes how to use MaxCompute to analyze data concerning house buyers with bank loans. After you purchase a MaxCompute project with your Alibaba Cloud account, you are added to the project and authorized to create a table. Afterwards, you can complete client configurations on your local PC and operate MaxCompute.

**Note:** **If you are using MaxCompute for the first time, be sure to complete all [preparations](../../../../reseller.en-US/Prepare/Create an Alibaba Cloud account.md#) before you get started.** 

The MaxCompute quick start guide describes how to use the [MaxCompute client](../../../../reseller.en-US/Prepare/Install and configure a client.md#) and [MaxCompute Studio](../../../../reseller.en-US/Tools and Downloads/MaxCompute Studio/Tools Installation and version history/Install IntelliJ IDEA.md#) to create tables and how to upload, process, and export data. Alternatively, you can use DataWorks to perform these operations. For more information, see [DataWorks quick start](../../../../reseller.en-US/Quick Start/Instructions.md#).

In MaxCompute, the basic operations \(input and output\) objects are tables, so you need to create tables and partitions before processing data. You can use any of the following methods to create, view, and delete tables:

-   Run [common commands](../../../../reseller.en-US/User Guide/Common commands/Table operations.md#) on the MaxCompute client.
-   Follow the instructions provided in [Visualization of operating the tables](../../../../reseller.en-US/Tools and Downloads/MaxCompute Studio/Manage data and resources/Visualization of operating the tables.md) to use MaxCompute Studio.
-   Follow the instructions provided in [Create a table](../../../../reseller.en-US/User Guide/Data management/Create a table.md#) and [Delete a table](../../../../reseller.en-US/User Guide/Data management/Table management.md#) to use DataWorks.

The following describes how to use the **[MaxCompute client](../../../../reseller.en-US/Prepare/Install and configure a client.md#)** to create and view a table.

## Create a table {#section_lz3_tcy_5db .section}

Log on to the MaxCompute client, and then create a table by running the following command:

``` {#codeblock_3pa_06a_lqf}
CREATE TABLE [IF NOT EXISTS] table_name 
                    [(col_name data_type [COMMENT col_comment], ...)] 
                    [COMMENT table_comment] 
                    [PARTITIONED BY (col_name data_type [COMMENT col_comment], ...)] 
                    [LIFECYCLE days] 
                    [AS select_statement]
                    CREATE TABLE [IF NOT EXISTS] table_name 
                    LIKE existing_table_name
```

**Note:** For more information, see [Table operations](../../../../reseller.en-US/User Guide/SQL/DDL SQL/Table operations.md#).

After logging on to the MaxCompute client, you need to confirm that you are in the correct project. In this example, the project name is MaxCompute\_DOC, so you can run `use MaxCompute_DOC;` to switch to the project.

In this example, the **bank\_data** table is used to store service data, and the **result\_table** table is used to store data processing results.

Create the **bank\_data** table by running the following command:

``` {#codeblock_4sp_jmu_rvo .language-sql}
CREATE TABLE IF NOT EXISTS bank_data
                (
                age             BIGINT COMMENT 'age',
                job             STRING COMMENT 'job type',
                marital         STRING COMMENT 'marital status',
                education       STRING COMMENT 'educational level',
                default         STRING COMMENT 'credit card ownship',
                housing         STRING COMMENT 'mortgate',
                loan            STRING COMMENT 'loan',
                contact         STRING COMMENT 'contact information',
                month           STRING COMMENT 'month',
                day_of_week     STRING COMMENT 'day of the week',
                duration        STRING COMMENT 'Duration',
                campaign        BIGINT COMMENT 'contact times during the campaign',
                pdays           DOUBLE COMMENT 'time interval from the last contact',
                previous        DOUBLE COMMENT 'previous contact times before this activity',
                poutcome        STRING COMMENT 'market activity results before this activity',
                emp_var_rate    DOUBLE COMMENT 'employment change rate',
                cons_price_idx  DOUBLE COMMENT 'consumer price index',
                cons_conf_idx   DOUBLE COMMENT 'consumer confidence index',
                euribor3m       DOUBLE COMMENT 'euro deposit rate',
                nr_employed     DOUBLE COMMENT 'number of employees',
                y               BIGINT COMMENT 'whether a fixed deposit holder'
                );
```

The command output **OK** is displayed, as shown in the following figure.

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/11950/156592737736984_en-US.png)

Create the **result\_table** table by running the following command:

``` {#codeblock_kle_9n8_l3q}
CREATE TABLE IF NOT EXISTS result_table
                (  
                education   STRING COMMENT 'education level',
                num         BIGINT COMMENT 'number of people'
                );
```

## View a table {#section_ln3_j2y_5db .section}

Run `desc <table_name>;` to view information related to a table.

For example, you can run `desc bank_data;` to view information related to the **bank\_data** table that was created in the previous step.

The following figure shows the command output.

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/11950/156592737736985_en-US.png)

For more information, see [Table operations](../../../../reseller.en-US/User Guide/SQL/DDL SQL/Table operations.md).

## \(Optional\) Create a partition {#section_gn3_j2y_5db .section}

The table used in this example is a non-partition table.

If you want to [run the Tunnel command to import data of different partitions](reseller.en-US/Quick Start/Import data.md#) to a [partition table](../../../../reseller.en-US/Product Introduction/Definitions/Partition.md#), you must create a partition by running the following commands:

``` {#codeblock_ckp_jfb_l58}
alter table table_name add [if not exists] partition partition(partition_col1 = partition_col_value1, partition_col2 = partiton_col_value2, ...);
```

You do not need to create an independent partition for different operations, such as [data integration](../../../../reseller.en-US/User Guide/Data integration/Data integration introduction/Data integration overview.md#) and insert operations.

## \(Optional\) Delete a partition {#section_pp3_5fy_5db .section}

Delete a partition by running the following command:

``` {#codeblock_yxc_4ji_u7a}
alter table table_name drop [if exists] partition(partition_col1 = partition_col_value1, partition_col2 = partiton_col_value2, ...);
```

For example, if you want to delete a partition with the date of `20180923` and region of `hangzhou`, run the following command:

``` {#codeblock_f4q_1ib_udy}
alter table user drop if exists partition(region='hangzhou',dt='20180923');
```

## \(Optional\) Delete a table {#section_up3_5fy_5db .section}

Create a table by running the following command:

``` {#codeblock_a6h_t8t_94i}
DROP TABLE [IF EXISTS] table_name;
```

For more information, see [Table operations](../../../../reseller.en-US/User Guide/SQL/DDL SQL/Table operations.md#).

## What to do next {#section_lcv_yh4_jgb .section}

You can refer to [Import data](reseller.en-US/Quick Start/Import data.md#) for data processing.

