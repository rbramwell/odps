# Development and debugging {#concept_tlp_v4m_vdb .concept}

MaxCompute does not provide Graph development plugins for users. However, you can develop the MaxCompute Graph program based on Eclipse. The development process is as follows:

1.  Compile Graph codes and perform basic tests using local debugging.
2.  Perform cluster debugging and verify the result.

## Example {#section_zvq_z4m_vdb .section}

This section uses the [SSSP](reseller.en-US/User Guide/Graph/Examples/SSSP.md) algorithm as an example to describe how to use Eclipse to develop and debug a Graph program.

**Procedure**

1.  Create a Java project, for example, graph\_examples.
2.  Add the JAR package in the lib directory of the MaxCompute client to Build Path of the Eclipse project. The following figure shows a configured Eclipse project:

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12048/15580610072233_en-US.png)

3.  Develop a MaxCompute Graph program.

    In the actual development process, an example \(such as [SSSP](reseller.en-US/User Guide/Graph/Examples/SSSP.md)\) is often copied and then modified. In this example, only the package path is changed to package com.aliyun.odps.graph.example.

4.  Compile and build the package.

    In an Eclipse environment, right-click the source code directory \(the src directory in the figure\) and select Export \> Java \> JAR file to generate a JAR package. Select the path for storing the target JAR package, for example, `D:\\odps\\clt\\odps-graph-example-sssp.jar`.

5.  Use the MaxCompute console to run SSSP. For more information about the related operations, see [Run Graph](../../../../reseller.en-US/Quick Start/(Optional) Submit Graph jobs.md) in “Quick start”.

**Note:** For more information about the related development procedure, see Introduction on the [Graph development plug-in](https://www.alibabacloud.com/help/doc-detail/27985.html)*.*

## Local debugging {#section_dll_yqm_vdb .section}

MaxCompute Graph supports the local debugging mode. Use Eclipse to perform breakpoint debugging.

**Procedure**

1.  Download an odps-graph-local maven package.
2.  Select the Eclipse project, right-click the main program file \(including the main function\) of the Graph job, and configure its running parameters \(by selecting Run As \> Run Configurations\).
3.  On the Arguments tab page, set Program arguments to `1 sssp_in sssp_out` as the input parameter of the main program.
4.  On the Arguments tab page, set VM arguments to the following:

    ```
    -Dodps.runner.mode=local
    -Dodps.project.name=<project.name>
    -Dodps.end.point=<end.point>
    -Dodps.access.id=<access.id> 
    -Dodps.access.key=<access.key>
    						
    ```

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12048/15580610072234_en-US.png)

5.  If MapReduce is in local mode \(the value of odps.end.point is not specified\), you must create the sssp\_in and sssp\_out tables in the warehouse and add data for sssp\_in. Input data is listed as follows:

    ```
    1,"2:2,3:1,4:4"
    2,"1:2,3:2,4:1"
    3,"1:1,2:2,5:1"
    4,"1:4,2:1,5:1"
    5,"3:1,4:1"
    ```

    For more information about the warehouse, see [MapReduce local running](reseller.en-US/User Guide/MapReduce/Function Introduction/Local run.md).

6.  Click **Run**.

    **Note:** Check the settings of conf/odps\_config.ini in the MaxCompute client to set parameters. The preceding parameters are commonly used. Other parameters are described as follows:

    -   odps.runner.mode: The parameter value is local. This parameter is required for the local debugging function.
    -   odps.project.name: \(Required\). Specifies the current project.
    -   odps.end.point: \(Optional\). Specifies the address of the current MaxCompute service. If this parameter is not specified, metadata of tables or resources is only read from the warehouse, and an exception is thrown when the address does not exist. If this parameter is specified, data is read from the warehouse first, and then from remote MaxCompute if the address does not exist.
    -   odps.access.id: Indicates the ID to connect to the MaxCompute service. This parameter is valid only when odps.end.point is specified.
    -   odps.access.key: Indicates the key to connect to the MaxCompute service. This parameter is valid only when odps.end.point is specified.
    -   odps.cache.resources: Specifies the resource list in use. This parameter has the same effect as -resources of the JAR command.
    -   odps.local.warehouse: Specifies the local warehouse path. This parameter is set to ./warehouse by default ,if not specified.
    After SSSP debugging is implemented locally in Eclipse, the following information is output:

    ```
    Counters: 3
             com.aliyun.odps.graph.local.COUNTER
                     TASK_INPUT_BYTE=211
                     TASK_INPUT_RECORD=5
                     TASK_OUTPUT_BYTE=161
                     TASK_OUTPUT_RECORD=5
     graph task finish
    ```

    **Note:** In the preceding example, the sssp\_in and sssp\_out tables must exist in the local warehouse. For more information about the sssp\_in and sssp\_out tables, see [Run Graph](https://www.alibabacloud.com/help/doc-detail/27813.htm) in “Quick start”.


## Temporary directory of local job {#section_ijb_rrm_vdb .section}

A temporary directory is created in the Eclipse project directory when local debugging runs each time, as shown in the following figure.

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12048/15580610072235_en-US.png)

The temporary directory of a locally running Graph job contains the following directories and files:

-   counters: Stores counting information about job running.
-   inputs: Stores input data of the job. Data is preferentially obtained from the local warehouse. If such data does not exist locally, the MaxCompute SDK reads data from the server \(if odps.end.point is set\). An input reads only 10 data records by default. This threshold can be modified in the `-Dodps.mapred.local.record.limit` parameter, of which the maximum value is 10,000.
-   outputs: Stores output data of the job. If the local warehouse has an output table, the result data in the outputoverwrites the corresponding table in the local warehouse after job running is complete.
-   resources: Stores resources used by the job. Similar to inputs, data is preferentially obtained from the local warehouse. If such data does not exist locally, the data is read from the server using MaxCompute SDK \(when odps.end.point is set\).
-   job.xml: Indicates job configuration.
-   superstep: Stores information about message persistence in each iteration.

