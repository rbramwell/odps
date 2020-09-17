---
keyword: configure Spark on MaxCompute to access OSS resources
---

# Configure Spark on MaxCompute to access OSS resources

This topic describes how to configure Spark on MaxCompute to access Object Storage Service \(OSS\) resources.

## Configure the OSS endpoint

Use the public endpoint of OSS in the target region when you debug features. Use the OSS endpoint in the virtual private cloud \(VPC\) if MaxCompute is deployed in a production environment. For more information, see [Regions and endpoints](/intl.en-US/Developer Guide/Endpoint/Regions and endpoints.md).

## Configure the OSS access method

-   Access OSS by using the AccessKey ID and AccessKey secret of an account.

    ```
    spark.hadoop.fs.oss.accessKeyId = xxxxxx
    spark.hadoop.fs.oss.accessKeySecret = xxxxxx
    spark.hadoop.fs.oss.endpoint = oss-xxxxxx-internal.aliyuncs.com
    ```

-   Access OSS by using a Security Token Service \(STS\) token.

    Spark on MaxCompute can access OSS by using the AccessKey ID and AccessKey secret of an account. In this case, you must write the plaintext AccessKey ID and AccessKey secret in the configuration, which incurs security risks. We recommend that you configure Spark on MaxCompute to access OSS by using an STS token.

    1.  Go to the [Cloud Resource Access Authorization](https://ram.console.aliyun.com/?spm=a2c4g.11186623.2.9.3bf06a064lrBYN#/role/authorize?request=%7B%22Requests%22:%20%7B%22request1%22:%20%7B%22RoleName%22:%20%22AliyunODPSDefaultRole%22,%20%22TemplateId%22:%20%22DefaultRole%22%7D%7D,%20%22ReturnUrl%22:%20%22https:%2F%2Fram.console.aliyun.com%2F%22,%20%22Service%22:%20%22ODPS%22%7D) page and click Confirm Authorization Policy. Then, the MaxCompute project can access OSS resources of the current Alibaba Cloud account by using an STS token.

        **Note:** You can authorize a MaxCompute project to access OSS resources by using this method only when the owner of the MaxCompute project is an Alibaba Cloud account that owns the OSS resources to be accessed.

    2.  Obtain the Alibaba Cloud Resource Name \(ARN\) of the role that Spark on MaxCompute assumes.
        1.  Log on to the [RAM console](https://ram.console.aliyun.com/).
        2.  In the left-side navigation pane, click **RAM Roles**.
        3.  On the **RAM Roles** page, search for AliyunODPSDefaultRole.
        4.  In the search result, click **AliyunODPSDefaultRole** in the RAM Role Name column. On the page that appears, obtain the value of **ARN** in the **Basic Information** section. The value is in the `acs:ram::xxxxxxxxxxxxxxx:role/aliyunodpsdefaultrole` format.
    3.  Add the following content to the configurations of Spark on MaxCompute:

        ```
        # Configure Spark on MaxCompute to access OSS resources by using an STS token.
        spark.hadoop.fs.oss.credentials.provider=org.apache.hadoop.fs.aliyun.oss.AliyunStsTokenCredentialsProvider
        
        # Configure the ARN of the role that Spark on MaxCompute assumes.
        spark.hadoop.fs.oss.ststoken.roleArn=acs:ram::xxxxxxxxxxxxxxx:role/aliyunodpsdefaultrole
        
        # Configure the OSS endpoint in the VPC.
        spark.hadoop.fs.oss.endpoint=oss-cn-hangzhou-internal.aliyuncs.com
        ```


## Configure a whitelist

```
spark.hadoop.odps.cupid.vpc.domain.list = {"regionId":"xxxxxx", "vpcs":[{"zones":[{"urls":[{ "domain":"bucketname.oss-xxxxxx-internal.aliyuncs.com", "port":80} ] }]}]}
```

