# 计算计费项（按CU预付费） {#concept_ihl_cvc_5db .concept}

本文为您介绍MaxCompute的按CU预付费的计算计费说明。

## 计算计费分类 {#section_zw5_hvc_5db .section}

MaxCompute分为以下两种计算计费方式。

-   按CU预付费方式：即您提前预定一部分资源，先付费后使用。按CU预付费方式仅[大数据计算服务MaxCompute](https://www.aliyun.com/product/odps)中提供。
-   [按量后付费方式](cn.zh-CN/产品定价/计算计费项（按量付费）.md#)：即以作业的消耗作为计量指标，在作业执行后收取费用。

    目前MaxCompute开放的计算任务类型有SQL、UDF、MapReduce、Graph、Lightning、Spark、[机器学习](https://data.aliyun.com/product/learn)等。其中，SQL（不包括UDF）、MapReduce计算任务已经收费；New SQL（MaxCompute2.0）任务在2018年5月底启动收费；从2018年10月31日开始，MaxCompute SQL外表功能开始计费；从2019年2月1日开始Lightning及Spark任务收费；其他类型暂无收费计划。

    **说明：** 

    -   有关UDF、Graph及机器学习的收费请关注阿里云相关公告。 PyODPS任务底层执行的是SQL任务，费用参考SQL的计量计费逻辑。
    -   **标准版**与**开发者版**的SQL计费方式不同，**标准版**即过去您购买的MaxCompute资源版本。

## 按CU预付费 {#section_rx5_hvc_5db .section}

按CU预付费的方式仅[大数据计算服务MaxCompute](https://common-buy.aliyun.com/?commodityCode=odpsplus#/buy)提供，包括SQL、MapReduce、Spark等类型的计算任务。您可以预先购买一部分资源，MaxCompute会为您预留您所购买的资源。

|资源定义|内存|CPU|售价|
|:---|:-|:--|:-|
|1CU|4GB|1CPU|150元/月|

如果您是新用户，建议您先采用按I/O后付费的方式进行结算。您初期使用MaxCompute时，消耗的资源较少，采购CU预留资源会出现资源闲置。相对而言，按I/O后付费方式成本会更低。

如果您的任务量较大，您可以考虑对于消耗资源较少的任务采取预付费，资源较大的任务采取按I/O后付费，这样可以保证任务运行时一直有CU资源。

**说明：** 

预付费10CU起购。您可以考虑最少购买50CU，以充分发挥MaxCompute集群性能。

当预付费购买60CU或以上，可以通过MaxCompute预付费资源监控工具-CU管家进行资源监控管理，详情请参见[MaxCompute预付费资源监控工具-CU管家](../../../../cn.zh-CN/使用指南/MaxCompute管家/MaxCompute预付费资源监控工具-CU管家.md#)。

