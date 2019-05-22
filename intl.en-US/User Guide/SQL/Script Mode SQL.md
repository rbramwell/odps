# Script Mode SQL {#concept_f4h_hnc_bfb .concept}

This topic describes Script Mode SQL and how to run this language.

**Note:** The fees information detailed in the instructions section of [Other operations](reseller.en-US/User Guide/Common commands/Other operations.md#section_xm2_sgf_vdb) for Script Mode SQL is provided as reference purposes only. The fees displayed on the billing console are the actual fees charged to your account. For more information, see [View billing details](../../../../reseller.en-US/Pricing/View billing details.md#).

## What is Script Mode SQL {#section_4vb_lpu_mch .section}

MaxCompute supports Script Mode SQL. In script mode type of SQL, statements are compiled as a whole according to the logic in an SQL script file. Then the script file is submitted to MaxCompute to generate one execution plan, so that these statements are scheduled in one queue and run all at once.

Script Mode SQL is used similarly to other common programming languages. This language can assist in compiling scripts more efficiently by eliminating the need to organize statements for these scripts.

Script Mode SQL supports the following tools:

-   MaxCompute Studio: For more information, see [What is Studio](../../../../reseller.en-US/Tools and Downloads/MaxCompute Studio/What is Studio.md#).
-   MaxCompute command line interface \(CLI\): For more information, see [Install and configure a client](../../../../reseller.en-US/Prepare/Install and configure a client.md#).

## Syntax {#section_mrl_g4c_bfb .section}

```
--set
set odps.sql.type.system.odps2=true;
[set odps.stage.reducer.num=xxx;]
[...]
--ddl
create table table1 xxx;
[create table table2 xxx;]
[...]
--dml
@var1 := SELECT [ALL | DISTINCT] select_expr, select_expr, ...
        FROM table3
        [WHERE where_condition];
@var2 := SELECT [ALL | DISTINCT] select_expr, select_expr, ...
        FROM table4
        [WHERE where_condition];
@var3 := SELECT [ALL | DISTINCT] var1.select_expr, var2.select_expr, ...
        FROM @var1 join @var2 on ...;
INSERT OVERWRITE|INTO TABLE [PARTITION (partcol1=val1, partcol2=val2 ...)]
        SELECT [ALL | DISTINCT] select_expr, select_expr, ...
        FROM @var3;    
[@var4 := SELECT [ALL | DISTINCT] var1.select_expr, var.select_expr, ... FROM @var1 
        UNION ALL | UNION 
        SELECT [ALL | DISTINCT] var1.select_expr, var.select_expr, ... FROM @var2;    
  CREATE [EXTERNAL] TABLE [IF NOT EXISTS] table_name 
        AS 
        SELECT [ALL | DISTINCT] select_expr, select_expr, ...
        FROM var4;]
[...]
```

-   Script Mode SQL supports `SET` statements, data manipulation language \(DML\) statements, and data definition language \(DDL\) statements \(excluding such statements as `DESC` and `SHOW` whose results are displayed on your screen\).
-   A complete script must start with `SET` statements, followed by DDL statements, and end with DML statements. One part may either be empty or contain several statements. Different types of statements cannot co-occur in one part.
-   The at sign `@` indicates a variable.
-   A script can contain up to one statement, such as a `SELECT` statement, whose result is displayed on your screen. If one script contains two or more such statements, errors are more likely to occur. Therefore, we recommend that you do not run such `SELECT` statements.
-   A script can contain only one `CREATE TABLE AS` statement, and must end with this statement. We recommend that you separate the `CREATE TABLE AS` statement from `INSERT` statements.
-   If a statement fails to run, all the other statements in the same script also fail.
-   A job is generated to process data only after all the input data is prepared.
-   If a table is written and then read, the following error occurs:

    ```
    insert overwrite table src2 select * from src where key > 0;
    @a := select * from src2;
    select * from @a;
    ```

    To avoid this error, modify your SQL script as follows:

    ```
    @a := select * from src where key > 0;
     insert overwrite table src2 select * from @a;
     select * from @a;
    ```


**Example** 

```
create table if not exists dest(key string , value bigint) partitioned by (d string);
create table if not exists dest2(key string,value bigint) partitioned by (d string);
@a := select * from src where value >0;
@b := select * from src2 where key is not null;
@c := select * from src3 where value is not null;
@d := select a.key,b.value from @a left outer join @b on a.key=b.key and b.value>0;
@e := select a.key,c.value from @a inner join @c on a.key=c.key;
@f := select * from @d union select * from @e union select * from @a;
insert overwrite table dest partition (d='20171111') select * from @f;
@g := select e.key,c.value from @e join @c on e.key=c.key;
insert overwrite table dest2 partition (d='20171111') SELECT * from @g;
```

**Scenarios**

We recommend that you use Script Mode SQL in the two following scenarios:

-   You require multiple layers of query clauses to be nested into one statement, or require a complicated script to be split into multiple simpler statements.
-   You require the input data obtained from multiple sources at long intervals to be combined into a script. However, the input data cannot be combined by using table variables alone.

## Run Script Mode SQL by using MaxCompute Studio {#section_od2_yqc_bfb .section}

Before running Script Mode SQL, you must first install MaxCompute Studio, add your project link, and create a MaxCompute SQL script file. For more information, see [Install IntelliJ IDEA](../../../../reseller.en-US/Tools and Downloads/MaxCompute Studio/Tools Installation and version history/Install IntelliJ IDEA.md#), [Project space connection management](../../../../reseller.en-US/Tools and Downloads/MaxCompute Studio/Project space connection management.md#), and [Create MaxCompute Script module](../../../../reseller.en-US/Tools and Downloads/MaxCompute Studio/Develop SQL procedure/Create MaxCompute Script module.md#).

After submitting your script, you can view the corresponding plan for running the script in a directed acyclic graph \(DAG\). This is the case even though the script contains multiple statements.

## Run Script Mode SQL by using MaxCompute Console \(odpscmd\) {#section_arx_3xc_bfb .section}

For the following examples, we recommend that you use MaxCompute CLI \(odpscmd\) v0.27.0 or a later version, which can be downloaded from [Directory Listing For odpscmd](http://odps.alibaba-inc.com/official_downloads/odpscmd/). After installing the odpscmd tool, you can use the `-s` parameter to submit your script.

To edit the myscript.sql source code file in script mode, run the following command:

 `odpscmd -s myscript.mxql;` 

**Note:** `-s` is a parameter similar to `-f` and `-e` rather than a command in the odpscmd tool. The odpscmd tool does not support Script Mode SQL or table variables.

## Run Script Mode SQL by using DataWorks {#section_p91_9ik_im6 .section}

You can create a node that runs in script mode, as shown in the following figure.

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/20231/155850687844790_en-US.png)

On this node, you can compile your script in script mode and then click **Run** on the toolbar to submit your script to MaxCompute. You can visit the URL of Logview in the command output to obtain the script execution plan and result.

