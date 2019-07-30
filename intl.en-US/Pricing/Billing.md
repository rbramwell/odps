# Billing {#concept_f32_fpm_xdb .concept}

This topic provides an overview of MaxCompute billable items.

Overview:

-   MaxCompute is billed according to the projects you create and the items you use within each project.
-   Billable items include the storage, computing resources and data downloads of a project.
-   MaxCompute fees are calculated daily.

## Storage pricing {#section_efw_kpm_xdb .section}

The data that is stored in MaxCompute, including tables and resources, is billed according to the storage used. The billing cycle is **one day**.

MaxCompute records the storage used for each project on an hourly basis. The average storage over the day is then calculated for each project space at the end of each day.

The daily MaxCompute fee is calculated by applying the tiered unit prices in the table below to the average storage used. Up to 1 GB of storage is free each day, while storage used between 1 GB and 100 GB costs 0.0028 USD for each gigabyte and so on. If you require more than 1 PB of storage per day, you can open a ticket to get a quote for the price.

|Less than 1 GB USD/GB/Day|1 GB to 100 GB USD/GB/Day|100 GB to 1 TB USD/GB/Day|1 TB to 10 TB USD/GB/Day|10 TB to 100 TB USD/GB/Day|100 TB to 1 PB USD/GB/Day|More than 1 PB USD/GB/Day|
|:------------------------|:------------------------|:------------------------|:-----------------------|:-------------------------|:------------------------|-------------------------|
|Free|0.0028|0.0014|0.0013|0.0011|0.0009|Please contact us|

For example, if you store 50 TB data in MaxCompute, the bill is calculated as follows.

``` {#codeblock_vxb_qfi_73f}
(100GB - 1) × 0.0028 USD/GB/day
+ (1024 - 100) GB × 0.0014 USD/GB/day
+ (10240 - 1024) GB × 0.0013 USD/GB/day
+ (50 x 1024 - 10240) GB × 0.0011 USD/GB/day
= 58.61 USD/day
```

**Note:** 

-   Because MaxCompute compresses and stores user data, the bill is based on the capacity size of the data after compresssion. This means the size of the stored data is different from the size of the data as measured by you before storage. The compression ratio is generally about 5.
-   Generally, MaxCompute fees are deducted no more than 6 hours after the daily fee calculation is completed, and are automatically deducted from the corresponding account balance.
-   On the MaxCompute console, you can view your consumption details under **Bill Management**.

## Computation pricing {#section_lfw_kpm_xdb .section}

MaxCompute supports two kinds of billing methods.

-   Pay-As-You-Go: Each task is measured according to the input size by job cost.
-   Subscription \(CU cost\): Users can subscribe the usage of a part of the resource in advance. This method is only supported on **Alibaba Cloud DTPlus Platform**.

Currently, MaxCompute supports the following computing task types: SQL, UDF, MapReduce, Graph, and machine learning. Charges apply for SQL \(excluding UDF\) computing tasks and for MapReduce tasks \(charges introduced 2017-12-19\). There are no plans to charge for other types in future.

Pay-As-You-Go

Pay-As-You-Go is for SQL tasks and MapReduce tasks.

SQL tasks are paid after volume, that is, SQL is charged after I/O:

Every SQL task is billed according to **Data Input Size** and **SQL Complexity**. Once the SQL task is completed, MaxCompute sends its metering information to the billing system, which calculates the fee and adds it to the next payment.

The MaxCompute SQL task is charged according to I/O for each job. All daily measurement information is paid next day.

The bill for SQL tasks is calculated as follows.

``` {#codeblock_2iv_uc9_h9m}
 Computing Cost of One SQL Task = Data Input Size × SQL Complexity × SQL Price
```

The price is as follows.

|Item|Price|
|:---|:----|
|SQL task|0.0438 USD/GB|

