---
keyword: UDT usage example
---

# Usage examples

This topic describes common usage examples of user-defined type \(UDT\). For example, you can use Java arrays, JSON, complex data types, and table-valued functions in UDT. You can also aggregate data, overload functions, and reference embedded code in UDT.

## Use Java arrays

```
set odps.sql.type.system.odps2=true;
set odps.sql.udt.display.tostring=true;
SELECT
    new Integer[10],    -- Create an array that contains 10 elements.
    new Integer[] {c1, c2, c3},  -- Create an array that contains three elements by initializing an ArrayList.
    new Integer[][] { new Integer[] {c1, c2}, new Integer[] {c3, c4} },  -- Create a multidimensional array.
    new Integer[] {c1, c2, c3} [2], -- Access the elements in the array by using indexes.
    java.util.Arrays.asList(c1, c2, c3);    -- Create a list of the List<Integer> type, which can be used as an array of the Array<Int> type. This is another way to create a built-in array.
FROM VALUES (1,2,3,4) AS t(c1, c2, c3, c4);
```

## Use JSON

The runtime of UDT carries a JSON dependency \(version 2.2.4\), which can be directly used in JSON.

**Note:** In addition to JSON dependencies, MaxCompute runtime also carries other dependencies, including `commons-logging (1.1.1)`, `commons-lang (2.5)`, `commons-io (2.4)`, and `protobuf-java (2.4.1)`.

```
set odps.sql.type.system.odps2=true;
set odps.sql.session.java.imports=java.util.*,java,com.google.gson. *; -- To import multiple packages at a time, separate the packages with commas (,).
@a := select new Gson() gson;   -- Create a Gson object.
select 
gson.toJson(new ArrayList<Integer>(Arrays.asList(1, 2, 3))), -- Convert an object to a JSON string.
cast(gson.fromJson('["a","b","c"]', List.class) as List<String>) -- Deserialize the JSON string. Gson forcibly converts the deserialization result from the List<Object> type to the List<String> type.
from @a;
```

Compared with the built-in function [GET\_JSON\_OBJECT](/intl.en-US/Development/SQL/Builtin functions/String functions.md), this UDT-based method is simpler. In addition, this method deserializes the content that is extracted from the JSON string to a supported data type, thereby improving efficiency.

## Use complex data types

The built-in data types ARRAY and MAP map the `java.util.List` and `java.util.Map` methods, respectively.

-   Java objects in classes that implement `java.util.List` or `java.util.Map` can be used in complex-type data processing in MaxCompute SQL.
-   MaxCompute can directly call java.util.List or java.util.Map to process data of the ARRAY or MAP type.

```
set odps.sql.type.system.odps2=true;
set odps.sql.session.java.imports=java.util.*;
select
    size(new ArrayList<Integer>()),        -- Call the built-in function size() to obtain the size of the ArrayList.
    array(1,2,3).size(),                   -- Call java.util.List to process data of the ARRAY type.
    sort_array(new ArrayList<Integer>()),  -- Sort the data in the ArrayList.
    al[1],                                 -- java.util.List does not support indexing but can process data of the ARRAY type that supports indexing.
    Objects.toString(a),        -- Convert data from the ARRAY type to the STRING type.
    array(1,2,3).subList(1, 2)             -- Obtain a sublist.
from (select new ArrayList<Integer>(array(1,2,3)) as al, array(1,2,3) as a) t;
```

## Aggregate data

To use UDT to achieve aggregation, you must first use the built-in function [COLLECT\_SET](/intl.en-US/Development/SQL/Builtin functions/Aggregate functions.md) or [COLLECT\_LIST](/intl.en-US/Development/SQL/Builtin functions/Aggregate functions.md) to aggregate the data to a list, and then call the scalar method of UDT to calculate the aggregate value.

The following example shows how to calculate the median of BigInteger data. You cannot directly call the built-in function [MEDIAN](/intl.en-US/Development/SQL/Builtin functions/Aggregate functions.md) because the data is of the `java.math.BigInteger` type.

```
set odps.sql.session.java.imports=java.math.*;
@test_data := select * from values (1),(2),(3),(5) as t(value);
@a := select collect_list(new BigInteger(value)) values from @test_data;  -- Aggregate the data to a list.
@b := select sort_array(values) as values, values.size() cnt from @a;  -- Sort the data.
@c := select if(cnt % 2 == 1, new BigDecimal(values[cnt div 2]), new BigDecimal(values[cnt div 2 - 1].add(values[cnt div 2])).divide(new BigDecimal(2))) med from @b;
-- The final output.
select med.toString() from @c;
```

