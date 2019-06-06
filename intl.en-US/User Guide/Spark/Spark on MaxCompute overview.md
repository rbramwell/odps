# Spark on MaxCompute overview {#concept_jzp_wlb_kgb .concept}

This topic provides an overview of Spark on MaxCompute. It is an open-source framework that functions on the service level to support data processing and analysis operations. Equipped with unified computing resources and data set permissions, Spark on MaxCompute allows you to submit and run jobs while using your preferred development methods.

## Features {#section_bdx_d0a_sup .section}

-   Supports different versions of native Spark jobs.

    MaxCompute is compatible with the APIs of all native Spark versions that run in MaxCompute. Different versions of Spark can run in MaxCompute at the same time. Spark on MaxCompute provides you with native Spark Web UIs.

-   Runs in unified computing resources.

    Similar to MaxCompute SQL and MaxCompute MapReduce, Spark on MaxCompute runs in the unified computing resources activated for MaxCompute projects.

-   Supports unified data and permission management.

    Spark on MaxCompute complies with the permissions you set for MaxCompute projects, allowing you to query data without any additional permission modifications required.

-   Provides the same experience as open-source systems.

    Spark on MaxCompute provides the same user experience as open-source systems, both in terms of an open-source application UI and online interactions. Specifically, it supports native, open-source, and real-time UIs that are essential for debugging open-source applications, while also provides the historical log query function.


## Architecture {#section_svc_5ux_kny .section}

The Spark on MaxCompute architecture solution allows native Spark engines to run in MaxCompute. The following figure shows this architecture.

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/92656/155978768036635_en-US.png)

The left half of the diagram shows the architecture of a native Spark engine, and the right half shows the architecture of Spark on MaxCompute. This second architecture runs on the Cupid platform developed by Alibaba Cloud. This platform also supports computing frameworks such as native Spark engines.

## Supported services {#section_reu_7ts_ksv .section}

MaxCompute Spark supports the following services:

-   All Java and Scala offline jobs such as GraphX, MLlib, RDD, Spark SQL, and PySpark
-   Read and write operations on MaxCompute tables
-   Unstructured storage in OSS

MaxCompute Spark will support the following services in its later versions:

-   Read and write operations on services such as RDS, Redis, and ECS in VPCs
-   Streaming services
-   Interactive services such as Spark shell, Spark SQL shell, and PySpark shell

