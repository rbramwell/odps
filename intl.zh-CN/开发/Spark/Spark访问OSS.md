---
keyword: Spark访问OSS
---

# Spark访问OSS

本文为您介绍使用Spark访问OSS时需要的相关配置。

## OSS Endpoint配置

调试时请使用OSS服务所在地域的外网Endpoint，提交集群需替换为VPC内网Endpoint。详情请参见[访问域名和数据中心](/intl.zh-CN/开发指南/访问域名（Endpoint）/访问域名和数据中心.md)。

## OSS访问方式配置

-   以AccessKey ID和AccessKey Secret方式访问OSS。

    ```
    spark.hadoop.fs.oss.accessKeyId = xxxxxx
    spark.hadoop.fs.oss.accessKeySecret = xxxxxx
    spark.hadoop.fs.oss.endpoint = oss-xxxxxx-internal.aliyuncs.com
    ```

-   以StsToken的方式访问OSS。

    以AccessKey ID和AccessKey Secret方式访问OSS，需要明文将AccessKey ID和AccessKey Secret写在配置中，存在一定的安全风险。因此建议您以StsToken的方式访问OSS。

    1.  单击[一键授权](https://ram.console.aliyun.com/?spm=a2c4g.11186623.2.9.3bf06a064lrBYN#/role/authorize?request=%7B%22Requests%22:%20%7B%22request1%22:%20%7B%22RoleName%22:%20%22AliyunODPSDefaultRole%22,%20%22TemplateId%22:%20%22DefaultRole%22%7D%7D,%20%22ReturnUrl%22:%20%22https:%2F%2Fram.console.aliyun.com%2F%22,%20%22Service%22:%20%22ODPS%22%7D)，将当前云账号的OSS资源通过StsToken的方式授权给MaxCompute项目直接访问。

        **说明：** 当MaxCompute的ProjectOwner为OSS云账号时，才可以执行一键授权。

    2.  获取roleArn。
        1.  登录[RAM控制台](https://ram.console.aliyun.com/)。
        2.  在左侧导航栏上，单击**RAM角色管理**。
        3.  在**RAM角色管理**页面，搜索AliyunODPSDefaultRole。
        4.  单击**AliyunODPSDefaultRole**，在**基本信息**区域获取**ARN**。格式为`acs:ram::xxxxxxxxxxxxxxx:role/aliyunodpsdefaultrole`。
    3.  在Spark配置中添加如下内容即可访问OSS资源。

        ```
        # 此配置表明Spark是通过StsToken去访问OSS资源。
        spark.hadoop.fs.oss.credentials.provider=org.apache.hadoop.fs.aliyun.oss.AliyunStsTokenCredentialsProvider
        
        # 此配置是一键授权后产生的一个roleArn。
        spark.hadoop.fs.oss.ststoken.roleArn=acs:ram::xxxxxxxxxxxxxxx:role/aliyunodpsdefaultrole
        
        # 此配置是OSS资源对应的VPC访问Endpoint。
        spark.hadoop.fs.oss.endpoint=oss-cn-hangzhou-internal.aliyuncs.com
        ```


## 网络白名单配置

```
spark.hadoop.odps.cupid.vpc.domain.list = {"regionId":"xxxxxx", "vpcs":[{"zones":[{"urls":[{ "domain":"bucketname.oss-xxxxxx-internal.aliyuncs.com", "port":80} ] }]}]}
```

