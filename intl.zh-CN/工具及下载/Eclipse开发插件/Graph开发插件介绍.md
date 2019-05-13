# Graph开发插件介绍 {#concept_gcz_wcd_wdb .concept}

创建ODPS项目后，用户可以编写自己的Graph程序，参照下文步骤操作完成本地调试。

在此示例中，我们选用插件提供的PageRank.java来完成本地调试工作。选中**examples**下的PageRank.java文件，如下图。

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12154/15577294843218_zh-CN.png)

右键单击，选择**Debug As** \> **ODPS MapReduce|Graph**，如下图。

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12154/15577294853220_zh-CN.png)

单击后出现对话框，作如下配置。

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12154/15577294853221_zh-CN.png)

查看作业运行结果，如下图。

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12154/15577294853222_zh-CN.png)

可以查看在本地的计算结果，如下图。

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12154/15577294853223_zh-CN.png)

调试通过后，用户可以将程序打包，并以Jar资源的形式上传到ODPS，并提交Graph作业。

**说明：** 

-   程序打包过程以及有关本地结果的目录结构介绍，请参见[MapReduce开发插件介绍](intl.zh-CN/工具及下载/Eclipse开发插件/MapReduce开发插件介绍.md#)。
-   关于上传Jar资源的详细介绍，请参见[资源操作](../../../../intl.zh-CN/用户指南/常用命令/资源操作.md#)中创建Jar资源部分。
-   有关提交Graph作业的操作，请参见[Graph功能介绍](../../../../intl.zh-CN/用户指南/图模型/功能概述.md#)中Jar命令的介绍。

