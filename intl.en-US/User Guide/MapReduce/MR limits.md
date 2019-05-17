# MR limits {#concept_m4c_4lg_vdb .concept}

In order to avoid that you have not paid attention to restrictions so that business stops after the business starts, this article will summarize the MaxCompute MR restrictions to help you.

The restrictions of MaxCompute MapReduce are as follows:

|Restricted item|Value|Type|Configuration item|Default value|Configurable?|Description|
|:--------------|:----|:---|:-----------------|:------------|:------------|:----------|
|Memory occupied by the instance|\[256MB,12GB\]|Memory limit|odps.stage.mapper\(reducer\).mem and odps.stage.mapper\(reducer\).jvm.mem|2048M＋1024M|Yes|Memory occupied by a single map instance or reduce instance, including the framework memory \(2,048 MB by default\) and heap memory of the Java virtual machine \(JVM\) \(1,024 MB by default\).|
|Number of resources|256|Number limit|N/A|None|No|The number of resources referenced by a single job cannot exceed 256. The table and archive are regarded as a unit.|
|Numbers of inputs and outputs|1024 and 256|Number limit|N/A|None|No|The number of inputs of one job cannot exceed 1024. \(A partition of a table is regarded as one input. The number of input tables cannot exceed 64\). The number of outputs of one job cannot exceed 256.|
|Number of counters|64|Number limit|N/A|None|No|The number of custom counters in one job cannot exceed 64. The group name and counter name of a counter must not contain \#. The overall length of the group name and the counter name of a counter must be within 100.|
|map instance|\[1,100000\]|Number limit|odps.stage.mapper.num|None|Yes|The number of map instances of one job is calculated by the framework based on the split size. If no input table exists, you can set the value directly in odps.stage.mapper.num. The final number ranges from 1 to 100,000.|
|reduce instance|\[0,2000\]|Number limit|odps.stage.reducer.num|None|Yes|The number of reduce instances of one job is 1/4 of that of map instances by default. The reduce instance number configured by the user ranges from 0 to 2,000. It may occur that the data volume processed by reduce is several times that processed by map. In this case, the reduce phase gets slower and can initiate at most 2000 instances.|
|Number of retries|3|Number limit|N/A|None|No|The maximum number of retries allowed for a single map instance or reduce instance is 3. Some exceptions that do not allow retries may cause task execution failures.|
|Local debug mode|100|Number limit|N/A|None|No|In local debug mode, the number of map instances is 2 by default and cannot exceed 100. The number of reduce instances is 1 by default and cannot exceed 100. The number of download records of one input is 1 by default and cannot exceed 100.|
|Number of times of reading a resource repeatedly|64|Number limit|N/A|None|No|The number of times that a map instance or reduce instance reads one resource repeatedly cannot exceed 64 .|
|Resource length|2G|Length limit|N/A|None|No|The total length of a resource referenced by a job cannot exceed 2 GB.|
|split size|\[1,\)|Length limit|odps.stage.mapper.split.size|256M|Yes|The framework splits the map based on the configured split size, of which the number of maps is then determined.|
|Content length of the string column|8 MB|Length limit|N/A|None|No|The content in the string column of the MaxCompute table cannot exceed 8 MB.|
|Worker running timeout period|［1,3600］|Time limit|odps.function.timeout|600|Yes|Timeout period for the worker when the map or reduce worker does not read or write data or actively send heartbeat data by using context.progress\(\). The default value is 600s.|
|The supported field types of table referenced by MR|BIGINT,DOUBLE,STRING,DATETIME,BOOLEAN|Data type limit|N/A|None|No|When the MR task refers to a table, an error occurs if the table contains other types of fields.|
|Read data from OSS| |Feature limit|N/A|None|No|Not supported|
|MaxCompute 2.0 new types| |Feature limit|N/A|None|No|Not supported|

