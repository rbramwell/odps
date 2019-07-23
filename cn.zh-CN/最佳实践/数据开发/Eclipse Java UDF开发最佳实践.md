# Eclipse Java UDF开发最佳实践 {#concept_zkf_m1v_cgb .concept}

本文将为您介绍如何使用Eclipse开发工具，配合ODPS插件进行Java UDF开发的全流程操作。

## 准备工作 {#section_xxl_q1v_cgb .section}

开始使用Eclipse进行Java UDF开发前，您需要进行如下准备工作：

1.  使用Eclipse安装[ODPS插件](../../../../cn.zh-CN/工具及下载/Eclipse开发插件/安装Eclipse插件.md#)。
2.  创建ODPS Project。
    1.  在Eclipse中选择**File** \> **New** \> **ODPS Project**，输入项目名称，单击**Config ODPS console installation path**，配置[odpscmd客户端](../../../../cn.zh-CN/工具及下载/客户端.md#)安装路径。

        ![配置路径](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/79958/156384817834302_zh-CN.png)

    2.  输入客户端整体安装包的路径后，单击**Apply**，ODPS插件会为您自动解析出客户端Version。

        ![Apply](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/79958/156384817834304_zh-CN.png)

    3.  单击**Finish**，即可完成项目的创建。

## 开发步骤 {#section_ngg_tcv_cgb .section}

1.  在ODPS Project中创建Java UDF。

    1.  在左侧**Package Exploer**中右键单击新建的ODPS Java UDF项目，选择**New** \> **UDF**。

        ![UDF](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/79958/156384817834311_zh-CN.png)

    2.  输入UDF的Package名称（本例中为com.aliyun.example.udf）和Name（本例中为Upper2Lower），单击**Finish**，即可完成UDF的创建。

        ![Finish](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/79958/156384817934316_zh-CN.png)

    完成UDF创建后，您即可看到生成的默认Java代码，请注意不要改变evaluate\(\)方法的名称。

    ![默认代码](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/79958/156384817934317_zh-CN.png)

2.  实现UDF类文件中的evaluate方法。

    将您想要实现的功能代码写到evaluate方法中，且不要改变evaluate\(\)方法的名称。此处以实现大写字母转化为小写字母为例。

    ![示例](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/79958/156384817934318_zh-CN.png)

    ``` {#codeblock_ly6_qfy_w0l .language-json}
    package com.aliyun.example.udf;
    
    import com.aliyun.odps.udf.UDF;
    
    public class Upper2Lower extends UDF {
        public String evaluate(String s) {
            if (s == null) { return null; }
            return s.toLowerCase();
        }
    }
    ```

    代码编写完成后，请及时保存。


## 测试Java UDF {#section_bfw_zgv_cgb .section}

为了测试Java UDF代码，您可以首先在MaxCompute上存放一些大写字母作为输入数据。您可以利用odpscmd客户端使用SQL语句`create table upperABC(upper string);`新建一个名为upperABC的测试表格。

![测试表格](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/79958/156384817934320_zh-CN.png)

使用SQL语句`insert into upperABC values('ALIYUN');`在表格中插入测试用的大写字母“ALIYUN”。

完成测试数据准备后，单击**Run** \> **Run Configurations**配置测试参数。

![配置测试参数](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/79958/156384817934322_zh-CN.png)

配置测试参数：**Project**一栏中填写我们创建的Java ODPS Project名称，**Select ODPS project**中填写您的MaxCompute项目名称（请注意与odpscmd客户端当前连接的MaxCompute项目保持一致），**Table**一栏填写我们刚才创建的测试表格名称。完成配置后点击**Run**进行测试。

![RUN](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/79958/156384818034324_zh-CN.png)

您可以在下方的**Console**中查看测试结果。

![查看测试结果](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/79958/156384818034326_zh-CN.png)

**说明：** 测试结果只是Eclipse获取表格中的数据后在本地转换的结果，并不代表MaxCompute中的数据已经转换为小写的aliyun了。

## 使用Java UDF {#section_url_sqv_cgb .section}

确定测试结果正确后，可以开始正式使用Java UDF了，操作步骤如下。

1.  导出Jar包

    在左侧新建的ODPS Project上右键单击，选择**Export**。

    ![Export](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/79958/156384818034328_zh-CN.png)

    在弹框中选择**JAR file**，单击**Next**。

    ![Next](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/79958/156384818034329_zh-CN.png)

    在对话框中JAR file处填写Jar包名称，单击**Finish**即可导出至当前workspace目录下。

    ![finish](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/79958/156384818034330_zh-CN.png)

2.  使用DataWorks引用Jar包

    登录DataWorks控制台，进入同一个项目（本例中为项目MaxCompute\_DOC）的[数据开发](../../../../cn.zh-CN/使用指南/数据开发/界面功能/界面功能点介绍.md#)页面。选择**业务流程** \> **资源** \> **新建资源** \> **JAR**，新建一个[JAR类型资源](../../../../cn.zh-CN/使用指南/数据开发/业务流程/资源.md#ul_u5d_411_t2b)。

    ![新建资源](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/79958/156384818134331_zh-CN.png)

    在弹窗中上传您刚导出的JAR资源。

    ![上传资源](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/79958/156384818134334_zh-CN.png)

    刚才只是将Jar资源上传到DataWorks，接下来您需要单击进入JAR资源，单击**提交并解锁（提交）**按钮将资源上传至MaxCompute。

    ![上传资源](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/79958/156384818134335_zh-CN.png)

    完成上传后，您可以在odpscmd客户端使用list resources命令查看您上传的JAR资源。

3.  创建资源函数

    现在Jar资源已经存在于您的MaxCompute项目中了，接下来您需要单击**业务流程** \> **函数** \> **新建函数**，新建一个Jar资源对应的[函数](../../../../cn.zh-CN/使用指南/数据开发/业务流程/注册函数.md#)，本例中命名函数名称为upperlower\_Java。完成后，依次单击**保存**和**提交并解锁（提交）**

    ![保存](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/79958/156384818134337_zh-CN.png)

    完成提交后，您可以在odpscmd客户端使用list functions命令查看已注册的函数。至此，您使用Eclipse开发工具完成注册的Java UDF函数upperlower\_Java已经可用了。


## 使用Java UDF结果验证 {#section_pkt_mvv_cgb .section}

打开您的odpscmd命令行界面，运行`select upperlower_Java('ABCD') from dual;`命令，可以观察到该Java UDF已经可以转换字母的大小写了，函数运行正常。

![结果验证](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/79958/156384818134338_zh-CN.png)

## 更多信息 {#section_uzm_jnb_pgb .section}

更多Java UDF开发示例请参见[Java UDF](../../../../cn.zh-CN/开发/SQL及函数/UDF/Java UDF.md#)。

如果您要使用IntelliJ IDEA开发工具完成完整的Java UDF开发过程，请参见[IntelliJ IDEA Java UDF开发最佳实践](cn.zh-CN/最佳实践/数据开发/IntelliJ IDEA Java UDF开发最佳实践.md#)。

