# Configure Endpoint {#concept_m2j_h1y_5db .concept}

This sections details the regions where MaxCompute is available, and connection methods. Problems such as network connectivity and downloading charges related to use of other Alibaba Cloud products \(such as ECS, TableStore, OSS, and so on\) are also detailed.

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/11949/15671479141423_en-US.png)

MaxCompute provides two types of service connection addresses as follows:

-   **The connection address of the MaxCompute service itself**: All requests except uploading and downloading data can be sent to MaxCompute, such as creating a table, deleting a function, creating a job, and so on.
-   **The connection address of MaxCompute Tunnel service**: The ability to upload and download data is provided by MaxCompute Tunnel service. When you want to upload and download data through the Tunnel, you can initiate a request through the link address provided by the Tunnel.

    **Note:** Because the deployment and network connection status for every region are different, the billing rules for downloading data through Tunnel are also different.


## Accessing method and data downloading billing rule {#section_ydd_51y_5db .section}

According to the deployment and network conditions of Alibaba Cloud regions, it mainly contains three connection methods as follows:

-   Access MaxCompute Service or Tunnel Service from the Internet.
-   Access MaxCompute service and Tunnel service from Alibaba Cloud classic network.
-   Access MaxCompute service and Tunnel service from Alibaba Cloud VPC network.

**Note:** You only need to specify a network when you want to connect to a project, not during MaxCompute project creation.

**data upload**

Data uploading is free-of-charge, regardless of which kind of network configuration is adopted.

**Data download**

According to the Tunnel service request to download data, and from which region of the ECS server, the charging rules are as follows.

-   If requester and requestee are both in the same region, the tunnel download request through Alibaba Cloud classic network or VPC network is **free**. **Note that the tunnel endpoint must be configured at this time, otherwise the data is routed to the outer network for download across the region, the cost is generated**.
-   If requester and requestee are not in the same region or cannot meet the same region access, then the request is a cross-region access, and data download **fees** are calculated under this condition.

    **Note:** Due to deployment and network conditions of cross-regional **access of Alibaba Cloud products and services**, MaxCompute cannot be **guaranteed** a permanent connection. Some fees may apply.


## The connectivity configuration {#section_d2d_51y_5db .section}

MaxCompute 2.0 supports reading and writing OSS object, and also supports reading and writing tables of Table Store. For more information, see [Access OSS Unstructured Data](../../../../intl.en-US/User Guide/External table/Access OSS unstructured data.md) and [Access Table Store Data](../../../../intl.en-US/User Guide/External table/Access Table Store (OTS) data.md).

Configuration instructions about network connectivity are as follows:

-   If MaxCompute and TableStore/OSS are in the same region, we recommend to configure the Alibaba Cloud classic network or VPC network connection method. The external network method can also be selected.
-   If MaxCompute and TableStore/OSS are not in the same region, select the external network method to connect. In the case of cross-regional access, connectivity is not permanently guaranteed if you select the Alibaba Cloud classic network or VPC network method.

## MaxCompute activation and service connections by region {#section_f2d_51y_5db .section}

The table as follows gives information about deployment of MaxCompute in different regions. On the International Site, MaxCompute is currently only available in the following regions. You can apply to use MaxComputer in this region only. In this case, your data storage and computing consumption all occur in the Asia Pacific SE 1 \(Singapore\) region.

**Note:** External endpoints\(aliyun\) support both HTTP and HTTPS. If you need to encrypt your requests, use HTTPS. Internal endpoints\(aliyun-inc\) only support HTTP.

