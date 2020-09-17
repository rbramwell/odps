---
keyword: [create a table, obtain table data, manage table partitions, synchronize table updates]
---

# Tables

This topic describes the basic operations as well as other operations that you can perform on a MaxCompute table. For example, you can create a table, create a table schema, synchronize table updates, obtain table data, delete a table, manage table partitions, and convert a table to a DataFrame.

## Basic operations

A table is the data storage unit in MaxCompute. For more information, see [Table](/intl.en-US/Product Introduction/Definitions/Table.md). You can perform the following basic operations on tables:

-   Obtain all tables in the current project by calling the `list_tables()` method of a MaxCompute entry object.

    ```
    # Obtain all tables in the current project.
    for table in o.list_tables():
    ```

-   Check whether a specific table exists by calling the `exist_table()` method of a MaxCompute entry object.
-   Obtain a specific table by calling the `get_table()` method of a MaxCompute entry object.

    ```
    t = o.get_table('dual')
    t.schema
    odps.Schema {
      c_int_a                 bigint
      c_int_b                 bigint
      c_double_a              double
      c_double_b              double
      c_string_a              string
      c_string_b              string
      c_bool_a                boolean
      c_bool_b                boolean
      c_datetime_a            datetime
      c_datetime_b            datetime
    }
    t.lifecycle
    -1
    print(t.creation_time)
    2014-05-15 14:58:43
    t.is_virtual_view
    False
    t.size
    1408
    t.comment
    'Dual Table Comment'
    t.schema.columns
    [<column c_int_a, type bigint>,
     <column c_int_b, type bigint>,
     <column c_double_a, type double>,
     <column c_double_b, type double>,
     <column c_string_a, type string>,
     <column c_string_b, type string>,
     <column c_bool_a, type boolean>,
     <column c_bool_b, type boolean>,
     <column c_datetime_a, type datetime>,
     <column c_datetime_b, type datetime>]
    t.schema['c_int_a']
    <column c_int_a, type bigint>
    t.schema['c_int_a'].comment
    'Comment of column c_int_a'
    ```

    You can obtain tables across projects by specifying the `project` parameter.

    ```
    t = o.get_table('dual', project='other_project')
    ```


## Create a table schema

You can use one of the following methods to create a table schema:

-   Create a schema based on table columns and optional partitions.

    ```
    from odps.models import Schema, Column, Partition
    columns = [Column(name='num', type='bigint', comment='the column'),
               Column(name='num2', type='double', comment='the column2')]
    partitions = [Partition(name='pt', type='string', comment='the partition')]
    schema = Schema(columns=columns, partitions=partitions)
    schema.columns
    [<column num, type bigint>,
     <column num2, type double>,
     <partition pt, type string>]
    schema.partitions
    [<partition pt, type string>]
    schema.names  # Obtain the names of non-partitioned fields.
    ['num', 'num2']
    schema.types  # Obtain the data types of non-partitioned fields.
    [bigint, double]
    ```

-   Create a schema by calling the `Schema.from_lists()` method. This method is more convenient, but you cannot directly set comments for columns and partitions.

    ```
    schema = Schema.from_lists(['num', 'num2'], ['bigint', 'double'], ['pt'], ['string'])
    schema.columns
    [<column num, type bigint>,
     <column num2, type double>,
     <partition pt, type string>]
    ```


## Create a table

You can call the `create_table()` method to create a table by using one of the following methods:

-   Use a table schema to create a table.

    ```
    table = o.create_table('my_new_table', schema)
    table = o.create_table('my_new_table', schema, if_not_exists=True)  # Create the table only if no table with the same name exists.
    table = o.create_table('my_new_table', schema, lifecycle=7)  # Set the lifecycle of the table.
    ```

-   Create a table by specifying the names and data types of the fields to be contained in the table.

    ```
    # Create a non-partitioned table.
    table = o.create_table('my_new_table', 'num bigint, num2 double', if_not_exists=True)
    # Create a partitioned table with the specified table columns and partition fields.
    table = o.create_table('my_new_table', ('num bigint, num2 double', 'pt string'), if_not_exists=True)
    ```

    By default, when you create a table, you can only use the BIGINT, DOUBLE, DECIMAL, STRING, DATETIME, BOOLEAN, MAP, and ARRAY data types. If you need to use other data types such as TINYINT and STRUCT, you must set `options.sql.use_odps2_extension` to True. Example:

    ```
    from odps import options
    options.sql.use_odps2_extension = True
    table = o.create_table('my_new_table', 'cat smallint, content struct<title:varchar(100), body string>')
    ```


