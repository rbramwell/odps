---
keyword: [Python 3, UDF, UDAF, UDTF]
---

# Python 3 UDFs

Python Software Foundation announces the End of Life \(EOL\) for Python 2. Due to this reason, MaxCompute supports Python 3 and uses CPython 3.7.3. MaxCompute supports three types of Python 3 user-defined functions \(UDFs\): UDFs, user-defined aggregate functions \(UDAFs\), and user-defined table-valued functions \(UDTFs\). This topic describes how to implement these Python 3 UDFs.

## Limits

Python 3 is incompatible with Python 2. You can select Python 3 or Python 2 to execute an SQL statement, but cannot use both Python 3 and Python 2 code in a single SQL statement.

## Enable Python 3

You can enable Python 3 only for jobs. To enable Python 3 for a job, insert the following statement before the corresponding SQL statement and execute them together:

```
set odps.sql.python.version=cp37;
```

## Third-party libraries

MaxCompute Python UDFs support third-party libraries. In addition to the standard library, the third-party library NumPy is installed in the Python 2 runtime environment to supplement the standard library.

NumPy is not installed in the Python 3 runtime environment in MaxCompute. To use a NumPy UDF, you must manually upload a NumPy WHL file. If you download a NumPy WHL file from Python Package Index \(PyPI\) or an image, the file name is numpy-<Version\>-cp37-cp37m-manylinux1\_x86\_64.whl. For more information about how to upload a file, see [Resource operations](/intl.en-US/Development/Common commands/Resource operations.md).

## Port Python 2 UDFs

Python Software Foundation announces the EOL for Python 2. Therefore, we recommend that you port Python 2 UDFs. The method of porting Python 2 UDFs varies based on the project.

