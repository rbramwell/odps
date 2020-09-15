---
keyword: [GCP, BigQuery, MaxCompute]
---

# BigQuery数据迁移至MaxCompute

本文为您介绍如何通过公网环境将谷歌云GCP（Google Cloud Platform）的BigQuery数据集迁移至阿里云MaxCompute。

|类别|平台|要求|参考文档|
|--|--|--|----|
|环境及数据|谷歌云GCP|-   已开通谷歌BigQuery服务，并准备好环境及待迁移的数据集。
-   已开通谷歌Cloud Storage服务，并创建存储分区（Bucket）。

|如果您没有相关环境及数据集，可参考如下内容准备：-   BigQuery：[BigQuery快速入门](https://cloud.google.com/bigquery/docs/quickstarts)和[创建数据集](https://cloud.google.com/bigquery/docs/datasets)
-   Cloud Storage：[Cloud Storage快速入门](https://cloud.google.com/storage/docs/quickstart-console)和[创建存储分区](https://cloud.google.com/storage/docs/creating-buckets) |
|阿里云|-   已开通MaxCompute、DataWorks服务并创建项目空间。

以**印度尼西亚（雅加达）**区域为例，创建作为迁移目标的MaxCompute项目。

-   已开通对象存储服务OSS并创建存储分区（Bucket）。
-   已开通OSS的在线迁移上云服务。

|如果您没有相关环境，可参考如下内容准备：-   MaxCompute、DataWorks：[准备工作](/cn.zh-CN/准备工作/准备阿里云账号.md)和[创建项目空间](/cn.zh-CN/准备工作/创建项目空间.md)
-   OSS：[开通OSS服务](/cn.zh-CN/快速入门/开通OSS服务.md)和[创建存储空间](/cn.zh-CN/快速入门/创建存储空间.md)
-   在线迁移上云服务：[提工单](https://selfservice.console.aliyun.com/ticket/createIndex)或[在线申请](https://page.aliyun.com/form/act998591440/index.htm?spm=5176.a2c3g.0.0.46583d89fy9Fpo) |
|账号|谷歌云GCP|已创建具备访问谷歌Cloud Storage权限的IAM用户。|[通过JSON使用IAM权限](https://cloud.google.com/storage/docs/access-control/iam-json)|
|阿里云|已创建具备存储空间读写权限和在线迁移权限的RAM用户及RAM角色。|[创建RAM用户](/cn.zh-CN/用户管理/创建RAM用户.md)和[STS模式授权](/cn.zh-CN/开发/外部表/STS模式授权.md)|
|区域|谷歌云GCP|无。|无|
|阿里云|开通OSS服务的区域与MaxCompute项目在同一区域。|无|

将BigQuery数据集迁移至阿里云MaxCompute的流程如下。

![迁移流程](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/3091410061/p143579.png)

|序号|描述|
|--|--|
|①|将BigQuery数据集导出至谷歌Cloud Storage。|
|②|通过对象存储服务OSS的**在线迁移上云服务**，将数据从谷歌Cloud Storage迁移至OSS。|
|③|将数据从OSS迁移至同区域的MaxCompute项目中，并校验数据完整性和正确性。|

## 步骤一：将BigQuery数据集导出至谷歌Cloud Storage

您可以使用bq命令行工具执行`bq extract`命令，将BigQuery数据集导出至谷歌Cloud Storage。

1.  登录[Google Cloud控制台](https://console.cloud.google.com/storage)，创建存储迁移数据的分区（Bucket）。操作详情请参见[创建存储分区](https://cloud.google.com/storage/docs/creating-buckets)。

    ![存储分区](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/7991559951/p149249.png)

2.  使用bq命令行工具，查询TPC-DS数据集中表的DDL脚本并下载至本地设备。操作详情请参见[使用INFORMATION\_SCHEMA获取表元数据](https://cloud.google.com/bigquery/docs/information-schema-tables)。

    如果您需要了解bq命令行工具，请参见[使用bq命令行工具](https://cloud.google.com/bigquery/docs/bq-command-line-tool)。

    BigQuery不提供诸如`show create table`之类的命令来查询表的DDL脚本。BigQuery允许您使用内置的用户自定义函数UDF来查询特定数据集中表的DDL脚本。DDL脚本示例如下。

    ![DDL](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/7991559951/p149250.png)

3.  通过bq命令行工具执行`bq extract`命令，将BigQuery数据集中的表依次导出至谷歌Cloud Storage的目标存储分区（Bucket）。相关操作及导出的数据格式和压缩类型详情请参见[导出表数据](https://cloud.google.com/bigquery/docs/exporting-data)。

    导出命令示例如下。

    ```
    bq extract
    --destination_format AVRO 
    --compression SNAPPY
    tpcds_100gb.web_site
    gs://bucket_name/web_site/web_site-*.avro.snappy;
    ```

4.  查看存储分区，检查数据导出结果。


## 步骤二：将导出至谷歌Cloud Storage的数据迁移至对象存储服务OSS

对象存储服务OSS支持通过**在线迁移上云服务**，将谷歌Cloud Storage的数据迁移至OSS，详情请参见[谷歌云GCP迁移教程]()。**在线迁移上云服务**处于公测状态，您需要[提工单](https://selfservice.console.aliyun.com/ticket/createIndex)联系客服，并由在线服务团队开通后才可使用。

1.  预估需要迁移的数据，包括迁移存储量和迁移文件个数。您可以使用gsutil工具或通过存储日志查看待迁移存储分区（Bucket）的存储量。详情请参见[获取存储分区信息](https://cloud.google.com/storage/docs/getting-bucket-information)。

2.  如果您未创建存储分区，请登录[OSS管理控制台](https://oss.console.aliyun.com/)，创建保存迁移数据的分区（Bucket），详情请参见[创建存储空间](/cn.zh-CN/快速入门/创建存储空间.md)。

    ![OSS分区](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/7991559951/p149259.png)

3.  如果您未创建RAM用户，请创建RAM用户并授予相关权限。

    1.  登录[RAM访问控制台](https://ram.console.aliyun.com)，创建RAM用户，详情请参见[创建RAM用户](/cn.zh-CN/用户管理/创建RAM用户.md)。

    2.  选中新创建的用户登录名称，单击**添加权限**，为新创建的RAM用户授予**AliyunOSSFullAccess**（存储空间读写权限）和**AliyunMGWFullAccess**（在线迁移权限），单击**确定** \> **完成**。

    3.  在左侧导航栏，单击**概览**。在**概览**页面的**账号管理**区域，单击**用户登录地址**链接，使用新创建的RAM用户登录[阿里云控制台](https://homenew.console.aliyun.com/)。

4.  在GCP侧准备一个以编程方式访问谷歌Cloud Storage的用户。操作详情请参见[通过JSON使用IAM权限](https://cloud.google.com/storage/docs/access-control/iam-json)。

    1.  登录[IAM用户控制台](https://console.cloud.google.com/iam-admin/serviceaccounts)，选择一个有权限访问BigQuery的用户。在**操作**列，单击**![操作](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/7991559951/p143506.png)** \> **创建密钥**。

    2.  在弹出的对话框，选择**JSON**，单击**创建**。将JSON文件保存至本地设备，并单击**完成**。

    3.  在**创建服务账号**页面，单击**选择角色**，选择**Cloud Storage** \> **Storage Admin**，授予用户访问谷歌Cloud Storage的权限。

5.  创建在线迁移数据地址。

    1.  登录[阿里云数据在线迁移控制台](https://mgw.console.aliyun.com/#/job)。在左侧导航栏，单击**数据地址**。

    2.  如果您未开通在线迁移服务，请在弹出的对话框中单击**去申请**。在**在线迁移公测申请**页面，填写信息，并单击**提交**。

        ![在线迁移公测申请](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/7991559951/p146824.png)

        **说明：** **在线迁移公测申请**页面的**迁移数据源类型**中如果没有谷歌云GCP，请您选择其中一种数据源类型，并在**备注**中指明实际数据源类型。

    3.  在**管理数据地址**页面，单击**创建数据地址**，配置数据源及目标地址相关参数，单击**确定**。参数详情请参见[迁移实施]()。

        -   数据源

            ![数据源](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/7991559951/p143531.png)

            **说明：** **Key File**是[步骤4](#step_9xl_wyx_oe6)下载的JSON文件。

        -   目标地址

            ![目标地址](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/7991559951/p143538.png)

            **说明：** **Access Key ID**和**Access Key Secret**为RAM用户的密钥信息。

6.  创建在线迁移任务。

    1.  在左侧导航栏，单击**迁移任务**。

    2.  在**迁移任务列表**页面，单击**创建迁移任务**，配置相关信息后，单击**创建**。参数详情请参见[迁移实施]()。

        -   **任务配置**

            ![任务配置](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/7991559951/p143543.png)

        -   **性能调优**

            ![性能优化](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/8991559951/p143545.png)

            **说明：** **待迁移存储量**和**待迁移文件个数**是您通过Google Cloud控制台获取到的待迁移数据大小和文件个数。

    3.  创建的迁移任务会自动运行。请您确认**任务状态**为**已完成**，表示迁移任务成功结束。

    4.  在迁移任务的右侧单击**管理**，查看迁移任务报告，确认数据已经全部迁移成功。

    5.  登录[OSS管理控制台](https://oss.console.aliyun.com/)。

    6.  在左侧导航栏，单击**Bucket列表**。在**Bucket列表**区域单击创建的Bucket。在Bucket页面，单击**文件管理**，查看数据迁移结果。


## 步骤三：将数据从OSS迁移至同区域的MaxCompute项目并验证数据完整性和正确性

您可以通过MaxCompute的LOAD命令将OSS数据迁移至同区域的MaxCompute项目中。

LOAD命令支持STS认证和AccessKey认证两种方式，AccessKey认证方式需要使用明文AccessKey ID和AccessKey Secret。STS认证方式不会暴露AccessKey信息，具备高安全性。本文以STS认证方式为例介绍数据迁移操作。

1.  在DataWorks的临时查询界面或MaxCompute客户端（odpscmd），修改已获取到的BigQuery数据集中表的DDL，适配MaxCompute数据类型，创建与迁移数据相对应的表。

    临时查询功能详情请参见[使用临时查询运行SQL语句（可选）](/cn.zh-CN/快速入门/使用临时查询运行SQL语句（可选）.md)。命令示例如下。

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

    BigQuery和MaxCompute数据类型的对应关系如下。

    |BigQuery数据类型|MaxCompute数据类型|
    |------------|--------------|
    |INT64|BIGINT|
    |FLOAT64|DOUBLE|
    |NUMERIC|DECIMAL、DOUBLE|
    |BOOL|BOOLEAN|
    |STRING|STRING|
    |BYTES|VARCHAR|
    |DATE|DATE|
    |DATETIME|DATETIME|
    |TIME|DATETIME|
    |TIMESTAMP|TIMESTAMP|
    |STRUCT|STRUCT|
    |GEOGRAPHY|STRING|

2.  如果您未创建RAM角色，请创建具备访问OSS权限的RAM角色并授权。详情请参见[STS模式授权](/cn.zh-CN/开发/外部表/STS模式授权.md)。

3.  执行LOAD命令，将OSS的全部数据加载至创建的MaxCompute表中，并执行SQL命令查看和校验数据导入结果。LOAD命令一次只能加载一张表，有多个表时，需要执行多次。LOAD命令详情请参见[LOAD](/cn.zh-CN/开发/SQL及函数/LOAD.md)。

    ```
    LOAD OVERWRITE TABLE web_site
    FROM  LOCATION 'oss://oss-<your_region_id>-internal.aliyuncs.com/bucket_name/tpc_ds_100gb/web_site/' --OSS存储空间位置。
    ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.avro.AvroSerDe'
    WITH SERDEPROPERTIES ('odps.properties.rolearn'='<Your_RoleARN>','mcfed.parquet.compression'='SNAPPY')
    STORED AS AVRO;
    ```

    **说明：** 如果导入失败，请[提工单](https://selfservice.console.aliyun.com/ticket/createIndex)联系MaxCompute团队处理。

    执行如下命令查看和校验数据导入结果。

    ```
    SELECT * FROM web_site;
    ```

    返回结果示例如下。

    ![返回结果](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/8991559951/p143575.png)

4.  通过表的数量、记录的数量和典型作业的查询结果，校验迁移至MaxCompute的数据是否和BigQuery的数据一致。


