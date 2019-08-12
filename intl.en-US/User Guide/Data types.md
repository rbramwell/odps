# Data types {#concept_jhp_4bb_5db .concept}

This topic describes the data types supported by MaxCompute 2.0, including basic data types and complex data types.

## Basic data types {#section_b53_rbb_5db .section}

The following table lists the basic data types supported by MaxCompute 2.0. To use a new data type, you must configure the **set odps.sql.type.system.odps2=true;** or **setproject odps.sql.type.system.odps2=true;** statement as needed, and pay attention to the impact on the existing tasks. If you do not set the statement, an error `xxxx type is not enabled in current mode` will occur.

**Note:** The statement `odps.sql.type.system.odps2` for enabling new data types supports only lowercase letters.

|Type|New in MaxCompute 2.0?|Constant|Description|
|:---|:---------------------|:-------|:----------|
|TINYINT|Yes|1Y, -127Y|The 8-bit signed integer type. Range: -128 to 127|
|SMALLINT|Yes|32767S, -100S|The 16-bit signed integer type. Range: -32768 to 32767|
|INT|Yes|1000, -15645787|The 32-bit signed integer type. Range: \(-2\) 31 to 2 31 - 1 \(note 1 and note 2\)|
|BIGINT|No|100000000000L, -1L|The 64-bit signed integer type. Range: \(-2\) 63 + 1 to 2 63 - 1|
|FLOAT|Yes|N/A|The 32-bit binary float type|
|DOUBLE|No|3.1415926 1E+7|The 64-bit binary float type|
|DECIMAL|No|3.5BD, 99999999999.9999999BD|The decimal precision number type. Integer range: \(-10\) 36 + 1 to 10 36 - 1; decimal part: accurate to 10 -18 \(note 5\)|
|VARCHAR\(n\)|Yes|N/A|The variable-length character type. 'n' is the length, and the range is 1 to 65535 \(note 3\).|
|STRING|No|"abc", 'bcd', "alibaba", 'inc'|The string type. Maximum length: 8 MB \(note 4\)|
|BINARY|Yes|N/A|The binary data type. Maximum length: 8 MB|
|DATETIME|No|DATETIME '2017-11-11 00:00:00'|The date time type. Range: January 1, 0000 to December 31, 9999, accurate to milliseconds \(note 6\)|
|TIMESTAMP|Yes|TIMESTAMP '2017-11-11 00:00:00.123456789'|The timestamp type that is irrelevant to time zones. Range: January 1, 0000 to 23.59:59.999999999 December 31, 9999 **Note:** For some time zone-related functions, such as cast\(<a timestamp\> as string\), timestamps that are irrelevant to the time zone must be displayed in accordance with the current time zone.

 |
|BOOLEAN|No|True, False|The Boolean type. Value: True, False|

**Note:** The data types listed in the preceding table can be NULL.

When you use new data types in MaxCompute, pay attention to the following information:

-   Applicable scenarios of new data types:
    -   MaxCompute SQL
    -   The latest version of MapReduce
    -   SDK 0.27.2 \(public and later versions\), and client 0.27.0 and later versions
