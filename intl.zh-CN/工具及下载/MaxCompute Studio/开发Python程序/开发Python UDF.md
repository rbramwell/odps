---
keyword: [开发Python UDF, 测试Python UDF, 发布Python UDF]
---

# 开发Python UDF

MaxCompute Studio支持Python UDF开发，本文为您介绍如何开发、测试和注册发布Python UDF。

您必须完成以下操作：

-   [管理项目连接](/intl.zh-CN/工具及下载/MaxCompute Studio/项目空间连接管理.md)
-   [配置Python开发环境](/intl.zh-CN/工具及下载/MaxCompute Studio/开发Python程序/配置Python开发环境.md)

## 开发Python UDF

1.  在**Project**区域**MaxCompute Studio**目录下，右键单击**scripts**，选择**New** \> **MaxCompute Python**。

2.  在**Create new MaxCompute python class**对话框中输入类名**Name**，选择类型为**Python UDF**，单击**OK**完成。

    ![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/4545819951/p3402.png)

3.  在编辑框中编写UDF代码。

    ![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/4545819951/p3403.png)


## 测试UDF

UDF开发完成后，需要测试代码是否符合预期。MaxCompute Studio支持本地测试，即下载表的部分示例数据在本地运行并进行调试。

1.  右键单击已经编辑完成的Python UDF脚本，选择**RUN**。

2.  在**Edit configuration**页面，配置相关参数，单击**OK**。

    ![**](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/4545819951/p96067.png)

    -   **MaxCompute project**：UDF运行使用的MaxCompute空间。本地运行时选择**local**。
    -   **MaxCompute table**：UDF运行时需要使用的MaxCompute表的名称。
    -   **Table columns**：UDF运行时需要使用的MaxCompute表的列信息。
    -   **Download Record limit**：下载数据记录限制。默认为100条。
    **说明：**

    -   如果已经下载数据，则不会再次重复下载。如果需要再次下载，请在MaxCompute客户端使用Tunnel命令下载数据。
    -   默认下载100条数据，如果需要更多数据测试，请在MaxCompute客户端使用Tunnel命令或者MaxCompute Studio的表下载功能下载数据。
    -   下载完成后，您可以在warehouse目录下该表的data文件中看到下载的示例数据。
3.  本地运行框架会根据您指定的列，获取data文件中指定列的数据，调用UDF本地运行。

    **说明：** 本地运行是通过PyODPS的pyou脚本实现的，命令为`pyou hello.Plus<data`。安装完PyODPS后可以使用相应的命令检查该脚本是否存在。

    -   如果您是Windows系统，请运行`${python}/../Scripts/pyou`命令。
    -   如果您是mac OS系统，请运行`${python}/../pyou`命令。
4.  您可以在控制台查看打印结果。

    ![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/4545819951/p3412.png)


## 发布Python UDF

Python UDF测试通过后，即可发布到生产环境上使用。详情请参见[打包、上传和注册](/intl.zh-CN/工具及下载/MaxCompute Studio/开发Java程序/打包、上传和注册.md)。

