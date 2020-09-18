---
keyword: configure a Python development environment
---

# Configure a Python development environment

MaxCompute Studio allows you to develop Python programs, such as user defined functions \(UDFs\) and MaxCompute SDK for Python \(PyODPS\) scripts, in Intellij IDEA. This topic describes how to configure a Python development environment.

## Install PyODPS

1.  Install PyODPS. For more information, see [Installation guide and limits](/intl.en-US/Development/PyODPS/Installation guide and limits.md).

2.  Start Intellij IDEA. In the top navigation bar, choose **File** \> **Settings**.

3.  On the **Settings** page, click **MaxCompute Studio** in the left-side navigation pane.

4.  Set the **Python path to resolve UDF** parameter to the local path of the Python.exe file.

    ![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/7144130061/p34672.png)


## Install the Python plug-in

In the plug-in repository of Intellij IDEA, search for Python Community Edition and install the Python plug-in.

1.  In the top navigation bar of Intellij IDEA, choose **File** \> **Settings**.

2.  On the **Settings** page, click **Plugins** in the left-side navigation pane.

3.  In the search box, enter Python Community Edition to search for the Python plug-in. Then, install the Python plug-in.

    ![**](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/7144130061/p95688.png)


## Configure the Python dependency

After you configure the Python dependency for the MaxCompute script module, you can develop Python programs by using MaxCompute Studio.

1.  In the top navigation bar, choose **File** \> **Project Structure**.

2.  Add the Python SDK.

    1.  In the left-side navigation pane, click **SDKs** under Platform Settings.

    2.  Click the plus sign \(**+**\) at the top and add the Python SDK.

        ![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/7144130061/p3391.png)

    3.  Click **Apply**.

3.  Configure the Python dependency for the MaxCompute script module.

    1.  In the left-side navigation pane, click **Modules** under Project Settings.

    2.  Click the plus sign \(**+**\) at the top and add the dependency on the Python SDK for the MaxCompute script module.

        ![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/7144130061/p3393.png)

    3.  Click **Apply**.


