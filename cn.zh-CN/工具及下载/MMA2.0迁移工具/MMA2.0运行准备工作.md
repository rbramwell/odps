---
keyword: [MMA2.0运行环境, 数据准备]
---

# MMA2.0运行准备工作

本文为您介绍MMA2.0运行前的环境准备和迁移数据预处理。

## 准备运行环境

-   下载与Hive版本对应的MMA工具。下载方式请[提工单](https://selfservice.console.aliyun.com/ticket/createIndex)获取。
-   MMA所服务器上需要安装JDK1.8及以上版本的Java。
-   安装Beeline客户端。
-   确认MaxCompute所在地域并获取该地域的Endpoint，详情请参见[配置Endpoint](/cn.zh-CN/准备工作/配置Endpoint.md)。
-   获取Hive Metastore URI。

    **说明：** 在hive-site.xml中查找`"hive.metastore.uris"`即可获取Hive Metastore URI。

-   获取Hive JDBC连接信息。Hive JDBC的格式为`jdbc:hive2://localhost:10000/default`。
-   确保Hive集群和MMA所在机器与MaxCompute服务所在地域保持网络连通。

    **说明：** 专线场景路由配置说明

    例如，本地IDC通过专线访问MaxCompute的Endpoint，需要在边界路由器（VBR）中将100.64.0.0/10网段的路由条目指向VPC方向的路由器接口，并在本地数据中心的网关设备上将100.64.0.0/10网段的路由指向VBR的阿里云侧互联IP，详情请参见[本地IDC通过专线访问云服务器ECS](/cn.zh-CN/最佳实践/本地IDC通过专线访问云服务器ECS.md)。

-   确认Hive Metastore是否有安全配置。

    请准备一个有权限访问Hive Metastore服务和执行Hive SQL的用户在hive-site.xml中查看`"hive.security.authorization.enabled"`的值。如果值为True，需要配置安全信息，详情请参见[基于Kerberos身份认证的配置](#section_sq4_8rb_z9s)。


## 预处理待迁移数据

您可以通过如下方法对待迁移数据进行预处理，可以提升迁移效率、提升数据进入MaxCompute后的查询效率以及提前发现并解决MaxCompute与Hive的不兼容问题。

-   分区合并

    尽可能减少分区数，可以加速迁移。例如，7 TB非分区表迁移用时15分钟，而30 GB、3万分区表迁移用时为1小时。

-   类型转换

    -   MaxCompute在数据类型上与Hive存在不完全兼容的情况，例如，STRING类型不支持超8 MB。
    -   MMA会自动进行类型转换。例如，Hive上的DATE类型分区字段，在MaxCompute中会自动转换成STRING类型分区字段。
    **说明：** MMA默认会打开新类型，即`set odps.sql.type.system.odps2=true;` ，以2.0新类型来创建表，详情请参见[数据类型版本说明](/cn.zh-CN/开发/数据类型/数据类型版本说明.md)。

-   使用[闪电立方](/cn.zh-CN/工具及下载/MMA2.0迁移工具/MMA2.0迁移概述.md)从HDFS上传数据到OSS时，存储路径格式为oss://bucket\_name/database\_name/table\_name/partition\_name/ 。

    **说明：** MMA2.0默认以2.0新数据类型创建表（即`set odps.sql.type.system.odps2=true;`）。详情请参见[2.0数据类型版本](/cn.zh-CN/开发/数据类型/2.0数据类型版本.md)。


## 基于Kerberos身份认证的配置

在mma\_server\_config.json和mma\_client\_config.json文件中添加krbPrincipal、keyTab、krbSystemProperties配置信息，如下所示。

```
{
  "dataSource": "Hive",
  "hiveConfig": {
    "jdbcConnectionUrl": "jdbc:hive2://127.0.0.1:10000/default",
    "user": "Hive",
    "password": "",
    "hmsThriftAddr": "thrift://127.0.0.1:9083",
    "krbPrincipal": "xxx",
    "keyTab": "xxx",
    "krbSystemProperties": "xxx=xxx,xxx=xxx",
    "hiveJdbcExtraSettings": [
      "hive.fetch.task.conversion=none",
      "hive.execution.engine=mr",
      "mapreduce.job.name=data-carrier",
      "mapreduce.max.split.size=512000000",
      "mapreduce.task.timeout=3600000",
      "mapreduce.map.maxattempts=0",
      "mapred.map.tasks.speculative.execution=false"
    ]
  },
  "odpsConfig": {
    "accessId": "your_access_id",
    "accessKey": "your_access_key",
    "endpoint": "your_endpoint",
    "projectName": "your_project_name"
  }
}
```

