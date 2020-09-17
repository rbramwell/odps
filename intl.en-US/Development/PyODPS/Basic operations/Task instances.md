---
keyword: [task, instance]
---

# Task instances

A task, such as an SQL task, is the basic compute unit in MaxCompute. This topic describes the basic operations on task instances in PyODPS.

## Basic operations

When a task is triggered, a MaxCompute instance is created for the task. For more information, see [Instance](/intl.en-US/Product Introduction/Definitions/Instance.md). You can perform the following operations on MaxCompute instances:

-   `list_instances()`: obtains all instances in a specific project.
-   `exist_instance()`: checks whether a specific instance exists.
-   `get_instance()`: obtains a specific instance.
-   `stop_instance()`: stops a specific instance.

    **Note:** You can also call the `stop()` method to stop a specific instance.


**Example**

```
for instance in o.list_instances():
    print(instance.id)
    o.exist_instance('my_instance_id')
```

## Obtain the Logview URL of an instance

-   For an SQL task instance, you can call the `get_logview_address` method to obtain its Logview URL.

    ```
    # Execute an SQL statement to obtain a specific instance. Then, call the get_logview_address method to obtain the Logview URL of the instance.
    instance = o.run_sql('desc pyodps_iris')
    print(instance.get_logview_address())
    # Obtain the Logview URL of an instance with the specified instance ID.
    instance = o.get_instance('2016042605520945g9k5pvyi2')
    print(instance.get_logview_address())
    ```

-   For an XFlow task instance, you must enumerate its sub-instances and then obtain the Logview URL of each sub-instance.

    ```
    instance = o.run_xflow('AppendID', 'algo_public', {'inputTableName': 'input_table', 'outputTableName': 'output_table'})
    for sub_inst_name, sub_inst in o.get_xflow_sub_instances(instance).items():
        print('%s: %s' % (sub_inst_name, sub_inst.get_logview_address()))
    ```


## Obtain the status of an instance

An instance can be in the `Running`, `Suspended`, or `Terminated` state. You can use the following methods to obtain the status of an instance:

-   `status`: obtains the status of an instance.
-   `is_terminated`: checks whether the execution of the current instance is completed.
-   `is_successful`: checks whether the execution of the current instance is successful. If the instance is running or fails, False is returned.

```
instance = o.get_instance('2016042605520945g9k5pvyi2')
instance.status
<Status.TERMINATED: 'Terminated'>
    from odps.models import Instance
    instance.status == Instance.Status.TERMINATED
    True
    instance.status.value
    'Terminated'
```

**Note:** You can call the `wait_for_completion` method to ask the system to return the status until the execution of the instance is completed. The `wait_for_success` method functions similarly. The difference is that if the instance fails, an exception is thrown.

## Perform operations on sub-instances

A running instance may contain one or more sub-instances. You can call the following methods to perform specific operations on sub-instances:

-   `get_task_names`: obtains all sub-instances. This method returns the names of all sub-instances.

    ```
    instance.get_task_names()
    ['SQLDropTableTask']
    ```

-   `get_task_result`: obtains the execution result of a specific sub-instance. This method returns the execution result of each sub-instance in the form of a dictionary.

    ```
    instance = o.execute_sql('select * from pyodps_iris limit 1')
    instance.get_task_names()
    ['AnonymousSQLTask']
    instance.get_task_result('AnonymousSQLTask')
    '"sepallength","sepalwidth","petallength","petalwidth","name"\n5.1,3.5,1.4,0.2,"Iris-setosa"\n'
    instance.get_task_results()
    OrderedDict([('AnonymousSQLTask', "sepallength","sepalwidth","petallength","petalwidth","name"\n5.1,3.5,1.4,0.2,"Iris-setosa"\n')])
    ```

-   `get_task_progress`: obtains the current running progress of a specific sub-instance when the instance is running.

    ```
    while not instance.is_terminated():
        for task_name in instance.get_task_names():
            print(instance.id, instance.get_task_progress(task_name).get_stage_progress_formatted_string())
            time.sleep(10)
    20160519101349613gzbzufck2 2016-05-19 18:14:03 M1_Stg1_job0:0/1/1[100%]
    ```


