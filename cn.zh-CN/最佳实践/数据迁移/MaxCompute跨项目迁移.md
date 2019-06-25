# MaxCompute跨项目迁移 {#task_828066 .task}

本文为您介绍同Region下不同的MaxCompute项目如何实现配置与数据的迁移。

请您首先完成教程《搭建互联网在线运营分析平台》的全部步骤，详情请参见[业务场景与开发流程](../../../../cn.zh-CN/使用教程/搭建互联网在线运营分析平台/业务场景与开发流程.md#)。

本文使用的被迁移的原始项目为教程《搭建互联网在线运营分析平台》中的bigdata\_DOC项目，您需要再创建一个迁移目标项目，用于存放原始项目的表、资源、配置及数据。

1.   创建迁移目标项目 本文中的MaxCompute项目即DataWorks的工作空间。
    1.   进入[DataWorks工作空间列表](https://workbench.data.aliyun.com/consolenew#/projectlist)，选择区域为**华东1**，单击**创建工作空间**。 

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/669740/156145359750071_zh-CN.png)

    2.   选择**计算引擎服务**为MaxCompute、按量付费。由于原始项目bigdata\_DOC为简单模式，为方便起见，本文中DataWorks工作空间模式也为**简单模式（单环境）**。在简单模式下，DataWorks工作空间与MaxCompute项目一一对应，详情请参见[简单模式和标准模式的区别](../../../../cn.zh-CN/产品简介/简单模式和标准模式的区别.md#)。 

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/669740/156145359750069_zh-CN.png)

        **说明：** 工作空间名称全局唯一，建议您使用易于区分的名称，本例中使用的名称为clone\_test\_doc。

2.   跨项目克隆 您可以通过**跨项目克隆**功能将原始项目bigdata\_DOC的节点配置和资源复制到当前项目，详情请参见[跨项目克隆实践](../../../../cn.zh-CN/使用指南/数据开发/发布管理/跨项目克隆实践.md#)。

    **说明：** 

    -   跨项目克隆无法复制表结构与数据。
    -   跨项目克隆无法复制组合节点，需要您手动创建。
    1.   在单原始的项目bigdata\_DOC中击右上角的**跨项目克隆**，跳转至相应的克隆页面。 

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/669740/156145359750087_zh-CN.png)

    2.   选择**克隆目标工作空间**为clone\_test\_doc，**业务流程**为您需要克隆的业务流程Workshop，勾选所有节点，单击**添加到待克隆**后单击右侧的**待克隆列表**。 

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/669740/156145359750089_zh-CN.png)

    3.   单击**全部克隆**，将选中节点克隆至工作空间clone\_test\_DOC。 

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/669740/156145359850090_zh-CN.png)

    4.   切换到您新建的项目，检查节点是否已完成克隆。 

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/669740/156145359850091_zh-CN.png)

3.   新建数据表 跨项目克隆功能无法克隆您的表结构，因此您需要手动新建表。您可以参见[新建数据表](../../../../cn.zh-CN/使用教程/搭建互联网在线运营分析平台/数据建模与开发/新建数据表.md#)完成表的创建。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/669740/156145359850092_zh-CN.png)

    **说明：** 新建表后请将表提交到生产环境。

4.   数据同步 跨项目克隆功能无法复制原始项目的数据到新项目，因此您需要手动同步数据，本文中仅同步表rpt\_user\_trace\_log的数据。
    1.   新建数据源。 
        1.  在**数据集成**页面单击**同步资源管理** \> **数据源**，选择**MaxCompute**。

            ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/669740/156145359850093_zh-CN.png)

        2.  填写您的**数据源名称**、**ODPS项目名称**、AccessKey等信息，单击**完成**，详情请参见[配置MaxCompute数据源](../../../../cn.zh-CN/使用指南/数据集成/数据源配置/配置MaxCompute数据源.md#)。

            ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/669740/156145359950094_zh-CN.png)

    2.   创建数据同步任务。 
        1.  在您的**数据开发**页面右键您克隆的业务流程**Workshop**下的**数据集成**，单击**新建数据集成节点** \> **数据同步**。

            ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/669740/156145359950095_zh-CN.png)

        2.  编辑您新建的数据同步任务节点，填写参数如下图所示。其中数据源bigdata\_DOC是您的原始项目，数据源**odps\_first**代表您当前的新建项目，表名是您需要同步数据的表rpt\_user\_trace\_log。完成后单击**调度配置**。

            ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/669740/156145359950096_zh-CN.png)

        3.  单击**使用工作空间根节点**后，提交数据同步任务。

            ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/669740/156145359950097_zh-CN.png)

    3.   补数据 
        1.  切换到**运维中心**后，单击**周期任务**。右键您的数据同步任务，单击**补数据** \> **当前节点**。

            ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/669740/156145360050098_zh-CN.png)

        2.  本例中，需要补数据的日期分区为2019年6月11日到17日，您可以直接选择业务日期，进行多个分区的数据同步。完成设置后，单击**确定**。

            ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/669740/156145360050099_zh-CN.png)

        3.  在**任务运维** \> **补数据实例**页面，您可以查看您的补数据实例任务运行状态，待显示**运行成功**则说明完成数据同步。

            ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/669740/156145360050100_zh-CN.png)

    4.   结果验证 您可以在**业务流程** \> **数据开发**中新建**ODPS SQL**类型节点，通过运行`select * from rpt_user_trace_log where dt BETWEEN '20190611' and '20190617';`语句查看数据是否完成同步。

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/669740/156145360050101_zh-CN.png)