## Synchronize table updates

After another program updates a table, for example, the table schema, you can call the `reload()` method to synchronize the update.

```
table.reload()
```

## Insert a record to a table

A record refers to a single row in a table. You can call the `new_record()` method to insert a record to a specific table.

```
t = o.get_table('mytable')
r = t.new_record(['val0', 'val1'])  # The number of values must be equal to the number of fields in the table schema.
r2 = t.new_record()     # You can leave the value empty.
r2[0] = 'val0' # Set a value based on an offset.
r2['field1'] = 'val1'  # Set a value based on the field name.
r2.field1 = 'val1'  # Set a value based on an attribute.

print(record[0])  # Obtain the value at position 0.
print(record['c_double_a'])  # Obtain a value based on a field.
print(record.c_double_a)  # Obtain a value based on an attribute.
print(record[0: 3])  # Slice data.
print(record[0, 2, 3])  # Obtain values at multiple positions.
print(record['c_int_a', 'c_double_a'])  # Obtain values based on multiple fields.
```

## Write data to a table

-   Call the `write_table()` method of a MaxCompute entry object to write data to a table.

    ```
    records = [[111, 'aaa', True],                 # A list can be specified.
              [222, 'bbb', False],
              [333, 'ccc', True],
              [444, 'Chinese', False]]
    o.write_table('test_table', records, partition='pt=test', create_partition=True)
    ```

    **Note:**

    -   Each time you call the `write_table()` method, MaxCompute generates a file on the server. This operation is time-consuming. If a large number of files are concurrently generated, subsequent queries will be less efficient. We recommend that you write multiple records at a time or provide a generator object if you use the write\_table\(\) method.
    -   If you call the `write_table()` method to write data to a table, new data will be appended to existing data. PyODPS does not provide options to overwrite existing data. You must manually delete the data that you want to overwrite. For a non-partitioned table, you must call the `table.truncate()` method to delete data. For a partitioned table, you must delete partitions first and then create partitions again.
-   Call the `open_writer()` method to write data to a table.

    ```
    with t.open_writer(partition='pt=test') as writer:
    records = [[111, 'aaa', True],                 # A list can be specified.
              [222, 'bbb', False],
              [333, 'ccc', True],
              [444, 'Chinese', False]]
    writer.write(records)  # records can be iterable objects.
    
    with t.open_writer(partition='pt1=test1,pt2=test2') as writer: # Write data in multi-level partitioning mode.
    
    records = [t.new_record([111, 'aaa', True]),   # Record objects can be used.
               t.new_record([222, 'bbb', False]),
               t.new_record([333, 'ccc', True]),
               t.new_record([444, 'Chinese', False])]
    writer.write(records) 
    ```

    If the specified partition does not exist, set the `create_partition` parameter to True to create a partition. Example:

    ```
    with t.open_writer(partition='pt=test', create_partition=True) as writer:
         records = [[111, 'aaa', True],                 # A list can be specified.
                    [222, 'bbb', False],
                    [333, 'ccc', True],
                    [444, 'Chinese', False]]
       writer.write(records)  # records can be iterable objects.
    ```

-   Use multiple processes to concurrently write data to a table.

    If multiple processes concurrently write data to a table, each process shares the same session ID but writes data in a separate block. Each block maps a file on the server. After all the processes finish writing data, the main process submits the data.

    ```
    import random
    from multiprocessing import Pool
    from odps.tunnel import TableTunnel
    
    def write_records(session_id, block_id):
        # Create a session with the specified session ID.
        local_session = tunnel.create_upload_session(table.name, upload_id=session_id)
        # Create a writer with the specified block ID.
        with local_session.open_record_writer(block_id) as writer:
            for i in range(5):
                # Generate data and write the data to the corresponding block.
                record = table.new_record([random.randint(1, 100), random.random()])
                writer.write(record)
    
    if __name__ == '__main__':
        N_WORKERS = 3
    
        table = o.create_table('my_new_table', 'num bigint, num2 double', if_not_exists=True)
        tunnel = TableTunnel(o)
        upload_session = tunnel.create_upload_session(table.name)
    
        # Each process uses the same session ID.
        session_id = upload_session.id
    
        pool = Pool(processes=N_WORKERS)
        futures = []
        block_ids = []
        for i in range(N_WORKERS):
            futures.append(pool.apply_async(write_records, (session_id, i)))
            block_ids.append(i)
        [f.get() for f in futures]
    
        # Submit the data in all the blocks.
        upload_session.commit(block_ids)
    ```


