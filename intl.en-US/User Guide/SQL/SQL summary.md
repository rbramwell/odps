# SQL summary {#concept_awk_jmb_5db .concept}

This article introduces you to MaxCompute SQL keywords, type conversion instructions, partition tables, UNION ALL operations and use restrictions.

## SQL summary {#section_asv_hvg_hfb .section}

MaxCompute SQL is suitable for various scenarios. The massive data \(GB, TB, or EB level\) must be processed based on an offline batch calculation. It takes several seconds or even minutes to schedule after a job is submitted. Therefore, MaxCompute SQL is preferred for services that process tens of thousands of transactions per second.

The MaxCompute SQL syntax is similar to SQL and can be considered as a subset of standard SQL. However, the MaxCompute SQL must not be confused with a database. It does not have database characteristics including transactions, primary key constraints, indexes, and so on. The maximum size of SQL in MaxCompute is 2 MB.

## Reserved words {#section_nzx_m2l_vdb .section}

MaxCompute SQL considers the keywords of SQL statement as reserved words. If you use keywords for name tables, columns, or partitions, you must escape the keywords with the ```` symbol, otherwise an error is occurred. Reserved words are case insensitive and the most common words used are as follows: \(For a complete reserved word list, see [MaxCompute SQL Reserved Word](reseller.en-US/User Guide/SQL/Appendix/Reserved words and keywords.md#)\).

``` {#codeblock_izf_gmr_yg5}
% & && ( ) * +  
 - . / ; < <= <>  
 =    >    >=    ?    ADD ALL ALTER  
 AND AS ASC BETWEEN BIGINT BOOLEAN BY  
 CASE CAST COLUMN COMMENT CREATE DESC DISTINCT  
 DISTRIBUTE DOUBLE DROP ELSE FALSE FROM FULL  
 GROUP IF IN INSERT INTO IS JOIN  
 LEFT LIFECYCLE LIKE LIMIT MAPJOIN NOT NULL  
 ON OR ORDER OUTER OVERWRITE PARTITION RENAME  
 REPLACE RIGHT RLIKE SELECT SORT STRING TABLE  
 THEN TOUCH TRUE UNION VIEW WHEN WHERE
```

## Type conversion {#section_k1n_qfl_vdb .section}

MaxCompute SQL allows conversion between data types. The conversion methods include explicit type conversion and implicit type conversion. For more information, see [Type Conversion](reseller.en-US/User Guide/SQL/Type conversions.md#).

-   Explicit conversions: Uses CAST to convert a value type.
-   Implicit conversions: MaxCompute automatically performs implicit conversions while running based on the context environment and conversion rules. Implicit conversion scope includes various operators, built-in functions, and so on.

## Partitioned table {#section_m1n_qfl_vdb .section}

MaxCompute SQL supports partitioned tables. Specify the partition as it simplifies the operation. For example, improve SQL running efficiency, reduce the cost, and so on. For more information, see [Basic concept\>Partition](reseller.en-US/Product Introduction/Definitions/Partition.md#).

## UNION ALL {#section_n1n_qfl_vdb .section}

To be involved in a UNION ALL operation, the data type of columns, column numbers, and column names must be consistent, otherwise an error occurs.

