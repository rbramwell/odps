# Develop a PyODPS script

This topic describes how to develop a MaxCompute SDK for Python \(PyODPS\) script.

The Python development environment is configured. For more information, see [Configure a Python development environment](/intl.en-US/Tools and Downloads/MaxCompute Studio/Develop a Python program/Configure a Python development environment.md).

PyODPS supports basic operations on MaxCompute objects and allows you to use DataFrames. You can use PyODPS to analyze data in MaxCompute.

1.  In the Project section, right-click **scripts** and choose **New** \> **MaxCompute Python**.

    ![MaxCompute Python](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/5725819951/p3447.png)

2.  In the **Create new MaxCompute python class** dialog box, enter a class name in the **Name** field, select **PyODPS Script** from the **Kind** drop-down list, and then click OK.

    ![Create new MaxCompute python class](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/5725819951/p3452.png)

3.  Write the script code in the code editor. When you create a PyODPS script, the `odps` and `o` objects are automatically initialized in the PyODPS script template by using a PyODPS room.

    When you develop a PyODPS script in DataWorks, a room is automatically created. When you develop a PyODPS script in Intellij IDEA, you must create a room yourself. For more information, see the [PyODPS documentation](https://pyodps.readthedocs.io/).

    ![Template](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/5725819951/p3455.png)


