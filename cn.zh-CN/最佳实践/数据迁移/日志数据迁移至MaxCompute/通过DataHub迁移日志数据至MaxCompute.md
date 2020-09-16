---
keyword: [迁移日志数据, DataHub]
---

# 通过DataHub迁移日志数据至MaxCompute

本文为您介绍如何通过DataHub迁移日志数据至MaxCompute。

授权访问MaxCompute的账号已开通以下权限：

-   MaxCompute中项目的CreateInstance权限。
-   MaxCompute中表的查看、修改和更新权限。

授权操作详情请参见[授权](/cn.zh-CN/管理/安全管理详解/用户及授权管理/授权.md)。

DataHub是阿里云流式数据（Streaming Data）的处理平台。数据上传至DataHub后保存在实时表里，后续会在5分钟内通过定时任务的形式同步到MaxCompute离线表里，供离线计算使用。

您只需要创建DataHub Connector，指定相关配置，即可创建将DataHub中流式数据定期归档到MaxCompute的同步任务。

1.  在MaxCompute客户端（odpscmd）执行如下语句创建表，用于存储DataHub同步的日志数据。示例语句如下。

    ```
     CREATE TABLE test(f1 string, f2 string, f3 double) partitioned by (ds string);
    ```

2.  在DataHub上创建项目。

    1.  登录[DataHub控制台](https://datahub.console.aliyun.com/datahub)，在左上角选择地域。

    2.  在左侧导航栏，单击**项目管理**。

    3.  在**项目列表**区域，单击右上角**新建项目**。

    4.  在**新建项目**页面，填写**名称**及**描述**，单击**创建**。

3.  创建Topic。

    1.  在**项目列表**区域，单击项目右侧的**查看**。

    2.  在项目详情页面，单击右上角**新建Topic**，进入**新建Topic**页面，填写配置参数。

        ![新建Topic](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/2993359951/p73849.png)

    3.  单击**下一步**，完善Topic信息。

        **说明：**

        -   **Schema**与MaxCompute表为对应关系，DataHub Topic Schema的字段名称、类型和顺序必须与MaxCompute表字段完全一致，如果三个条件中的任意一个不满足，则Connector创建失败。
        -   支持将**TUPLE**和**BLOB**类型的DataHub Topic同步到MaxCompute表中。
        -   系统默认最多可以创建20个Topic。如果需要创建更多，请提交[工单](https://selfservice.console.aliyun.com/ticket/createIndex)申请。
        -   DataHub Topic的Owner或Creator账号有操作Connector的权限，包括创建和删除等。
4.  向新建的Topic中写入数据。

    1.  单击新建Topic右侧的**查看**。

    2.  在Topic详情页面，单击右上角**同步**。

    3.  在**新建Connector**页面，单击**MaxCompute**，配置相关参数后，单击**创建**。

5.  查看Connector详细信息。

    1.  在左侧导航栏，单击**项目管理**。

    2.  在**项目列表**区域，单击项目右侧的**查看**。

    3.  在**Topic列表**区域，单击Topic右侧的**查看**。

    4.  在Topic详情页面，单击**Connector**页签，查看创建好的Connector。

    5.  单击Connector右侧的**查看**，即可查看Connector详细信息。

        DataHub默认每五分钟或当数据达到60 MB时强行向MaxCompute离线表中投递一次数据。**同步点位**代表已经同步数据的条数。

        ![DataConnector详情](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/2993359951/p73847.png)

6.  执行如下语句测试日志数据是否投递成功。

    ```
    SELECT * FROM test;
    ```

    返回结果如下，表示投递成功。

    ![测试结果](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/2993359951/p73848.png)