## Obtain table data

You can use one of the following methods to obtain data from a table:

-   Call the `read_table()` method of a MaxCompute entry object.

    ```
    for record in o.read_table('test_table', partition='pt=test'):
    # Process a record.
    ```

-   Call the `head()` method to obtain less than 10,000 data records from the beginning of a table.

    ```
    t = o.get_table('dual')
    # Process each record.
    for record in t.head(3):
    ```

-   Call the `open_reader()` method.
    -   Open the reader with a WITH clause.

        ```
        with t.open_reader(partition='pt=test') as reader:
        count = reader.count
        for record in reader[5:10]  # You can execute the statement multiple times until all records are read. The number of records is specified by count. You can change it to parallel-operation code.
        # Process a record.
        ```

    -   Open the reader without a WITH clause.

        ```
        reader = t.open_reader(partition='pt=test')
        count = reader.count
        for record in reader[5:10]  # You can execute the statement multiple times until all records are read. The number of records is specified by count. You can change it to parallel-operation code.
        # Process a record.
        ```


## Delete a table

You can call the `delete_table()` method to delete an existing table.

```
o.delete_table('my_table_name', if_exists=True)  # Delete a table only if the table exists.
 t.drop()  # Call the drop() method to delete a table if the table exists.
```

## Create a DataFrame

PyODPS provides the DataFrame framework, which allows you to conveniently query and manage MaxCompute data. For more information, see [DataFrame](). You can call the `to_df()` method to convert a table to a DataFrame.

```
table = o.get_table('my_table_name')
df = table.to_df()
```

## Manage table partitions

-   Check whether a table is partitioned.

    ```
    if table.schema.partitions:
     print('Table %s is partitioned.' % table.name)
    ```

-   Iterate over all the partitions in a table.

    ```
    for partition in table.partitions:
         print(partition.name)
    for partition in table.iterate_partitions(spec='pt=test'):
    # Iterate over level-2 partitions.
    ```

-   Check whether a partition exists.

    ```
    table.exist_partition('pt=test,sub=2015')
    ```

-   Obtain a partition.

    ```
    partition = table.get_partition('pt=test')
     print(partition.creation_time)
    2015-11-18 22:22:27
     partition.size
    0
    ```

-   Create a partition.

    ```
    t.create_partition('pt=test', if_not_exists=True)  # Create a partition only if no partition with the same name exists.
    ```

-   Delete a partition.

    ```
    t.delete_partition('pt=test', if_exists=True)  # Delete a partition only if the partition exists.
    partition.drop()  # Call the drop() method to delete a partition if the partition exists.
    ```


## Upload and download data by using MaxCompute Tunnel

MaxCompute Tunnel is the data tunnel of MaxCompute. You can use Tunnel to upload data to or download data from MaxCompute.

-   Example of data upload

    ```
    from odps.tunnel import TableTunnel
    
    table = o.get_table('my_table')
    
    tunnel = TableTunnel(odps)
    upload_session = tunnel.create_upload_session(table.name, partition_spec='pt=test')
    
    with upload_session.open_record_writer(0) as writer:
        record = table.new_record()
        record[0] = 'test1'
        record[1] = 'id1'
        writer.write(record)
    
        record = table.new_record(['test2', 'id2'])
        writer.write(record)
    
    upload_session.commit([0])
    ```

-   Example of data download

    ```
    from odps.tunnel import TableTunnel
    
    tunnel = TableTunnel(odps)
    download_session = tunnel.create_download_session('my_table', partition_spec='pt=test')
    
    with download_session.open_record_reader(0, download_session.count) as reader:
        for record in reader:
    # Process each record.
    ```


**Note:**

-   PyODPS does not allow you to upload data by using foreign tables. For example, you cannot upload data from Object Storage Service \(OSS\) or Tablestore by using foreign tables.
-   We recommend that you use the write and read interfaces of tables instead of Tunnel.
-   In a CPython environment, PyODPS compiles C programs during installation to accelerate Tunnel-based upload and download.