**Note:** If a detailed log must be output during local debugging, the following log4j configuration file must be placed in the src directory: `log4j.properties_odps_graph_cluster_debug`.

## Cluster debugging {#section_mfh_zrm_vdb .section}

After local debugging, submit the job to a cluster for testing.

**Procedure**

1.  Configure the MaxCompute client.
2.  Run the `add jar /path/work.jar -f;` command to update the JAR package.
3.  Run a JAR command to run the job, and view the running log and result data.

**Note:** For more information about how to run Graph in a cluster, see [Run Graph](../../../../reseller.en-US/Quick Start/(Optional) Submit Graph jobs.md) in “Quick start”.

## Performance Tuning {#section_pxb_fsm_vdb .section}

The following section describes common performance tuning methods on the MaxCompute Graph framework.

**Job Parameter configuration**

GraphJob configurations that have an impact on performance include:

-   setSplitSize\(long\): Indicates the split size of an input table. The unit is in MB. Its value must be greater than 0, and the default value is 64.
-   setNumWorkers\(int\): Specifies the number of Workers for a job. The value range is \[1, 1000\], and the default value is –1. The number of Workers varies depending on the number of input bytes of the job and split size.
-   setWorkerCPU\(int\): Indicates CPU resources of the Map. A one-core CPU contains 100 resources. The value range is \[50, 800\], and the default value is 200.
-   setWorkerMemory\(int\): Indicates memory resources of the Map. The unit is MB. The value range is \[256 MB, 12 GB\], and the default value is 4,096 MB.
-   setMaxIteration\(int\): Specifies the maximum number of iterations. The default value is –1. If the value is smaller than or equal to 0, the maximum number of iterations is not a condition for job termination.
-   setJobPriority\(int\): Specifies the job priority. The value range is \[0, 9\], and the default value is 9. A larger value indicates a smaller priority.

Additional actions that increase overall processing capabilities are as follows:

-   You can use the setNumWorkers\(\) method to increase the number of Workers.
-   You can use the setSplitSize\(\) method to reduce the split size and increase the speed for a job to load data.
-   Increase the CPU or memory of Workers.
-   Set the maximum number of iterations. If applications do not have high requirements on result precision, you can reduce the number of iterations to speed up the process.

The interfaces setNumWorkers and setSplitSize can be used together to speed up data loading. Assume that setNumWorkers is workerNum and setSplitSize is splitSize, and the total number of input bytes is inputSize. The number of splits is calculated using the formula: splitNum = inputSize /splitSize. The relationship between workerNum and splitNum is as follows:

-   If splitNum == workerNum, each Worker is responsible for loading one split.
-   If splitNum \> workerNum, each Worker is responsible for loading one or multiple splits.
-   If splitNum < workerNum, each Worker is responsible for loading zero or one split.

Therefore, if the first two conditions are met, you can adjust workerNum and splitSize to enable fast data loading. In the iteration phase, you only need to adjust workerNum.

If you set runtime partitioning to false, we recommend that you use setSplitSize to control the number of Workers. Regarding the third condition, the number of vertices on some Worker may be 0. You can use set odps.graph.split.size=<m\>; set odps.graph.worker.num=<n\>; before the JAR command, which has the same effect as setNumWorkers and setSplitSize.

Another common performance problem is data skew. For example, on Counters, the number of vertices or edges processed by some Workers is much greater than that processed by other Workers.

Data skew occurs usually when the number of vertices, edges, or messages corresponding to some keys is much greater than that corresponding to other keys. Such keys with the large data volume are processed by a small number of Workers, resulting in a long run time of these Workers.

To resolve this problem, we recommend the following steps:

-   Use a combiner to locally aggregate messages of vertices corresponding to such keys to reduce the number of sent messages.
-   Improve the service logic.

**Use a Combiner**

Define a Combiner to reduce memory that stores messages and network data traffic volume and shortens the job execution time. For more information, see introduction to Combiner in MaxCompute SDK.

**Reduce the Data Input Volume**

When the data volume is large, reading data in a disk may extend the processing time. Therefore, reducing the number of data bytes to be read can increase the overall throughput, thereby improving job performance. You can use either of the following methods:

-   Reduce the input data volume: For decision-making applications, results obtained from processing subsets after data sampling only affect the result precision, instead of the overall accuracy. Therefore, you can perform special data sampling and import the data to the input table for processing.
-   Avoid reading fields that are not used: The TableInfo class of the MaxCompute Graph framework supports reading specific columns \(transmitted using column name arrays\), rather than reading the entire table or table partition. This reduces the input data volume and improves job performance.

## Built-in JAR Packages {#section_cpx_qsm_vdb .section}

The following JAR packages are loaded to JVMs running the Graph program by default. You do not have to upload these resources or carry these JAR packages when running -libjars on the command line.

-   commons-codec-1.3.jar
-   commons-io-2.0.1.jar
-   commons-lang-2.5.jar
-   commons-logging-1.0.4.jar
-   commons-logging-api-1.0.4.jar
-   guava-14.0.jar
-   json.jar
-   log4j-1.2.15.jar
-   slf4j-api-1.4.3.jar
-   slf4j-log4j12-1.4.3.jar
-   xmlenc-0.52.jar

**Note:** In a classpath that runs a JVM, the preceding built-in JAR packages are placed before users’ JAR packages, which may result in a version conflict. For example, if your program uses a function of a class in commons-codec-1.5.jar but this function is not in commons-codec-1.3.jar. Check whether an implementation method exists in commons-codec-1.3.jar or wait for MaxCompute to upgrade to a supported version.