-   **Data Input Size**: The actual size of the data that an SQL statement scans. Most SQL statements have partition filtering and column pruning, so this value is generally less than the source table data size.
    -   Column pruning: For example, the submitted SQL is `select f1,f2,f3 from t1;`. Only the data size of three columns \(f1, f2, and f3\) in t1 are charged.
    -   Partition filtering: For example, a SQL statement includes ds\>”20130101”. The “ds” is a partition column. The data size is calculated only according to the read partition, rather than the data of other partitions, and then billed.
-   **SQL Complexity**: First, MaxCompute counts keywords in SQL statements, and then converts to SQL complexity.
    -   SQL keyword number = join Number + group by number + order by number + distinct number + window function number + max \(insert into Number -1, 1\)
    -   SQL complexity calculation:
        -   If SQL keyword number is less than or equal to 3, the complexity is 1.
        -   If SQL keyword number is less than or equal to 6, the complexity is 1.5.
        -   If SQL keyword number is less than or equal to 19, the complexity is 2.
        -   If SQL keyword number is greater than or equal to 20, the complexity is 4.

The input SQL statement for calculating **SQL Complexity** is as follows:

``` {#codeblock_3nl_mw2_78z}
cost sql <SQL Sentence>;
```

An example of a SQL statement is as follows:

``` {#codeblock_u5t_0oq_7ha}
odps@ $odps_project >cost sql SELECT DISTINCT total1 FROM
(SELECT id1, COUNT(f1) AS total1 FROM in1 GROUP BY id1) tmp1
ORDER BY total1 DESC LIMIT 100;

Intput:1825361100.8 Bytes
Complexity:1.5
```

The preceding SQL includes 4 keywords \(one DISTINCT, one COUNT, one GROUP BY, and one ORDER\), so the SQL complexity is 1.5. If the data volume of table “in1” is 1.7 GB, then the actual consumption is as follows:

``` {#codeblock_bqa_286_fsh}
1.7 × 1.5 × 0.0438 = 0.11 USD
```

**Note:** 

-   The bill invoicing time is usually before 06:00 the next day. After the computing task successfully ends, the system counts the data size and SQL complexity. After the bill is generated, the fee is automatically deducted from your account. If the SQL task is unsuccessful, no fee is deducted.
-   As with storage, SQL computing also calculates and bills the data size after compression.

**Note:** When you need to mix the internal and external tables, you will pay separately.

The billing for MaxCompute SQL external tables commenced on 24th July, 2019. The billing for one external table SQL task is calculated as follows:

``` {#codeblock_zvm_zft_ao6}
 Computing Cost of One SQL Task = Data Input Size × SQL Price
```

The SQL price is as follows.

|Item|Price|
|:---|:----|
|SQL task|0.0044 USD/GB|

The price for an SQL task is 0.0044 USD/GB/Complexity. The complexity coefficient is 1. All metering information is calculated and summarized by the end of the day and billing is then generated for the next day.

**Note:** If you use internal and external tables at the same time, the two types of tables are charged separately.

Pay-As-You-Go for MapReduce

In December 19, 2017, MaxCompute began charging for MapReduce \(MR\) tasks. The billing of an MR task is calculated as follows:

``` {#codeblock_8u4_xhi_tm9}
Computation Cost of One MR task = Total Time × MR Price (USD)
```

The price is as follows.

|Item|Price|
|:---|:----|
|MR task|0.0690 USD/Hour/Task|

The calculating time for each successful MR task is: Execution time \(hours\) × Number of cores that task calls.

If one MR task calls 100 cores, and the task takes 30 minutes to complete, the calculating time for the MR task is: 0.5 hours × 100 cores = 50 hours.

After the MR task is finished, MaxCompute sends its metering information to the billing system, which calculates the fee and adds it to the next payment.

**Note:** 

-   You are not charged if a task fails to run.
-   The calculating time does not include the time waiting for execution.
-   If you purchased the MaxCompute Subscription service, you can use MR tasks for free within the range of the services you purchased. No additional fee is charged.

Subscription \(Spark on MaxCompute\)