-   In a new project or an existing project where no Python UDFs exist, we recommend that you use Python 3 to write all Python UDFs.
-   In an existing project where a large number of Python 2 UDFs exist, exercise caution when you enable Python 3. You cannot use both Python 2 and Python 3 code in a single SQL statement. If you plan to gradually replace Python 2 UDFs with Python 3 UDFs, use the following methods:
    -   Use Python 3 to write new UDFs and enable Python 3 for new jobs.
    -   Rewrite Python 2 UDFs to make them compatible with both Python 2 and Python 3. For more information about how to rewrite UDFs, see [Porting Python 2 Code to Python 3](https://docs.python.org/3/howto/pyporting.html).

        **Note:** When you write a public UDF that needs to be shared among multiple projects, we recommend that you write the UDF to be compatible with both Python 2 and Python 3.


## Data types of parameters and return values

The data types of parameters and return values are specified by the following method:

```
@odps.udf.annotate(signature)
```

Python UDFs support the following MaxCompute SQL data types: BIGINT, STRING, DOUBLE, BOOLEAN, DATETIME, DECIMAL, ARRAY, MAP, STRUCT, and nested complex data types. Complex data types include ARRAY, MAP, and STRUCT. Before you execute an SQL statement, the data types of parameters and return values of all functions must be determined. Python is a dynamically typed language. You must add decorators to the UDF class to specify the function signature.

The function signature is specified by a string. The following syntax is supported:

```
arg_type_list '->' type_list
arg_type_list: type_list | '*' | '' | 'char(n)' | 'varchar(n)'
type_list: [type_list ','] type
type: 'bigint' | 'string' | 'double' | 'boolean' | 'datetime' | 'float' | 'binary' | 'date' | 'decimal' | 'decimal(precision,scale)'
```

-   The part to the left of the arrow indicates the data types of parameters. The part to the right of the arrow indicates the data types of return values.
-   The return value of a UDTF can contain multiple columns, whereas that of a UDF or UDAF contains only one column.
-   An asterisk \(`*`\) represents variable-length parameters. If variable-length parameters are used, UDFs, UDTFs, and UDAFs can match input parameters of any data type.
-   In a project that uses the MaxCompute V2.0 data type edition, you can set the decimal parameter to precision or scale. For more information, see [MaxCompute V2.0 data type edition](/intl.en-US/Development/Data types/MaxCompute V2.0 data type edition.md).
-   You cannot specify the CHAR or VARCHAR type for return values.

The following example shows a valid signature:

```
'bigint,double->string'            # The parameters are of the BIGINT and DOUBLE types and the return values are of the STRING type.
'bigint,boolean->string,datetime'  # The parameters are of the BIGINT and BOOLEAN types and the return values are of the STRING and DATETIME types.
'*->string'                        # The parameters are variable-length parameters and can be of any data type, and the return values are of the STRING type.
'->double'                         # The parameters are left empty and the return values are of the DOUBLE type.
'array<bigint>->struct<x:string, y:int>' # The parameters are of the ARRAY<BIGINT> type and the return values are of the STRUCT<x:STRING, y:INT> type.
'->map<bigint, string>'            # The parameters are left empty and the return values are of the MAP<BIGINT, STRING> type.
```

If an invalid signature is found during query parsing, an error is returned and the execution of the function is prohibited. During the execution of the function, the parameters of the type that is specified by the function signature are transferred. The return values must be of the same type as that specified by the function signature. Otherwise, an error is returned.

The following table lists the mappings between MaxCompute SQL data types and Python 3 data types.

|MaxCompute SQL data type|Python 3 data type|
|------------------------|------------------|
|BIGINT|INT|
|STRING|UNICODE|
|DOUBLE|FLOAT|
|BOOLEAN|BOOL|
|DATETIME|DATETIME.DATETIME|
|FLOAT|FLOAT|
|CHAR|UNICODE|
|VARCHAR|UNICODE|
|BINARY|BYTES|
|DATE|DATETIME.DATE|
|DECIMAL|DECIMAL.DECIMAL|
|ARRAY|LIST|
|MAP|DICT|
|STRUCT|COLLECTIONS.NAMEDTUPLE|

**Note:**

-   A value of the DATETIME type is converted to a value of the INT type. The value of the INT type indicates the number of milliseconds that have elapsed since the epoch time January 1, 1970, 00:00:00 UTC. You can process data of the DATETIME type by using the datetime module in the Python standard library.
-   `silent` is added to `odps.udf.int(value,[silent=True])`. If `silent` is set to True and `value` cannot be converted to the INT type, None is returned, and no error is reported.
-   NULL in MaxCompute SQL corresponds to None in Python.

## UDFs

To implement Python UDFs, you must define a new-style class and implement the `evaluate` method.

```
from odps.udf import annotate
@annotate("bigint,bigint->bigint")
class MyPlus(object):
   def evaluate(self, arg0, arg1):
       if None in (arg0, arg1):
           return None
       return arg0 + arg1
```

**Note:** A Python UDF must have its signature specified by using `annotate`.

## UDAFs

-   `class odps.udf.BaseUDAF`: the base class of Python UDAFs. It can be inherited to implement Python UDAFs.
-   `BaseUDAF.new_buffer()`: returns `buffer` of the intermediate value of a UDAF. `buffer` must be a [marshallable](https://docs.python.org/3.3/library/marshal.html#module-marshal) object such as a list or a dictionary, and the `buffer` size cannot increase with the data volume. Under extreme circumstances, the `buffer` size cannot exceed 2 MB after the marshal operation.
-   `BaseUDAF.iterate(buffer[, args, ...])`: aggregates `args` to `buffer` of the intermediate value.
-   `BaseUDAF.merge(buffer, pbuffer)`: aggregates `buffer` of two intermediate values. It merges `pbuffer` to `buffer`.
-   `BaseUDAF.terminate(buffer)`: converts `buffer` of the intermediate value to a value of a basic data type in MaxCompute SQL.

The following example shows how to use a UDAF to obtain the average value:

```
@annotate('double->double')
class Average(BaseUDAF):
    def new_buffer(self):
        return [0, 0]
    def iterate(self, buffer, number):
        if number is not None:
            buffer[0] += number
            buffer[1] += 1
    def merge(self, buffer, pbuffer):
        buffer[0] += pbuffer[0]
        buffer[1] += pbuffer[1]
    def terminate(self, buffer):
        if buffer[1] == 0:
            return 0.0
        return buffer[0] / buffer[1]
```

## UDTFs

-   `class odps.udf.BaseUDTF`: the base class of Python UDTFs. It can be inherited to implement the `process` and `close` methods.
-   `BaseUDTF.__init__()`: the initialization method. To implement this method for a derived class, you must call the `super(BaseUDTF, self).__init__()` initialization method of the base class at the beginning of code execution. Throughout the lifecycle of a UDTF, the `init` method is called only once. It is called only before the first record is processed. If a UDTF needs to save internal states, you can call the init method to initialize all states.
-   `BaseUDTF.process([args, ...])`: is called by the MaxCompute SQL framework. The `process` method is called to process each record involved in the SQL statements. The UDTF input parameters that are specified in the SQL statements are used as the parameters of the `process` method.
-   `BaseUDTF.forward([args, ...])`: the UDTF output method. This method is called by user code. One record is generated each time the `forward` method is called. The UDTF output parameters that are specified in the SQL statements are used as the parameters of the `forward` method.
-   `BaseUDTF.close()`: the UDTF termination method. Throughout the lifecycle of a UDTF, this method is called only once by the MaxCompute SQL framework. It is called only after the last record is processed.

The following example shows how to use a UDTF:

```
#coding:utf-8
# explode.py
from odps.udf import annotate
from odps.udf import BaseUDTF
@annotate('string -> string')
class Explode(BaseUDTF):
   """ Use commas (,) to separate the string into multiple records.
   def process(self, arg):
       props = arg.split(',')
       for p in props:
           self.forward(p)
```

**Note:** The data types of parameters and return values of a Python UDTF can be specified without the need to use `annotate` to specify the UDTF signature. In this case, the UDTF can match input parameters of any data type in SQL statements. However, the data types of return values cannot be inferred. All output parameters are considered to be of the STRING type. Therefore, when the `forward` method is called, all output values must be converted to the STRING type.

## Resource reference

You can reference file and table resources in Python UDFs by using the `odps.distcache` module.

-   `odps.distcache.get_cache_file(resource_name)`: returns the content of a specific file resource.

    -   `resource_name` is a string that corresponds to the name of an existing file resource in the current project. If the resource name is invalid or the file resource does not exist, an error is returned.

        **Note:** To reference a file resource in a UDF, you must declare the file resource when you create the UDF. Otherwise, an error is returned when the UDF is called.

    -   The return value is a file-like object. If this object is no longer used, the caller must call the `close` method to release the open file resource.
    Example:

    ```
    @annotate('bigint->string')
    class DistCacheExample(object):
    def __init__(self):
        cache_file = get_cache_file('test_distcache.txt')
        kv = {}
        for line in cache_file:
            line = line.strip()
            if not line:
                continue
            k, v = line.split()
            kv[int(k)] = v
        cache_file.close()
        self.kv = kv
    def evaluate(self, arg):
        return self.kv.get(arg)
    ```

-   `odps.distcache.get_cache_table(resource_name)`: returns the content of a specific table resource.
    -   `resource_name` can be of the BIGINT, STRING, DOUBLE, BOOLEAN, DATETIME, FLOAT, CHAR, VARCHAR, BINARY, DATE, DECIMAL, ARRAY, MAP, or STRUCT type. It corresponds to the name of an existing table resource in the current project. If the resource name is invalid or the table resource does not exist, an error is returned.
    -   The return value is of the generator type. The caller traverses the table to obtain the content. Each time the caller traverses the table, a record of the ARRAY type is obtained.

Example:

```
from odps.udf import annotate
from odps.distcache import get_cache_table
@annotate('->string')
class DistCacheTableExample(object):
    def __init__(self):
        self.records = list(get_cache_table('udf_test'))
        self.counter = 0
        self.ln = len(self.records)
    def evaluate(self):
        if self.counter > self.ln - 1:
            return None
        ret = self.records[self.counter]
        self.counter += 1
        return str(ret)
```

