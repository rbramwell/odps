# VALUES {#concept_uhn_rdb_wdb .concept}

This topic describes the `INSERT … VALUES` command operation.

In the test phase, you need to prepare some data for simple testing:

-   If you want to create a few data records \(such as 10 or fewer records\), you can use the `INSERT … VALUES` statement to quickly write the data.
-   If you want to create a large number of data records \(such as more than 10 records\), you can incorporate the data records into a .txt or .csv file and then upload the file by using Tunnel. For more information about this method, see [Import data](../../../../reseller.en-US/Quick Start/Import data.md#). Alternatively, you can incorporate the data records into a file and then import the file by using DataWorks. For more information about this method, see [Introduction to console](../../../../reseller.en-US/User Guide/Data development/Console functions/Introduction to console.md#).

**Note:** 

Currently, the `INSERT OVERWRITE` statement cannot insert columns, therefore we recommend that you use the `INSERT INTO` statement instead.

 **Statement format**:

``` {#codeblock_fjd_k1v_iiz}
INSERT INTO TABLE tablename 
[PARTITION (partcol1=val1, partcol2=val2 ...)][co1name1,colname2...] 
[VALUES (col1_value,col2_value,...),(col1_value,col2_value,...),...]
```

 **Example 1:**:

``` {#codeblock_isu_qcx_3a1}
drop table if exists srcp;
create table if not exists srcp (key string ,value bigint) partitioned by (p string);
insert into table srcp partition (p='abc') values ('a',1),('b',2),('c',3);
```

After the preceding statements are complete, the result of Partition `abc` is as follows:

``` {#codeblock_52r_572_nfi}





| key | value | p |

| a | 1 | abc |
| b | 2 | abc |
| c | 3 | abc |
			
```

If you want to run the `INSERT` statement to write data into only some columns of a table, use the `insert list` function as follows.

 **Example 2:** 

``` {#codeblock_wpw_yp6_mqw}
drop table if exists srcp;
create table if not exists srcp (key string ,value bigint) partitioned by (p string);
insert into table srcp partition (p)(key,p) values ('d','20170101'),('e','20170101'),('f','20170101');
```

After the preceding statements are complete, the result of Partition `20170101` is as follows:

``` {#codeblock_d82_se5_c97}





| key | value | p |

| d | NULL | 20170101 |
| e | NULL | 20170101 |
| f | NULL | 20170101 |
			
```

For columns not specified in values, the default value is NULL. The `insert list` function is not necessarily used with values, and can also be used with the `insert into…select…` function.

 **In fact, the values is not only used in the `INSERT` statement, any DML statement can also be used.** 

The `INSERT…VALUES` method has a limitation: values must be constants. You can use the values table function of MaxCompute to perform some simple operations on the inserted data. For more information, see Example 3.

 **Example 3:** 

``` {#codeblock_tjx_0gr_a03}
drop table if exists srcp;
create table if not exists srcp (key string ,value bigint) partitioned by (p string);
insert into table srcp partition (p) select concat(a,b), length(a)+length(b),'20170102' from values ('d',4),('e',5),('f',6) t(a,b);
```

The `values (…), (…) t(a, b)` is used to define a table named `t` whose columns are `a` and `b`and data types are STRING for column `a` and BIGINT for column `b`. The data types are derived from the values list. In this way, with no physical table prepared, it is possible to simulate a multi-row table with arbitrary data and perform arbitrary calculations.

After the preceding statements are complete, the result of Partition `20170102` is as follows:

``` {#codeblock_sd8_46f_qqc}





| key | value | p |

| d4 | 2 | 20170102 |
| e5 | 2 | 20170102 |
| f6 | 2 | 20170102 |
			
```

The use of `VALUES TABLE` can also replace the combination of `select * from dual` and to spell out the constants as follows:

``` {#codeblock_uog_nz5_v1h}
select 1 c from dual 
union all
select 2 c from dual;
--The same as: 
select * from values (1), (2) as t (c);
```

A special usage of values is as follows.

``` {#codeblock_s92_xzg_zrb}
select abs(-1), length('abc'), getdate();
```

As the preceding statement shows, the `SELECT` statement can be run without the `FROM` statement, if the expression list of the `SELECT` statement does not use any upstream table data. The underlying implementation is selecting from an anonymous values table in one row and zero columns. In this way, to test some functions, such as your UD, you do not need to manually create DUAL tables.

**Note:** 

-   Values only support constants and do not support functions including ARRAY complex types. Currently, MaxCompute cannot construct corresponding constants. Therefore, we recommend that you modify the statement as follows:

    ``` {#codeblock_evm_j5w_lcj}
    insert into table srcp (p ='abc') select 'a',array('1', '2', '3');.
    ```

    which can provide the same effect.

-   To write the DATETIME and TIMESTAMP data types through values, specify the type names in the `VALUES` statement, for example:

    ``` {#codeblock_f7p_6ux_839}
    insert into table srcp (p ='abc') values (datetime'2017-11-11 
                00:00:00',timestamp'2017-11-11 00:00:00.123456789');
    ```


