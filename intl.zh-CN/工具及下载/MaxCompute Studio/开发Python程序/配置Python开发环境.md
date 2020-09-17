---
keyword: 配置Python开发环境
---

# 配置Python开发环境

MaxCompute Studio支持您在Intellij IDEA中完成Python开发，例如UDF和PyODPS脚本。本文为您介绍如何配置Python开发环境。

## 安装PyODPS

1.  PyODPS是MaxCompute的PyODPS SDK，详情请参见[安装指南及使用限制]()。

2.  运行Intellij IDEA，在顶部菜单栏上，单击**File** \> **Settings**。

3.  在**Settings**页面左侧导航栏，单击**MaxCompute Studio**。

4.  设置**Python path to resolve UDF**为本地Python安装目录。

    ![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/3545819951/p34672.png)


## 安装Python插件

在Intellij IDEA的插件仓库中搜索Python或者Python Community Edition插件并安装。

1.  在Intellij IDEA顶部菜单栏上，单击**File**，选择**Settings**。

2.  在**Settings**页面左侧导航栏，单击**Plugins**。

3.  在搜索框搜索Python Community Edition，并安装。

    ![**](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/3545819951/p95688.png)


## 配置Python依赖

配置Studio Module对Python的依赖，即可进行MaxCompute Python的开发。

1.  在顶部菜单栏，单击**File** \> **Project Structure**。

2.  增加SDK。

    1.  在左侧导航栏上，单击**SDKs**。

    2.  单击上方的**加号**添加Python SDK。

        ![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/3545819951/p3391.png)

    3.  单击**Apply**完成添加。

3.  配置Modules。

    1.  在左侧导航栏上，单击**Modules**。

    2.  单击**+**添加Module依赖Python Facets。

        ![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/3545819951/p3393.png)

    3.  单击**Apply**保存配置。