-   **External MaxCompute activation and service connections by region** 

    |Region name|City|Service status|Internet endpoint|Internet tunnel endpoint|
    |:----------|:---|:-------------|:----------------|:-----------------------|
    |China East 1|Hangzhou|In service|http://service.cn.maxcompute.aliyun.com/api|http://dt.cn-hangzhou.maxcompute.aliyun.com|
    |China East 2|Shanghai|In service|http://service.cn.maxcompute.aliyun.com/api|http://dt.cn-shanghai.maxcompute.aliyun.com|
    |China North 2|Beijing|In service|http://service.cn.maxcompute.aliyun.com/api|http://dt.cn-beijing.maxcompute.aliyun.com|
    |China South 1|Shenzhen|In service|http://service.cn.maxcompute.aliyun.com/api|http://dt.cn-shenzhen.maxcompute.aliyun.com|
    |Hong Kong|Hong Kong|In service|http://service.cn-hongkong.maxcompute.aliyun.com/api|http://dt.cn-hongkong.maxcompute.aliyun.com|
    |Asia Pacific SE 1|Singapore|In service|http://service.ap-southeast-1.maxcompute.aliyun.com/api|http://dt.ap-southeast-1.maxcompute.aliyun.com|
    |Asia Pacific SE 2|Sydney|In service|http://service.ap-southeast-2.maxcompute.aliyun.com/api|http://dt.ap-southeast-2.maxcompute.aliyun.com|
    |Asia Pacific SE 3|Kuala Lumpur|In service|http://service.ap-southeast-3.maxcompute.aliyun.com/api|http://dt.ap-southeast-3.maxcompute.aliyun.com|
    |Asia Pacific SE 5|Jakarta|In service|http://service.ap-southeast-5.maxcompute.aliyun.com/api|http://dt.ap-southeast-5.maxcompute.aliyun.com|
    |Asia Pacific NE 1|Tokyo|In service|http://service.ap-northeast-1.maxcompute.aliyun.com/api|http://dt.ap-northeast-1.maxcompute.aliyun.com|
    |Central Europe 1|Frankfurt|In service|http://service.eu-central-1.maxcompute.aliyun.com/api|http://dt.eu-central-1.maxcompute.aliyun.com|
    |West USA 1|Silicon Valley|In service|http://service.us-west-1.maxcompute.aliyun.com/api|http://dt.us-west-1.maxcompute.aliyun.com|
    |East USA 1|Virginia|In service|http://service.us-east-1.maxcompute.aliyun.com/api|http://dt.us-east-1.maxcompute.aliyun.com|
    |Asia Pacific SOU 1|Mumbai|In service|http://service.ap-south-1.maxcompute.aliyun.com/api|http://dt.ap-south-1.maxcompute.aliyun.com|
    |Middle East 1|Dubai|In service|http://service.me-east-1.maxcompute.aliyun.com/api|http://dt.me-east-1.maxcompute.aliyun.com|
    |UK|London|In service|http://service.eu-west-1.maxcompute.aliyun.com/api|http://dt.eu-west-1.maxcompute.aliyun.com|

-   **Classic network MaxCompute activation and service connections by region** 

    |Region name|City|Service status|Classic network endpoint|Classic Network tunnel endpoint|
    |-----------|----|--------------|------------------------|-------------------------------|
    |China East 1|Hangzhou|In service|http://service.cn.maxcompute.aliyun-inc.com/api|http://dt.cn-hangzhou.maxcompute.aliyun-inc.com|
    |China East 2|Shanghai|In service|http://service.cn.maxcompute.aliyun-inc.com/api|http://dt.cn-shanghai.maxcompute.aliyun-inc.com|
    |China North 2|Beijing|In service|http://service.cn.maxcompute.aliyun-inc.com/api|http://dt.cn-beijing.maxcompute.aliyun-inc.com|
    |China South 1|Shenzhen|In service|http://service.cn.maxcompute.aliyun-inc.com/api|http://dt.cn-shenzhen.maxcompute.aliyun-inc.com|
    |Hong Kong|Hong Kong|In service|http://service.cn-hongkong.maxcompute.aliyun-inc.com/api|http://dt.cn-hongkong.maxcompute.aliyun-inc.com|
    |Asia Pacific SE 1|Singapore|In service|http://service.ap-southeast-1.maxcompute.aliyun-inc.com/api|http://dt.ap-southeast-1.maxcompute.aliyun-inc.com|
    |Asia Pacific SE 2|Sydney|In service|http://service.ap-southeast-2.maxcompute.aliyun-inc.com/api|http://dt.ap-southeast-2.maxcompute.aliyun-inc.com|
    |Asia Pacific SE 3|Kuala Lumpur|In service|http://service.ap-southeast-3.maxcompute.aliyun-inc.com/api|http://dt.ap-southeast-3.maxcompute.aliyun-inc.com|
    |Asia Pacific SE 5|Jakarta|In service|http://service.ap-southeast-5.maxcompute.aliyun-inc.com/api|http://dt.ap-southeast-5.maxcompute.aliyun-inc.com|
    |Asia Pacific NE 1|Tokyo|In service|http://service.ap-northeast-1.maxcompute.aliyun-inc.com/api|http://dt.ap-northeast-1.maxcompute.aliyun-inc.com|
    |Central Europe 1|Frankfurt|In service|http://service.eu-central-1.maxcompute.aliyun-inc.com/api|http://dt.eu-central-1.maxcompute.aliyun-inc.com|
    |West USA 1|Silicon Valley|In service|http://service.us-west-1.maxcompute.aliyun-inc.com/api|http://dt.us-west-1.maxcompute.aliyun-inc.com|
    |East USA 1|Virginia|In service|http://service.us-east-1.maxcompute.aliyun-inc.com/api|http://dt.us-east-1.maxcompute.aliyun-inc.com|
    |Asia Pacific SOU 1|Mumbai|In service|http://service.ap-south-1.maxcompute.aliyun-inc.com/api|http://dt.ap-south-1.maxcompute.aliyun-inc.com|
    |Middle East 1|Dubai|In service|http://service.me-east-1.maxcompute.aliyun-inc.com/api|http://dt.me-east-1.maxcompute.aliyun-inc.com|
    |UK|London|In service|http://service.uk-all.maxcompute.aliyun-inc.com/api|http://dt.uk-all.maxcompute.aliyun-inc.com|