-   New data type flag

    Currently, new data types cannot be directly used by default. To use a new data type, run a flag command. Note that only lowercase letters are supported in the flag command.

    -   **Session level:** To use new data types \(such as TINYINT, SMALLINT, INT, FLOAT, VARCHAR, TIMESTAMP, and BINARY\) in SQL or MapReduce \(the latest version\), add a set statement before the table statement `set odps.sql.type.system.odps2=true;` and submit the two statements. The SQL statements submitted through MaxCompute Studio are performed automatically. For more information, see [Create and view a table](../../../../reseller.en-US/Quick Start/Create and view a table.md#).
    -   **Project level:** New data types at the project level can be used. To use the new data types at the project level, the project owner must run the following command:

        ``` {#codeblock_3qg_yam_6o1}
        setproject odps.sql.type.system.odps2=true;
        ```

        For more information about the setproject command, see [Other operations](reseller.en-US/User Guide/Common commands/Other operations.md#).

-   Impact of using the new data type odps.sql.type.system.odps2 :
    -   **Note 1:** 

        If the new data type flag is enabled, INT in the SQL statement indicates a 32-bit integer. For example, if you run the cast\(a as INT\) command, type a is converted to a 32-bit integer. If the new data type flag is not enabled, type a is converted to BIGINT \(64-bit\).

        For example, the cast\(a as INT\) command is equivalent to cast\(a as BIGINT\), create table t\(a INT\), and create table a \(BIGINT\). If you do not set **odps.sql.type.system.odps2** to **true**, you can also convert data types in MaxCompute. However, a warning will be prompted, indicating that INT is processed as BIGINT. Therefore, we recommend that you write BIGINT in your scripts.

    -   The syntax of an integer constant changes.

        In `SELECT 1 + a;`, the integer constant is 1.

        -   If the new data type flag is not enabled, the integer constant is processed as BIGINT. If the constant length \(such as 1000000000000000000000000\) exceeds the value range of BIGINT, the integer constant is processed as DOUBLE.
        -   If the new data type flag is enabled, the integer constant is expressed as 1 \(a 32-digit integer\). If the constant value is greater than the maximum INT value but smaller than the maximum BIGINT value, the integer is converted to BIGINT. If the constant value is greater than the maximum BIGINT value, the integer is converted to DOUBLE.
        -   **Possible compatibility issues:** The INT type may lead to inconsistencies in function prototypes during subsequent operations. For example, the actions of peripheral tools and subsequent jobs might be changed by new type tables generated after data is written to disk.
    -   The rules for converting implicit data types change.

        If the new data type flag is enabled, some implicit types may not be converted. For example, the precision of conversions from STRING to BIGINT, STRING to DATETIME, DOUBLE to BIGINT, DECIMAL to DOUBLE, and DECIMAL to BIGINT might be reduced. In this case, you can use the CAST function to convert the data type.

        Converting implicit data types affects insert and the calling of functions. You can run the SQL statements that meet the requirements before enabling the new data type flag. However, an error might be displayed after the new data type flag is enabled.

    -   The supported operations, built-in functions, and UDFs are different.

        If the new type flag is not enabled, some operations and built-in functions that use new types as parameters and response values are not available.

        1.  Some built-in functions can be performed only after the new type flag is enabled, for example, most of the functions that use INT as parameters have BIGINT overloading. Functions such as YEAR, QUARTER, MONTH, DAY, HOUR, MINUTE, SECOND, MILLISECOND, NANOSECOND, DAYOFMONTH, and WEEKOFYEAR can be implemented by using DATEPART [built-in functions](reseller.en-US/User Guide/SQL/Builtin functions/Date functions.md#).
        2.  The UDF resolutions are different. For example, a UDF contains both BIGINT and INT. Before the new type flag is enabled, the resolution is BIGINT. After the new type flag is enabled, the resolution is INT.
    -   The BIGINT resolutions are different.

        An integer constant, such as 123, is of the BIGINT type before a new data type is used. The integer constant becomes INT type after a new data type is used.

        **Possible compatibility issues:** The INT type may lead to inconsistencies in function prototypes during subsequent operations. For example, the actions of peripheral tools and subsequent jobs might be changed by new type tables generated after data is written to disk.

    -   Types of supported partition columns are different.
        -   Before the new type flag is enabled, only STRING is supported by partition columns.
        -   After the new type flag is enabled, data types such as STRING, VARCHAR, CHAR, TINYINT, SMALLINT, INT, and BIGINT are supported by partition columns.
        -   If the new type flag is not enabled, partition fields in [INSERT operations](reseller.en-US/User Guide/SQL/Insert Operation/INSERT OVERWRITE__INTO.md#) must be processed as STRING. For example, you can enter `insert overwrite table t partition (pt = 045) select ...`. Note that 045 is not enclosed by quotation marks \(""\) and is resolved as an integer \(45\).
    -   Action of the LIMIT statement changes.

        Take the `SELECT * FROM t1 UNION ALL SELECT * FROM t2 limit 10;` statement as an example:

        -   Before the new type flag is enabled, the statement is `SELECT * FROM t1 UNION ALL SELECT * FROM ( SELECT * FROM t2 limit 10) t2;`.
        -   After the new type flag is enabled, the statement is `SELECT * FROM (SELECT * FROM t1 UNION ALL SELECT * FROM t2 ) t limit 10;`.
        Actions of the [UNION, INTERSECT, and EXCEPT](reseller.en-US/User Guide/SQL/Select Operation/UNION, INTERSECT, and EXCEPT.md#), INTERSECT, EXCEPT, LIMIT, ORDER BY, DISTRIBUTE BY, SORT BY, and CLUSTER BY statements are also changed if the new type flag is enabled.

    -   The resolutions of the IN expression are different.

        Take the IN expression, such as a in \(1, 2, 3\), as an example:

        -   Before the new type flag is enabled, all the values in the parentheses \(\) must be of the same type.
        -   After the new type flag is enabled, all the values in the parentheses \(\) can be converted to the same implicit type.
-   **Note 2:** If a constant value is greater than the maximum INT value but smaller than the maximum BIGINT value, the integer is converted to BIGINT. If the constant value is greater than the maximum BIGINT value, the integer is converted to DOUBLE. In MaxCompute versions earlier than 2.0, all INT types in SQL scripts are converted to BIGINT. The following is an example:

    ``` {#codeblock_feq_trs_rmz}
    create table a_bigint_table(a int); -- Here INT is processed as BIGINT.
    select cast(id as int) from mytable; -- Here INT is processed as BIGINT.
    ```

-   **Note 3:** VARCHAR constants can be expressed through implicit transformation of STRING constants.
-   **Note 4:** STRING constants can be connected to different strings, for example, `abc` and `xyz` are resolved as `abcxyz`, and can be written into different row.
-   **Note 5:**: When you insert a constant to the DECIMAL field, ensure that the constant is in the same format as that in the constant definition, for example, 3.5BD in the following command:

    ``` {#codeblock_kvr_o7b_xc3}
    insert into test_tb(a) values (3.5BD)
    ```

-   **Note 6:** The displayed time value does not contain milliseconds. You can use the tunnel command and `-dfp` to specify the time format, for example, `tunnel upload -dfp 'yyyy-MM-dd HH:mm:ss.SSS'`. For more information about tunnel commands, see [Tunnel commands](reseller.en-US/User Guide/Data upload and download/Tunnel commands.md#).

## Complex data types {#section_r53_rbb_5db .section}

The following table lists the complex data types that are supported by MaxCompute 2.0.

|Type|Definition method|Construction method|
|:---|:----------------|:------------------|
|ARRAY| -   array<int\>
-   array<struct<a:int, b:string\>\>

 | -   array\(1, 2, 3\)
-   array\(array\(1, 2\), array\(3, 4\)\)

 |
|MAP| -   map<string, string\>
-   map<smallint, array<string\>\>

 | -   map\(“k1”, “v1”, “k2”, “v2”\)
-   map\(1S, array\(‘a’, ‘b’\), 2S, array\(‘x’, ‘y’\)\)

 |
|STRUCT| -   struct<x:int, y:int\>
-   struct<field1:bigint, field2:array<int\>, field3:map<int, int\>\>

 | -   named\_struct\(‘x’, 1, ‘y’, 2\)
-   named\_struct\(‘field1’, 100L, ‘field2’, array\(1, 2\), ‘field3’, map\(1, 100, 2, 200\)\)

 |

**Note:** Complex data types are nestable. For the related built-in functions, see [ARRAY](reseller.en-US/User Guide/SQL/Builtin functions/Other functions.md#section_zcz_4lb_wdb), [MAP](reseller.en-US/User Guide/SQL/Builtin functions/Other functions.md#section_bzn_hcb_wdb), [STRUCT](reseller.en-US/User Guide/SQL/Builtin functions/Other functions.md#section_ifb_2nb_wdb).

