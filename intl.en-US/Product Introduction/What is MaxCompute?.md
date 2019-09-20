# What is MaxCompute? {#concept_qbk_1kv_tdb .concept}

The big data computing service \(MaxCompute, formerly called ODPS\) is a fast and fully hosted GB/TB/PB level data warehouse solution.

MaxCompute supports a variety of classic distributed computing models that enable you to solve massive data calculation problems while reducing business costs, and maintaining data security.

MaxCompute seamlessly integrates with DataWorks, which provides one-stop data synchronization, task development, data workflow development, data operation and maintenance, and data management for MaxCompute. For more information, see [What is DataWorks?](../../../../intl.en-US/Product Introduction/What is DataWorks?.md#).

MaxCompute is mainly used to store and compute batches of structured data. It provides a massive range of data warehouse solutions as well as big data analysis and modeling services. As data collection techniques are becoming increasingly diverse and comprehensive,industries are amassing larger and larger volumes of data. The scale of data has increased to the level of massive data \(100 GB, TB and even PB\) that traditional software industry can not carry.

Given these massive data volumes, the limited processing capacity of a single server has prompted analysts to move towards distributed computing. However, distributed computing models are not easy to maintain and demand highly-qualified data analysts. When using a distributed model,data analysts not only need to understand their business needs, but also must be familiar with the underlying computing model. The purpose of MaxCompute is to provide you with a convenient way of analyzing and processing mass data, and you can achieve the purpose of analyzing large data without having to care about the details of distributed computing.

**Note:** MaxCompute is widely used by Alibaba Group in scenarios such as data warehouse and BI analysis, web log analysis, transaction analysis of e-commerce sites, and customer behavior analysis.

## MaxCompute learning path {#section_learningpath .section}

You can quickly learn about MaxCompute's related concepts, basic operations and advanced operations through [MaxCompute learning path](https://www.alibabacloud.com/getting-started/learningpath/maxcompute).

## Product advantage {#section_ufc_wrv_tdb .section}

-   **Large-scale computing and storage** 

    MaxCompute is suitable for the storage and processing of large volumes of data \(up to PB-level\).

-   **Multiple computational models** 

    MaxCompute supports data processing methods based on SQL, MapReduce,Graph, MPI iteration algorithm, and other programming models.

-   **Strong data security** 

    MaxCompute has stabilized alloffline analysis for all Alibaba Group's business for more than seven years, providing multilayer sandbox protection and monitoring.

-   **Cost-effective** 

    MaxCompute can help reduce procurement costs by 20%-30% compared with on-premises private cloud models.


## Function {#section_cnm_c5v_tdb .section}

-   **Data tunnel** 
    -   Supports large volumes of historical data channels

        [TUNNEL](../../../../intl.en-US/User Guide/Data upload and download/Tunnel SDK/Tunnel overview.md) provides high concurrency data upload and download services. This service supports the import and export of terabytes or petabytes of data on a daily basis, which is particularly useful for the batch import of full or historical data. Tunnel Provides you with a Java programming interface, and in the MaxCompute client tool, there are corresponding commands for local file and service data interchange.

    -   Real-time and incremental data channels

        For real-time data upload scenarios, MaxCompute provides DataHub services with low latency and convenient usage. It is especially suitable for incremental data imports. DataHub also supports a variety of data transmission plug-ins, such as Logstash, Flume, Fluentd, and Sqoop, it supports Log. Service's delivery log to MaxCompute, and then use DataWorks to do log analysis and mining.

-   **Computing and analysis tasks** 

    MaxCompute provides multiple computing models.

    -   SQL: In MaxCompute, data is stored in tables. MaxCompute provides an SQL query function for the external interface. You can operate MaxCompute similarly to a traditional database software but with the ability to process PB-level data.

**Note:** 

-   MaxCompute SQL does not support transactions, index, or Update/Delete operations.
-   MaxCompute SQL syntax differs from Oracle and MySQL, notably, you cannot seamlessly migrate SQL statements of other databases into MaxCompute.
-   In terms of usage, MaxCompute SQL can complete queries at the second- to millisecond-level,and can not return results at milliseconds.
-   The advantage of MaxCompute SQL is low learning cost. You don't need to understand the concept of complex distributed computing. If you have experience in database operations, you can familiarize yourself with MaxCompute SQL quickly.
    -   [UDF](../../../../intl.en-US/User Guide/SQL/UDF/UDF Summary.md): A user-defined function.

        MaxCompute provides numerous[built-in functions](../../../../intl.en-US/User Guide/SQL/Builtin functions/Date functions.md)to meet your computing needs, while also supporting the creation of custom functions.

    -   [MapReduce](../../../../intl.en-US/User Guide/MapReduce/Summary/MapReduce.md): MapReduce is a Java MapReduce programming model provided by MaxCompute. It uses the Java programming interface and is designed to simplify the development process. However, users are recommended to have a basic understanding of the concept of distribution, and relevant programming experience before using MapReduce. MaxCompute MapReduce provides you with Java programming interface.
    -   [Graph](../../../../intl.en-US/User Guide/Graph/Summary.md)：Graph in MaxCompute is a processing framework designed for iterative graph computing. Graph computing jobs use graphs to build models. Graphs are composed of vertices and edges. Vertices and edges contain values. After performing iterative graph editing and evolution, you can get the final result. Typical applications include PageRank, SSSP algorithm, and K-Means algorithm.The graph is edited and evolved through an iteration, and the results are finally solved. Typical applications: [PageRank](../../../../intl.en-US/User Guide/Graph/Examples/PageRank.md), [single source shortest distance algorithm](../../../../intl.en-US/User Guide/Graph/Examples/SSSP.md), [K-means clustering algorithm](../../../../intl.en-US/User Guide/Graph/Examples/Kmeans.md), and so on.
-   **SDK** 

    A convenient toolkit provided for developers. For more information, see [MaxCompute SDK](../../../../intl.en-US/SDK Reference /Java SDK/Java SDK.md).

-   **Secure** 

    Maxcompute offers powerful security services to protect your data, for more information, see the [security guide](../../../../intl.en-US/Management/Configure security features/Target users.md).


## What to do next {#section_akn_wvv_tdb .section}

Now, you have learned about MaxCompute's product advantages, functional features and other related profiles, you can continue to learn the next tutorial. In this tutorial, you will understand the related charges of MaxCompute. For more information, see[Product Pricing](../../../../intl.en-US/Pricing/Computation pricing.md#) .

