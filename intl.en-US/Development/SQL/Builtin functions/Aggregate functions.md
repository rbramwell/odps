---
keyword: aggregate function
---

# Aggregate functions

Aggregate functions group multiple input records together to form a single output record. The input and output records have a many-to-one relationship. Aggregate functions can be used together with the GROUP BY clause of SQL. This topic describes the syntax, functionality, parameters, and return value of each aggregate function that MaxCompute supports. This topic also provides examples that show how to use these aggregate functions.

## COUNT

-   Syntax

    ```
    BIGINT COUNT([distinct|all] value)
    ```

-   Description

    This function counts the number of records that match the specified criteria.

-   Parameters
    -   distinct\|all: specifies whether to remove duplicate records during counting. The default value is all, which indicates that all records are counted. If this parameter is set to distinct, only records with distinct values are counted.
    -   value: the column for the function to count the number of records. The column can be of any data type. If a value in the specified column is NULL, the row that contains this value is not counted. You can set value to an asterisk \(`*`\). This means that you can run `count(*)` to count all records.
-   Return value

    This function returns a value of the BIGINT type.

-   Examples

    ```
    -- Assume that the tbla table has a col1 column whose data type is BIGINT.
    +------+
    | COL1 |
    +------+
    | 1 |
    +------+
    | 2 |
    +------+
    | NULL |
    +------+
    SELECT COUNT(*) FROM tbla; -- The return value is 3.
    SELECT COUNT(col1) FROM tbla; -- The return value is 2.
    ```

    The COUNT function can be used together with the `GROUP BY` clause. Assume that the `test_src` table has two columns: `key` of the STRING type and `value` of the DOUBLE type.

    ```
    -- The test_src table contains the following data:
    +-----+-------+
    | key | value |
    +-----+-------+
    | a | 2.0 |
    +-----+-------+
    | a | 4.0 |
    +-----+-------+
    | b | 1.0 |
    +-----+-------+
    | b | 3.0 |
    +-----+-------+
    -- Execute the following statement:
    SELECT key, COUNT(value) AS count FROM test_src GROUP BY key;
    +-----+-------+
    | key | count |
    +-----+-------+
    | a | 2 |
    +-----+-------+
    | b | 2 |
    +-----+-------+
    -- The COUNT function separately counts the number of records with the same key. The usage of the following aggregate functions is similar to that of the COUNT function, and therefore is not described in detail in this topic.
    ```


## AVG

-   Syntax

    ```
    DOUBLE AVG(DOUBLE value)
    DECIMAL AVG(DECIMAL value)
    ```

-   Description

    This function calculates the average value of a column.

-   Parameters

    value: the column whose average value is to be calculated. The column must be of the DOUBLE or DECIMAL type. If the specified column is of the STRING or BIGINT type, the values in the column are implicitly converted to those of the DOUBLE type before calculation. If the specified column is of another data type, an error is returned. If a value in the specified column is NULL, the value is not included in the calculation. Values of the BOOLEAN type are not included in the calculation.

-   Return value

    If the specified column is of the DECIMAL type, this function returns a value of the DECIMAL type. If the specified column is of another valid data type, this function returns a value of the DOUBLE type.

-   Examples

    ```
    -- Assume that the tbla table has a value column whose data type is BIGINT.
    +-------+
    | value |
    +-------+
    | 1 |
    | 2 |
    | NULL |
    +-------+
    -- The average value of this column is: (1 + 2)/2 = 1.5.
    SELECT AVG(value) AS avg FROM tbla;
    +------+
    | avg |
    +------+
    | 1.5 |
    +------+
    ```


## MAX

-   Syntax

    ```
    MAX(value)
    ```

-   Description

    This function calculates the maximum value of a column.

-   Parameters

    value: the column whose maximum value is to be calculated. The column can be of any data type. If a value in the specified column is NULL, the value is not included in the calculation. Values of the BOOLEAN type are not included in the calculation.

-   Return value

    This function returns a value of the same type as value.

-   Examples

    ```
    -- Assume that the tbla table has a col1 column whose data type is BIGINT.
    +------+
    | col1 |
    +------+
    | 1 |
    +------+
    | 2 |
    +------+
    | NULL |
    +------+
    SELECT MAX(value) FROM tbla; -- The return value is 2.
    ```


## MIN

-   Syntax

    ```
    MIN(value)
    ```

-   Description

    This function calculates the minimum value of a column.

