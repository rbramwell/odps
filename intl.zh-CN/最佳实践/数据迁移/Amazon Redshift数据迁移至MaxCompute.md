---
keyword: [Amazon Redshift, MaxCompute]
---

# Amazon Redshift数据迁移至MaxCompute

本文为您介绍如何通过公网环境将Amazon Redshift数据迁移至MaxCompute。

-   准备Amazon Redshift集群环境及数据环境。

    您可以登录AWS官网，获取创建Redshift集群的详细操作内容，详情请参见[Amazon Redshift集群管理指南](https://docs.aws.amazon.com/zh_cn/redshift/latest/mgmt/welcome.html)。

    1.  创建Redshift集群。如果已有Redshift集群，您可以直接使用已有的Redshift集群。

        ![Redshift集群](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/5659559951/p139376.png)

    2.  在Redshift集群中准备好需要迁移的Amazon Redshift数据。

        假设，已在public schema中准备好了TPC-H数据集。数据集使用MaxCompute 2.0数据类型和Decimal 2.0数据类型。

-   准备MaxCompute的项目环境

    操作详情请参见[准备工作](/intl.zh-CN/准备工作/创建项目空间.md)。

    以**新加坡（新加坡）**区域为例，创建作为迁移目标的MaxCompute项目。由于TPC-H数据集使用MaxCompute 2.0数据类型和Decimal 2.0数据类型，因此本文创建的项目为MaxCompute 2.0版本。

    ![创建项目空间](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/5659559951/p139385.png)

-   开通阿里云OSS服务。

    开通阿里云OSS服务详情请参见[开通OSS服务](/intl.zh-CN/快速入门/开通OSS服务.md)。


将Amazon Redshift数据迁移至MaxCompute的流程如下。

![迁移流程](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/5659559951/p139418.png)

|序号|描述|
|--|--|
|①|将Amazon Redshift数据导出至Amazon S3数据湖（简称S3）。|
|②|通过对象存储服务OSS的**在线迁移上云服务**，将数据从S3迁移至OSS。|
|③|将数据从OSS迁移至同区域的MaxCompute项目中，并校验数据完整性和正确性。|

## 步骤一：将Amazon Redshift数据导出至S3

Amazon Redshift支持IAM角色和临时安全凭证（AccessKey）认证方式。您可以基于这两种认证方式通过Redshift UNLOAD命令将数据导出至S3。将Amazon Redshift数据导出至S3的详细操作内容请参见[卸载数据](https://docs.aws.amazon.com/zh_cn/redshift/latest/dg/c_unloading_data.html)。

两种认证方式的UNLOAD命令格式如下：

-   基于IAM角色的UNLOAD命令

    ```
    -- 通过UNLOAD命令将表customer的内容导出至S3。
    UNLOAD ('SELECT * FROM customer')
    TO 's3://bucket_name/unload_from_redshift/customer/customer_' --S3 Bucket。
    IAM_ROLE 'arn:aws:iam::****:role/MyRedshiftRole'; --角色ARN。
    ```

-   基于AccessKey的UNLOAD命令

    ```
    -- 通过UNLOAD命令将表customer的内容导出至S3。
    UNLOAD ('SELECT * FROM customer')
    TO 's3://bucket_name/unload_from_redshift/customer/customer_' --S3 Bucket。
    Access_Key_id '<access-key-id>'  --IAM用户的Access Key ID。
    Secret_Access_Key '<secret-access-key>'  --IAM用户的Access Key Secret。
    Session_Token '<temporary-token>';  --IAM用户的临时访问令牌。
    ```


UNLOAD命令导出的数据格式如下：

-   默认格式

    命令示例如下。

    ```
    UNLOAD ('SELECT * FROM customer')
    TO 's3://bucket_name/unload_from_redshift/customer/customer_'
    IAM_ROLE 'arn:aws:iam::****:role/redshift_s3_role';
    ```

    执行成功后，导出以竖线（\|）分隔的文本文件。您可以登录[S3控制台](https://s3.console.aws.amazon.com/s3/home)，在对应Bucket中查看导出的文本文件。

    ![导出位置](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/6659559951/p139554.png)

    导出的文本文件格式如下。

    ![默认导出格式](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/5834500061/p139557.png)

-   PARQUET格式

    以PARQUET格式导出，便于其它引擎直接读取数据。命令示例如下。

    ```
    UNLOAD ('SELECT * FROM customer')
    TO 's3://bucket_name/unload_from_redshift/customer_parquet/customer_'
    FORMAT AS PARQUET
    IAM_ROLE 'arn:aws:iam::xxxx:role/redshift_s3_role';
    ```

    执行成功后，您可以在对应Bucket中查看导出的文件。PARQUET文件比文本文件更小，数据压缩率更高。

    ![导出位置](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/6659559951/p139570.png)


本文以IAM角色认证及导出PARQUET格式为例介绍数据迁移操作。

1.  新建Redshift类型的IAM角色。

    1.  登录[IAM控制台](https://console.aws.amazon.com/iam/home?#/roles)，单击**创建角色**。

        ![创建IAM Role](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/6659559951/p139611.png)

    2.  在**创建角色**页面的**选择一个使用案例**区域，单击**Redshift**。在**选择您的使用案例**区域单击**Redshift-Customizable**后，单击**下一步：权限**。

        ![选择案例](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/6659559951/p139628.png)

2.  添加读写S3的权限策略。在**创建角色**页面的**Attach权限策略**区域，输入**S3**，选中**AmazonS3FullAccess**，单击**下一步：标签**。

    ![选择策略](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/6659559951/p139686.png)

3.  为IAM角色命名并完成IAM角色创建。

    1.  单击**下一步：审核**，在**创建角色**页面的**审核**区域，配置**角色名称**和**角色描述**，单击**创建角色**，完成IAM角色创建。

        ![角色审核](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/6659559951/p139687.png)

    2.  返回[IAM控制台](https://console.aws.amazon.com/iam/home?#/roles)，在搜索框输入**redshift\_s3\_role**，单击**redshift\_s3\_role**角色名称，获取并记录**角色ARN**。

        执行UNLOAD命令迁移数据时会使用**角色ARN**访问S3。

        ![获取角色RAN](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/6659559951/p139689.png)

4.  为Redshift集群添加创建好的IAM角色，获取访问S3的权限。

    1.  登录[Amazon Redshift控制台](https://console.aws.amazon.com/redshiftv2/home)，在右上角选择区域为**亚太地区（新加坡）**。

    2.  在左侧导航栏，单击**集群**，选中已创建好的Redshift集群，在**操作**下拉列表选择**管理IAM角色**。

    3.  在**管理IAM角色**页面，单击搜索框右侧的![下拉图标](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/6659559951/p139692.png)图标，选择**redshift\_s3\_role**。单击**添加IAM角色** \> **完成**，将具备S3访问权限的**redshift\_s3\_role**添加至Redshift集群。

5.  将Amazon Redshift数据导出至S3。

    1.  返回[Amazon Redshift控制台](https://console.aws.amazon.com/redshiftv2/home)。

    2.  在左侧导航栏，单击**编辑器**，执行UNLOAD命令将Amazon Redshift数据以PARQUET格式导出至S3的Bucket目录。

        命令示例如下。

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

        **说明：** **编辑器**支持一次提交多条UNLOAD命令。

    3.  登录[S3控制台](https://s3.console.aws.amazon.com/s3/home)，在S3的Bucket目录下检查导出的数据。

        格式为符合预期的PARQUET格式。

        ![检查导出数据](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/6659559951/p139753.png)


## 步骤二：将导出至S3的数据迁移至对象存储服务OSS

MaxCompute支持通过OSS的**在线迁移上云服务**，将S3的数据迁移至OSS，详情请参见[AWS S3迁移教程]()。**在线迁移上云服务**处于公测状态，您需要[提工单](https://selfservice.console.aliyun.com/ticket/createIndex)联系客服，并由在线服务团队开通后才可使用。

1.  登录[OSS管理控制台](https://oss.console.aliyun.com/)，创建保存迁移数据的Bucket，详情请参见[创建存储空间](/intl.zh-CN/快速入门/创建存储空间.md)。

    ![Bucket](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/6659559951/p139787.png)

2.  创建RAM用户并授予相关权限。

    1.  登录[RAM访问控制台](https://ram.console.aliyun.com)，创建RAM用户，详情请参见[创建RAM用户](/intl.zh-CN/用户管理/创建RAM用户.md)。

    2.  选中新创建的用户登录名称，单击**添加权限**，为新创建的RAM用户授予**AliyunOSSFullAccess**（存储空间读写权限）和**AliyunMGWFullAccess**（在线迁移权限），单击**确定** \> **完成**。

    3.  在左侧导航栏，单击**概览**。在**概览**页面的**账号管理**区域，单击**用户登录地址**链接，使用新创建的RAM用户登录阿里云控制台。

3.  在AWS侧准备可编程及访问S3的IAM用户。

    1.  登录[S3控制台](https://s3.console.aws.amazon.com/s3/home)。

    2.  在导出的数据目录上单击右键，选择**获取总大小**，获取迁移目录的数据大小和文件个数。

        -   获取总大小

            ![获取总大小](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/6659559951/p139988.png)

        -   获取迁移目录的数据大小和文件个数

            ![数据大小和文件数量](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/7659559951/p139994.png)

    3.  登录[IAM用户控制台](https://console.aws.amazon.com/iam/home?#/users)，单击**添加用户**。

        ![IAM用户控制台](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/7659559951/p139842.png)

    4.  在**添加用户**页面，配置**用户名**。在**选择AWS访问类型**区域，选中**编程访问**，单击**下一步：权限**。

        ![配置用户信息](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/7659559951/p139852.png)

    5.  在**添加用户**页面，单击**直接附加现有策略**。在搜索框中输入**S3**，选中**AmazonS3ReadOnlyAccess**策略，单击**下一步：标签**。

        ![配置用户策略](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/7659559951/p139861.png)

    6.  单击**下一步：审核** \> **创建用户**，完成IAM用户创建并记录密钥信息。

        创建在线迁移任务时会使用该密钥信息。

        ![完成用户创建](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/7659559951/p139865.png)

4.  创建在线迁移数据地址。

    1.  登录[阿里云数据在线迁移控制台](https://mgw.console.aliyun.com/#/job)。在左侧导航栏，单击**数据地址**。

    2.  如果未开通在线迁移服务，请在弹出的对话框中单击**去申请**。在**在线迁移公测申请**页面，填写信息，并单击**提交**。

        ![在线迁移公测申请](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/7659559951/p139884.png)

    3.  在**管理数据地址**页面，单击**创建数据地址**，配置数据源及目标地址相关参数，单击**确定**。参数详情请参见[迁移实施]()。

        -   数据源

            ![创建源数据地址](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/7659559951/p139902.png)

            **说明：** **Access Key ID**和**Access Key Secret**为IAM用户的密钥信息。

        -   目标地址

            ![创建目标地址](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/7659559951/p139909.png)

            **说明：** **Access Key ID**和**Access Key Secret**为RAM用户的密钥信息。

5.  创建在线迁移任务。

    1.  在左侧导航栏，单击**迁移任务**。

    2.  在**迁移任务列表**页面，单击**创建迁移任务**，配置相关信息后，单击**创建**。参数详情请参见[迁移实施]()。

        -   **任务配置**

            ![任务配置](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/7659559951/p139935.png)

        -   **性能调优**

            ![性能调优](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/7659559951/p139937.png)

            **说明：** **待迁移存储量**和**待迁移文件个数**是您通过S3控制台获取到的待迁移数据大小和文件个数。

    3.  创建的迁移任务会自动运行。请您确认**任务状态**为**已完成**，表示迁移任务成功结束。

        ![迁移任务完成](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/7659559951/p139991.png)

    4.  在迁移任务的右侧单击**管理**，查看迁移任务报告，确认数据已经全部迁移成功。

        ![迁移报告](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/7659559951/p139998.png)

    5.  登录[OSS管理控制台](https://oss.console.aliyun.com/)。

    6.  在左侧导航栏，单击**Bucket列表**。在**Bucket列表**区域单击创建的Bucket。在Bucket页面，单击**文件管理**，查看文件迁移结果。

        ![OSS目录迁移结果](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/7659559951/p140005.png)


## 步骤三：将数据从OSS迁移至同区域的MaxCompute项目并验证数据完整性和正确性

您可以通过MaxCompute的LOAD命令将OSS数据迁移至同区域的MaxCompute项目中。

LOAD命令支持STS认证和AccessKey认证两种方式，AccessKey认证方式需要使用明文AccessKey ID和AccessKey Secret。STS认证方式不会暴露AccessKey信息，具备高安全性。本文以STS认证方式为例介绍数据迁移操作。

1.  在DataWorks的临时查询界面或MaxCompute客户端（odpscmd），使用Redshift集群数据的DDL，创建与迁移数据相对应的表。

    临时查询功能详情请参见[使用临时查询运行SQL语句（可选）](/intl.zh-CN/快速入门/使用临时查询运行SQL语句（可选）.md)。命令示例如下。

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

    本文的TPC-H数据集使用MaxCompute 2.0数据类型和Decimal 2.0数据类型，所以创建的项目为MaxCompute 2.0版本。如果您的项目需要设置使用2.0数据类型，请在命令前添加如下语句一起提交执行。

    ```
    setproject odps.sql.type.system.odps2=true;
    setproject odps.sql.decimal.odps2=true;
    ```

2.  创建具备访问OSS权限的RAM角色并授权。详情请参见[STS模式授权](/intl.zh-CN/开发/外部表/STS模式授权.md)。

3.  分次执行LOAD命令，将OSS的全部数据加载至创建的MaxCompute表中，并执行SQL命令查看和校验数据导入结果。LOAD命令详情请参见[LOAD](/intl.zh-CN/开发/SQL及函数/LOAD.md)。

    ```
    LOAD OVERWRITE TABLE orders 
    FROM  LOCATION 'oss://endpoint/oss_bucket_name/unload_from_redshift/orders_parquet/' --OSS存储空间位置。
    ROW FORMAT SERDE 'org.apache.hadoop.hive.ql.io.parquet.serde.ParquetHiveSerDe'
    WITH SERDEPROPERTIES ('odps.properties.rolearn'='acs:ram::xxx:role/xxx_role')
    STORED AS PARQUET;
    ```

    **说明：** 如果导入失败，请[提工单](https://workorder-intl.console.aliyun.com/)联系MaxCompute团队处理。

    执行如下命令查看和校验数据导入结果。

    ```
    SELECT * FROM orders limit 100;
    ```

    返回结果示例如下。

    ![查询导入结果](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/8659559951/p140826.png)

4.  通过表的数量、记录的数量和典型作业的查询结果，校验迁移至MaxCompute的数据是否和Redshift集群的数据一致。

    1.  登录[Amazon Redshift控制台](https://console.aws.amazon.com/redshiftv2/home)，在右上角选择区域为**亚太地区（新加坡）**。在左侧导航栏，单击**编辑器**，输入如下命令执行查询操作。

        ```
        SELECT l_returnflag, l_linestatus, SUM(l_quantity) as sum_qty,
        SUM(l_extendedprice) AS sum_base_price, SUM(l_extendedprice*(1-l_discount)) AS sum_disc_price,
        SUM(l_extendedprice*(1-l_discount)*(1+l_tax)) AS sum_charge, AVG(l_quantity) AS avg_qty,
        AVG(l_extendedprice) AS avg_price, AVG(l_discount) AS avg_disc,  COUNT(*) AS count_order
        FROM lineitem
        GROUP BY l_returnflag, l_linestatus
        ORDER BY l_returnflag,l_linestatus;
        ```

        返回结果示例如下。

        ![AWS查询结果](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/8659559951/p140170.png)

    2.  通过DataWorks的临时查询界面或MaxCompute客户端（odpscmd），执行上述命令，验证返回结果是否与Redshift集群的返回结果一致。

        返回结果示例如下。

        ![临时查询结果](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/8659559951/p140803.png)


