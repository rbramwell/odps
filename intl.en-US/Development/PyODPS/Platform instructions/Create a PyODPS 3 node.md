---
keyword: PyODPS 3
---

# Create a PyODPS 3 node

This topic describes the usage limits of PyODPS 3 nodes and how to create a PyODPS 3 node in the DataWorks console.

## Limits

-   Python 3 defines bytecode differently in its different subversions such as Python 3.7 and Python 3.8.

    MaxCompute is compatible with Python 3.7. A MaxCompute client that uses another subversion of Python 3 will return an error when it executes code with specific syntax. For example, a MaxCompute client that uses Python 3.8 will return an error when it executes code with the finally block syntax. We recommend that you use a MaxCompute client with Python 3.7.

-   Each PyODPS 3 node can process a maximum of 50 MB data and can occupy a maximum of 1 GB memory. Otherwise, DataWorks terminates the PyODPS 3 node. Do not write Python code that will process an extra large amount of data in a PyODPS 3 node.

## Create a PyODPS 3 node

1.  Go to the **DataStudio** page.

    1.  Log on to the [DataWorks console](https://workbench.data.aliyun.com/console).

    2.  In the left-side navigation pane, click **Workspaces**.

    3.  In the top navigation bar, select the region where the target workspace resides. Find the target workspace and click **Data Analytics** in the Actions column.

2.  On the Data Development tab, move the pointer over ![New](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/2491659951/p72306.png) and choose **MaxCompute** \> **PyODPS 3**.

    You can also click the target workflow, right-click **MaxCompute**, and then choose **New** \> **PyODPS 3**.

3.  In the **Create Node** dialog box that appears, set **Node Name** and **Location**.

    **Note:** The node name can be up to 128 characters in length and can contain letters, digits, underscores \(\_\), and periods \(.\).

4.  Click **OK**.

5.  Enter the code of the PyODPS 3 node.

    Enter the code of the PyODPS 3 node based on your needs. For example, if you need to use the execute\_sql\(\) method to execute SQL statements in the node, you must specify runtime parameters for the SQL statements. For more information, see [SQL]().

    ```
    hints={'odps.sql.python.version': 'cp37', 'odps.isolation.session.enable': True}
    ```

    If you need to use a user defined function \(UDF\), such as df.map, df.map\_reduce, df.apply, or df.agg, to call the DataFrame API in the node, specify the following setting:

    ```
    hints={'odps.isolation.session.enable': True}
    ```

    PyODPS determines the runtime environment of the UDF and commits SQL statements based on the Python version that the MaxCompute client uses. Assume that the MaxCompute client uses Python 3. If the UDF executes a print statement specific to Python 2 to call the DataFrame API, the MaxCompute client returns the ScriptError error.

6.  On the configuration tab of the PyODPS 3 node, click the **Scheduling configuration** tab in the right-side navigation pane. On the Scheduling configuration tab, set the scheduling properties of the node. For more information, see [Basic attributes]().

7.  Save and commit the node.

    **Note:** You need to set the node's **re-run attribute** and **dependent upstream node** to commit the node.

    1.  Click ![Save icon](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/8828477951/p113864.png)icon to save the node.

    2.  Click ![Commit dialog box](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/0881659951/p98393.png) in the toolbar.

    3.  In **submit New version** dialog box, enter **remarks**.

    4.  Click **OK**.

    In a workspace in the standard mode, you need to click **Publish** in the upper-right corner after you commit the real-time sync node. For more information, see [Deploy a node]().

8.  The test node. For more information, see [Auto triggered nodes]().