-   Parameters

    value: the column whose minimum value is to be calculated. The column can be of any data type. If a value in the specified column is NULL, the value is not included in the calculation. Values of the BOOLEAN type are not included in the calculation.

-   Examples

    ```
    -- Assume that the tbla table has a value column whose data type is BIGINT.
    +------+
    | value|
    +------+
    | 1 |
    +------+
    | 2 |
    +------+
    | NULL |
    +------+
    SELECT MIN(value) FROM tbla; -- The return value is 1.
    ```


## MEDIAN

-   Syntax

    ```
    DOUBLE MEDIAN(DOUBLE number)
    DECIMAL MEDIAN(DECIMAL number)
    ```

-   Description

    This function calculates the median value of a column.

-   Parameters

    value: the column whose median value is to be calculated. The column must be of the DOUBLE or DECIMAL type. If the specified column is of the STRING or BIGINT type, the values in the column are implicitly converted to those of the DOUBLE type before calculation. If the specified column is of another data type, an error is returned.

-   Return value

    This function returns a value of the DOUBLE or DECIMAL type.

-   Examples

    ```
    -- Assume that the tbla table has a value column whose data type is BIGINT.
    +------+
    | value|
    +------+
    | 1 |
    +------+
    | 2 |
    +------+
    | 3 |
    +------+
    | 4 |
    +------+
    | 5 |
    +------+
    SELECT MEDIAN(value) FROM tbla; -- The return value is 3.0.
    ```


## STDDEV

-   Syntax

    ```
    DOUBLE STDDEV(DOUBLE number)
    DECIMAL STDDEV(DECIMAL number)
    ```

-   Description

    This function calculates the population standard deviation of a column.

-   Parameters

    value: the column whose population standard deviation is to be calculated. The column must be of the DOUBLE or DECIMAL type. If the specified column is of the STRING or BIGINT type, the values in the column are implicitly converted to those of the DOUBLE type before calculation. If the specified column is of another data type, an error is returned.

-   Return value

    This function returns a value of the DOUBLE or DECIMAL type.

-   Examples

    ```
    -- Assume that the tbla table has a value column whose data type is BIGINT.
    +------+
    | value|
    +------+
    | 1 |
    +------+
    | 2 |
    +------+
    | 3 |
    +------+
    | 4 |
    +------+
    | 5 |
    +------+
    SELECT STDDEV(value) FROM tbla; -- The return value is 1.4142135623730951.
    ```


## STDDEV\_SAMP

-   Syntax

    ```
    DOUBLE STDDEV_SAMP(DOUBLE number)
    DECIMAL STDDEV_SAMP(DECIMAL number)
    ```

-   Description

    This function calculates the sample standard deviation of a column.

-   Parameters

    value: the column whose sample standard deviation is to be calculated. The column must be of the DOUBLE or DECIMAL type. If the specified column is of the STRING or BIGINT type, the values in the column are implicitly converted to those of the DOUBLE type before calculation. If the specified column is of another data type, an error is returned.

-   Return value

    This function returns a value of the DOUBLE or DECIMAL type.

-   Examples

    ```
    -- Assume that the tbla table has a value column whose data type is BIGINT.
    +------+
    | value|
    +------+
    | 1 |
    +------+
    | 2 |
    +------+
    | 3 |
    +------+
    | 4 |
    +------+
    | 5 |
    +------+
    SELECT STDDEV_SAMP(value) FROM tbla; -- The return value is 1.5811388300841898.
    ```


## SUM

-   Syntax

    ```
    SUM(value)
    ```

-   Description

    This function calculates the sum of a column.

-   Parameters

    value: the column whose sum is to be calculated. The column must be of the DOUBLE, DECIMAL, or BIGINT type. If the specified column is of the STRING type, the values in the column are implicitly converted to those of the DOUBLE type before calculation. If a value in the specified column is NULL, the value is not included in the calculation. Values of the BOOLEAN type are not included in the calculation.

-   Return value

    If the specified column is of the BIGINT type, this function returns a value of the BIGINT type. If the specified column is of the DOUBLE or STRING type, this function returns a value of the DOUBLE type. If the specified column is of the DECIMAL type, the function returns a value of the DECIMAL type.

-   Examples

    ```
    -- Assume that the tbla table has a value column whose data type is BIGINT.
    +------+
    | value|
    +------+
    | 1 |
    +------+
    | 2 |
    +------+
    | NULL |
    +------+
    SELECT SUM(value) FROM tbla; -- The return value is 3.
    ```


## WM\_CONCAT

-   Syntax

    ```
    STRING WM_CONCAT(STRING separator, STRING str)
    ```

