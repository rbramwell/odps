# Configure endpoints {#concept_m2j_h1y_5db .concept}

This topic describes the regions that MaxCompute is available in, MaxCompute connection methods, and issues arising from use with other Alibaba Cloud services such as ECS, Table Store, and OSS.

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/11949/15687717541423_en-US.png)

MaxCompute provides two types of endpoints:

-   MaxCompute endpoint: All requests except for upload and download requests can be sent to MaxCompute. For example, you can send a request to create a table, delete a function, or create a job.
-   MaxCompute Tunnel endpoint: The MaxCompute Tunnel endpoint is used to upload and download data. To upload and download data through MaxCompute Tunnel, you can initiate a request through the MaxCompute Tunnel endpoint.

    **Note:** 

    -   MaxCompute Tunnel downloads are charged differently in each region due to different deployments and network connection states for MaxCompute Tunnel.
    -   If your Tunnel endpoint needs to connect over the internal network, you must configure an internal MaxCompute endpoint. Otherwise, traffic may be routed to the public network and you may be charged for downloads over the public network.

## Connection methods and billing rules for data download {#section_ydd_51y_5db .section}

You can access MaxCompute and MaxCompute Tunnel through the following connection methods:

-   Public network
-   Alibaba Cloud classic network
-   Alibaba Cloud VPC

**Note:** You only need to specify a network when you want to connect to a project. You do not need to specify a network when you create a project.

Data uploads

Uploading data through MaxCompute Tunnel is free of charge regardless of which connection method is used, as shown in the preceding figure.

Data downloads

The billing rules for downloading data through MaxCompute Tunnel depend on whether the source and destination are in the same region:

-   If the source and destination are in the same region, downloading data through MaxCompute Tunnel is **free of charge** over both the classic network and VPC.
-   If the source and destination are not in the same region or do not meet the requirements for the same region access, downloading data through MaxCompute Tunnel over the public network is billed.

    **Note:** Because the MaxCompute Tunnel deployment and network connection status in every region are different, we cannot guarantee permanent connectivity for MaxCompute Tunnel if you select the Alibaba Cloud classic network or VPC.


## Connectivity configurations for accessing external tables {#section_d2d_51y_5db .section}

MaxCompute 2.0 supports reading and writing OSS data and Table Store data. For more information, see [Access OSS unstructured data](../../../../reseller.en-US/User Guide/External table/Access OSS unstructured data.md) and [Access Table Store unstructured data](../../../../reseller.en-US/User Guide/External table/Access Table Store (OTS) data.md).

Connectivity configurations are as follows:

