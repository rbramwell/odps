---
keyword: [execute\_sql, run\_sql]
---

# SQL

PyODPS supports MaxCompute SQL queries and provides methods to obtain execution results.

## Execute SQL statements

You can call the `execute_sql()` and `run_sql()` methods of a MaxCompute entry object to execute SQL statements. The methods return the task instances that execute the SQL statements. For more information, see [Task instances](/intl.en-US/Development/PyODPS/Basic operations/Task instances.md).

Parameters

-   statement: the SQL statement to execute.

    **Note:** Not all SQL statements can be executed by using methods of the MaxCompute entry object. Some SQL statements that are executable on the MaxCompute client may fail to be executed by using the `execute_sql()` and `run_sql()` methods. You must use other methods to execute non-DDL or non-DML statements. For example, you must use the `run_security_query` method to execute GRANT or REVOKE statements, and use the `run_xflow` or `execute_xflow` method to run Machine Learning Platform for Artificial Intelligence \(PAI\) commands.

-   hints: the runtime parameters. The hints parameter is of the DICT type.

Examples

-   Execute SQL statements.

    ```
    o.execute_sql('select * from dual')  # Execute the statement in synchronous mode. Other instances are blocked until the SQL statement is executed.
    instance = o.run_sql('select * from dual')  # Execute the statement in asynchronous mode.
    print(instance.get_logview_address())  # Obtain the Logview URL of an instance.
    instance.wait_for_success()  # Other instances are blocked until the SQL statement is executed.
    ```

-   Set runtime parameters for SQL statements.

    ```
    o.execute_sql('select * from pyodps_iris', hints={'odps.sql.mapper.split.size': 16})
    ```

    If you set the `sql.settings` parameter that is effective globally, the hints parameter is automatically set each time you execute the SQL statement.

    ```
    from odps import options
    options.sql.settings = {'odps.sql.mapper.split.size': 16}
    o.execute_sql('select * from pyodps_iris')  # The hints parameter is automatically set based on your global settings.
    ```


## Obtain the execution results of SQL statements

You can call the `open_reader` method to obtain execution results of SQL statements in the following scenarios:

-   The SQL statements return structured data.

    ```
    with o.execute_sql('select * from dual').open_reader() as reader:
        for record in reader:
        # Process each record.
    ```

-   The `DESC` statement is executed. In this case, you can call the `reader.raw` method to obtain the raw execution result.

    ```
    with o.execute_sql('desc dual').open_reader() as reader:
        print(reader.raw)
    ```


If you specify `options.tunnel.use_instance_tunnel == True` when you call the `open_reader` method, PyODPS calls InstanceTunnel by default. However, if you are using an earlier version of MaxCompute or an error occurs when PyODPS calls InstanceTunnel, PyODPS generates an alert and automatically downgrades the call object to the old Result interface. You can identify the cause of the downgrade based on the alert information. If the result of InstanceTunnel does not meet your expectation, set this option to False. When you call the `open_reader` method, you can specify whether to call InstanceTunnel or the Result interface by setting the tunnel parameter.

```
# Call InstanceTunnel.
with o.execute_sql('select * from dual').open_reader(tunnel=True) as reader:
    for record in reader:
    # Process each record.
# Call the Result interface.
with o.execute_sql('select * from dual').open_reader(tunnel=False) as reader:
    for record in reader:
    # Process each record.
```

By default, PyODPS does not limit the amount of data that can be read from an instance. However, for a protected project, the amount of data that can be downloaded by using Tunnel is limited. If you do not specify `options.tunnel.limit_instance_tunnel`, the limit is automatically enabled, and the number of data records that can be downloaded is limited based on the project configurations. Generally, a maximum of 10,000 data records can be downloaded. You can add a limit option to the `open_reader` method or set `options.tunnel.limit_instance_tunnel` to True to manually limit the amount.

If your MaxCompute version only supports the old Result interface and you need to read all data, you can export the execution result of the SQL statement to another table and then use the table read interface to read data. This may be limited by project security settings.

In PyODPS V0.7.7.1 and later, you can use the `open_reader` method to call InstanceTunnel to obtain all data. For more information, see [InstanceTunnel](/intl.en-US/Development/Data upload and download/Tunnel SDK/InstanceTunnel.md).

```
instance = o.execute_sql('select * from movielens_ratings limit 20000')
with instance.open_reader() as reader:
    print(reader.count)
    # for record in reader: Traverse the 20,000 data records. In this example, only 10 data records are obtained based on data slicing.
    for record in reader[:10]:  
        print(record)
```

## Set an alias

If the resource referenced by a UDF changes dynamically, you can set an alias for the old resource and use the alias as the name of the new resource. This way, you do not need to delete the UDF or create another UDF.

```
from odps.models import Schema
myfunc = '''\
from odps.udf import annotate
from odps.distcache import get_cache_file

@annotate('bigint->bigint')
class Example(object):
    def __init__(self):
        self.n = int(get_cache_file('test_alias_res1').read())

    def evaluate(self, arg):
        return arg + self.n
'''
res1 = o.create_resource('test_alias_res1', 'file', file_obj='1')
o.create_resource('test_alias.py', 'py', file_obj=myfunc)
o.create_function('test_alias_func',
                  class_type='test_alias.Example',
                  resources=['test_alias.py', 'test_alias_res1'])

table = o.create_table(
    'test_table',
    schema=Schema.from_lists(['size'], ['bigint']),
    if_not_exists=True
)

data = [[1, ], ]
# Write a row of data that contains only one value: 1.
o.write_table(table, 0, [table.new_record(it) for it in data])

with o.execute_sql(
    'select test_alias_func(size) from test_table').open_reader() as reader:
    print(reader[0][0])
res2 = o.create_resource('test_alias_res2', 'file', file_obj='2')
# Set the alias of resource res1 as the name of resource res2 without the need to modify the UDF or resource.
with o.execute_sql(
    'select test_alias_func(size) from test_table',
    aliases={'test_alias_res1': 'test_alias_res2'}).open_reader() as reader:
    print(reader[0][0])
```

## Execute SQL statements in an interactive environment

You can use SQL plug-ins to execute SQL statement or parameterized queries in IPython and Jupyter. For more information, see [t21194.md\#]() and [t21195.md\#]().

## Set biz\_id

Occasionally, you must specify `biz_id` for the SQL statement to execute. Otherwise, an error occurs. In this case, you can set `biz_id` in the global `options` to troubleshoot the error.

```
from odps import options
options.biz_id = 'my_biz_id'
o.execute_sql('select * from pyodps_iris')
```