-   **VPC MaxCompute activation and service connections by region** 

    In a VPC, you can only access MaxCompute using the endpoint and tunnel endpoint below. Endpoint

    |Region name|City|Service status|VPC endpoint|VPC tunnel endpoint|
    |:----------|:---|:-------------|:-----------|:------------------|
    |China East 1|Hangzhou|In service|http://service.cn.maxcompute.aliyun-inc.com/api|http://dt.cn-hangzhou.maxcompute.aliyun-inc.com|
    |China East 2|Shanghai|In service|http://service.cn.maxcompute.aliyun-inc.com/api|http://dt.cn-shanghai.maxcompute.aliyun-inc.com|
    |China North 2|Beijing|In service|http://service.cn.maxcompute.aliyun-inc.com/api|http://dt.cn-beijing.maxcompute.aliyun-inc.com|
    |China South 1|Shenzhen|In service|http://service.cn.maxcompute.aliyun-inc.com/api|http://dt.cn-shenzhen.maxcompute.aliyun-inc.com|
    |Hong Kong|Hong Kong|In service|http://service.cn-hongkong.maxcompute.aliyun-inc.com/api|http://dt.cn-hongkong.maxcompute.aliyun-inc.com|
    |Asia Pacific SE 1|Singapore|In service|http://service.ap-southeast-1.maxcompute.aliyun-inc.com/api|http://dt.ap-southeast-1.maxcompute.aliyun-inc.com|
    |Asia Pacific SE 2|Sydney|In service|http://service.ap-southeast-2.maxcompute.aliyun-inc.com/api|http://dt.ap-southeast-2.maxcompute.aliyun-inc.com|
    |Asia Pacific SE 3|Kuala Lumpur|In service|http://service.ap-southeast-3.maxcompute.aliyun-inc.com/api|http://dt.ap-southeast-3.maxcompute.aliyun-inc.com|
    |Asia Pacific SE 5|Jakarta|In service|http://service.ap-southeast-5.maxcompute.aliyun-inc.com/api|http://dt.ap-southeast-5.maxcompute.aliyun-inc.com|
    |Asia Pacific NE 1|Tokyo|In service|http://service.ap-northeast-1.maxcompute.aliyun-inc.com/api|http://dt.ap-northeast-1.maxcompute.aliyun-inc.com|
    |Central Europe 1|Frankfurt|In service|http://service.eu-central-1.maxcompute.aliyun-inc.com/api|http://dt.eu-central-1.maxcompute.aliyun-inc.com|
    |West USA 1|Silicon Valley|In service|http://service.us-west-1.maxcompute.aliyun-inc.com/api|http://dt.us-west-1.maxcompute.aliyun-inc.com|
    |East USA 1|Virginia|In service|http://service.us-east-1.maxcompute.aliyun-inc.com/api|http://dt.us-east-1.maxcompute.aliyun-inc.com|
    |Asia Pacific SOU 1|Mumbai|In service|http://service.ap-south-1.maxcompute.aliyun-inc.com/api|http://dt.ap-south-1.maxcompute.aliyun-inc.com|
    |Middle East 1|Dubai|In service|http://service.me-east-1.maxcompute.aliyun-inc.com/api|http://dt.me-east-1.maxcompute.aliyun-inc.com|
    |UK|London|In service|http://service.uk-all.maxcompute.aliyun-inc.com/api|http://dt.uk-all.maxcompute.aliyun-inc.com|


**Note:** Scenarios that need to configure Endpoint and Tunnel Endpoint:

-   MaxCompute client \(console\) configuration. See[Install and configure client](../../../../intl.en-US/Tools and Downloads/MaxCompute Studio/Project space connection management.md#).
-   MaxCompute studio project connection configuration. See[Project space management connection](../../../../intl.en-US/Tools and Downloads/Client.md#).
-   SDK connects to MaxCompute configuration. See[Java SDK](../../../../intl.en-US/SDK Reference /Java SDK/Java SDK.md#) and[Python SDK](../../../../intl.en-US/SDK Reference /Python SDK.md#) connect the MaxCompute interface configuration.
-   The DataWorks data integration script pattern connects the MaxComputer data source configuration and uses the DataX open source tool to connect the MaxComputer data source. See[Export SQL operation results](../../../../intl.en-US/Best Practices/SQL/Export SQL operation result.md#).

## Access concepts {#section_hgt_i9r_dj2 .section}

-   In regions where the service is available, you can connect to the MaxCompute service through the Internet, classic network, or VPC.
-   Fees will be charged for the data downloaded through the configured Internet tunnel endpoint, at a rate of 0.1166 USD/GB.

