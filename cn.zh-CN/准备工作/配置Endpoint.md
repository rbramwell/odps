# 配置Endpoint {#concept_m2j_h1y_5db .concept}

本文将为您介绍MaxCompute Region的开通情况和连接方式，解答您在与其他云产品（ECS、TableStore、OSS）互访场景中遇到的网络连通性和下载数据收费等问题。

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/11949/15628089741423_zh-CN.png)

从服务层面来看，MaxCompute为您提供了两大类服务连接地址：

-   MaxCompute服务本身的连接地址：您可以向MaxCompute发出除数据上传、下载外的所有请求，例如创建表、删除某个函数、创建一个作业等。
-   MaxCompute Tunnel服务的连接地址：上传、下载数据的能力是通过MaxCompute Tunnel服务提供的。当您想通过Tunnel上传、下载数据时，可以通过Tunnel提供的链接地址发起请求。

    **说明：** 

    -   由于各Region部署和网络连接状况不一致，在Tunnel数据的下载计费规则上也不统一。
    -   如果您不配置Tunnel Endpoint，Tunnel会自动路由到Service Endpoint所在的网络对应的Tunnel Endpoint，例如Service Endpoint为公网就路由到公网的Tunnel Endpoint；Service Endpoint为内网就路由到内网的Tunnel Endpoint。若您继续手动配置，则以手动配置为准，不进行自动路由。

## 访问来源及下载数据收费规则说明 {#section_ydd_51y_5db .section}

根据阿里云各Region的部署及网络情况，您可以通过以下三种连接方式访问MaxCompute服务和Tunnel服务：

-   从外网访问MaxCompute服务和Tunnel服务。
-   从阿里云经典网络访问MaxCompute服务和Tunnel服务。
-   从阿里云VPC网络访问MaxCompute服务和Tunnel服务。

**说明：** 创建MaxCompute的项目时无需指定网络，只在连接项目时才需指定通过什么网络进行连接。

数据上传

Tunnel数据上传无论走哪一种网络形态都免费，如上文示意图所示。

数据下载

您无论是在ECS云服务器的哪一个Region上进行Tunnel服务请求进行下载数据，网络连通性设置都需满足如下形态定义。

-   两者在同一Region内，Tunnel下载请求走阿里云经典网络/VPC网络都**免费**。
-   两者不在同一Region内或没有条件满足同Region访问，则需走外网跨Region访问请求，此条件下的数据下载将会进行计费。

    **说明：** 由于阿里云数据中心各个Region部署和网络情况不一致，若您选择通过阿里云经典网络/VPC网络进行跨Region的访问，则MaxCompute产品方不承诺、不保证其永久连通性。


## MaxCompute访问外部表的连通性 {#section_d2d_51y_5db .section}

MaxCompute 2.0版支持读写OSS对象存储数据，同时也支持读写TableStore表格存储数据，详情请参见[访问OSS非结构化数据](../../../../cn.zh-CN/开发/外部表/访问OSS非结构化数据.md)和[访问OTS非结构化数据](../../../../cn.zh-CN/开发/外部表/访问OTS非结构化数据.md)。

网络连通性的配置说明，如下所示：

