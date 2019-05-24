# Spark常见问题 {#concept_y5c_w5c_kgb .concept}

本文为您介绍使用Spark过程中可能遇到的常见问题。

## 如何将开源Spark代码迁移到Spark on MaxCompute {#section_t1w_jjg_b2s .section}

分以下三种情形：

-   作业无需访问MaxCompute表和OSS。

    用户jar包可直接运行，具体步骤请参见[搭建开发环境](cn.zh-CN/开发/Spark/搭建开发环境.md#)。注意，对于Spark或Hadoop的依赖必须设成provided。

-   作业需要访问MaxCompute表。

    配置相关依赖后重新打包即可。配置依赖的步骤请参见[配置依赖](cn.zh-CN/开发/Spark/搭建开发环境.md#section_oey_mqy_cq1)。

-   作业需要访问OSS。

    配置相关依赖后重新打包即可。配置依赖的步骤请参见[配置依赖](cn.zh-CN/开发/Spark/搭建开发环境.md#section_oey_mqy_cq1)。


## 如何通过Spark访问VPC环境内服务 {#section_l39_8cg_g2o .section}

Spark默认支持对MaxCompute表的访问，支持对OSS、OTS等外部数据源的访问处理。对VPC内的服务访问，目前默认暂不支持访问，如有需求可以通过工单和我们联系。

## spark-defaults.conf提供的id、key错误 {#section_08p_gq5_tqa .section}

``` {#codeblock_485_yyx_pdh .language-java}
Stack:
com.aliyun.odps.OdpsException: ODPS-0410042:
Invalid signature value - User signature dose not match
```

请检查spark-defaults.conf提供的id、key和在阿里云官网管理控制台[用户信息管理](https://account.aliyun.com/login/login.htm?oauth_callback=https://usercenter.console.aliyun.com/)中的**AccessKey ID**、**Access Key Secret**是否一致。

## 提示权限不足 {#section_ybh_d0v_mtc .section}

``` {#codeblock_afm_rvm_a7m .language-java}
Stack:
com.aliyun.odps.OdpsException: ODPS-0420095: 
Access Denied - Authorization Failed [4019], You have NO privilege 'odps:CreateResource' on {acs:odps:*:projects/*}
```

请project owner授权grant resource的read以及create权限。

## 项目未支持Spark任务运行 {#section_4ci_td1_5wn .section}

``` {#codeblock_1m6_c2r_mp3 .language-java}
Exception in thread "main" org.apache.hadoop.yarn.exceptions.YarnException: com.aliyun.odps.OdpsException: ODPS-0420095: Access Denied - The task is not in release range: CUPID
```

首先需要确认项目所在的region中，是否已经提供了MaxCompute Spark服务。同时，检查Spark-defaults.conf配置信息是否与产品文档中的要求一致。如果region已经支持，请通过工单或加入钉钉群：21969532（MaxCompute Spark支持群）进行咨询。

## 运行时报错`No space left on device` {#section_grc_i85_1x2 .section}

Spark使用网盘进行本地存储。Shuffle数据和BlockManager溢出的数据均存储在网盘上。网盘的大小通过参数spark.hadoop.odps.cupid.disk.driver.device\_size控制，默认20g，最大100g。如果调整到100g仍然报出此错误，则需要分析具体原因。常见的原因为数据倾斜，在Shuffle或者Cache过程中数据集中分布在某些Block，此时可以缩小单个Executor的并发（spark.executor.cores），增加Executor的数量（spark.executor.instances）。

