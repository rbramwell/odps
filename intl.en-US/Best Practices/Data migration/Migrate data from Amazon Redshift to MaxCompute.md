---
keyword: [Amazon Redshift, MaxCompute]
---

# Migrate data from Amazon Redshift to MaxCompute

This topic describes how to migrate data from Amazon Redshift to MaxCompute over the Internet.

-   Prepare an Amazon Redshift cluster and the data for migration.

    For more information about how to create an Amazon Redshift cluster, see [Amazon Redshift Cluster Management Guide](https://docs.aws.amazon.com/redshift/latest/mgmt/welcome.html).

    1.  Create an Amazon Redshift cluster. If you already have an Amazon Redshift cluster, skip this step.

        ![Amazon Redshift cluster](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/3559559951/p139376.png)

    2.  Prepare data that you want to migrate in the Amazon Redshift cluster.

        In this example, a TPC-H dataset is available in public schema. The dataset uses MaxCompute V2.0 data types such as DECIMAL.

-   Prepare a MaxCompute project.

    For more information, see [Prepare](/intl.en-US/Prepare/Create a project.md).

    In this example, a MaxCompute project is created as the migration destination in the **Singapore \(Singapore\)** region. The project is created in MaxCompute V2.0 because the TPC-H dataset uses the MaxCompute V2.0 data types and the Decimal V2.0 data type.

    ![Create a project](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/4559559951/p139385.png)

-   Activate Alibaba Cloud Object Storage Service \(OSS\).

    For more information, see [Activate OSS](/intl.en-US/Quick Start/Sign up for OSS.md).


The following figure shows the process to migrate data from Amazon Redshift to MaxCompute.

![Migration process](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/7931410061/p139418.png)

|No.|Description|
|---|-----------|
|①|Unload data from Amazon Redshift to a data lake on Amazon Simple Storage Service \(S3\).|
|②|Migrate data from Amazon S3 to an OSS bucket by using the **Data Online Migration** service of OSS.|
|③|Migrate data from the OSS bucket to a MaxCompute project in the same region, and then verify the integrity and accuracy of the migrated data.|

## Step 1: Unload data from Amazon Redshift to Amazon S3

Amazon Redshift supports the authentication methods based on Identity and Access Management \(IAM\) roles and temporary security credentials \(AccessKey pairs\). You can use the UNLOAD command of Amazon Redshift to unload data to Amazon S3 based on these two authentication methods. For more information, see [Unloading data](https://docs.aws.amazon.com/redshift/latest/dg/c_unloading_data.html).

The syntax of the UNLOAD command varies with the authentication method.

-   UNLOAD command based on an IAM role

    ```
    -- Run the UNLOAD command to unload data in the customer table to Amazon S3.
    UNLOAD ('SELECT * FROM customer')
    TO 's3://bucket_name/unload_from_redshift/customer/customer_' --The Amazon S3 bucket.
    IAM_ROLE 'arn:aws:iam::****:role/MyRedshiftRole'; --The ARN of the IAM role.
    ```

-   UNLOAD command based on an AccessKey pair

    ```
    -- Run the UNLOAD command to unload data in the customer table to Amazon S3.
    UNLOAD ('SELECT * FROM customer')
    TO 's3://bucket_name/unload_from_redshift/customer/customer_' --The Amazon S3 bucket.
    Access_Key_id '<access-key-id>' -- The AccessKey ID of the IAM user.
    Secret_Access_Key '<secret-access-key>' -- The AccessKey secret of the IAM user.
    Session_Token '<temporary-token>'; -- The temporary access token of the IAM user.
    ```


The UNLOAD command allows you to unload data in one of the following formats:

-   Default format

    The following sample command shows how to unload data in the default format:

    ```
    UNLOAD ('SELECT * FROM customer')
    TO 's3://bucket_name/unload_from_redshift/customer/customer_'
    IAM_ROLE 'arn:aws:iam::****:role/redshift_s3_role';
    ```

    After the command is run, data is unloaded to text files in which values are separated with vertical bars \(\|\). You can log on to the [Amazon S3 console](https://s3.console.aws.amazon.com/s3/home) and view the unloaded text files in the specified bucket.

    ![Destination location](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/4559559951/p139554.png)

    The following figure shows the unloaded text files in the default format.

    ![Unloaded files in the default format](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/0034500061/p139557.png)

-   Apache Parquet format

    Data unloaded in the Apache Parquet format can be directly read by other engines. The following sample command shows how to unload data in Apache Parquet format:

    ```
    UNLOAD ('SELECT * FROM customer')
    TO 's3://bucket_name/unload_from_redshift/customer_parquet/customer_'
    FORMAT AS PARQUET
    IAM_ROLE 'arn:aws:iam::xxxx:role/redshift_s3_role';
    ```

    After the command is run, you can view the unloaded Parquet files in the specified bucket. Parquet files are smaller than text files and have a higher data compression ratio.

    ![Destination location](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/4559559951/p139570.png)


This section describes how to authenticate requests based on IAM roles and unload data in Apache Parquet format.

1.  Create an IAM role for Amazon Redshift.

    1.  Log on to the [IAM console](https://console.aws.amazon.com/iam/home?#/roles). In the left-side navigation pane, choose Access Management \> Roles. On the Roles page, click **Create role**.

        ![Create an IAM Role](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/4559559951/p139611.png)

    2.  In the **Common use cases** section of the **Create role** page, click **Redshift**. In the **Select your use case** section, click **Redshift-Customizable**, and then click **Next: Permissions**.

        ![Select a case](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/4559559951/p139628.png)

2.  Add an IAM policy that grants the read and write permissions on Amazon S3. In the **Attach permissions policies** section of the **Create role** page, enter **S3**, select **AmazonS3FullAccess**, and then click **Next: Tags**.

    ![Select a policy](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/4559559951/p139686.png)

3.  Assign a name to the IAM role and complete the IAM role creation.

    1.  Click **Next: Review**. In the **Review** section of the **Create role** page, specify **Role name** and **Role description**, and click **Create role**. The IAM role is then created.

        ![Review the role information](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/4559559951/p139687.png)

    2.  Go to the [IAM console](https://console.aws.amazon.com/iam/home?#/roles), and enter **redshift\_s3\_role** in the search box to search for the role. Then, click the role name **redshift\_s3\_role**, and copy the value of **Role ARN**.

        When you run the UNLOAD command to unload data, you must provide the **Role ARN** value to access Amazon S3.

        ![Obtain the ARN of the IAM role](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/4559559951/p139689.png)

4.  Associate the created IAM role with the Amazon Redshift cluster to authorize the cluster to access Amazon S3.

    1.  Log on to the [Amazon Redshift console](https://console.aws.amazon.com/redshiftv2/home). In the upper-right corner, select **Asia Pacific \(Singapore\)** from the drop-down region list.

    2.  In the left-side navigation pane, click **CLUSTERS**, find the created Amazon Redshift cluster, click **Actions**, and then click **Manage IAM roles**.

    3.  On the **Manage IAM roles** page, click the ![Drop-down icon](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/0034500061/p139692.png) icon next to the search box, and select **redshift\_s3\_role**. Click **Add IAM role** \> **Done** to associate the **redshift\_s3\_role** role with the Amazon Redshift cluster.

5.  Unload data from Amazon Redshift to Amazon S3.

    1.  Go to the [Amazon Redshift console](https://console.aws.amazon.com/redshiftv2/home).

    2.  In the left-side navigation pane, click **EDITOR**. Run the UNLOAD command to upload data from Amazon Redshift to each destination bucket on Amazon S3 in the Apache Parquet format.

        The following code shows an example of unloading data from Amazon Redshift to Amazon S3:

        ```
        UNLOAD ('SELECT * FROM customer')   
        TO 's3://bucket_name/unload_from_redshift/customer_parquet/customer_' 
        FORMAT AS PARQUET 
        IAM_ROLE 'arn:aws:iam::xxxx:role/redshift_s3_role';
        UNLOAD ('SELECT * FROM orders')   
        TO 's3://bucket_name/unload_from_redshift/orders_parquet/orders_' 
        FORMAT AS PARQUET 
        IAM_ROLE 'arn:aws:iam::xxxx:role/redshift_s3_role';
        UNLOAD ('SELECT * FROM lineitem')   
        TO 's3://bucket_name/unload_from_redshift/lineitem_parquet/lineitem_' 
        FORMAT AS PARQUET 
        IAM_ROLE 'arn:aws:iam::xxxx:role/redshift_s3_role';
        UNLOAD ('SELECT * FROM nation')   
        TO 's3://bucket_name/unload_from_redshift/nation_parquet/nation_' 
        FORMAT AS PARQUET 
        IAM_ROLE 'arn:aws:iam::xxxx:role/redshift_s3_role';
        UNLOAD ('SELECT * FROM part')   
        TO 's3://bucket_name/unload_from_redshift/part_parquet/part_' 
        FORMAT AS PARQUET 
        IAM_ROLE 'arn:aws:iam::xxxx:role/redshift_s3_role';
        UNLOAD ('SELECT * FROM partsupp')   
        TO 's3://bucket_name/unload_from_redshift/partsupp_parquet/partsupp_' 
        FORMAT AS PARQUET 
        IAM_ROLE 'arn:aws:iam::xxxx:role/redshift_s3_role';
        UNLOAD ('SELECT * FROM region')   
        TO 's3://bucket_name/unload_from_redshift/region_parquet/region_' 
        FORMAT AS PARQUET 
        IAM_ROLE 'arn:aws:iam::xxxx:role/redshift_s3_role';
        UNLOAD ('SELECT * FROM supplier')   
        TO 's3://bucket_name/unload_from_redshift/supplier_parquet/supplier_' 
        FORMAT AS PARQUET 
        IAM_ROLE 'arn:aws:iam::xxxx:role/redshift_s3_role';
        ```

        **Note:** You can submit multiple UNLOAD commands at a time in **EDITOR**.

    3.  Log on to the [Amazon S3 console](https://s3.console.aws.amazon.com/s3/home) and check the unloaded data in the directory of each destination bucket on Amazon S3.

        The unloaded data is available in the Apache Parquet format.

        ![Verify the unloaded data](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/5559559951/p139753.png)


## Step 2: Migrate the unloaded data from Amazon S3 to OSS

In MaxCompute, you can use the **Data Online Migration** service of OSS to migrate data from Amazon S3 to OSS. For more information, see [Migrate data from Amazon Simple Storage Service \(Amazon S3\) to OSS](). The **Data Online Migration** service is in public preview. Before you use the service, you must [submit a ticket](https://selfservice.console.aliyun.com/ticket/createIndex) to contact the Customer Service to activate the service.

1.  Log on to the [OSS console](https://oss.console.aliyun.com/), and create a bucket to save the migrated data. For more information, see [Create buckets](/intl.en-US/Quick Start/Create buckets.md).

    ![Bucket](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/5559559951/p139787.png)

2.  Create a Resource Access Management \(RAM\) user and grant relevant permissions to the user.

    1.  Log on to the [RAM console](https://ram.console.aliyun.com) and create a RAM user. For more information, see [Create a RAM user](/intl.en-US/RAM User Management/Create a RAM user.md).

    2.  Find the newly created RAM user, and click **Add Permissions** in the Actions column. On the page that appears, select **AliyunOSSFullAccess** and **AliyunMGWFullAccess**, and click **OK** \> **Complete**. The AliyunOSSFullAccess permission authorizes the RAM user to read and write OSS buckets. The AliyunMGWFullAccess permission authorizes the RAM user to perform online migration jobs.

    3.  In the left-side navigation pane, click **Overview**. In the **Account Management** section of the **Overview** page, click the link under **RAM user logon**, and use the credentials of the RAM user to log on to the Alibaba Cloud Management Console.

3.  On the Amazon Web Services \(AWS\) platform, create an IAM user who uses the programmatic access method to access Amazon S3.

    1.  Log on to the [Amazon S3 console](https://s3.console.aws.amazon.com/s3/home).

    2.  Right-click the exported folder and select **Get total size**. You can then obtain the total size of the folder and the number of files in the folder.

        -   Obtain the total size.

            ![Obtain the total size](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/5559559951/p139988.png)

        -   Obtain the total size of the folder and the number of files in the folder.

            ![The total size of the folder and the number of files in the folder](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/5559559951/p139994.png)

    3.  Log on to the [IAM console](https://console.aws.amazon.com/iam/home?#/users) and click **Add user**.

        ![IAM console](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/5559559951/p139842.png)

    4.  On the **Add user** page, specify the **User name**. In the **Select AWS access type** section, select **Programmatic access** and then click **Next: Permissions**.

        ![Specify user information](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/5559559951/p139852.png)

    5.  On the **Add user** page, click **Attach existing policies directly**. Enter **S3** in the search box, select the **AmazonS3ReadOnlyAccess** policy, and then click **Next: Tags**.

        ![Configure user policies](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/5559559951/p139861.png)

    6.  Click **Next: Review** \> **Create user**. The IAM user is created. Obtain the AccessKey information.

        When you create an online migration job, you must provide this AccessKey information.

        ![Complete the user creation](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/5559559951/p139865.png)

4.  Create a source data address and a destination data address for online data migration.

    1.  Log on to the [Alibaba Cloud Data Transport console](https://mgw.console.aliyun.com/#/job). In the left-side navigation pane, click **Data Address**.

    2.  If you have not activated Data Online Migration, click **Application** in the dialog box that appears. On the **Online Migration Beta Test** page, specify the required information and click **Submit**.

        ![Online Migration Beta Test](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/5559559951/p139884.png)

    3.  On the **Data Address** page, click **Create Data Address**. In the Create Data Address dialog box, set the required parameters and click **OK**. For more information, see [Create a migration job]().

        -   Source data address

            ![Create a source data address](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/5559559951/p139902.png)

            **Note:** In the **Access Key Id** and **Access Key Secret** fields, enter the AccessKey ID and AccessKey secret of the IAM user.

        -   Destination data address

            ![Create a destination data address](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/5559559951/p139909.png)

            **Note:** In the **Access Key Id** and **Access Key Secret** fields, enter the AccessKey ID and the AccessKey secret of the RAM user.

5.  Create an online migration job.

    1.  In the left-side navigation pane, click **Migration Jobs**.

    2.  On the **File Sync Management** page, click **Create Job**. In the Create Job wizard, set the required parameters and click **Create**. For more information about the parameters, see [Create a migration job]().

        -   **Job Config**

            ![Job configurations](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/5559559951/p139935.png)

        -   **Performance**

            ![Performance](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/5559559951/p139937.png)

            **Note:** In the **Data Size** and **File Count** fields, enter the size and the number of files that were migrated from Amazon S3.

    3.  The created migration job is automatically run. If **Finished** is displayed in the **Job Status** column, the migration job is complete.

        ![Migration job completed](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/6559559951/p139991.png)

    4.  In the Operation column of the migration job, click **Manage** to view the migration report and confirm that all data is migrated.

        ![Migration report](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/6559559951/p139998.png)

    5.  Log on to the [OSS console](https://oss.console.aliyun.com/).

    6.  In the left-side navigation pane, click **Buckets**. On the **Buckets** page, click the created bucket. In the left-side navigation pane of the bucket, details page, choose Files \> **Files** to view the migration results.

        ![OSS migration result](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/6559559951/p140005.png)


## Step 3: Migrate data from the OSS bucket to a MaxCompute project in the same region and verify data integrity and accuracy

You can execute the LOAD statement of MaxCompute to migrate data from an OSS bucket to a MaxCompute project in the same region.

The LOAD statement supports Security Token Service \(STS\) and AccessKey for authentication. If you use AccessKey for authentication, you must provide the AccessKey ID and AccessKey secret of your account in plaintext. STS authentication is highly secure because it does not expose the AccessKey information. In this section, STS authentication is used as an example to show how to migrate data.

1.  On the Ad-Hoc Query tab of DataWorks or the MaxCompute client odpscmd, perform Data Definition Language \(DDL\) operations on the data in the Amazon Redshift cluster, and create tables that store the migrated data in MaxCompute.

    For more information about ad hoc queries, see [\(Optional\) Use an ad-hoc query to run SQL statements](/intl.en-US/Quick Start/(Optional) Use an ad-hoc query to run SQL statements.md). The following code shows a configuration example:

    ```
    CREATE TABLE customer(
    C_CustKey int ,
    C_Name varchar(64) ,
    C_Address varchar(64) ,
    C_NationKey int ,
    C_Phone varchar(64) ,
    C_AcctBal decimal(13, 2) ,
    C_MktSegment varchar(64) ,
    C_Comment varchar(120) ,
    skip varchar(64)
    );
    CREATE TABLE lineitem(
    L_OrderKey int ,
    L_PartKey int ,
    L_SuppKey int ,
    L_LineNumber int ,
    L_Quantity int ,
    L_ExtendedPrice decimal(13, 2) ,
    L_Discount decimal(13, 2) ,
    L_Tax decimal(13, 2) ,
    L_ReturnFlag varchar(64) ,
    L_LineStatus varchar(64) ,
    L_ShipDate timestamp ,
    L_CommitDate timestamp ,
    L_ReceiptDate timestamp ,
    L_ShipInstruct varchar(64) ,
    L_ShipMode varchar(64) ,
    L_Comment varchar(64) ,
    skip varchar(64)
    );
    CREATE TABLE nation(
    N_NationKey int ,
    N_Name varchar(64) ,
    N_RegionKey int ,
    N_Comment varchar(160) ,
    skip varchar(64)
    );
    CREATE TABLE orders(
    O_OrderKey int ,
    O_CustKey int ,
    O_OrderStatus varchar(64) ,
    O_TotalPrice decimal(13, 2) ,
    O_OrderDate timestamp ,
    O_OrderPriority varchar(15) ,
    O_Clerk varchar(64) ,
    O_ShipPriority int ,
    O_Comment varchar(80) ,
    skip varchar(64)
    );
    CREATE TABLE part(
    P_PartKey int ,
    P_Name varchar(64) ,
    P_Mfgr varchar(64) ,
    P_Brand varchar(64) ,
    P_Type varchar(64) ,
    P_Size int ,
    P_Container varchar(64) ,
    P_RetailPrice decimal(13, 2) ,
    P_Comment varchar(64) ,
    skip varchar(64)
    );
    CREATE TABLE partsupp(
    PS_PartKey int ,
    PS_SuppKey int ,
    PS_AvailQty int ,
    PS_SupplyCost decimal(13, 2) ,
    PS_Comment varchar(200) ,
    skip varchar(64)
    );
    CREATE TABLE region(
    R_RegionKey int ,
    R_Name varchar(64) ,
    R_Comment varchar(160) ,
    skip varchar(64)
    );
    CREATE TABLE supplier(
    S_SuppKey int ,
    S_Name varchar(64) ,
    S_Address varchar(64) ,
    S_NationKey int ,
    S_Phone varchar(18) ,
    S_AcctBal decimal(13, 2) ,
    S_Comment varchar(105) ,
    skip varchar(64)
    );
    ```

    In this example, the project is created in MaxCompute V2.0 because the TPC-H dataset uses the MaxCompute 2.0 data types and the Decimal 2.0 data type. If you want to configure the project to use the MaxCompute 2.0 data types and the Decimal 2.0 data type, add the following statements at the beginning of the CREATE TABLE statements:

    ```
    setproject odps.sql.type.system.odps2=true;
    setproject odps.sql.decimal.odps2=true;
    ```

2.  Create a RAM role that has the OSS access permissions and assign the RAM role to the RAM user. For more information, see [STS authorization](/intl.en-US/Development/External table/STS authorization.md).

3.  Execute the LOAD statement multiple times to load all data from OSS to the MaxCompute tables that you created, and execute the SELECT statement to query and verify the imported data. For more information about the LOAD statement, see [LOAD](/intl.en-US/Development/SQL/LOAD.md).

    ```
    LOAD OVERWRITE TABLE orders 
    FROM  LOCATION 'oss://endpoint/oss_bucket_name/unload_from_redshift/orders_parquet/' --The endpoint of the OSS bucket
    ROW FORMAT SERDE 'org.apache.hadoop.hive.ql.io.parquet.serde.ParquetHiveSerDe'
    WITH SERDEPROPERTIES ('odps.properties.rolearn'='acs:ram::xxx:role/xxx_role')
    STORED AS PARQUET;
    ```

    **Note:** If the data import fails, [submit a ticket](https://workorder-intl.console.aliyun.com/) to contact the MaxCompute team.

    Execute the following statement to query and verify the imported data:

    ```
    SELECT * FROM orders limit 100;
    ```

    The statement returns the following output.

    ![Query the imported data](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/6559559951/p140826.png)

4.  Verify that the data migrated to MaxCompute is the same as the data in Amazon Redshift. This verification is based on the number of tables, the number of rows, and the query results of typical jobs.

    1.  Log on to the [Amazon Redshift console](https://console.aws.amazon.com/redshiftv2/home). In the upper-right corner, select **Asia Pacific \(Singapore\)** from the drop-down region list. In the left-side navigation pane, click **EDITOR**. Execute the following statements to query data:

        ```
        SELECT l_returnflag, l_linestatus, SUM(l_quantity) as sum_qty,
        SUM(l_extendedprice) AS sum_base_price, SUM(l_extendedprice*(1-l_discount)) AS sum_disc_price,
        SUM(l_extendedprice*(1-l_discount)*(1+l_tax)) AS sum_charge, AVG(l_quantity) AS avg_qty,
        AVG(l_extendedprice) AS avg_price, AVG(l_discount) AS avg_disc,  COUNT(*) AS count_order
        FROM lineitem
        GROUP BY l_returnflag, l_linestatus
        ORDER BY l_returnflag,l_linestatus;
        ```

        The statement returns the following output.

        ![AWS query result](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/7559559951/p140170.png)

    2.  On the Ad-Hoc Query tab of DataWorks or the MaxCompute client odpscmd, execute the preceding statement and check whether the returned result is consistent with the data that is queried from the Amazon Redshift cluster.

        The statement returns the following output.

        ![Ad hoc query result](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/7559559951/p140803.png)


