# Develop MapReduce {#concept_nvh_s2c_5db .concept}

After the [Create MaxCompute Java Module](reseller.en-US/Tools and Downloads/MaxCompute Studio/Developing Java/Create MaxCompute Java Module.md#) is created, [MR](../../../../reseller.en-US/User Guide/MapReduce/Summary/MapReduce.md) can be developed.

## Develop the MR program {#section_gr4_mgg_vdb .section}

1.  Right-click the module source code directory **src** \> **main**, select **New** \> **java**, and select **MaxCompute Java**.

2.  Create Driver, Mapper, and Reducer.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12131/15603073921997_en-US.png)

3.  Set the input/output table and Mapper/Reducer class. The framework code is automatically filled in the template.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12131/15603073921998_en-US.png)


For details of developing MR, see [To write MapReduce](../../../../reseller.en-US/Quick Start/(Optional) Use MapReduce.md#).

## Debug the MR program {#section_jr4_mgg_vdb .section}

After the MR program is developed, test your code and check whether it meets the expectations. The following two methods are supported:

**Unit test \(UT\)**: There are WordCount UT examples in the examples directory. You can refer to them to compile your UT.

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12131/15603073931999_en-US.png)

**Local MR running**: During local running, the running data source must be specified. The following two methods are provided to set the test data source:

-   MaxCompute Studio uses the Tunnel Service to automatically download table data of a specific MaxCompute project to the warehouse directory. By default, 100 data records are downloaded. If more data is required for testing, use the Tunnel Command of the console or table downloading function of MaxCompute Studio.

-   Provide the mock project \(example\_project\) and table data. You can see example\_project in warehouse to set it by yourself.


1.  Run the MR program. Right-click the Driver class and select **Run**. In the displayed Run Configuration dialog box, configure the MaxCompute project on which the MR program runs.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12131/15603073932001_en-US.png)

2.  Click **OK**. If table data of the specified MaxCompute project is not downloaded to warehouse, download data first. If a mock project is used or the MaxCompute project table data is downloaded, skip this step. Then, the MR local run framework reads specified table data in warehouse as the MR input and runs the MR program locally. You can view log output and result display on the console.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12131/15603073942002_en-US.png)


## Run the MR program in the production environment {#section_or4_mgg_vdb .section}

After local debugging is complete, release the MR program to the server and run it in the MaxCompute distributed environment.

1.  Use the MaxCompute console integrated with MaxCompute Studio in seamless mode, that is, in the Project Explorer window, right-click Project and select Open in Console, and input the commands similar to the following [JAR command](../../../../reseller.en-US/User Guide/MapReduce/Function Introduction/Commands.md) in the console command line:

    ``` {#codeblock_3bb_2ub_s0i}
    jar -libjars wordcount.jar -classpath D:\odps\clt\wordcount.jar com.aliyun.odps.examples.mr.WordCount wc_in wc_out;
    ```


