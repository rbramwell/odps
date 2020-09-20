# Lifecycle of table {#concept_axp_nj1_wdb .concept}

## Modify lifecycle of table {#section_u1v_dz1_wdb .section}

MaxCompute provides a function to manage data lifecycle so that user can release storage space and simplify the data recycle flow.

**Statement format:**

```
ALTER TABLE table_name SET lifecycle days;
```

**Note:** 

-   The parameter ‘days’ refers to the time required to complete the lifecycle. It must be a positive integer and its unit is ‘day’.
-   Suppose that the table ‘table\_name’ is a no-partition table. Calculated from the last updated date, the data is still not modified after N \(days\) days, then MaxCompute automatically recycles the table without user intervention \(similar to  ‘drop table’ operation\).
-   In MaxCompute, once the data in the table is modified, the LastDataModifiedTime is updated. So MaxCompute judges whether to recycle this table based on the LastDataModifiedTime setting and lifecycle.
-   · Suppose the table ‘table\_name’ is a partition table. MaxCompute determines whether to recycle the table according to LastDataModifiedTime of each partition.
-   · Unlike no-partition table, after the last partition of a partitioned table has been recycled, the table is not deleted.
-   · The lifecycle can be set for a table, not for the partition.
-   · It can be specified while creating a table.

**Example:**

```
create table test_lifecycle(key string) lifecycle 100;
 -- Create a new table test_lifecycle and the lifecycle is 100 days.
 alter table test_lifecycle set lifecycle 50;
 -- Alter the lifecycle for the table test_lifecycle and set it to be 50 days.
```

## Disable lifecycle of table {#section_zbv_kz1_wdb .section}

In some cases, the data in specified partitions do not need to be recycled by the lifecycle function. For example, data in the beginning of the month, or the data during the Global Shopping Day period. You can disable the lifecycle function using some specific partitions.

**Statement format:**

```
ALTER TABLE table_name [PARTITION partition_spec] ENABLE|DISABLE LIFECYCLE;
```

An example is shown as follows.

```
ALTER TABLE trans PARTITION(dt='20141111') DISABLE LIFECYCLE;
```