You cannot use the `collect_list` function to aggregate partial data because it can only aggregate all data in a specific group. It is less efficient than built-in aggregate functions of MaxCompute or user-defined aggregate functions \(UDAFs\). We recommend that you use built-in aggregate functions if possible. Aggregating all data in a group increases the risk of data skew.

The UDT-based method produces a higher efficiency than a built-in aggregate function such as `WM_CONCAT` in aggregating all data in a group.

## Implement table-valued functions

Table-valued functions allow you to specify multiple input rows and columns, and can generate multiple output rows and columns. To implement a table-valued function, perform the following steps:

-   Specify multiple input rows or columns. For more information, see [Aggregate data](#section_jei_63w_tbz).
-   Call the java.util.List or java.util.Map method to generate a data collection, and then call the explode function to split the collection into multiple rows.
-   Call different getter methods to obtain data from different fields in UDT. The obtained data is returned in multiple columns.

The following example shows how to split a JSON string and return the splitting result in multiple columns:

```
@a := select '[{"a":"1","b":"2"},{"a":"1","b":"2"}]' str; -- The sample data.
@b := select new com.google.gson.Gson().fromJson(str, java.util.List.class) l from @a; -- Deserialize the JSON string.
@c := select cast(e as java.util.Map<Object,Object>) m from @b lateral view explode(l) t as e;  -- Call the explode function to split the string.
@d := select m.get('a') as a, m.get('b') as b from @c; -- Return the splitting result in multiple columns.
select a.toString() a, b.toString() b from @d; -- The final output. Columns a and b in the variable d are of the Object type.
```

## Overload functions

In MaxCompute, user-defined functions \(UDFs\) overload functions by overloading the `evaluate` method. Generics are not supported in this mode. Therefore, you must write an `evaluate` method for each supported data type when you define a function. Functions with some input types such as ARRAY cannot be overloaded in this mode. If the Resolve annotation is unavailable, Python UDFs or user-defined table-valued functions \(UDTFs\) determine input parameters based on the number of parameters and support variable-length parameters. However, when this mechanism is used, the compiler does not detect specific errors in static mode.

In this case, you can use UDT to overload functions. UDT supports generics, class inheritance, and variable-length parameters, and offer flexible methods to define functions. Example:

```
public class UDTClass {
    // This function supports a numeric-type parameter and returns a value of the DOUBLE type. The parameter can be of the TINYINT, SMALLINT, INT, BIGINT, FLOAT, or DOUBLE type. You can also specify a user-defined data type that is defined based on a numeric type in the parameter.
    public static Double doubleValue(Number input) {
        return input.doubleValue();
    }
    // This function supports a numeric-type parameter and a parameter of any data type. The return value is of the same type as the second parameter.
    public static <T extends Number, R> R nullOrValue(T a, R b) {
        return a.doubleValue() > 0 ? b : null;
    }
    // This function supports a parameter of the ARRAY or LIST type. The parameter value can contain any type of elements. The function returns a value of the BIGINT type.
    public static Long length(java.util.List<? extends Object> input) {
        return input.size();
    }
    // If the code does not forcibly convert the data type, you must specify the default map object java.util.Map<Object, Object> of UDT. To transfer another map object, for example, map<bigint,bigint>, consider using the following method:
    // 1. When you define the function, use java.util.Map<? extends Object, ? extends Object>.
    // 2. When you call the function, implement a cast function to forcibly convert the type, for example, UDTClass.mapSize(cast(mapObj as java.util.Map<Object, Object>)).
    public static Long mapSize(java.util.Map<Object, Object> input) {
        return input.size();
    }
}
```

In specific scenarios, a UDF can use `com.aliyun.odps.udf.ExecutionContext` that is transferred by the `setup` method to obtain the context. UDT can call the `com.aliyun.odps.udt.UDTExecutionContext.get()` method to obtain the `ExecutionContext` object.

## Reference embedded code in UDT

Code-embedded UDFs allow you to place SQL scripts and third-party code lines in the same source code file. This simplifies the usage of UDT and facilitates daily development and maintenance. For more information, see [Code-embedded UDFs](/intl.en-US/Development/SQL/UDF/Code-embedded UDFs.md).

