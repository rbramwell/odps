---
keyword: [Mars cluster operations, read/write operations on MaxCompute tables, obtain the Mars UI URL, obtain the Logview URL, obtain the Jupyter Notebook URL]
---

# Instructions

This topic describes how to perform operations on Mars clusters, read or write MaxCompute tables, and obtain Mars UI, Logview, and Jupyter Notebook URLs.

## Mars cluster operations

-   Create a Mars cluster

    Run the following commands to create a Mars cluster. This process takes a while to complete.

    ```
    from odps import options
    options.verbose = True  
    # If the preceding two commands are set in the DataWorks PyODPS 3 node, you do not need to run the commands.
    client = o.create_mars_cluster(5, 4, 16, min_worker_num=3)
    ```

    -   5: the number of worker nodes in the cluster. In this example, the cluster consists of five worker nodes.
    -   4: the number of CPU cores of each worker node. In this example, each worker node has four CPU cores.
    -   16: the memory size of each worker node. In this example, each worker node has 16 GB memory.

        **Note:**

        -   The memory size that you request for each worker node must be larger than 1 GB. The optimal ratio of CPU core count to memory size is 1:4. For example, configure a worker node with 4 CPU cores and 16 GB memory.
        -   You can create at most 30 worker nodes. Otherwise, the image server may be overloaded. If you want to create more than 30 worker nodes, [submit a ticket](https://workorder-intl.console.aliyun.com/).
    -   min\_worker\_num: the minimum number of worker nodes. When the number of worker nodes that are started reaches the number specified by min\_worker\_num, the system returns a client object without waiting until all worker nodes are started.
    If you specify `options.verbose = True` when you create a Mars cluster, the Logview, Mars UI, and Jupyter Notebook URLs of the MaxCompute instance are displayed in the command output. You can use Mars UI to connect to Mars clusters and query the statuses of clusters and jobs.

-   Submit a job

    When you create a Mars cluster, the cluster creates a default session that connects to the cluster. You can call the `execute()` method, which uses the default session as the argument, to submit a job to the cluster and run the job.

    ```
    import mars.dataframe as md
    import mars.tensor as mt
    md.DataFrame(mt.random.rand(10, 3)).execute()  # Call the execute() method to submit the job to the created cluster.
    ```

-   Stop and release a cluster

    A Mars cluster is automatically released after it is created for three days. If you no longer use a Mars cluster, you can call the `client.stop_server()` method to release the cluster.

    ```
    client.stop_server()
    ```


## Read/write operations on MaxCompute tables

Mars can directly read and write MaxCompute tables.

-   Read MaxCompute tables

    Mars uses the `o.to_mars_dataframe` method to read a MaxCompute table and return a [Mars DataFrame](https://docs.pymars.org/en/latest/#mars-dataframe).

    ```
    In [1]: df = o.to_mars_dataframe('test_mars')
    In [2]: df.head(6).execute()
    Out[2]:
           col1  col2
    0        0    0
    1        0    1
    2        0    2
    3        1    0
    4        1    1
    5        1    2
    ```

-   Write MaxCompute tables

    Mars uses the `o.persist_mars_dataframe(df, 'table_name')` method to save a Mars DataFrame as a MaxCompute table.

    ```
    In [3]: df = o.to_mars_dataframe('test_mars')
    In [4]: df2 = df + 1
    In [5]: o.persist_mars_dataframe(df2, 'test_mars_persist')  # Save the Mars DataFrame as a MaxCompute table.
    In [6]: o.get_table('test_mars_persist').to_df().head(6)  # Query data by using the PyODPS DataFrame API.
           col1  col2
    0        1    1
    1        1    2
    2        1    3
    3        2    1
    4        2    2
    5        2    3
    ```

-   Use the Jupyter Notebook of the Mars cluster

    When you create a Mars cluster, the cluster creates a Jupyter Notebook for writing code. The new Jupyter Notebook creates a session and submits a job to the Mars cluster.

    ```
    import mars.dataframe as md
    md.DataFrame(mt.random.rand(10, 3)).sum().execute() # Call the execute() method in the Jupyter Notebook to submit the job to the current cluster. You do not need to manually create a session in the Jupyter Notebook.
    ```

    **Note:**

    -   The Jupyter Notebook is not saved automatically. We recommend that you manually save the Jupyter Notebook as needed.
    -   You can connect your Jupyter Notebook to an existing Mars cluster. For more information, see [Use an existing Mars cluster](#section_mxb_4df_rm2).

## Other operations

-   Use an existing Mars cluster
    -   Recreate an existing Mars cluster based on the instance ID.

        ```
        client = o.create_mars_cluster(instance_id=**instance-id**)
        ```

    -   To use an existing Mars cluster, create a Mars session to visit the Mars UI URL.

        ```
        from mars.session import new_session
        new_session('**Mars UI address**').as_default() # Set the session as default.
        ```

-   Obtain the Mars UI URL

    If you specify `options.verbose = True` when you create a Mars cluster, the Mars UI URL is automatically displayed in the command output. You can use `client.endpoint` to obtain the Mars UI URL.

    ```
    print(client.endpoint)
    ```

-   Obtain the Logview URL

    If you specify `options.verbose = True` when you create a Mars cluster, the Logview URL is automatically displayed in the command output. You can use `client.get_logview_address()` to obtain the Logview URL.

    ```
    print(client.get_logview_address())
    ```

-   Obtain the Jupyter Notebook URL

    If you specify `options.verbose = True` when you create a Mars cluster, the Jupyter Notebook URL is automatically displayed in the command output. You can use `client.get_notebook_endpoint()` to obtain the Jupyter Notebook URL.

    ```
    print(client.get_notebook_endpoint())
    ```


