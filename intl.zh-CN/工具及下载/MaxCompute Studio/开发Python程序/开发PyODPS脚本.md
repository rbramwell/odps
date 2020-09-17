# 开发PyODPS脚本

本文为您介绍如何开发PyODPS脚本。

已配置Python开发环境，详情请参见[配置Python开发环境](/intl.zh-CN/工具及下载/MaxCompute Studio/开发Python程序/配置Python开发环境.md)。

PyODPS是MaxCompute Python版本的SDK，提供对MaxCompute对象的基本操作和DataFrame框架。您可以通过PyODPS在MaxCompute上分析数据。

1.  在Project区域右键单击**scripts**，选择**New** \> **MaxCompute Python**。

    ![MaxCompute Python](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/5545819951/p3447.png)

2.  在**Create new MaxCompute python class**对话框，填写**Name**，从**Kind**列表中选择**PyODPS Script**。

    ![Create new MaxCompute python class](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/5545819951/p3452.png)

3.  新建MaxCompute PyODPS脚本后，PyODPS脚本模板会通过PyODPS Room自动初始化`odps`和`o`两个对象。

    通过DataWorks开发PyODPS脚本时，系统会自动创建Room。通过Intellij IDEA开发PyODPS脚本时，需要创建Room，详情请参见[PyODPS文档](https://pyodps.readthedocs.io/zh_CN/latest/)。

    ![模板](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/5545819951/p3455.png)


