---
keyword: [Logview 2.0, 作业运行信息]
---

# 使用Logview 2.0查看Job运行信息

本文为您介绍Logview 2.0的入口及功能。您可以通过Logview 2.0 URL查看作业运行信息。

## 概述

Logview 2.0在Logview基础上，进行了全新的页面设计，优化了加载速度，并新增了如下功能：

-   支持以交互式DAG图展示作业处理逻辑架构，您还可以查看相应的Operation层级。
-   支持回放作业运行过程。
-   支持通过Fuxi Sensor查看内存及CPU使用情况。

## 入口

您使用MaxCompute客户端（odpscmd）提交作业后，系统会生成Logview URL。在搜索引擎中打开Logview URL界面，单击**体验Logview 2.0**即可进入Logview 2.0界面。

![URL](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/8039459951/p161396.png)

Logview 2.0界面如下。

![URL界面](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/7653089951/p161406.png)

|序号|区域|
|--|--|
|①|标题与功能区，详情请参见[标题和功能区](#section_rok_7rf_7dw)。|
|②|Basic Info，详情请参见[Basic Info](#section_cpk_fnd_egg)。|
|③|作业详情，详情请参见[作业详情](#section_j30_f4e_sej)。|

## 标题和功能区

标题与功能区包含您提交MaxCompute作业时生成的唯一作业ID及您自定义的作业名称（通过SDK方式提交的作业包含作业名称信息）。您还可以执行如下操作。

|图标|功能|
|--|--|
|![打开](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/9039459951/p161417.png)|打开本地保存的作业详情文件Logview\_detail.txt。|
|![返回](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/9039459951/p161420.png)|返回Logview旧版界面。|
|![保存](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/9039459951/p161422.png)|将作业详情文件保存至本地设备。|

## Basic Info

Basic Info区域展示的作业基本信息如下。

|参数名|描述|
|---|--|
|MaxCompute Service|作业使用的MaxCompute服务的Endpoint。Endpoint详情请参见[配置Endpoint](/cn.zh-CN/准备工作/配置Endpoint.md)。|
|Project|作业所属的MaxCompute项目名称。|
|Cloud account|提交作业的阿里云账号信息。|
|Type|作业的类型。例如SQL、SQLRT、LOT、XLib、CUPID、AlgoTask和Graph。|
|Status|作业的状态。状态取值如下：-   Success：作业执行成功。
-   Failed：作业执行失败。
-   Canceled：作业执行取消。
-   Waiting：作业正在MaxCompute中处理，并没有提交至Fuxi中运行。
-   Running：作业正在Fuxi中处理。
-   Terminated：作业已执行结束。 |
|Start Time|作业提交时间。|
|End Time|作业执行结束时间。|
|Latency|作业执行消耗的时长。|
|Progress|作业执行进度。|
|Priority|作业优先级。|
|Queue|作业在资源配额组内的排队位置。|

## 作业详情

您可以通过作业详情区域全方位了解作业，作业详情区域包含如下功能区：

-   **Job Details**
    -   作业执行图

        **Job Detail**页签的上半部分为作业执行图。执行图以可视化方式展示三个维度的子任务依赖关系：Fuxi Job层、Fuxi Task层和Operation层，同时提供一系列辅助工具，为排查问题提供帮助。界面构成如下。

        ![作业执行图](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/9039459951/p161495.png)

        |序号|描述|
        |--|--|
        |①|面包屑导航，用于切换Fuxi Job。JOB:\_SQL\_0\_0\_0\_job\_0为Fuxi Job名称。|
        |②|排查问题辅助工具。包含Progress Chart、Input Heat Chart、Output Heat Chart、TaskTime Heart Chart和InstanceTime Heart Chart。|
        |③|您可以在此区域刷新作业执行状态（![刷新](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/9039459951/p161505.png)）、全屏或缩放显示作业执行图（![全屏](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/9039459951/p161507.png)）、获取MaxCompute Studio文档（![帮助](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/9039459951/p161508.png)）和切换至任务上一层级（![切换层级](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/9039459951/p161509.png)）。|
        |④|缩放辅助工具。|
        |⑤|Fuxi Task。一个MaxCompute作业由一个或多个Fuxi Job组成。每个Fuxi Job由一个或多个Fuxi Task组成。每个Fuxi Task由一个或者多个Fuxi Instance组成，当用户的输入数据量变大时，MaxCompute会在每个Task启动更多的节点来处理数据，每个节点就是一个Fuxi Instance。例如，简单的MapReduce通常会产生两个Fuxi Task，一个是Map一个是Reduce，两个Fuxi Task的名称分别为M1和R2，当SQL比较复杂时，可能会产生多个Fuxi Task。您可以在执行界面上看到每个Fuxi Task的名称。例如M1，表示一个Map Task；R4\_3\_9的3、9表示它依赖M3、C9\_3执行结束才能开始执行。同理，M2\_4\_9\_10\_16表示M2要依赖R4\_3\_9、C9\_3、R10\_1\_16、C16\_1四个Task执行结束后才能开始执行。R/W表示Task读取和写的行数。

单击某个Task或在Task上单击右键可以查看Task对应的Operator算子依赖关系及全部Operator算子图。

Logview 2.0增加了表依赖关系，您可以快速查看输入和输出表。 |
        |⑥|Fuxi Task回放。单击![播放](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/9039459951/p161512.png)即可开始播放，再次单击则暂停。您也可以手动拖动进度条。进度条左边为开始时间，中间为播放时间，右边为结束时间。|
        |⑦|鹰眼。|

        **说明：**

        -   不支持回放Running状态的Fuxi Task。
        -   AlgoTask类型的作业（例如PAI机器学习），由于只有一个Fuxi Task，故不提供作业执行图。
        -   非SQL类型作业，仅能展示Fuxi Job和Fuxi Task层，不支持展示Operation层。
        -   如果只有一个Fuxi Job，作业执行图默认展示Fuxi Task层依赖关系；否则，默认展示Fuxi Job层依赖关系。
    -   作业运行情况

        **Job Detail**页签的下半部分为作业运行详细信息。界面构成如下。

        ![运行结果](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/9039459951/p161547.png)

        |序号|描述|
        |--|--|
        |①|Fuxi Job页签，您可以在此切换Fuxi Job。|
        |②|Fuxi Job对应的Fuxi Task详情。在某个Fuxi Job下，单击任一Fuxi Task，在下方会展开该Fuxi Task对应的Fuxi Instance信息。默认展开第一个Fuxi Job的第一个Fuxi Task的Fuxi Instance信息。|
        |③|Logview为不同阶段的Instance进行了分组。您可以单击Failed快速查看出错的节点。|
        |④|Fuxi Sensor。仅AlgoTask类型作业提供该功能，例如PAI。您可以查看Fuxi Instance的CPU及内存信息。Fuxi Sensor详情请参见[Fuxi Sensor](#section_2ue_j53_syz)。**说明：** Fuxi Sensor功能在西南1（成都）、华南1（深圳）、华东2（上海）、华东1（杭州）、华北3（张家口）和华北2（北京）区域已开放。 |
        |⑤|StdOut和StdErr。您可以查看输出和错误信息，您自己打印的信息也可以在这里查看，同时提供下载功能。|
        |⑥|Debug。调测并排除错误。|

-   **Result**

    Result页签用于展示作业运行结果，如果作业执行失败，系统默认会打开该页签展示失败原因。系统会根据MaxCompute设置的返回格式，显示为表格或纯文本，您可以通过MaxCompute客户端配置odps.sql.select.output.format参数设置显示格式，如果取值为HumanReadable表示纯文本，否则默认是CSV格式。

    -   表格

        ![Result](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/0139459951/p161841.png)

        您可以通过该界面执行如下操作：

        -   选中某一行可以导出该行数据。
        -   选中标题栏可以导出当前界面的数据。
        -   在标题栏单击![下拉箭头](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/8653089951/p165233.png)，选择**Select All Data**或**Deselect All**可以全选或取消全选全量数据。
        -   单击**export**即可导出CSV格式的数据。CSV仅仅为文件后缀，您可以使用文本文件或Sublime Text工具打开。
    -   纯文本

        ![Result](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/0139459951/p161843.png)

-   **SourceXML**

    您提交至MaxCompute的作业的SourceXML信息。

-   **SQL Script**

    您提交至MaxCompute的作业的SQL脚本信息。

-   **Summary**

    您提交至MaxCompute的作业运行概要信息。

-   **Json Summary**

    以JSON格式显示的作业运行概要信息。

-   **History**

    如果Fuxi Instance重新执行就会产生History信息。

-   **Substatus History**

    您可以查看作业执行的详细历史状态，包含状态码、状态描述、开始时间、持续时间和结束时间。


## Fuxi Sensor

Fuxi Sensor是展示MaxCompute作业全维度的资源视图。您可以通过Fuxi Sensor查看Fuxi Instance的CPU及内存消耗量。Fuxi Sensor是作业问题定位以及作业运行质量分析的重要工具。例如以下场景的作业问题可借助Fuxi Sensor进行分析：

-   作业内存溢出时，分析实际使用的内存量。
-   对比作业申请的资源量与实际使用资源量，优化资源申请。

例如，您可以通过Fuxi Sensor查看机器学习作业Fuxi Instance的以下资源使用情况：

-   CPU使用量

    CPU指标包括两条线，一条显示申请的CPU量（cpu\_plan），另一条显示实际的CPU使用量（cpu\_usage）。纵坐标中的400，表示4个Processor。

    ![CPU](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/8653089951/p161866.png)

-   内存使用量

    内存指标包括两条线，一条显示申请的内存量（mem\_plan），另一条显示实际的内存使用量（mem\_usage）。

    内存使用量（mem\_usage）包括两部分信息RSS（Resident Set Size）和PageCache。RSS是Malloc（非文件映射）发生缺页之后的内存，该部分内存，在内存紧张时无法被回收。PageCache是内核缓存文件占用的内存，例如读写日志文件，PageCache在内存紧张时通常是可以被回收。

    -   内存详情

        ![内存](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/9653089951/p161869.png)

    -   RSS内存使用量

        ![RSS](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/0139459951/p161943.png)

    -   PageCache内存使用量

        ![Page-cache](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/0139459951/p161945.png)


