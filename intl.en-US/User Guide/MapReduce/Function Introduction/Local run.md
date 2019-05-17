# Local run {#concept_etd_q3g_vdb .concept}

## Basic stages introduction {#section_zjr_r3g_vdb .section}

**Local run prerequisite**: By setting –local parameter in the jar command, user can simulate MapReduce running process on the local to initiate local debugging.

**At local operation time**: The client downloads required meta information of input tables, resources, and meta information of output tables from MaxCompute, and saves them into a local directory named ‘warehouse’.

**After running the program**: The calculation result is output into a file in the ‘warehouse’. If the input table and referenced resources have been downloaded in the local warehouse directory, the data and files in ‘warehouse’ directory are referenced directly during the next run time, and the downloading process does not need to be repeated.

## Difference between running locally and running distributed environments {#section_k3l_t3g_vdb .section}

In the local operation course, multiple Map and Reduce workers are yet to start data processing. But these workers do not run concurrently and run serially.

The distinguishing points between the simulation process and real distributed operation are as follows:

-   A limit on the row number of input table exists. Currently, up to 100 rows of data can be downloaded.
-   Usage of resources: In a distributed environment, MaxCompute limits the size of the referenced resource. For more information, see [Application Restriction](reseller.en-US/User Guide/MapReduce/MR limits.md). Note that in the local running environment, the resource size has no limits.
-   Security limits: MaxCompute, MapReduce, and UDF program running in a distributed environment are limited by [Java Sandbox](reseller.en-US/User Guide/Java Sandbox.md). Note that in local operations this limit is not applicable.

## Example: {#section_e44_v3g_vdb .section}

A local operation example is as follows:

```
    odps:my_project> jar -l com.aliyun.odps.mapred.example.WordCount wc_in wc_out
    Summary:
    counters: 10
        map-reduce framework
                combine_input_groups=2
                combine_output_records=2
                map_input_bytes=4
                map_input_records=1
                map_output_records=2
                map_output_[wc_out]_bytes=0
                map_output_[wc_out]_records=0
                reduce_input_groups=2
                reduce_output_[wc_out]_bytes=8
                reduce_output_[wc_out]_records=2
    OK
```

For a detailed WordCount example, see [WordCount Code example](reseller.en-US/User Guide/MapReduce/Program Example/WordCount samples.md).

If a user runs local debugging command for the first time, a path named ‘warehouse’ appears in the current path after the command is executed successfully. The directory structure of warehouse is as follows:

```
<warehouse>
   |____my_project(project directory)
          |____ <__tables__>
          | |__wc_in(table directory)
          | | |____ data(file)
          |       |      |
          | | |____ <__schema__> (file)
          | |__wc_out(table data directory)
          | |____ data(file)
          |               |
          | |____ <__schema__> (file)
          |
          |____ <__resources__>
                  |
                  |___table_resource_name (table resource)
                  | |____<__ref__>
                  |
                  |___ file_resource_name (file resource)
```

-   The same level directory of myproject indicates the project. ‘wcin’ and ‘wc\_out’ indicate tables. The table files read by user in JAR command is downloaded into this directory.
-   The contents in <\_\_schema\_\_\> indicate table meta information. The format is defined as follows:

    ```
      project=local_project_name
      table=local_table_name
      columns=col1_name:col1_type,col2_name:col2_type
      partitions=p1:STRING,p2:BIGINT
    ```

    Columns and column types are separated by colons \(:\), and columns are separated by commas \(,\). Corresponding to <\_\_schema\_\_\> file, the Project name and Table name must be declared, such as `project_name.table_name`, and separated by a comma \(,\) and column definition. `project_name.table_name,col1_name:col1_type,col2_name:col2_type,……`

-   The file ‘data; indicates the table data. The column quantity and corresponding data must comply with the definition in \_schema\_. Moreover, extra columns and missing columns are not allowed.

    The content of \_schema\_ in wc\_in is as follows:

    ```
    my_project.wc_in,key:STRING,value:STRING
    ```

    The content of ‘data’ is as follows:

    ```
    0,2
    ```

    The client downloads the meta information of table and part of the data from MaxCompute, and save them into the two preceding files. If you run this example again, the data in the directory ‘wc\_in’ is used directly and will not be downloaded again.

    **Note:** The function to download the data from MaxCompute is only supported in MapReduce local operation mode. If the local debugging is executed in [Eclipse development plug-in](https://www.alibabacloud.com/help/doc-detail/27981.html), the data of MaxCompute cannot be downloaded to local.

    The content of ‘\_schema\_’ in wc\_out is as follows:

    ```
    my_project.wc_out,key:STRING,cnt:BIGINT
    ```

    The content of ‘data’ is as follows:

    ```
      0,1
      2,1
    ```

    The client downloads the meta information of wc\_out from MaxCompute and saves it to the file \_schema\_. The file ‘data’ is a result data file generated after the local operation.

    **Note:** 

    -   Users can also edit \_schema\_ file and ‘data’ and then place these two files into the corresponding table directory.
    -   When running on the local, the client can detect the table directory already exists, and does not download the information of this table from MaxCompute. The table directory on the local can be a table that does not exist in MaxCompute.

