# Graph {#concept_gcz_wcd_wdb .concept}

After creating a MaxCompute project, you can write Graph program and complete the local debugging according to the following steps.

In this example, PageRank.java provided by the plug-in is selected to complete the debugging. Select PageRank.java in **examples**, as follows.

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12154/15577294903218_en-US.png)

Right-click and select **Debug As** \> **ODPS MapReduce|Graph**,as follows.

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12154/15577294903220_en-US.png)

The dialog box appears, and configure it as follows.

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12154/15577294903221_en-US.png)

View the running result,as follows.

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12154/15577294903222_en-US.png)

You can view the computing result on the local,as follows.

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12154/15577294903223_en-US.png)

After the debugging is complete, you can package the program and upload it to MaxCompute as a Jar resource. Then submit Graph job.

**Note:** 

-   For the package process, see [MapReduce Eclipse Plug-in Introduction](intl.en-US/Tools and Downloads/Eclipse Plugins/MapReduce.md).
-   For the structure introduction of local result, see [MapReduce Eclipse Plug-in Introduction](intl.en-US/Tools and Downloads/Eclipse Plugins/MapReduce.md).
-   For the detailed introduction of uploading Jar resource, see Add Resource in [Basic Introduction](../../../../intl.en-US/User Guide/Common commands/Resources operations.md) .
-   For submitting the Graph job, see [Graph Function](../../../../intl.en-US/User Guide/Graph/Function overview.md).

