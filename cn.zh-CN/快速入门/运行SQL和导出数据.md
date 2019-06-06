# 运行SQL和导出数据 {#concept_tgf_v5y_5db .concept}

在您的数据导入到MaxCompute后，即可在MaxCompute上运行SQL来处理数据。

您可以选择在[MaxCompute客户端](../../../../cn.zh-CN/准备工作/安装并配置客户端.md#)或[DataWorks](../../../../cn.zh-CN/使用指南/数据开发/节点类型/ODPS SQL节点.md#)上运行SQL语句，本文为您演示如何使用客户端完成SQL查询、写入等过程。

MaxCompute当前支持的SQL语法：

-   支持各类运算符。
-   通过DDL语句对表、分区以及视图进行管理。
-   通过Select语句查询表中的记录，通过Where语句过滤表中的记录。
-   通过Insert语句插入数据、更新数据。
-   通过等值连接Join操作，支持两张表的关联，并支持多张小表的Mapjoin。
-   支持通过内置函数和自定义函数来进行计算。
-   支持正则表达式。

**说明：** 

-   MaxCompute SQL不支持事务、索引及Update/Delete等操作，同时MaxCompute的SQL语法与Oracle，MySQL有一定差别，您无法将其他数据库中的SQL语句无缝迁移到MaxCompute上来，更多差异请参见[与其他SQL语法的差异](../../../../cn.zh-CN/开发/SQL及函数/与其他SQL语法的差异.md#)。
-   关于SQL操作的详细示例，请参见[SQL模块](../../../../cn.zh-CN/开发/SQL及函数/SQL概述.md#)。
-   在使用方式上，MaxCompute作业提交后会有几十秒到数分钟不等的排队调度，所以适合处理跑批作业，一次作业批量处理海量数据，不适合直接对接需要每秒处理几千至数万笔事务的前台业务系统。作业的优化请参见[SQL优化示例](../../../../cn.zh-CN/最佳实践/计算优化/SQL优化示例.md#)。
-   MaxCompute SQL的更多限制请参见[SQL限制项](../../../../cn.zh-CN/开发/SQL及函数/SQL限制项.md#)。

## 提取和分析数据 {#section_ynz_3mq_kgb .section}

用SQL代码查询不同学历的单身人士贷款买房的数量，并将结果保存下到result\_table以便分析或展现。

``` {#codeblock_rtl_jp2_0ih .language-sql}
INSERT OVERWRITE TABLE result_table  --数据插入到result_table中。
SELECT education,COUNT(marital) AS num
FROM bank_data
WHERE housing = 'yes'
    AND marital = 'single'
GROUP BY education;
```

您可以使用`select * from result_table;`查看result\_table表中的数据，如下图所示，可以看到当前受到各阶段教育的单身人士数量。

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/11952/155979766637072_zh-CN.png)

上述过程仅仅是一个最简单的数据加工举例，您在实际应用的过程中，可能需要使用多个SQL对多个表进行加工操作。推荐您使用DataWorks完成复杂的数据加工[业务流程](../../../../cn.zh-CN/使用指南/数据开发/业务流程/业务流程介绍.md#)。

## 导出数据 {#section_jvd_2j4_jgb .section}

您在完成SQL语句处理后，可以参考[上传下载命令](../../../../cn.zh-CN/开发/数据上传下载/上传下载命令.md#)将处理完的数据导出到本地。本例中，将result\_table中数据导出到本地D盘。

``` {#codeblock_n56_2l1_a9j}
tunnel download result_table D:\result.txt;
```

导出成功后如下图所示，可以看到**download OK**字样。

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/11952/155979766637074_zh-CN.png)

**说明：** 如果您需要将数据导出到MySQL或其他数据源，推荐您使用[数据集成](../../../../cn.zh-CN/使用指南/数据集成/数据集成简介/数据集成概述.md#)。

