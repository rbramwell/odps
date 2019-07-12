# SET操作 {#concept_yys_xby_rfb .concept}

本文向您介绍如何使用SET命令设置MaxCompute或用户自定义的系统变量，以及如何清除SET命令设置。

## SET {#section_pqs_bff_vdb .section}

命令格式

``` {#codeblock_8vb_luf_la6}
set <KEY>=<VALUE>
```

作用：设置MaxCompute或影响MaxCompute的行为。

MaxCompute支持的`set`命令Value如下所示。

|属性名称|属性描述|取值范围|
|:---|:---|:---|
|odps.sql.allow.fullscan|设置是否允许对分区表进行全表扫描，False不允许，True为允许。|True（允许）/False（禁止）。|
|odps.stage.mapper.mem|设置每个map worker的内存大小。|单位M，默认值1024M。|
|odps.stage.reducer.mem|设置每个reduce worker的内存大小。|单位M，默认值1024M。|
|odps.stage.joiner.mem|设置每个join worker的内存大小。|单位M，默认值1024M。|
|odps.stage.mem|设置MaxCompute 指定任务下所有worker的内存大小。优先级低于以上三个set key。|单位M，无默认值。|
|odps.stage.mapper.split.size|修改每个map worker的输入数据量，即输入文件的分片大小，从而间接控制每个map阶段下worker的数量。|单位M，默认值256M。|
|odps.stage.reducer.num|修改每个reduce阶段worker数量。|无默认值。|
|odps.stage.joiner.num|修改每个join阶段worker数量。|无默认值。|
|odps.stage.num|修改MaxCompute指定任务的所有阶段的worker的并发度，优先级低于以上三者。|无默认值。|
|odps.sql.reshuffle.dynamicpt|避免拆分动态分区时产生过多小文件。如果生成的动态分区个数很少，建议将值设为False，以避免数据倾斜。|True/False，默认值为True。|
|odps.sql.type.system.odps2|SQL操作中涉及到[新数据类型](cn.zh-CN/开发/数据类型.md#)（TINYINT、SMALLINT、INT、FLOAT、VARCHAR、TIMESTAMP、BINARY等）时需要设置为True。此设置为Session级别的设置。|True（允许）/False（禁止），默认为False。|
|odps.sql.hive.compatible|打开Hive兼容模式后，MaxCompute才可以支持Hive指定的各种语法，例如inputRecordReader、outputRecordReader、Serde等。此设置为Session级别的设置。|True（允许）/False（禁止），默认为False。|

**说明：** `set`操作命令仅支持字母小写。Project级别设置请参见[SetProject](cn.zh-CN/开发/常用命令/项目空间操作.md#section_pyt_mff_vdb)。

`set odps.sql.executionengine.coldata.deep.buffer.size.max`：

-   作用：调整MaxCompute写表过程中为单个复杂类型的列预先申请缓存大小，以便提高写入性能。
-   使用场景
    1.  输出的表中含有的复杂数据类型过多。
    2.  输出的表中含有的某个单独的复杂类型变量Size过大。
-   使用说明
    -   默认值为67108864（64M），默认单位为Byte。
    -   如果输出的表有3个列的Schema是复杂类型，例如列类型为（STRING、MAP、STRUCT、ARRAY、BINARY），则默认情况下MaxCompute将会为写表操作预留64M\*3大小的内存。为每一列预先申请的缓存将会用来存放这一列`batch row count`行的数据。
    -   如果预先可知道表的复杂类型变量占用的空间比较小，则建议调小此值。例如，如果每个复杂类型变量大小不会超过1024 Byte，同时`batch row count`值使用的是默认值（1024），则可以将Flag设置为1024\*1024，如下所示。

        ``` {#codeblock_x8i_loq_oyu .language-sql}
        set odps.sql.executionengine.coldata.deep.buffer.size.max=1048576;
        ```

    -   如果您预先知道每个复杂类型的值都在7M到8M之间，同时指定了`batch row count`为32，则这个值可以被调整为8M\*32。
    -   如果任务的输出带有复杂类型，或者任务的`mapjoin`小表带有复杂类型，这个值的调整会影响到任务执行过程中使用的内存。根据前面的计算方法，值设的过大有可能导致任务OOM内存溢出。

`set odps.stage.mapper.split.size`：

-   作用：等同于`odps.sql.mapper.split.size`，用于调整每个`mapper`读取数据的大小，单位MB，示例如下 。

    ``` {#codeblock_djz_s18_fs1}
    set odps.stage.mapper.split.size=256
    ```


## Show Flags {#section_ft4_jff_vdb .section}

命令格式

``` {#codeblock_1q2_gee_77w}
show flags; --显示set设置的参数。
```

**说明：** 运行`use project`命令会清除`set`命令设置的配置。

