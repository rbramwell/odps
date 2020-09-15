---
keyword: [GCP, BigQuery, MaxCompute]
---

# Migrate data from BigQuery to MaxCompute

This topic describes how to migrate data from BigQuery on Google Cloud Platform \(GCP\) to Alibaba Cloud MaxCompute over the Internet.

|Category|Platform|Requirement|Reference|
|--------|--------|-----------|---------|
|Environment and data|Google Cloud Platform|-   The Google BigQuery service is activated, and the environment and datasets for migration are prepared.
-   The Google Cloud Storage service is activated and a bucket is created.

|If you do not have the relevant environment and datasets, see the following references for preparation:-   BigQuery: [Quickstarts](https://cloud.google.com/bigquery/docs/quickstarts) and [Creating datasets](https://cloud.google.com/bigquery/docs/datasets)
-   Google Cloud Storage: [Quickstart: Using the Console](https://cloud.google.com/storage/docs/quickstart-console) and [Creating storage buckets](https://cloud.google.com/storage/docs/creating-buckets). |
|Alibaba Cloud|-   The MaxCompute and DataWorks services are activated and a project is created.

In this example, a MaxCompute project in the **Indonesia \(Jakarta\)** region is created as the migration destination.

-   Object Storage Service \(OSS\) is activated and a bucket is created.
-   The Data Online Migration service of OSS is activated.

|If you do not have the relevant environment, see the following references for preparation:-   MaxCompute and DataWorks: [Prepare](/intl.en-US/Prepare/Create an Alibaba Cloud account.md) and [Create a project](/intl.en-US/Prepare/Create a project.md).
-   OSS: [Activate OSS](/intl.en-US/Quick Start/Sign up for OSS.md) and [Create buckets](/intl.en-US/Quick Start/Create buckets.md).
-   Data Online Migration: [Submit a ticket](https://workorder-intl.console.aliyun.com/) or [Apply for the service online](https://page.aliyun.com/form/act998591440/index.htm?spm=5176.a2c3g.0.0.46583d89fy9Fpo). |
|Account|Google Cloud Platform|An Identity and Access Management \(IAM\) user is created and granted the permissions to access Google Cloud Storage.|[IAM permissions for JSON methods](https://cloud.google.com/storage/docs/access-control/iam-json)|
|Alibaba Cloud|A Resource Access Management \(RAM\) user and a RAM role are created. The RAM user is granted the read and write permissions on OSS buckets and the online migration permissions.|[Create a RAM user](/intl.en-US/RAM User Management/Create a RAM user.md) and [STS authorization](/intl.en-US/Development/External table/STS authorization.md)|
|Region|Google Cloud Platform|N/A|N/A|
|Alibaba Cloud|The OSS bucket and the MaxCompute project are in the same region.|N/A|

The following figure shows the process to migrate datasets from BigQuery to Alibaba Cloud MaxCompute.

![Migration process](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/6191410061/p143579.png)

|No.|Description|
|---|-----------|
|①|Export datasets from BigQuery to Google Cloud Storage.|
|②|Migrate data from Google Cloud Storage to an OSS bucket by using the **Data Online Migration** service of OSS.|
|③|Migrate data from the OSS bucket to a MaxCompute project in the same region, and then verify the integrity and accuracy of the migrated data.|

## Step 1: Export datasets from BigQuery to Google Cloud Storage

Use the bq command-line tool to run the `bq extract` command to export datasets from BigQuery to Google Cloud Storage.

1.  Log on to [Google Cloud Platform](https://console.cloud.google.com/storage), and create a bucket for the data you want to migrate. For more information, see [Creating storage buckets](https://cloud.google.com/storage/docs/creating-buckets).

    ![Storage bucket](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/1991559951/p149249.png)

2.  Use the bq command-line tool to query the Data Definition Language \(DDL\) scripts of tables in the TPC-DS datasets and download the scripts to an on-premises device. For more information, see [Getting table metadata using INFORMATION\_SCHEMA](https://cloud.google.com/bigquery/docs/information-schema-tables).

    For more information about the bq command-line tool, see [Using the bq command-line tool](https://cloud.google.com/bigquery/docs/bq-command-line-tool).

    BigQuery does not support commands such as `show create table` to query DDL scripts of tables. BigQuery allows you to use built-in user-defined functions \(UDFs\) to query the DDL scripts of the tables in a dataset. The following code shows examples of DDL scripts.

    ![DDL](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/1991559951/p149250.png)

3.  Use the bq command-line tool to run the `bq extract` command to export tables in BigQuery datasets to the destination bucket of Google Cloud Storage. For more information about the operations, formats of exported data, and compression types, see [Exporting table data](https://cloud.google.com/bigquery/docs/exporting-data).

    The following code shows a sample extract command:

    ```
    bq extract
    --destination_format AVRO 
    --compression SNAPPY
    tpcds_100gb.web_site
    gs://bucket_name/web_site/web_site-*.avro.snappy;
    ```

4.  View the bucket and check the data export result.


## Step 2: Migrate the exported data from Google Cloud Storage to OSS

You can use the **Data Online Migration** service to migrate data from Google Cloud Storage to OSS. For more information, see [Migrate data from Google Cloud Platform to OSS](). The **Data Online Migration** service is in public preview. Before you use the service, you must [submit a ticket](https://workorder-intl.console.aliyun.com/) to contact the Customer Service to activate the service.

1.  Estimate the size and the number of files that you want to migrate. You can query the data size in the bucket of Google Cloud Storage by using the gsutil tool or checking the storage logs. For more information, see [Getting bucket information](https://cloud.google.com/storage/docs/getting-bucket-information).

2.  If you do not have a bucket in OSS, log on to the [OSS console](https://oss.console.aliyun.com/) and create a bucket to store the migrated data. For more information, see [Create buckets](/intl.en-US/Quick Start/Create buckets.md).

    ![OSS bucket](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/2991559951/p149259.png)

3.  If you do not have a RAM user, create a RAM user and grant relevant permissions to the RAM user.

    1.  Log on to the [RAM console](https://ram.console.aliyun.com) and create a RAM user. For more information, see [Create a RAM user](/intl.en-US/RAM User Management/Create a RAM user.md).

    2.  Find the newly created RAM user, and click **Add Permissions** in the Actions column. On the page that appears, select **AliyunOSSFullAccess** and **AliyunMGWFullAccess**, and click **OK** \> **Complete**. The AliyunOSSFullAccess permission authorizes the RAM user to read and write OSS buckets. The AliyunMGWFullAccess permission authorizes the RAM user to perform online migration jobs.

    3.  In the left-side navigation pane, click **Overview**. In the **Account Management** section of the **Overview** page, click the link under **RAM user logon**, and use the credentials of the RAM user to log on to the [Alibaba Cloud Management Console](https://homenew.console.aliyun.com/).

4.  On Google Cloud Platform, create a user who uses the programmatic access method to access Google Cloud Storage. For more information, see [IAM permissions for JSON methods](https://cloud.google.com/storage/docs/access-control/iam-json).

    1.  Log on to the [IAM & Admin console](https://console.cloud.google.com/iam-admin/serviceaccounts), and find a user who has permissions to access BigQuery. In the **Actions** column, click **![Actions](../images/p143506.png)** \> **Create key**.

    2.  In the dialog box that appears, select **JSON**, and click **CREATE**. Save the JSON file to an on-premises device and click **CLOSE**.

    3.  In the **Create service account** wizard, click **Select a role**, and choose **Cloud Storage** \> **Storage Admin** to authorize the IAM user to access Google Cloud Storage.

5.  Create a source data address and a destination data address for online data migration.

    1.  Log on to the [Alibaba Cloud Data Transport console](https://mgw.console.aliyun.com/#/job). In the left-side navigation pane, click **Data Address**.

    2.  If you have not activated the Data Online Migration service, click **Application** in the dialog box that appears. On the **Online Migration Beta Test** page, specify the required information and click **Submit**.

        ![Online Migration Beta Test](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/2991559951/p146824.png)

        **Note:** On the **Online Migration Beta Test** page, if the **Source Storage Provider** options do not include Google Cloud Platform, select a source storage provider and specify the actual source storage provider in the **Notes** field.

    3.  On the **Data Address** page, click **Create Data Address**. In the Create Data Address dialog box, set the required parameters and click **OK**. For more information about the parameters, see [Create a migration job]().

        -   Source data address

            ![Source data address](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/2991559951/p143531.png)

            **Note:** For the **Key File** field, upload the JSON file that is downloaded in [Step 4](#step_9xl_wyx_oe6).

        -   Destination data address

            ![Destination data address](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/2991559951/p143538.png)

            **Note:** In the **Access Key Id** and **Access Key Secret** fields, enter the AccessKey ID and the AccessKey secret of the RAM user.

6.  Create an online migration job.

    1.  In the left-side navigation pane, click **Migration Jobs**.

    2.  On the **File Sync Management** page, click **Create Job**. In the Create Job wizard, set the required parameters and click **Create**. For more information about the parameters, see [Create a migration job]().

        -   **Job Config**

            ![Job configurations](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/2991559951/p143543.png)

        -   **Performance**

            ![Performance optimization](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/2991559951/p143545.png)

            **Note:** In the **Data Size** and **File Count** fields, enter the size and the number of files that were migrated from Google Cloud Platform.

    3.  The created migration job is automatically run. If **Finished** is displayed in the **Job Status** column, the migration job is complete.

    4.  In the Operation column of the migration job, click **Manage** to view the migration report and confirm that all data is migrated.

    5.  Log on to the [OSS console](https://oss.console.aliyun.com/).

    6.  In the left-side navigation pane, click **Buckets**. On the **Buckets** page, click the created bucket. In the left-side navigation pane of the bucket details page, choose Files \> **Files** to view the migration results.


## Step 3: Migrate data from the OSS bucket to a MaxCompute project in the same region and verify data integrity and accuracy

You can execute the LOAD statement of MaxCompute to migrate data from an OSS bucket to a MaxCompute project in the same region.

The LOAD statement supports Security Token Service \(STS\) and AccessKey for authentication. If you use AccessKey for authentication, you must provide the AccessKey ID and AccessKey secret of your account in plaintext. STS authentication is highly secure because it does not expose the AccessKey information. In this section, STS authentication is used as an example to show how to migrate data.

1.  On the Ad-Hoc Query tab of DataWorks or the MaxCompute client odpscmd, modify the DDL scripts of the tables in the BigQuery datasets, specify the MaxCompute data types, and create a destination table that stores the migrated data in MaxCompute.

    For more information about ad hoc queries, see [\(Optional\) Use an ad-hoc query to run SQL statements](/intl.en-US/Quick Start/(Optional) Use an ad-hoc query to run SQL statements.md). The following code shows a configuration example:

    ```
    CREATE OR REPLACE TABLE
    `****.tpcds_100gb.web_site`
    (
      web_site_sk INT64,
      web_site_id STRING,
      web_rec_start_date STRING,
      web_rec_end_date STRING,
      web_name STRING,
      web_open_date_sk INT64,
      web_close_date_sk INT64,
      web_class STRING,
      web_manager STRING,
      web_mkt_id INT64,
      web_mkt_class STRING,
      web_mkt_desc STRING,
      web_market_manager STRING,
      web_company_id INT64,
      web_company_name STRING,
      web_street_number STRING,
      web_street_name STRING,
      web_street_type STRING,
      web_suite_number STRING,
      web_city STRING,
      web_county STRING,
      web_state STRING,
      web_zip STRING,
      web_country STRING,
      web_gmt_offset FLOAT64,
      web_tax_percentage FLOAT64
    )
    
    Modify the INT64 and
    FLOAT64 fields to obtain the following DDL script:
    CREATE
    TABLE IF NOT EXISTS <your_maxcompute_project>.web_site_load
    (
    web_site_sk BIGINT,
    web_site_id STRING,
    web_rec_start_date STRING,
    web_rec_end_date STRING,
    web_name STRING,
    web_open_date_sk BIGINT,
    web_close_date_sk BIGINT,
    web_class STRING,
    web_manager STRING,
    web_mkt_id BIGINT,
    web_mkt_class STRING,
    web_mkt_desc STRING,
    web_market_manager STRING,
    web_company_id BIGINT,
    web_company_name STRING,
    web_street_number STRING,
    web_street_name STRING,`
    web_street_type STRING,
    web_suite_number STRING,
    web_city STRING,
    web_county STRING,
    web_state STRING,
    web_zip STRING,
    web_country STRING,
    web_gmt_offset DOUBLE,
    web_tax_percentage DOUBLE
    );
    ```

    The following table describes the mapping between BigQuery data types and MaxCompute data types.

    |BigQuery data type|MaxCompute data type|
    |------------------|--------------------|
    |INT64|BIGINT|
    |FLOAT64|DOUBLE|
    |NUMERIC|DECIMAL and DOUBLE|
    |BOOL|BOOLEAN|
    |STRING|STRING|
    |BYTES|VARCHAR|
    |DATE|DATE|
    |DATETIME|DATETIME|
    |TIME|DATETIME|
    |TIMESTAMP|TIMESTAMP|
    |STRUCT|STRUCT|
    |GEOGRAPHY|STRING|

2.  If you do not have a RAM role, create a RAM role that has the OSS access permissions and assign the role to the RAM user. For more information, see [STS authorization](/intl.en-US/Development/External table/STS authorization.md).

3.  Execute the LOAD statement to load all data from the OSS bucket to the MaxCompute table, and execute the SELECT statement to query and verify the imported data. You can only load one table at a time. To load multiple tables, you must execute the LOAD statement multiple times. For more information about the LOAD statement, see [LOAD](/intl.en-US/Development/SQL/LOAD.md).

    ```
    LOAD OVERWRITE TABLE web_site
    FROM  LOCATION 'oss://oss-<your_region_id>-internal.aliyuncs.com/bucket_name/tpc_ds_100gb/web_site/' --The endpoint of the OSS bucket
    ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.avro.AvroSerDe'
    WITH SERDEPROPERTIES ('odps.properties.rolearn'='<Your_RoleARN>','mcfed.parquet.compression'='SNAPPY')
    STORED AS AVRO;
    ```

    **Note:** If the data import fails, [submit a ticket](https://workorder-intl.console.aliyun.com/) to contact the MaxCompute team.

    Execute the following statement to query and verify the imported data:

    ```
    SELECT * FROM web_site;
    ```

    The statement returns the following output.

    ![Output](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/6084500061/p143575.png)

4.  Verify that the data migrated to MaxCompute is the same as the data in BigQuery. This verification is based on the number of tables, the number of rows, and the query results of typical jobs.