The billing for running Spark on MaxCompute tasks commenced on 24th July, 2019. The overall computation cost for your [Spark on MaxCompute](../../../../reseller.en-US/User Guide/Spark/Spark on MaxCompute overview.md#) tasks is calculated as follows:

``` {#codeblock_g9v_8b6_0lb}
Overall computation cost for all Spark on MaxCompute tasks on a day = Total number of compute hours for all tasks on the day × Unit price (0.1041 USD/Hour/Task)
```

The term compute hour used in Spark on MaxCompute tasks is defined as follows:

-   Compute hours are measured based on the number of CPU cores and memory space that are consumed.
-   One compute hour is equal to one CPU core plus 4-GB memory space.
-   Compute hours are calculated by using the following formula: `Max [Number of CPU cores consumed x Computation duration,roundup (Memory space consumed x Computation duration/4)]`.

For example, if you consume 2 CPU cores and 5-GB memory space for running your Spark on MaxCompute tasks for 1 hour, then the compute hours you need to pay are calculated as follows:

``` {#codeblock_zp4_ke8_5r8}
Max[2 × 1, roundup (5 × 1/4)] = 2
```

If you consume 2 CPU cores and 10-GB memory space for running your Spark on MaxCompute tasks for 1 hour, then the compute hours you need to pay are calculated as follows:

``` {#codeblock_gf4_97m_fxw}
Max [2 × 1, roundup (10 × 1/4)] = 3
```

After a Spark on MaxCompute task finishes, the system calculates the compute hours for the task. The metering information for all your Spark on MaxCompute tasks is summarized into your bill on the next day and the system automatically deducts the fees from your account balance.

**Note:** 

-   The time used for queuing is not counted into compute hours.
-   The fees for the same job vary depending on the size of resources you specify when submitting the job.
-   If you purchase the MaxCompute \(Subscription\) service, then you can run Spark on MaxCompute tasks for free within the scope of the service. No additional fees are charged.
-   If you have any questions about the fees charged for your Spark on MaxCompute tasks, you can open a ticket.
-   Spark on MaxCompute has been rolled out the Chian East 1 \(Hangzhou\), China North 2 \(Beijing\), China South 1 \(Shenzhen\), US West 1 \(Silicon Valley\), Hong Kong \(Hong Kong\), EU Central 1 \(Frankfurt\), Asia Pacific SE 1 \(Singapore\), and Asia Pacific SOU 1 \(Mumbai\) regions, and will be rolled out in the other regions soon.

Subscription \(CU cost\)

Payment by subscription is only available on the **Alibaba Cloud DTPlus Platform**. Subscription allows you to pay an initial fee \(monthly or annually\) for your entire reserved resources. The basic unit of such resources is defined as CU \(Compute Unit\). One CU includes 1 core CPU and 4 GB of memory.

|Resource definitions|Memory|CPU|Price|
|:-------------------|:-----|:--|:----|
|1 CU|4GB|1 CPU|22.0 USD/month|

We recommend that new users use the Pay-As-You-Go billing method, because this allows you to gauge your resource usage without unnecessary costs. Payment by subscription is only available on the Alibaba Cloud DTPlus platform.

## Download pricing {#section_fgw_kpm_xdb .section}

You can download data from the extranet through the MaxCompute Tunnel. The billing method for data downloads is Pay-As-You-Go. The calculation is as follows.

``` {#codeblock_q1b_qq8_skx}
Download Cost from Extranet/time = Downloaded Data Volume x Download Price
```

The price is as follows.

|Item|Price|
|:---|:----|
|Data download|0.1166 USD/GB|

**Note:** 

-   MaxCompute sends you messages to notify you of the size of your downloads, and to provide you with your download costs the next day.
-   Download data volume refers to the size of an HTTP body for one download request. The HTTP body that carries data uses protobuffer encoding, so it is generally smaller than the original data size, but larger than the data stored in MaxCompute after compression.
-   The different billing methods are applicable to different network environments, such as public networks, classic networks of Alibaba Cloud, or VPC networks. For more information about MaxCompute service connections, see [Configure Endpoint](../../../../reseller.en-US/Prepare/Configure Endpoint.md#).

