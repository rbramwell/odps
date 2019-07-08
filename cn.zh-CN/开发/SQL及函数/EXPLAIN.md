# EXPLAIN {#concept_s2f_wkb_wdb .concept}

本文为您介绍MaxCompute SQL提供的EXPLAIN操作。该操作用于显示对应的DML语句执行计划的结构（执行计划是指用于执行SQL语义的程序）。

命令格式如下。

``` {#codeblock_f81_5s7_x10}
EXPLAIN <DML query>;
```

`EXPLAIN`的执行结果包含：

-   对应于该DML语句的所有Task的依赖结构。
-   Task中所有Task的依赖结构。
-   Task中所有Operator的依赖结构。

示例如下。

``` {#codeblock_umr_pgx_bvj}
EXPLAIN
SELECT abs(a.key), b.value FROM src a JOIN src1 b ON a.value = b.value;
```

`EXPLAIN`的输出结果会有以下三个部分：

-   首先是Job间的依赖关系：`job0 is root job`。因为该Query只需要一个Job（`job0`），所以只需要一行信息。
-   其次是Task间的依赖关系：

    ``` {#codeblock_rgr_6tc_3mw}
    In Job job0:
    root Tasks: M1_Stg1, M2_Stg1
    J3_1_2_Stg1 depends on: M1_Stg1, M2_Stg1
    ```

    `job0`包含三个Task。`M1_Stg1`和`M2_Stg1`这两个Task会先执行，执行完成后，再执行`J3_1_2_Stg1`。

    Task的命名规则如下：

    -   在MaxCompute中，共有四种Task类型：MapTask、ReduceTask、JoinTask和LocalWork。
    -   Task名称的第一个字母表示了当前Task的类型，例如`M2Stg1`就是一个MapTask。
    -   紧跟着第一个字母后的数字，代表了当前Task的ID。这个ID在所有对应当前Query的Task中是唯一的。
    -   用下划线分隔的数字代表当前Task的直接依赖，例如`J3_1_2_Stg1`表示当前Task（ID为3）依赖ID为1、2的两个Task。
-   第三部分即Task中的Operator结构，Operator串描述了一个Task的执行语义。

    ``` {#codeblock_kv2_aj6_uhr}
    In Task M1_Stg1:
      Data source: yudi_2.src                       # "Data source"描述了当前Task的输入内容。
      TS: alias: a                                  # TableScanOperator
          RS: order: +                              # ReduceSinkOperator
              keys:
                   a.value
              values:
                   a.key
              partitions:
                   a.value
    In Task J3_1_2_Stg1:
      JOIN: a INNER JOIN b                          # JoinOperator
          SEL: Abs(UDFToDouble(a._col0)), b._col5   # SelectOperator
              FS: output: None                      # FileSinkOperator
    In Task M2_Stg1:
      Data source: yudi_2.src1
      TS: alias: b
          RS: order: +
              keys:
                   b.value
              values:
                   b.value
              partitions:
                   b.value
    ```

    -   各Operator的含义如下：
        -   TableScanOperator：描述了Query语句中的`FROM`语句块的逻辑。`EXPLAIN`结果中会显示输入表的名称（Alias）。
        -   SelectOperator：描述了Query语句中的`SELECT`语句块的逻辑。`EXPLAIN`结果中会显示向下一个Operator传递的列，多个列由逗号分隔。
            -   如果是列的引用，则显示成`< alias >.< column_name >`。
            -   如果是表达式的结果，则显示函数的形式，例如`func1(arg1_1, arg1_2, func2(arg2_1, arg2_2))`。
            -   如果是常量，则直接显示值内容。
        -   FilterOperator：描述了Query语句中的`WHERE`语句块的逻辑。`EXPLAIN`结果中会显示一个`WHERE`条件表达式，形式类似SelectOperator的显示规则。
        -   JoinOperator：描述了Query语句中的`JOIN`语句块的逻辑。`EXPLAIN`结果中会显示哪些表用哪种方式Join在一起。
        -   GroupByOperator：描述了聚合操作的逻辑。如果Query中使用了聚合函数，就会出现该结构，`EXPLAIN`结果中会显示聚合函数的内容。
        -   ReduceSinkOperator：描述了Task间数据分发操作的逻辑。如果当前Task的结果会传递给另一个Task，则必然需要在当前Task的最后，使用ReduceSinkOperator执行数据分发操作。`EXPLAIN`的结果中会显示输出结果的排序方式、分发的Key、Value以及用来求Hash值的列。
        -   FileSinkOperator：描述了最终数据的存储操作。如果Query中有`INSERT`语句块，`EXPLAIN`结果中会显示目标表名称。
        -   LimitOperator：描述了Query语句中的`LIMIT`语句块的逻辑。`EXPLAIN`结果中会显示`LIMIT`数。
        -   MapjoinOperator：类似JoinOperator，描述了大表的`JOIN`操作。

**说明：** 如果Query足够复杂，`EXPLAIN`的结果太多，则会触发API的限制，无法得到完整的`EXPLAIN`结果。此时您可以拆分Query，对各部分分别进行`EXPLAIN`操作，以了解Job的结构。

