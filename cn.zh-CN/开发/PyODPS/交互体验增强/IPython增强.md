# IPython增强 {#concept_ujw_pgg_cfb .concept}

PyODPS提供了IPython的插件，来更方便得操作ODPS。

## IPython增强 {#section_of1_cd1_hfb .section}

针对命令行增强，PyODPS也有相应的命令。

-   执行以下代码加载插件。

    ``` {#codeblock_nxl_rz7_1or .language-sql}
    %load_ext odps
    %enter
    ```

    执行结果如下。

    ``` {#codeblock_jt6_asw_vhj}
    <odps.inter.Room at 0x11341df10>
    ```

-   此时全局会包含`o`和`odps`变量，即ODPS入口。您可以通过`o.get_table`或`odps.get_table`命令调用表。

    ``` {#codeblock_9pg_4gu_a80 .language-ruby}
    o.get_table('dual')
    odps.get_table('dual')
    ```

    执行结果如下。

    ``` {#codeblock_0lf_qy3_5f2 .language-sql}
    odps.Table
      name: odps_test_sqltask_finance.`dual`
      schema:
        c_int_a                 : bigint
        c_int_b                 : bigint
        c_double_a              : double
        c_double_b              : double
        c_string_a              : string
        c_string_b              : string
        c_bool_a                : boolean
        c_bool_b                : boolean
        c_datetime_a            : datetime
        c_datetime_b            : datetime
    ```

-   调用以下命令将已储存的对象以表格形式打印出来。

    ``` {#codeblock_cm0_40w_6v0 .language-sql}
    %stores
    ```

    |default name|desc|
    |:-----------|:---|
    |iris|安德森鸢尾花卉数据集|


## 对象名补全 {#section_zct_lj4_cfb .section}

PyODPS拓展了IPython原有的代码补全功能，支持在书写类似`o.get_xxx`的语句时，自动补全对象名。以下示例中，<tab\>不是实际输入的字符，而是当所有输入完成后，将光标移动到相应位置， 并按Tab键。

-   直接使用Tab键补全对象。

    ``` {#codeblock_evx_jmu_46v .language-ruby}
    o.get_table(<tab>
    ```

-   如果已知需要补全对象的前缀，也可以在使用此功能时输入前缀。

    ``` {#codeblock_qnh_dao_02y .language-sql}
    o.get_table('tabl<tab>
    ```

    输入上述语句并按Tab键后IPython会自动补全前缀为`tabl`的表。

-   对象名补全也支持补全不同Project下的对象名，支持的用法如下。

    ``` {#codeblock_96w_u75_xnz .language-sql}
    o.get_table(project='project_name', name='tabl<tab>
    o.get_table('tabl<tab>', project='project_name')
    ```

-   如果匹配的对象有多个，IPython会给出一个列表。其最大长度由`options.completion_size`给出， 默认值为10。

## SQL命令 {#section_ecl_rj4_cfb .section}

PyODPS还提供了SQL插件，用于执行ODPS SQL。

-   您可以使用单个百分号输入是单行SQL命令。

    ``` {#codeblock_wwn_o38_wih .language-sql}
    In [37]: %sql select * from pyodps_iris limit 5
    |==========================================|   1 /  1  (100.00%)         3s
    Out[37]:
       sepallength  sepalwidth  petallength  petalwidth         name
    0          5.1         3.5          1.4         0.2  Iris-setosa
    1          4.9         3.0          1.4         0.2  Iris-setosa
    2          4.7         3.2          1.3         0.2  Iris-setosa
    3          4.6         3.1          1.5         0.2  Iris-setosa
    4          5.0         3.6          1.4         0.2  Iris-setosa
    ```

-   多行SQL可以使用`%%sql`命令。

    ``` {#codeblock_u9c_x27_s4z .language-sql}
    In [38]: %%sql
       ....: select * from pyodps_iris
       ....: where sepallength < 5
       ....: limit 5
       ....:
    |==========================================|   1 /  1  (100.00%)        15s
    Out[38]:
       sepallength  sepalwidth  petallength  petalwidth         name
    0          4.9         3.0          1.4         0.2  Iris-setosa
    1          4.7         3.2          1.3         0.2  Iris-setosa
    2          4.6         3.1          1.5         0.2  Iris-setosa
    3          4.6         3.4          1.4         0.3  Iris-setosa
    4          4.4         2.9          1.4         0.2  Iris-setosa
    ```

-   如果您需要执行参数化SQL查询，则需以`:参数`的方式替换需要的参数。

    ``` {#codeblock_b1w_8we_5o4 .language-sql}
    In [1]: %load_ext odps
    
    In [2]: mytable = 'dual'
    
    In [3]: %sql select * from :mytable
    |==========================================|   1 /  1  (100.00%)         2s
    Out[3]:
       c_int_a  c_int_b  c_double_a  c_double_b  c_string_a  c_string_b c_bool_a  \
    0        0        0       -1203           0           0       -1203     True
    
      c_bool_b         c_datetime_a         c_datetime_b
    0    False  2012-03-30 23:59:58  2012-03-30 23:59:59
    ```

-   设置SQL运行参数时，可以通过`%set`设置到全局，或者在SQL的cell里用SET进行局部设置。以下语句会进行局部设置，不会影响全局的配置。

    ``` {#codeblock_v85_0y5_0rf .language-sql}
    In [17]: %%sql
             set odps.sql.mapper.split.size = 16;
             select * from pyodps_iris;
    ```

    以下设置会进行全局设置，后续运行的SQL都会使用这个设置。

    ``` {#codeblock_mbb_8t1_yxs .language-sql}
    In [18]: %set odps.sql.mapper.split.size = 16
    ```


## 持久化Pandas DataFrame到ODPS表 {#section_m4k_2k4_cfb .section}

PyODPS还提供把Pandas DataFrame上传到ODPS表的命令。您只需使用`%persist`命令即可，此命令的第1个参数`df`是前面的变量名，第2个参数`pyodps_pandas_df`是ODPS表名。

``` {#codeblock_96f_3h8_7gk .language-sql}
import pandas as pd
import numpy as np

df = pd.DataFrame(np.arange(9).reshape(3, 3), columns=list('abc'))
%persist df pyodps_pandas_df
```

