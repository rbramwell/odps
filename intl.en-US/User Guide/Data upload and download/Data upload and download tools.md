# Data upload and download tools {#concept_fft_tkf_vdb .concept}

The MaxCompute platform supports a wide range of data upload and download tools. The source code for most of the tools can be found on GitHub, the open-source community to upload and download the data. You can select the tool according to the scenario. The tools are divided into two types: Alibaba Cloud DTplus products and open-source products. This article helps you learn more about these tools.

## Alibaba Cloud DTplus products {#section_g3j_jlf_vdb .section}

-   **Data integration of DataWorks** 

    Data Integration, or data synchronization, of DataWorks is a stable, efficient, and scalable data synchronization platform provided by Alibaba Cloud. It is designed to provide full offline and incremental real-time data synchronization, integration, and exchange services for the heterogeneous data storage systems on Alibaba Cloud.

    Data synchronization tasks support the following data types: MaxCompute, RDS \(MySQL, SQL Server, and PostgreSQL\), Oracle, FTP, AnalyticDB \(ADS\), OSS, Memcache, and DRDS. For more information, see [Data integration overview](../../../../reseller.en-US/User Guide/Data integration/Data integration introduction/Data integration overview.md#).

-   **MaxCompute console** 

    -   For information about console installation and basic use, see [Client introduction](../../../../reseller.en-US/Tools and Downloads/Client.md).
    -   Based on the [Batch data](reseller.en-US/User Guide/Data upload and download/Tunnel SDK/Tunnel overview.md) tunnel SDK, the client provides built-in Tunnel commands for data upload and download. For more information, see [Basic Tunnel command usage](reseller.en-US/User Guide/Data upload and download/Tunnel commands.md).
    **Note:** This is an open-source [aliyun-odps-console](https://github.com/aliyun/aliyun-odps-console).

-   **DTS** 

    [Data Transmission \(DTS\)](https://www.alibabacloud.com/help/doc-detail/26592.html) is a data service provided by Alibaba Cloud that supports data exchanges between RDBMS, NoSQL, OLAP, and other data sources. It provides data migration, real-time data subscription, real-time data synchronization, and other data transmission features.

    DTS supports data synchronization from ApsaraDB for RDS and MySQL instances to MaxCompute tables. Currently, other data source types are not supported.


## Open-source products {#section_pq1_fnf_vdb .section}

-   **Sqoop** 

    As a tool developed based on the Sqoop 1.4.6 community, Sqoop provides enhanced MaxCompute support with the ability to import and export data from MySQL and other relational databases to MaxCompute tables. Data in HDFS/Hive can also be imported to MaxCompute tables.

    **Note:** This is an open-source [aliyun-maxcompute-data-collectors](https://github.com/aliyun/aliyun-maxcompute-data-collectors).

-   **Kettle** 

    Kettle is an open-source ETL tool based on Java which can run on Windows, Unix, or Linux. It provides graphic interfaces for you to easily define data transmission topology using drag-and-drop components.

    **Note:** This is an open-source [aliyun-maxcompute-data-collectors](https://github.com/aliyun/aliyun-maxcompute-data-collectors?spm=a2c4g.11186623.2.15.oSXp9R).

-   **Flume** 

    Apache Flume is a distributed and reliable system, which efficiently collects, aggregates, and moves massive volumes of log data from different data sources to a centralized data storage system. It supports multiple Source and Sink plugins.

    The DataHub Sink plug-in of Apache Flume allows you to upload log data to DataHub in real time and archive the data in the MaxCompute tables.

    **Note:** This is an open-source [aliyun-maxcompute-data-collectors](https://github.com/aliyun/aliyun-maxcompute-data-collectors?spm=a2c4g.11186623.2.17.oSXp9R).

-   **Fluentd** 

    Fluentd is an open-source software product that collects logs, including Application Logs, System Logs, and Access Logs, from various sources. It allows you to select plug-ins to filter and store log data to different data processors, including MySQL, Oracle, MongoDB, Hadoop, and Treasure Data.

    The DataHub plug-in of Fluentd allows you to upload data to DataHub in real time and archive the data in MaxCompute tables.

-   **LogStash** 

    Logstash is an open-source log collection and processing framework. The logstash-output-datahub plugin allows you to import data to DataHub. This tool can be easily configured to collect and transmit data. When used together with MaxCompute or StreamCompute, it allows you to easily create an all-in-one streaming data solution right from data collection to analysis.

    The DataHub plug-in of Logstash allows you to upload data to DataHub in real time and archive the data in MaxCompute tables.

-   **OGG** 

    The DataHub plug-in of OGG allows you to incrementally synchronize the Oracle database data to DataHub in real time and archive the data in MaxCompute tables.

    **Note:** This is an open-source [aliyun-maxcompute-data-collectors](https://github.com/aliyun/aliyun-maxcompute-data-collectors?spm=a2c4g.11186623.2.21.oSXp9R).