-   Description

    This function uses the delimiter that is specified by separator to connect the values in str.

-   Parameters
    -   separator: the delimiter, which is a constant of the STRING type. If you specify a constant of another type or a non-constant, an error is returned.
    -   str: the values to be connected, which must be of the STRING type. If the specified values are of the BIGINT, DOUBLE, or DATETIME type, they are implicitly converted to those of the STRING type before connection. If other types of values are specified, an error is returned.
-   Return value

    This function returns a value of the STRING type.

    **Note:** If `test_src` in the `SELECT WM_CONCAT(',', name) FROM test_src;` statement is an empty set, NULL is returned.

-   Examples: Connect values after grouping and sorting.

    ```
    -- Create a table named test.
    CREATE TABLE test(id int , alphabet string);
    -- Insert data to the test table.
    INSERT INTO test VALUES (1,'a'),(1,'b'),(1,'c'),(2,'D'),(2,'E'),(2,'F');
    -- Group and sort values in the table based on the id column. Then, connect values in the same group.
    SELECT id,WM_CONCAT('',alphabet) FROM test GROUP BY id ORDER BY id LIMIT 100;
    +------------+------------+
    | id         | _c1        |
    +------------+------------+
    | 1          | abc        |
    | 2          | DEF        |
    +------------+------------+
    ```


## COLLECT\_LIST

-   Syntax

    ```
    ARRAY COLLECT_LIST(col)
    ```

-   Description

    This function aggregates the expressions that are specified by col in a given group into an array.

-   Parameters

    col: a table column, which can be of any data type.

-   Return value

    This function returns a value of the ARRAY type.


**Note:** If you need to use new data types, such as TINYINT, SMALLINT, INT, FLOAT, VARCHAR, TIMESTAMP, and BINARY, in MaxCompute SQL, you must enable the new data types.

-   Session level: To enable new data types at the session level, you must insert `set odps.sql.type.system.odps2=true;` before the corresponding SQL statement and execute them together.
-   Project level: The owner of a project can enable new data types for the project by running the following command:

    ```
    setproject odps.sql.type.system.odps2=true;
    ```

    For more information about `SETPROJECT`, see [Project operations](/intl.en-US/Development/Common commands/Project operations.md). For more information about how to enable data types at the project level, see [Date types](/intl.en-US/Development/Data types/Data type editions.md).


## COLLECT\_SET

-   Syntax

    ```
    ARRAY COLLECT_SET(col)
    ```

-   Description

    This function aggregates the expressions that are specified by col in a given group into an array without duplicate elements.

-   Parameters

    col: a table column, which can be of any data type.

-   Return value

    This function returns a value of the ARRAY type.


**Note:** If you need to use new data types, such as TINYINT, SMALLINT, INT, FLOAT, VARCHAR, TIMESTAMP, and BINARY, in MaxCompute SQL, you must enable the new data types.

-   Session level: To enable new data types at the session level, you must insert `set odps.sql.type.system.odps2=true;` before the corresponding SQL statement and execute them together.
-   Project level: The owner of a project can enable new data types for the project by running the following command:

    ```
    setproject odps.sql.type.system.odps2=true;
    ```

    For more information about `SETPROJECT`, see [Project operations](/intl.en-US/Development/Common commands/Project operations.md). For more information about how to enable data types at the project level, see [Date types](/intl.en-US/Development/Data types/Data type editions.md).


## NUMERIC\_HISTOGRAM

-   Syntax

    ```
    MAP<DOUBLE, DOUBLE> NUMERIC_HISTOGRAM(BIGINT buckets, DOUBLE value)
    ```

-   Description

    This function returns the approximate histogram of a specific column.

-   Parameters
    -   buckets: the number of buckets for grouping values in the column whose approximate histogram is to be returned. The value must be of the BIGINT type.
    -   value: the column whose approximate histogram is to be returned. The column must be of the DOUBLE type.
-   Return value

    This function returns a value of the `MAP<DOUBLE,DOUBLE>` type. In the return value, a key indicates an x coordinate, and its value indicates the height on the approximate histogram of the column.


## PERCENTILE\_APPROX

-   Syntax

    ```
    DOUBLE PERCENTILE_APPROX(DOUBLE col, p [, B])) 
    ARRAY<DOUBLE>  PERCENTILE_APPROX(DOUBLE col, ARRAY(p1 [, p2]...) [, B])
    ```

-   Description

    This function returns the approximate percentile value of column col at the percentage that is specified by p.

