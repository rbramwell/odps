---
keyword: [SQL function, ]
---

# SQL functions

SQL functions allow you to reference SQL user-defined functions \(UDFs\) in SQL scripts. This topic describes how to manage SQL functions.

## Background information

SQL functions of MaxCompute resolve the following issues:

-   Generally, a large amount of similar code exists, which is inconvenient to maintain and prone to errors. To use UDFs, you must develop code, compile the code in Java, and then create resources and functions. The process is complicated. In addition, UDFs cannot catch up with built-in functions in terms of performance. Example:

    ```
    SELECT
        NVL(STR_TO_MAP(GET_JSON_OBJECT(col, '$.key1')), 'default') AS key1,
        NVL(STR_TO_MAP(GET_JSON_OBJECT(col, '$.key2')), 'default') AS key2,
        ...
        NVL(STR_TO_MAP(GET_JSON_OBJECT(col, '$.keyN')), 'default') AS keyN
    FROM t;
    ```

-   One function can be transferred as a parameter to another only by using code similar to lambda expressions in Java.

## Features

As a type of UDFs, SQL functions allow you to create UDFs in SQL and use function type parameters and anonymous functions. In this way, you can define business logic more flexibly. You can use SQL functions to simplify feature implementation and improve code reuse. SQL functions provide the following features:

-   SQL functions allow you to reference and call SQL UDFs in SQL scripts.
-   You can specify built-in functions, UDFs, or SQL functions in function type parameters when you call SQL functions.
-   You can specify anonymous functions in function type parameters when you call SQL functions.

## Create a permanent SQL function

After you create a permanent SQL function and store it in the MaxCompute meta system, all query statements can reference this function. To reference an SQL function in a query statement, you must be granted relevant permissions on the function. For more information, see [Authorize users](/intl.en-US/Management/Configure security features/Manage users and permissions/Authorize users.md)and [Grant access to a specific UDF to a specified user](). Use the following syntax to create a permanent SQL function:

```
CREATE SQL FUNCTION function_name(@parameter_in1 datatype[, @parameter_in2 datatype...]) 
[RETURNS (@parameter_out1 datatype[, @parameter_out2 datatype...])] 
AS [BEGIN] 
function_expression 
[END];
```

-   function\_name: the name of the SQL function to create. The name must be unique in the system.
-   parameter\_in: the input parameters of the SQL function.
-   RETURNS: the variable to be returned by the SQL function. If you do not specify RETURNS, the value of function\_name is returned by default.
-   parameter\_out: the output parameters of the SQL function.
-   function\_expression: the expression of the SQL function.

Example:

```
CREATE SQL FUNCTION MY_ADD(@a BIGINT) AS @a + 1;
```

In the preceding example, `@a + 1` indicates the implementation logic of the SQL function. You can write it as an expression to specify a built-in operator, built-in function, or UDF.

If the implementation logic is complex, you can write multiple SQL statements and enclose them with BEGIN and END. RETURNS specifies the variable to be returned by the SQL function. If you do not specify RETURNS, the value of function\_name is returned by default. Example:

```
CREATE SQL FUNCTION MY_SUM(@a BIGINT, @b BIGINT, @c BIGINT) RETURNS @my_sum BIGINT
AS BEGIN 
    @temp := @a + @b;
    @my_sum := @temp + @c;
END;
```

**Note:** To write multiple SQL statements, you must use Script Mode SQL. For more information, see [Script Mode SQL](/intl.en-US/Development/SQL/Script Mode SQL.md).

## Query an SQL function

You can query an SQL function in the same way as querying a Java or Python UDF. Example:

```
DESC FUNCTION my_add;
```

**Note:** To query an SQL function on a client, make sure that the client version is later than 0.34.0.

## Delete an SQL function

You can delete an SQL function in the same way as deleting a Java or Python UDF. Example:

```
DROP FUNCTION my_add;
```

## Call an SQL function

You can call an SQL function in the same way as calling a built-in function. Example:

```
SELECT my_sum(col1, col2 ,col3) from t;
```

## Define a temporary SQL function

If you only need to call an SQL function in a specific SQL script, you can define a temporary SQL function in the script. The temporary SQL function is not stored in the MaxCompute meta system, and applies only to the current SQL script. Example:

```
FUNCTION MY_ADD(@a BIGINT) AS @a + 1;
SELECT MY_ADD(key), MY_ADD(value) FROM src;
```

**Note:** To write multiple SQL statements, you must use Script Mode SQL. For more information, see [Script Mode SQL](/intl.en-US/Development/SQL/Script Mode SQL.md).

## Use function type parameters

You can specify built-in functions, UDFs, or SQL functions in function type parameters when you call SQL functions. Example:

```
FUNCTION ADD(@a BIGINT) AS @a + 1;
FUNCTION OP(@a BIGINT, @fun FUNCTION (BIGINT) RETURNS BIGINT) AS @fun(@a);
SELECT OP(key, ADD), OP(key, ABS) FROM VALUES (1),(2) AS t (key);

-- Output
+------------+------------+
| _c0        | _c1        |
+------------+------------+
| 2          | 1          |
| 3          | 2          |
+------------+------------+
```

In the example, two input parameters are specified for the OP function. The `@a` parameter specifies a value of the BIGINT type. The `@fun` parameter specifies a function whose input and output parameters are both of the BIGINT type. The OP function transfers `@a` to the function that is specified by `@fun` and then to the ADD and ABS functions for processing.`` For more information about the ABS function, see [Mathematical functions](/intl.en-US/Development/SQL/Builtin functions/Mathematical functions.md).

## Use anonymous functions

You can specify anonymous functions in function type parameters when you call SQL functions. Example:

```
FUNCTION OP(@a BIGINT, @fun FUNCTION (BIGINT) RETURNS BIGINT) AS @fun(@a);
SELECT OP(key, FUNCTION (@a) AS @a + 1) FROM VALUES (1),(2) AS t (key);
```

In the example, `FUNCTION (@a) AS @a + 1` is an anonymous function. You do not need to specify a data type for the input parameter `@a`. The compiler will infer the data type of `@a` based on the parameter definition of the OP function.

