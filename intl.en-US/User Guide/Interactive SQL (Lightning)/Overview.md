# Overview {#concept_fvd_fqt_z2b .concept}

MaxCompute Lightning provides interactive query services for MaxCompute, and supports easy connection to MaxCompute projects based on the PostgreSQL protocol and syntax. This service allows you to quickly query and analyze MaxCompute project data using standard SQL and commonly used tools.

You can use major BI tools, such as Tableau and FineReport, to easily connect to MaxCompute projects, and perform BI analysis or ad hoc queries. The quick query feature in MaxCompute Lightning allows you to provide services by encapsulating project table data in APIs, supporting diverse application scenarios without data migration.

MaxCompute Lightning offers serverless computing services. No infrastructure is required and you pay only for queries.

## Key features {#section_fwm_w3j_z2b .section}

-   Compatibility with the PostgreSQL protocol

    MaxCompute Lightning provides Java Database Connectivity \(JDBC\) or Open Database Connectivity \(ODBC\) interfaces that are compatible with the PostgreSQL protocol. Tools or applications based on PostgreSQL databases can easily be connected to MaxCompute projects using default drivers. The easy connection enables diverse PostgreSQL tools to be used for analyzing MaxCompute project data.

-   Improved performance

    Quick query for MaxCompute tables is optimized, especially for small datasets and high query concurrency, supporting diverse application scenarios, such as regular reports and service APIs.

-   Unified permissions management

    MaxCompute Lightning is a product designed for MaxCompute products and provides access to MaxCompute projects. This service shares the same access control system with MaxCompute projects. This ensures that users can only query data that they are authorized to access.

-   Out-of-the-box feature and pay by queries

    MaxCompute Lightning provides serverless computing services based on existing MaxCompute computing resources. To perform queries, you only need to establish connections to MaxCompute projects using MaxCompute Lightning.

    You do not need to configure, manage, or maintain MaxCompute Lightning resources.When using MaxCompute Lightning, you only incur costs for the data amount processed for each query.


## System architecture {#section_qqv_mjj_z2b .section}

MaxCompute Lightning provides a method of connecting endpoints, clients, or applications to JDBC or ODBC interfaces using PostgreSQL drivers. This enables secure data access within the unified access control system for MaxCompute projects.

Query tasks, connected and submitted by JDBC or ODBC interfaces, use serverless computing resources of MaxCompute Lightning to ensure query service quality.

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/20113/156617780111152_en-US.jpg)

## Scenarios {#section_vdv_zjj_z2b .section}

-   Ad hoc query

    The query for small datasets \(less than 100 GB\) is optimized to allow you to easily query MaxCompute tables with low latency. You do not need to import the MaxCompute data into the AnalyticDB \(ADS\), Relational Database Service \(RDS\), or other systems, which reduces required resources and administration costs.

    This scenario has the following characteristics: flexible data objects for queries, complicated logic, quick query, easy adjustment of query logic, and low latency query requirements within one minute. Users are often data analysts who master SQL skills and want to use familiar client tools for query analysis.

-   Reporting and analysis

    Analysis reports are generated based on the MaxCompute project data consolidated in the Extract-Transform-Load \(ETL\) process. The reports are provided to managers and business users for regular checks.

    This scenario has the following characteristics: The queried data objects are usually the aggregated data. The queried data objects are included in small datasets. Queries are based on fixed and simple query logic. The scenario has low latency requirements. Latency for most queries is within 5 seconds. The query latency period varies greatly depending on the data volume and query complexity.

-   Online application

    MaxCompute project data can be encapsulated in RESTful APIs to support online applications.

    In this scenario, MaxCompute Lightning serves as an accelerated query engine to provide MaxCompute table data as API services with the least amount of manual intervention. This is enabled by integrating [data service components](reseller.en-US/User Guide/DataService studio/DataService studio overview.md#) of Alibaba Cloud DataWorks.