-   MaxCompute和TableStore/OSS在同一Region情况下，建议配置阿里云经典网络或VPC网络连接方式，其外网也可以进行连通。
-   MaxCompute和TableStore/OSS不在同一Region情况下，配置外网访问方式进行连通。在跨Region的情况下，您可选择配置阿里云经典网络或VPC网络，否则不保证其连通性。
-   对于通过物理专线访问VPC的场景，请参见[通过物理专线访问VPC中的云服务](https://www.alibabacloud.com/help/zh/doc-detail/57195.html)保证网络连通性。

## MaxCompute已开服的Region和服务连接对照表 {#section_f2d_51y_5db .section}

从Region部署情况来看，MaxCompute目前国内国外陆续开服，您可以申请使用对应区域的MaxCompute，您的数据存储和计算消耗均发生在开通使用的区域。

**说明：** 公网Endpoint域名\(aliyun\)支持http和https，若需要请求加密，请用https。内网Endpoint域名\(aliyun-inc\)仅支持http，不支持https访问。

-   **外网网络下Region和服务连接对照表** 

    |Region名称|所在城市|开服状态|外网Endpoint|外网Tunnel Endpoint|
    |:-------|:---|:---|:---------|:----------------|
    |华东1|杭州|已开服|http://service.cn.maxcompute.aliyun.com/api|http://dt.cn-hangzhou.maxcompute.aliyun.com|
    |华东2|上海|已开服|http://service.cn.maxcompute.aliyun.com/api|http://dt.cn-shanghai.maxcompute.aliyun.com|
    |华北2|北京|已开服|http://service.cn.maxcompute.aliyun.com/api|http://dt.cn-beijing.maxcompute.aliyun.com|
    |华北2政务云|北京|已开服|http://service.cn-north-2-gov-1.maxcompute.aliyun.com/api|http://dt.cn-north-2-gov-1.maxcompute.aliyun.com|
    |华南1|深圳|已开服|http://service.cn.maxcompute.aliyun.com/api|http://dt.cn-shenzhen.maxcompute.aliyun.com|
    |西南1|成都|已开服|http://service.cn.maxcompute.aliyun.com/api|http://dt.cn-chengdu.maxcompute.aliyun.com|
    |香港|香港|已开服|http://service.cn-hongkong.maxcompute.aliyun.com/api|http://dt.cn-hongkong.maxcompute.aliyun.com|
    |亚太东南1|新加坡|已开服|http://service.ap-southeast-1.maxcompute.aliyun.com/api|http://dt.ap-southeast-1.maxcompute.aliyun.com|
    |亚太东南2|悉尼|已开服|http://service.ap-southeast-2.maxcompute.aliyun.com/api|http://dt.ap-southeast-2.maxcompute.aliyun.com|
    |亚太东南3|吉隆坡|已开服|http://service.ap-southeast-3.maxcompute.aliyun.com/api|http://dt.ap-southeast-3.maxcompute.aliyun.com|
    |亚太东南5|雅加达|已开服|http://service.ap-southeast-5.maxcompute.aliyun.com/api|http://dt.ap-southeast-5.maxcompute.aliyun.com|
    |亚太东北1|东京|已开服|http://service.ap-northeast-1.maxcompute.aliyun.com/api|http://dt.ap-northeast-1.maxcompute.aliyun.com|
    |欧洲中部1|法兰克福|已开服|http://service.eu-central-1.maxcompute.aliyun.com/api|http://dt.eu-central-1.maxcompute.aliyun.com|
    |美国西部1|硅谷|已开服|http://service.us-west-1.maxcompute.aliyun.com/api|http://dt.us-west-1.maxcompute.aliyun.com|
    |美国东部1|弗吉尼亚|已开服|http://service.us-east-1.maxcompute.aliyun.com/api|http://dt.us-east-1.maxcompute.aliyun.com|
    |亚太南部1|孟买|已开服|http://service.ap-south-1.maxcompute.aliyun.com/api|http://dt.ap-south-1.maxcompute.aliyun.com|
    |中东东部1|迪拜|已开服|http://service.me-east-1.maxcompute.aliyun.com/api|http://dt.me-east-1.maxcompute.aliyun.com|
    |英国|伦敦|已开服|http://service.eu-west-1.maxcompute.aliyun.com/api|http://dt.eu-west-1.maxcompute.aliyun.com|

-   **经典网络下Region和服务连接对照表** 

    |Region名称|所在城市|开服状态|经典网络Endpoint|经典网络Tunnel Endpoint|
    |--------|----|----|------------|-------------------|
    |华东1|杭州|已开服|http://service.cn.maxcompute.aliyun-inc.com/api|http://dt.cn-hangzhou.maxcompute.aliyun-inc.com|
    |华东2|上海|已开服|http://service.cn.maxcompute.aliyun-inc.com/api|http://dt.cn-shanghai.maxcompute.aliyun-inc.com|
    |华北2|北京|已开服|http://service.cn.maxcompute.aliyun-inc.com/api|http://dt.cn-beijing.maxcompute.aliyun-inc.com|
    |华北2政务云|北京|已开服|http://service.cn-north-2-gov-1-all.maxcompute.aliyun-inc.com/api|http://dt.cn-north-2-gov-1-all.maxcompute.aliyun-inc.com|
    |华南1|深圳|已开服|http://service.cn.maxcompute.aliyun-inc.com/api|http://dt.cn-shenzhen.maxcompute.aliyun-inc.com|
    |西南1|成都|已开服|http://service.cn.maxcompute.aliyun-inc.com/api|http://dt.cn-chengdu.maxcompute.aliyun-inc.com|
    |香港|香港|已开服|http://service.cn-hongkong.maxcompute.aliyun-inc.com/api|http://dt.cn-hongkong.maxcompute.aliyun-inc.com|
    |亚太东南1|新加坡|已开服|http://service.ap-southeast-1.maxcompute.aliyun-inc.com/api|http://dt.ap-southeast-1.maxcompute.aliyun-inc.com|
    |亚太东南2|悉尼|已开服|http://service.ap-southeast-2.maxcompute.aliyun-inc.com/api|http://dt.ap-southeast-2.maxcompute.aliyun-inc.com|
    |亚太东南3|吉隆坡|已开服|http://service.ap-southeast-3.maxcompute.aliyun-inc.com/api|http://dt.ap-southeast-3.maxcompute.aliyun-inc.com|
    |亚太东南5|雅加达|已开服|http://service.ap-southeast-5.maxcompute.aliyun-inc.com/api|http://dt.ap-southeast-5.maxcompute.aliyun-inc.com|
    |亚太东北1|东京|已开服|http://service.ap-northeast-1.maxcompute.aliyun-inc.com/api|http://dt.ap-northeast-1.maxcompute.aliyun-inc.com|
    |欧洲中部1|法兰克福|已开服|http://service.eu-central-1.maxcompute.aliyun-inc.com/api|http://dt.eu-central-1.maxcompute.aliyun-inc.com|
    |美国西部1|硅谷|已开服|http://service.us-west-1.maxcompute.aliyun-inc.com/api|http://dt.us-west-1.maxcompute.aliyun-inc.com|
    |美国东部1|弗吉尼亚|已开服|http://service.us-east-1.maxcompute.aliyun-inc.com/api|http://dt.us-east-1.maxcompute.aliyun-inc.com|
    |亚太南部1|孟买|已开服|http://service.ap-south-1.maxcompute.aliyun-inc.com/api|http://dt.ap-south-1.maxcompute.aliyun-inc.com|
    |中东东部1|迪拜|已开服|http://service.me-east-1.maxcompute.aliyun-inc.com/api|http://dt.me-east-1.maxcompute.aliyun-inc.com|
    |英国|伦敦|已开服|http://service.uk-all.maxcompute.aliyun-inc.com/api|http://dt.uk-all.maxcompute.aliyun-inc.com|

-   VPC网络下Region和服务连接对照表

    在VPC网络下访问MaxCompute，只能使用如下Endpoint和Tunnel Endpoint。

    |Region名称|所在城市|开服状态|VPC网络Endpoint|VPC网络Tunnel Endpoint|
    |:-------|:---|:---|:------------|:-------------------|
    |华东1|杭州|已开服|http://service.cn.maxcompute.aliyun-inc.com/api|http://dt.cn-hangzhou.maxcompute.aliyun-inc.com|
    |华东2|上海|已开服|http://service.cn.maxcompute.aliyun-inc.com/api|http://dt.cn-shanghai.maxcompute.aliyun-inc.com|
    |华北2|北京|已开服|http://service.cn.maxcompute.aliyun-inc.com/api|http://dt.cn-beijing.maxcompute.aliyun-inc.com|
    |华北2政务云|北京|已开服|http://service.cn-north-2-gov-1-all.maxcompute.aliyun-inc.com/api|http://dt.cn-north-2-gov-1-all.maxcompute.aliyun-inc.com|
    |华南1|深圳|已开服|http://service.cn.maxcompute.aliyun-inc.com/api|http://dt.cn-shenzhen.maxcompute.aliyun-inc.com|
    |香港|香港|已开服|http://service.cn-hongkong.maxcompute.aliyun-inc.com/api|http://dt.cn-hongkong.maxcompute.aliyun-inc.com|
    |亚太东南1|新加坡|已开服|http://service.ap-southeast-1.maxcompute.aliyun-inc.com/api|http://dt.ap-southeast-1.maxcompute.aliyun-inc.com|
    |亚太东南2|悉尼|已开服|http://service.ap-southeast-2.maxcompute.aliyun-inc.com/api|http://dt.ap-southeast-2.maxcompute.aliyun-inc.com|
    |亚太东南3|吉隆坡|已开服|http://service.ap-southeast-3.maxcompute.aliyun-inc.com/api|http://dt.ap-southeast-3.maxcompute.aliyun-inc.com|
    |亚太东南5|雅加达|已开服|http://service.ap-southeast-5.maxcompute.aliyun-inc.com/api|http://dt.ap-southeast-5.maxcompute.aliyun-inc.com|
    |亚太东北1|东京|已开服|http://service.ap-northeast-1.maxcompute.aliyun-inc.com/api|http://dt.ap-northeast-1.maxcompute.aliyun-inc.com|
    |欧洲中部1|法兰克福|已开服|http://service.eu-central-1.maxcompute.aliyun-inc.com/api|http://dt.eu-central-1.maxcompute.aliyun-inc.com|
    |美国西部1|硅谷|已开服|http://service.us-west-1.maxcompute.aliyun-inc.com/api|http://dt.us-west-1.maxcompute.aliyun-inc.com|
    |美国东部1|弗吉尼亚|已开服|http://service.us-east-1.maxcompute.aliyun-inc.com/api|http://dt.us-east-1.maxcompute.aliyun-inc.com|
    |亚太南部1|孟买|已开服|http://service.ap-south-1.maxcompute.aliyun-inc.com/api|http://dt.ap-south-1.maxcompute.aliyun-inc.com|
    |中东东部1|迪拜|已开服|http://service.me-east-1.maxcompute.aliyun-inc.com/api|http://dt.me-east-1.maxcompute.aliyun-inc.com|
    |英国|伦敦|已开服|http://service.uk-all.maxcompute.aliyun-inc.com/api|http://dt.uk-all.maxcompute.aliyun-inc.com|


**说明：** 需要配置Endpoint、Tunnel Endpoint的场景：

-   MaxCompute客户端（console）配置，请参见[安装并配置客户端](cn.zh-CN/准备工作/安装并配置客户端.md#)。
-   MaxCompute studio project连接配置，请参见[项目空间连接管理](../../../../cn.zh-CN/工具及下载/MaxCompute Studio/项目空间连接管理.md#)。
-   SDK连接MaxCompute配置，请参见[Java SDK介绍](../../../../cn.zh-CN/SDK参考/Java SDK/Java SDK介绍.md#)和[Python SDK](../../../../cn.zh-CN/SDK参考/Python SDK.md#)连接MaxCompute接口配置。
-   PyODPS创建MaxCompute入口对象、通用配置、数据上传下载配置，请参见[配置选项](../../../../cn.zh-CN/开发/PyODPS/配置选项.md#)。
-   以DataWorks的数据集成脚本模式连接MaxCompute数据源配置和使用DataX开源工具连接MaxCompute数据源，请分别参见[配置MaxCompute数据源](https://help.aliyun.com/knowledge_detail/74280.html)和[导出SQL的运行结果](../../../../cn.zh-CN/最佳实践/SQL/导出SQL的运行结果.md#)。

## 访问原则 {#section_pbt_7py_3s9 .section}

-   对于已开服的Region，您可以通过公网、经典网络、VPC网络方式连接MaxCompute服务。
-   通过配置外网Tunnel Endpoint地址下载数据，价格为0.8元/GB。

