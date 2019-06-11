# 开发MapReduce {#concept_nvh_s2c_5db .concept}

创建完成[MaxCompute Java Module](cn.zh-CN/工具及下载/MaxCompute Studio/开发Java程序/创建MaxCompute Java Module.md)后，即可以开始开发[MR](../../../../cn.zh-CN/开发/MapReduce/概要/MapReduce概述.md)了。

## 开发MR {#section_gr4_mgg_vdb .section}

1.  在Module的源码目录即**src** \> **main**上右键**new** \> **java**，选择**MaxCompute Java**。
2.  分别创建Driver、Mapper、Reducer。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12131/15602429901997_zh-CN.png)

3.  模板已自动填充框架代码，只需要设置输入/输出表，Mapper/Reducer类等即可。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12131/15602429901998_zh-CN.png)


开发MR详情请参见[编写MapReduce（可选）](../../../../cn.zh-CN/快速入门/编写MapReduce（可选）.md#)。

## 调试MR {#section_jr4_mgg_vdb .section}

MR开发好后，下一步就是要测试自己的代码，看是否符合预期，目前支持以下两种方式：

-   单元测试：在examples目录下有WordCount的单测实例，可参考例子编写自己的UT。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12131/15602429901999_zh-CN.png)

-   本地运行MR：本地运行时，需要指定运行数据源，有两种方式设定测试数据源：
    -   Studio通过Tunnel服务自动下载指定MaxCompute Project的表数据到warehouse目录下。默认下载100条，如需更多数据测试，请自行使用Console的Tunnel命令或者Studio的表下载功能。
    -   提供mock项目（example\_project）及表数据，用户可参考warehouse下example\_project自行设置。
    1.  运行MR：在Driver类上右键，点击**运行**菜单，弹出run configuration对话框，配置MR需要在哪个MaxCompute Project上运行即可。

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12131/15602429912001_zh-CN.png)

    2.  单击**OK**，如果指定MaxCompute Project的表数据未被下载到warehouse中，则首先下载数据；如果采用mock项目或已被下载则跳过。MR Local Run框架会读取warehouse中指定表的数据作为MR的输入，开始本地运行MR。用户可以在控制台看到日志输出和结果打印。

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12131/15602429912002_zh-CN.png)


## 生产运行MR {#section_or4_mgg_vdb .section}

本地调试通过后，把MR发布到服务端，在MaxCompute分布式环境下运行MR。

1.  首先，将自己的MR程序打成jar包，并发布到服务端。详细操作步骤请参见[打包、上传和注册](cn.zh-CN/工具及下载/MaxCompute Studio/开发Java程序/打包、上传和注册.md#)。
2.  通过MaxCompute Console（无缝集成于Studio），打开Project Explorer Window，右键单击**project**，选择**Open in Console**，可在Console命令行中输入类似如下的jar命令。

    ``` {#codeblock_wam_6n0_zvr}
    jar-libjars wordcount.jar -classpath D:\odps\clt\wordcount.jar com.aliyun.odps.examples.mr.WordCount wc_in wc_out;
    ```

    详细命令输入参请见[jar命令](../../../../cn.zh-CN/开发/MapReduce/功能介绍/作业提交.md)。


