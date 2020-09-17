---
keyword: [use PyODPS in DataWorks, DataWorks, PyODPS]
---

# Use PyODPS in DataWorks

This topic describes how to use PyODPS in DataWorks and the limits.

For a usage example, see [Use a PyODPS node to segment Chinese text based on Jieba]().

## Create a PyODPS node

You can create a PyODPS node for a workflow.

![New node dialog box](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/0551330061/p11645.png)

## MaxCompute entry

Each PyODPS node in DataWorks contains the global variable `odps` or `o`, which is the MaxCompute entry. You do not need to specify the MaxCompute entry.

```
print(o.exist_table('pyodps_iris'))
```

## Execute SQL statements

You can execute SQL statements in the PyODPS node. For more information, see [SQL]().

By default, InstanceTunnel is disabled in DataWorks. In this case, instance.open\_reader is executed by using the Result interface, and a maximum of 10,000 data records can be read. You can use reader.count to obtain the number of data records. If you need to obtain all data iteratively, you must disable the limit on the data volume. You can execute the following statements to enable InstanceTunnel and disable the limit.

```
options.tunnel.use_instance_tunnel = True
options.tunnel.limit_instance_tunnel = False  # Disable the limit on the data volume.

with instance.open_reader() as reader:
    # Use InstanceTunnel to read all data.
```

You can also add `tunnel=True` to open\_reader to enable InstanceTunnel for the current open\_reader operation. You can add `limit=False` to open\_reader to disable the limit on the data volume for the current open\_reader operation.

```
with instance.open_reader(tunnel=True, limit=False) as reader:
# The current open_reader operation is executed by using InstanceTunnel and all data can be read.
```

**Note:** If you do not enable InstanceTunnel, the format of the obtained data may be invalid. For more information about how to fix the issue, see [t1856162.md\#]().

## DataFrame

-   Perform operations on DataFrames

    To perform operations on [DataFrames]() in DataWorks, you must explicitly call automatically executed methods, such as execute and head. For more information, see [t21283.md\#]().

    ```
    from odps.df import DataFrame
    iris = DataFrame(o.get_table('pyodps_iris'))
    for record in iris[iris.sepal_width < 3].execute():  # Call an automatically executed method to process each data record.
    ```

    To call an automatically executed method for data display, set `options.interactive` to True.

    ```
    from odps import options
    from odps.df import DataFrame
    options.interactive = True  # Set options.interactive to True at the beginning of the code.
    iris = DataFrame(o.get_table('pyodps_iris'))
    print(iris.sepal_width.sum())  # The method is executed immediately when the system displays information.
    ```

-   Display details

    To display details, you must set `options.verbose` to True. By default, this parameter is set to True in DataWorks. The system displays details such as the Logview URL during the running process.


## Obtain scheduling parameters

Different from SQL nodes in DataWorks, a PyODPS node does not replace strings such as $\{param\_name\} in the code. Instead, it adds a dictionary named `args` as a global variable before it runs the code. You can obtain the scheduling parameters from the dictionary. This way, the Python code is not affected. For example, on the **Scheduling configuration** tab of a PyODPS node in DataWorks, you can specify `ds=${yyyymmdd}` in the **Parameters** field in the Basic properties section. Then, you can specify the following commands in the code of the node to obtain the parameter value:

```
print('ds=' + args['ds'])
ds=20161116
```

**Note:** You can run the following command to obtain the partition named `ds=${yyyymmdd}`:

```
o.get_table('table_name').get_partition('ds=' + args['ds'])
```

## Limits on functions

-   The Python version of a PyODPS node is 2.7.
-   Each PyODPS node supports processing a maximum of 50 MB data and can occupy a maximum of 1 GB memory. Otherwise, DataWorks terminates the PyODPS node. Do not write unnecessary Python data processing code in PyODPS nodes.
-   Writing and debugging code in DataWorks is inefficient. We recommend that you install an IDE locally to write code.
-   To avoid excess pressure on the gateway of DataWorks, DataWorks limits the CPU utilization and memory usage. If the system displays **Got killed**, the memory usage exceeds the limit and the system terminates the related processes. Therefore, we recommend that you do not perform local data operations. However, the limits on the memory usage and CPU utilization do not apply to SQL or DataFrame nodes, except to\_pandas, that are initiated by PyODPS.
-   Functions may be limited in the following aspects due to the lack of packages such as matplotlib:
    -   The use of the plot function of DataFrame is affected.
    -   DataFrame user defined functions \(UDFs\) can be executed only after they are submitted to MaxCompute. As required by the Python sandbox, you can only use pure Python libraries and the NumPy library to execute UDFs. Other third-party libraries such as pandas cannot be used.
    -   However, you can use the NumPy and pandas libraries that are pre-installed in DataWorks to execute non-UDFs. You are not allowed to use other third-party libraries that contain binary code.
-   For compatibility reasons, options.tunnel.use\_instance\_tunnel is set to False in DataWorks by default. If you want to enable InstanceTunnel globally, you must set this parameter to True.
-   For implementation reasons, the Python atexit package is not supported. You must use try-finally to implement related features.