-   If MaxCompute is in the same region as Table Store or OSS, we recommend that you select the Alibaba Cloud classic network or VPC connection method. The public network can also be selected.
-   If MaxCompute is not in the same region as Table Store or OSS, we recommend that you select the public network connection method. We cannot guarantee permanent connectivity for MaxCompute Tunnel if MaxCompute is in the same region as Table Store or OSS and you select the Alibaba Cloud classic network or VPC.
-   However, you can use a physical connection to access a VPC to guarantee MaxCompute Tunnel connectivity. For more information, see [Access cloud services in VPC through physical connections](https://www.alibabacloud.com/help/zh/doc-detail/57195.html).

## Regions where MaxCompute is available and corresponding endpoints {#section_f2d_51y_5db .section}

MaxCompute is available in regions both inside and outside Mainland China. You can apply for data storage and computing in these regions.

**Note:** Both HTTP and HTTPS are supported in public endpoints \(aliyun\). If you need to encrypt your requests, use HTTPS.

-   **Regions and endpoints for the public network connection method** 

    |Region name|City|MaxCompute available or not|Public endpoint|Public Tunnel endpoint|
    |:----------|:---|:--------------------------|:--------------|:---------------------|
    |China \(Hangzhou\)|Hangzhou|Available|http://service.cn-hangzhou.maxcompute.aliyun.com/api|http://dt.cn-hangzhou.maxcompute.aliyun.com|
    |China \(Shanghai\)|Shanghai|Available|http://service.cn-shanghai.maxcompute.aliyun.com/api|http://dt.cn-shanghai.maxcompute.aliyun.com|
    |China \(Beijing\)|Beijing|Available|http://service.cn-beijing.maxcompute.aliyun.com/api|http://dt.cn-beijing.maxcompute.aliyun.com|
    |China \(Shenzhen\)|Shenzhen|Available|http://service.cn-shenzhen.maxcompute.aliyun.com/api|http://dt.cn-shenzhen.maxcompute.aliyun.com|
    |China \(Hong Kong\)|Hong Kong|Available|http://service.cn-hongkong.maxcompute.aliyun.com/api|http://dt.cn-hongkong.maxcompute.aliyun.com|
    |Singapore|Singapore|Available|http://service.ap-southeast-1.maxcompute.aliyun.com/api|http://dt.ap-southeast-1.maxcompute.aliyun.com|
    |Australia \(Sydney\)|Sydney|Available|http://service.ap-southeast-2.maxcompute.aliyun.com/api|http://dt.ap-southeast-2.maxcompute.aliyun.com|
    |Malaysia \(Kuala Lumpur\)|Kuala Lumpur|Available|http://service.ap-southeast-3.maxcompute.aliyun.com/api|http://dt.ap-southeast-3.maxcompute.aliyun.com|
    |Indonesia \(Jakarta\)|Jakarta|Available|http://service.ap-southeast-5.maxcompute.aliyun.com/api|http://dt.ap-southeast-5.maxcompute.aliyun.com|
    |Japan \(Tokyo\)|Tokyo|Available|http://service.ap-northeast-1.maxcompute.aliyun.com/api|http://dt.ap-northeast-1.maxcompute.aliyun.com|
    |Germany \(Frankfurt\)|Frankfurt|Available|http://service.eu-central-1.maxcompute.aliyun.com/api|http://dt.eu-central-1.maxcompute.aliyun.com|
    |US \(Silicon Valley\)|Silicon Valley|Available|http://service.us-west-1.maxcompute.aliyun.com/api|http://dt.us-west-1.maxcompute.aliyun.com|
    |US \(Virginia\)|Virginia|Available|http://service.us-east-1.maxcompute.aliyun.com/api|http://dt.us-east-1.maxcompute.aliyun.com|
    |India \(Mumbai\)|Mumbai|Available|http://service.ap-south-1.maxcompute.aliyun.com/api|http://dt.ap-south-1.maxcompute.aliyun.com|
    |UAE \(Dubai\)|Dubai|Available|http://service.me-east-1.maxcompute.aliyun.com/api|http://dt.me-east-1.maxcompute.aliyun.com|
    |UK \(London\)|London|Available|http://service.eu-west-1.maxcompute.aliyun.com/api|http://dt.eu-west-1.maxcompute.aliyun.com|

-   **Regions and endpoints for the classic network connection method** 

    |Region name|City|MaxCompute available or not|Classic endpoint|Classic Tunnel endpoint|
    |-----------|----|---------------------------|----------------|-----------------------|
    |China \(Hangzhou\)|Hangzhou|Available|http://service.cn-hangzhou.maxcompute.aliyun-inc.com/api|http://dt.cn-hangzhou.maxcompute.aliyun-inc.com|
    |China \(Shanghai\)|Shanghai|Available|http://service.cn-shanghai.maxcompute.aliyun-inc.com/api|http://dt.cn-shanghai.maxcompute.aliyun-inc.com|
    |China \(Beijing\)|Beijing|Available|http://service.cn-beijing.maxcompute.aliyun-inc.com/api|http://dt.cn-beijing.maxcompute.aliyun-inc.com|
    |China \(Shenzhen\)|Shenzhen|Available|http://service.cn-shenzhen.maxcompute.aliyun-inc.com/api|http://dt.cn-shenzhen.maxcompute.aliyun-inc.com|
    |China \(Hong Kong\)|Hong Kong|Available|http://service.cn-hongkong.maxcompute.aliyun-inc.com/api|http://dt.cn-hongkong.maxcompute.aliyun-inc.com|
    |Singapore|Singapore|Available|http://service.ap-southeast-1.maxcompute.aliyun-inc.com/api|http://dt.ap-southeast-1.maxcompute.aliyun-inc.com|
    |Australia \(Sydney\)|Sydney|Available|http://service.ap-southeast-2.maxcompute.aliyun-inc.com/api|http://dt.ap-southeast-2.maxcompute.aliyun-inc.com|
    |Malaysia \(Kuala Lumpur\)|Kuala Lumpur|Available|http://service.ap-southeast-3.maxcompute.aliyun-inc.com/api|http://dt.ap-southeast-3.maxcompute.aliyun-inc.com|
    |Indonesia \(Jakarta\)|Jakarta|Available|http://service.ap-southeast-5.maxcompute.aliyun-inc.com/api|http://dt.ap-southeast-5.maxcompute.aliyun-inc.com|
    |Japan \(Tokyo\)|Tokyo|Available|http://service.ap-northeast-1.maxcompute.aliyun-inc.com/api|http://dt.ap-northeast-1.maxcompute.aliyun-inc.com|
    |Germany \(Frankfurt\)|Frankfurt|Available|http://service.eu-central-1.maxcompute.aliyun-inc.com/api|http://dt.eu-central-1.maxcompute.aliyun-inc.com|
    |US \(Silicon Valley\)|Silicon Valley|Available|http://service.us-west-1.maxcompute.aliyun-inc.com/api|http://dt.us-west-1.maxcompute.aliyun-inc.com|
    |US \(Virginia\)|Virginia|Available|http://service.us-east-1.maxcompute.aliyun-inc.com/api|http://dt.us-east-1.maxcompute.aliyun-inc.com|
    |India \(Mumbai\)|Mumbai|Available|http://service.ap-south-1.maxcompute.aliyun-inc.com/api|http://dt.ap-south-1.maxcompute.aliyun-inc.com|
    |UAE \(Dubai\)|Dubai|Available|http://service.me-east-1.maxcompute.aliyun-inc.com/api|http://dt.me-east-1.maxcompute.aliyun-inc.com|
    |UK \(London\)|London|Available|http://service.uk-all.maxcompute.aliyun-inc.com/api|http://dt.uk-all.maxcompute.aliyun-inc.com|

-   Regions and endpoints for the VPC connection method

    You can only use the following endpoints and Tunnel endpoints when you select the VPC connection method.

    |Region name|City|MaxCompute available or not|VPC endpoint|VPC Tunnel endpoint|
    |:----------|:---|:--------------------------|:-----------|:------------------|
    |China \(Hangzhou\)|Hangzhou|Available|http://service.cn-hangzhou.maxcompute.aliyun-inc.com/api|http://dt.cn-hangzhou.maxcompute.aliyun-inc.com|
    |China \(Shanghai\)|Shanghai|Available|http://service.cn-shanghai.maxcompute.aliyun-inc.com/api|http://dt.cn-shanghai.maxcompute.aliyun-inc.com|
    |China \(Beijing\)|Beijing|Available|http://service.cn-beijing.maxcompute.aliyun-inc.com/api|http://dt.cn-beijing.maxcompute.aliyun-inc.com|
    |China \(Shenzhen\)|Shenzhen|Available|http://service.cn-shenzhen.maxcompute.aliyun-inc.com/api|http://dt.cn-shenzhen.maxcompute.aliyun-inc.com|
    |China \(Hong Kong\)|Hong Kong|Available|http://service.cn-hongkong.maxcompute.aliyun-inc.com/api|http://dt.cn-hongkong.maxcompute.aliyun-inc.com|
    |Singapore|Singapore|Available|http://service.ap-southeast-1.maxcompute.aliyun-inc.com/api|http://dt.ap-southeast-1.maxcompute.aliyun-inc.com|
    |Australia \(Sydney\)|Sydney|Available|http://service.ap-southeast-2.maxcompute.aliyun-inc.com/api|http://dt.ap-southeast-2.maxcompute.aliyun-inc.com|
    |Malaysia \(Kuala Lumpur\)|Kuala Lumpur|Available|http://service.ap-southeast-3.maxcompute.aliyun-inc.com/api|http://dt.ap-southeast-3.maxcompute.aliyun-inc.com|
    |Indonesia \(Jakarta\)|Jakarta|Available|http://service.ap-southeast-5.maxcompute.aliyun-inc.com/api|http://dt.ap-southeast-5.maxcompute.aliyun-inc.com|
    |Japan \(Tokyo\)|Tokyo|Available|http://service.ap-northeast-1.maxcompute.aliyun-inc.com/api|http://dt.ap-northeast-1.maxcompute.aliyun-inc.com|
    |Germany \(Frankfurt\)|Frankfurt|Available|http://service.eu-central-1.maxcompute.aliyun-inc.com/api|http://dt.eu-central-1.maxcompute.aliyun-inc.com|
    |US \(Silicon Valley\)|Silicon Valley|Available|http://service.us-west-1.maxcompute.aliyun-inc.com/api|http://dt.us-west-1.maxcompute.aliyun-inc.com|
    |US \(Virginia\)|Virginia|Available|http://service.us-east-1.maxcompute.aliyun-inc.com/api|http://dt.us-east-1.maxcompute.aliyun-inc.com|
    |India \(Mumbai\)|Mumbai|Available|http://service.ap-south-1.maxcompute.aliyun-inc.com/api|http://dt.ap-south-1.maxcompute.aliyun-inc.com|
    |UAE \(Dubai\)|Dubai|Available|http://service.me-east-1.maxcompute.aliyun-inc.com/api|http://dt.me-east-1.maxcompute.aliyun-inc.com|
    |UK \(London\)|London|Available|http://service.uk-all.maxcompute.aliyun-inc.com/api|http://dt.uk-all.maxcompute.aliyun-inc.com|


**Note:** Scenarios that require you to configure the MaxCompute endpoint and Tunnel endpoint:

-   MaxCompute client \(console\) configurations. For more information, see [Install and configure a client](reseller.en-US/Prepare/Install and configure a client.md#).
-   MaxCompute Studio project connection configurations. For more information, see [Project space connection management](../../../../reseller.en-US/Tools and Downloads/MaxCompute Studio/Project space connection management.md#).
-   MaxCompute SDK connection configurations. For more information, see MaxCompute connection configurations in [Java SDK](../../../../reseller.en-US/SDK Reference /Java SDK/Java SDK.md#) and [Python SDK](../../../../reseller.en-US/SDK Reference /Python SDK.md#).
-   Configurations for connecting to MaxCompute data sources from the DataWorks data integration script method or the DataX open-source tool. For more information, see [Configure a MaxCompute data source](https://www.alibabacloud.com/help/zh/faq-detail/74280.htm) and [Export SQL operation result](../../../../reseller.en-US/Best Practices/SQL/Export SQL operation result.md#).

## Access rules {#section_pbt_7py_3s9 .section}

-   In the regions where MaxCompute has been available, you can connect to MaxCompute over the public network, classic network, or VPC.
-   When you use the Tunnel endpoint to download data over the public network, the price is USD 0.1166/GB.

