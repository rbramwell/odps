# Tunnel overview {#concept_lhh_2dw_tdb .concept}

This topic provides an overview of MaxCompute Tunnel, which is a data tunnel that is used to upload and download data to MaxCompute.

MaxCompute is based on Tunnel SDK and offers data upload and download tools. For more information, see [Client](../../../../reseller.en-US/Tools and Downloads/Client.md).

When using Maven, you can search for odps-sdk-core in the [Maven database](http://search.maven.org/) to find different versions of Java SDK. The configuration is as follows:

``` {#codeblock_jqs_4te_i8n}
<dependency>
    <groupId>com.aliyun.odps</groupId>
    <artifactId>odps-sdk-core</artifactId>
    <version>0.24.0-public</version>
</dependency>
```

The following table describes the interfaces of Tunnel SDK, which may differ according to the SDK version. For more information, see [SDK Java Doc](https://www.javadoc.io/doc/com.aliyun.odps/odps-sdk-core/0.31.3-public).

|Interface|Description|
|:--------|:----------|
|TableTunnel|The ingress-class interface that is used to access the MaxCompute Tunnel service. You can access MaxCompute and Tunnel through the Internet or an intranet network on Alibaba Cloud. Data downloaded through an intranet network is free of charge.|
|TableTunnel.UploadSession|A session that is for uploading data to a MaxCompute table.|
|TableTunnel.DownloadSession|A session that is for downloading data from a MaxCompute table.|
|InstanceTunnel|The ingress-class interface that is used to access the MaxCompute Tunnel service. You can access MaxCompute and Tunnel through the Internet or an intranet network on Alibaba Cloud. Data downloaded through an intranet network is free of charge.|
|InstanceTunnel.DownloadSession|A session that is for downloading data to a MaxCompute SQL instance. The SQL instance must start with the`SELECT`keyword and is used for querying data.|

**Note:** 

-   For information about Tunnel SDK, see [SDK Java Doc](https://www.javadoc.io/doc/com.aliyun.odps/odps-sdk-core/0.31.3-public).
-   For information about service connections, see [Configure Endpoint](../../../../reseller.en-US/Prepare/Configure Endpoint.md).