-   Parameters
    -   p: the percentage that must range from 0.0 to 1.0. You can specify whether to return an approximate percentile value or an array that consists of multiple percentile values.
    -   B: the accuracy of the return value. A higher accuracy indicates a more accurate value. The default value is 10000. If the number of distinct values in column col is less than B, the exact percentile value is returned.
-   Return value

    `percentile_approx(double col, p [, B]))` returns an approximate percentile value. `percentile_approx(double col, array(p1 [, p2]...) [, B])` returns an array that consists of multiple percentile values.

-   Examples

    ```
    SELECT PERCENTILE_APPROX(10.0, 0.5, 100); -- The return value is 10.0.
    SELECT PERCENTILE_APPROX(10.0, array(0.5, 0.4, 0.1), 100); -- The return value is [10.0,10.0,10.0].
    ```


## APPROX\_DISTINCT

-   Syntax

    ```
    APPROX_DISTINCT(value)
    ```

-   Description

    This function returns the approximate number of distinct input values.

-   Parameters

    value: the input values.

-   Return value

    This function returns a value of the BIGINT type. If all input values are NULL, this function returns zero. This function produces a standard error of 5%.

-   Examples

    ```
    SELECT key, APPROX_DISTINCT(value) 
    FROM  VALUES
        (1, 99),
        (1, 100),
        (2, 100),
        (2, 100),
        (3, NULL) 
    AS t(key,value) 
    GROUP BY key;
    -- Output
    +------------+------------+
    | key        | _c1        |
    +------------+------------+
    | 1          | 2          |
    | 2          | 1          |
    | 3          | 1          |
    +------------+------------+
    ```


## ANY\_VALUE

-   Syntax

    ```
    ANY_VALUE(value)
    ```

-   Description

    This function returns a non-deterministic value from a specific column.

-   Parameters

    value: the column from which a single non-deterministic value is to be returned. The column can be of any data type. If a value in the specified column is NULL, the value is ignored.

-   Return value

    This function returns a value of the same type as the input values.

-   Examples

    ```
    SELECT key, ANY_VALUE(value) 
    FROM VALUES 
        (1, 'value1'),
        (1, 'value2'),
        (2, 'value3'),
        (2, NULL),
        (3, NULL) 
    AS t(key, value) 
    GROUP BY key;
    -- Output
    +------------+------------+
    | key        | _c1        |
    +------------+------------+
    | 1          | value1     |
    | 2          | value3     |
    | 3          | NULL       |
    +------------+------------+
    ```


## ARG\_MAX

-   Syntax

    ```
    ARG_MAX(valueToMaximize, valueToReturn)
    ```

-   Description

    This function finds the row where the maximum value of valueToMaximize resides and then returns the value of valueToReturn in the row.

-   Parameters
    -   valueToMaximize: the column that contains the values to compare.
    -   valueToReturn: the column that contains the value to return.
-   Return value

    This function returns a value of the same type as valueToReturn.

-   Examples

    ```
    SELECT key, ARG_MAX(comp, value) 
    FROM VALUES 
        (1, 99,   'value1'),
        (1, 100,  'value2'),
        (2, 99,   'value3'),
        (2, 100,   NULL),
        (3, NULL, 'value4') 
    AS t(key,comp, value) 
    GROUP BY key;
    
    -- Output
    +------------+------------+
    | key        | _c1        |
    +------------+------------+
    | 1          | value2     |
    | 2          | NULL       |
    | 3          | NULL       |
    +------------+------------+
    ```


## ARG\_MIN

-   Syntax

    ```
    ARG_MIN(valueToMinimize, valueToReturn)
    ```

-   Description

    This function finds the row where the minimum value of valueToMinimize resides and then returns the value of valueToReturn in the row.

-   Parameters
    -   valueToMinimize: the column that contains the values to compare.
    -   valueToReturn: the column that contains the value to return.
-   Return value

    This function returns a value of the same type as valueToReturn.

-   Examples

    ```
    SELECT key, ARG_MIN(comp, value) 
    FROM VALUES 
      (1, 99, 'value1'),
      (1, 100,  'value2'),
      (2, 99, 'value3'),
      (2, 100, NULL),
      (3, NULL, 'value4') 
    AS t(key,comp, value) 
    GROUP BY key;
    
    -- Output
    +------------+------------+
    | key        | _c1        |
    +------------+------------+
    | 1          | value1     |
    | 2          | value3     |
    | 3          | NULL       |
    +------------+------------+
    ```


