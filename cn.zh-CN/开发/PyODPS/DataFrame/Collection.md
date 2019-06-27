# Collection {#concept_nsj_ft4_cfb .concept}

DataFrame中所有二维数据集上的操作都属于CollectionExpr，可视为一张ODPS表或一张电子表单，DataFrame对象也是CollectionExpr的特例。CollectionExpr中包含针对二维数据集的列操作、筛选、变换等大量操作。

## 获取类型 {#section_grp_nt4_cfb .section}

`dtypes`可以用来获取CollectionExpr中所有列的类型。`dtypes` 返回的是[Schema类型](cn.zh-CN/开发/PyODPS/基本操作/表.md#) 。

``` {#codeblock_g9g_f0k_9by .language-sql}
iris.dtypes
odps.Schema {
  sepallength           float64
  sepalwidth            float64
  petallength           float64
  petalwidth            float64
  name                  string
}
```

## 列选择和增删 {#section_prv_tt4_cfb .section}

-   列选择

    如果您需要从一个CollectionExpr中选取部分列，产生新的数据集，可以使用`expr[columns]`语法。

    ``` {#codeblock_aef_xwv_kd6 .language-sql}
    iris['name', 'sepallength'].head(5)
              name  sepallength
    0  Iris-setosa          5.1
    1  Iris-setosa          4.9
    2  Iris-setosa          4.7
    3  Iris-setosa          4.6
    4  Iris-setosa          5.0
    ```

    **说明：** 

    如果需要选择的列只有一列，需要在Columns后加上逗号或者显示标记为列表，例如`iris[iris.sepal_length, ]`或`iris[[iris.sepal_length]]`，否则返回的将是一个Sequence对象，而不是Collection。

-   列删除

    如果您需要在新的数据集中排除已有数据集的某些列，可使用`exclude`方法。

    ``` {#codeblock_nac_vab_ddt .language-sql}
    iris.exclude('sepallength', 'petallength')[:5]
       sepalwidth  petalwidth         name
    0         3.5         0.2  Iris-setosa
    1         3.0         0.2  Iris-setosa
    2         3.2         0.2  Iris-setosa
    3         3.1         0.2  Iris-setosa
    4         3.6         0.2  Iris-setosa
    ```

    PyODPS 0.7.2版（及以上版本）支持另一种写法，即在数据集上直接排除相应的列。

    ``` {#codeblock_56g_4ow_muc .language-sql}
    del iris['sepallength']
    del iris['petallength']
    iris[:5]
       sepalwidth  petalwidth         name
    0         3.5         0.2  Iris-setosa
    1         3.0         0.2  Iris-setosa
    2         3.2         0.2  Iris-setosa
    3         3.1         0.2  Iris-setosa
    4         3.6         0.2  Iris-setosa
    ```

-   列增加

    如果您需要在已有Collection中添加某一列变换的结果，可以使用`expr[expr, new_sequence]`语法， 新增的列会作为新Collection的一部分。

    以下示例将`iris`中的`sepalwidth`列加一后重命名为`sepalwidthplus1`并追加到数据集末尾，形成新的数据集。

    ``` {#codeblock_e4s_tmp_yeb .language-sql}
    iris[iris, (iris.sepalwidth + 1).rename('sepalwidthplus1')].head(5)
       sepallength  sepalwidth  petallength  petalwidth         name  \
    0          5.1         3.5          1.4         0.2  Iris-setosa
    1          4.9         3.0          1.4         0.2  Iris-setosa
    2          4.7         3.2          1.3         0.2  Iris-setosa
    3          4.6         3.1          1.5         0.2  Iris-setosa
    4          5.0         3.6          1.4         0.2  Iris-setosa
    
       sepalwidthplus1
    0              4.5
    1              4.0
    2              4.2
    3              4.1
    4              4.6
    ```

    使用`expr[expr, new_sequence]`语法需要注意的是，变换后的列名与原列名可能相同。如果需要与原Collection合并， 请将该列重命名。

    PyODPS 0.7.2版（及以上版本）支持直接在当前数据集中追加。

    ``` {#codeblock_vi5_b21_rmo .language-sql}
    iris['sepalwidthplus1'] = iris.sepalwidth + 1
    iris.head(5)
       sepallength  sepalwidth  petallength  petalwidth         name  \
    0          5.1         3.5          1.4         0.2  Iris-setosa
    1          4.9         3.0          1.4         0.2  Iris-setosa
    2          4.7         3.2          1.3         0.2  Iris-setosa
    3          4.6         3.1          1.5         0.2  Iris-setosa
    4          5.0         3.6          1.4         0.2  Iris-setosa
    
       sepalwidthplus1
    0              4.5
    1              4.0
    2              4.2
    3              4.1
    4              4.6
    ```

-   同时增删列

    您也可以先将原列通过`exclude`方法进行排除，再将变换后的新列并入，而不必担心重名。

    ``` {#codeblock_4uf_a37_kki .language-sql}
    iris[iris.exclude('sepalwidth'), iris.sepalwidth * 2].head(5)
       sepallength  petallength  petalwidth         name  sepalwidth
    0          5.1          1.4         0.2  Iris-setosa         7.0
    1          4.9          1.4         0.2  Iris-setosa         6.0
    2          4.7          1.3         0.2  Iris-setosa         6.4
    3          4.6          1.5         0.2  Iris-setosa         6.2
    4          5.0          1.4         0.2  Iris-setosa         7.2
    ```

    PyODPS 0.7.2版（及以上版本）支持直接在当前数据集上覆盖的操作，写法如下。

    ``` {#codeblock_s6p_xxj_75u .language-sql}
    iris['sepalwidth'] = iris.sepalwidth * 2
    iris.head(5)
       sepallength  sepalwidth  petallength  petalwidth         name
    0          5.1         7.0          1.4         0.2  Iris-setosa
    1          4.9         6.0          1.4         0.2  Iris-setosa
    2          4.7         6.4          1.3         0.2  Iris-setosa
    3          4.6         6.2          1.5         0.2  Iris-setosa
    4          5.0         7.2          1.4         0.2  Iris-setosa
    ```

    增删列以创建新Collection的另一种实现方式是调用`select`方法，将需要选择的列作为参数输入。如果您需要重命名，可以使用`keyword`参数输入，并将新的列名作为参数名。

    ``` {#codeblock_wvr_b5n_e7s .language-sql}
    iris.select('name', sepalwidthminus1=iris.sepalwidth - 1).head(5)
              name  sepalwidthminus1
    0  Iris-setosa               2.5
    1  Iris-setosa               2.0
    2  Iris-setosa               2.2
    3  Iris-setosa               2.1
    4  Iris-setosa               2.6
    ```

    您也可以传入一个Lambda表达式，它接收的参数为上一步的运算结果。在执行时，PyODPS会检查这些Lambda表达式，传入上一步生成的Collection并将其替换为正确的列。

    ``` {#codeblock_guu_jf7_ane .language-sql}
    iris['name', 'petallength'][[lambda x: x.name]].head(5)
              name
    0  Iris-setosa
    1  Iris-setosa
    2  Iris-setosa
    3  Iris-setosa
    4  Iris-setosa
    ```

    PyODPS 0.7.2版（及以上版本）支持对数据进行条件赋值。

    ``` {#codeblock_w9g_4sf_ltk .language-sql}
    iris[iris.sepallength > 5.0, 'sepalwidth'] = iris.sepalwidth * 2
    iris.head(5)
       sepallength  sepalwidth  petallength  petalwidth         name
    0          5.1        14.0          1.4         0.2  Iris-setosa
    1          4.9         6.0          1.4         0.2  Iris-setosa
    2          4.7         6.4          1.3         0.2  Iris-setosa
    3          4.6         6.2          1.5         0.2  Iris-setosa
    4          5.0         7.2          1.4         0.2  Iris-setosa
    ```


## 引入常数和随机数 {#section_sbm_j54_cfb .section}

-   DataFrame支持在Collection中追加一列常数。追加常数需要使用 [Scalar](cn.zh-CN/开发/PyODPS/API概述.md#)，引入时需要手动指定列名。

    ``` {#codeblock_fqm_20a_sd1 .language-sql}
    from odps.df import Scalar
    iris[iris, Scalar(1).rename('id')][:5]
       sepallength  sepalwidth  petallength  petalwidth         name  id
    0          5.1         3.5          1.4         0.2  Iris-setosa   1
    1          4.9         3.0          1.4         0.2  Iris-setosa   1
    2          4.7         3.2          1.3         0.2  Iris-setosa   1
    3          4.6         3.1          1.5         0.2  Iris-setosa   1
    4          5.0         3.6          1.4         0.2  Iris-setosa   1
    ```

    如果需要指定一个空值列，您可以使用[NullScalar](cn.zh-CN/开发/PyODPS/API概述.md#) ，但需要提供字段类型。

    ``` {#codeblock_rl7_7ke_2ge .language-sql}
    from odps.df import NullScalar
    iris[iris, NullScalar('float').rename('fid')][:5]
       sepal_length  sepal_width  petal_length  petal_width     category   fid
    0           5.1          3.5           1.4          0.2  Iris-setosa  None
    1           4.9          3.0           1.4          0.2  Iris-setosa  None
    2           4.7          3.2           1.3          0.2  Iris-setosa  None
    3           4.6          3.1           1.5          0.2  Iris-setosa  None
    4           5.0          3.6           1.4          0.2  Iris-setosa  None
    ```

    在PyODPS 0.7.12（及以上版本）中，引入了简化写法。

    ``` {#codeblock_yul_p30_l5m .language-sql}
    iris['id'] = 1
    iris
       sepallength  sepalwidth  petallength  petalwidth         name  id
    0          5.1         3.5          1.4         0.2  Iris-setosa   1
    1          4.9         3.0          1.4         0.2  Iris-setosa   1
    2          4.7         3.2          1.3         0.2  Iris-setosa   1
    3          4.6         3.1          1.5         0.2  Iris-setosa   1
    4          5.0         3.6          1.4         0.2  Iris-setosa   1
    ```

    需要注意的是，这种写法无法自动识别空值的类型，所以在增加空值列时，仍然要使用如下代码。

    ``` {#codeblock_y56_6b6_f8u .language-sql}
    iris['null_col'] = NullScalar('float')
    iris
       sepallength  sepalwidth  petallength  petalwidth         name  null_col
    0          5.1         3.5          1.4         0.2  Iris-setosa      None
    1          4.9         3.0          1.4         0.2  Iris-setosa      None
    2          4.7         3.2          1.3         0.2  Iris-setosa      None
    3          4.6         3.1          1.5         0.2  Iris-setosa      None
    4          5.0         3.6          1.4         0.2  Iris-setosa      None
    ```

-   DataFrame也支持在Collection中增加一列随机数列，该列类型为FLOAT，范围为0~1，每行数值均不同。 追加随机数列需要使用[RandomScalar](cn.zh-CN/开发/PyODPS/API概述.md#)，参数为随机数种子，可省略。

    ``` {#codeblock_6wx_tf2_r1z .language-sql}
    from odps.df import RandomScalar
    iris[iris, RandomScalar().rename('rand_val')][:5]
       sepallength  sepalwidth  petallength  petalwidth         name  rand_val
    0          5.1         3.5          1.4         0.2  Iris-setosa  0.000471
    1          4.9         3.0          1.4         0.2  Iris-setosa  0.799520
    2          4.7         3.2          1.3         0.2  Iris-setosa  0.834609
    3          4.6         3.1          1.5         0.2  Iris-setosa  0.106921
    4          5.0         3.6          1.4         0.2  Iris-setosa  0.763442
    ```


## 过滤数据 {#section_ol1_bw4_cfb .section}

Collection提供了数据过滤的功能。查询`sepallength`大于5的数据，示例如下。

``` {#codeblock_pxq_j9r_1ta .language-sql}
iris[iris.sepallength > 5].head(5)
   sepallength  sepalwidth  petallength  petalwidth         name
0          5.1         3.5          1.4         0.2  Iris-setosa
1          5.4         3.9          1.7         0.4  Iris-setosa
2          5.4         3.7          1.5         0.2  Iris-setosa
3          5.8         4.0          1.2         0.2  Iris-setosa
4          5.7         4.4          1.5         0.4  Iris-setosa
```

多种查询方式的示例：

-   与（&）条件

    ``` {#codeblock_irx_fdm_is1 .language-sql}
    iris[(iris.sepallength < 5) & (iris['petallength'] > 1.5)].head(5)
       sepallength  sepalwidth  petallength  petalwidth             name
    0          4.8         3.4          1.6         0.2      Iris-setosa
    1          4.8         3.4          1.9         0.2      Iris-setosa
    2          4.7         3.2          1.6         0.2      Iris-setosa
    3          4.8         3.1          1.6         0.2      Iris-setosa
    4          4.9         2.4          3.3         1.0  Iris-versicolor
    ```

-   或（|）条件

    ``` {#codeblock_xlq_mo9_ko0 .language-sql}
    iris[(iris.sepalwidth < 2.5) | (iris.sepalwidth > 4)].head(5)
       sepallength  sepalwidth  petallength  petalwidth             name
    0          5.7         4.4          1.5         0.4      Iris-setosa
    1          5.2         4.1          1.5         0.1      Iris-setosa
    2          5.5         4.2          1.4         0.2      Iris-setosa
    3          4.5         2.3          1.3         0.3      Iris-setosa
    4          5.5         2.3          4.0         1.3  Iris-versicolor
    ```

    **说明：** 与和或条件必须使用`&`和`|`，不能使用`and`和`or`。

-   非（~）条件

    ``` {#codeblock_rwh_7ir_kmi .language-sql}
    iris[~(iris.sepalwidth > 3)].head(5)
       sepallength  sepalwidth  petallength  petalwidth         name
    0          4.9         3.0          1.4         0.2  Iris-setosa
    1          4.4         2.9          1.4         0.2  Iris-setosa
    2          4.8         3.0          1.4         0.1  Iris-setosa
    3          4.3         3.0          1.1         0.1  Iris-setosa
    4          5.0         3.0          1.6         0.2  Iris-setosa
    ```

-   显式调用`filter`方法，提供多个与条件。

    ``` {#codeblock_fz0_4x0_9c4 .language-sql}
    iris.filter(iris.sepalwidth > 3.5, iris.sepalwidth < 4).head(5)
       sepallength  sepalwidth  petallength  petalwidth         name
    0          5.0         3.6          1.4         0.2  Iris-setosa
    1          5.4         3.9          1.7         0.4  Iris-setosa
    2          5.4         3.7          1.5         0.2  Iris-setosa
    3          5.4         3.9          1.3         0.4  Iris-setosa
    4          5.7         3.8          1.7         0.3  Iris-setosa
    ```

-   对于连续的操作，使用Lambda表达式。

    ``` {#codeblock_9n8_ntb_3dm .language-sql}
    iris[iris.sepalwidth > 3.8]['name', lambda x: x.sepallength + 1]
              name  sepallength
    0  Iris-setosa          6.4
    1  Iris-setosa          6.8
    2  Iris-setosa          6.7
    3  Iris-setosa          6.4
    4  Iris-setosa          6.2
    5  Iris-setosa          6.5
    ```

-   对于Collection，如果它包含一个列是BOOLEAN类型，则可以直接使用该列作为过滤条件。

    ``` {#codeblock_qcv_ah5_5i1 .language-sql}
    df.dtypes
    odps.Schema {
      a boolean
      b int64
    }
    df[df.a]
          a  b
    0  True  1
    1  True  3
    ```

    因此，对Collection取单个Sequence的操作时，只有BOOLEAN列是合法的，即对Collection作过滤操作。

    ``` {#codeblock_ma3_t8q_ajb .language-sql}
    df[df.a, ]       # 取列操作。
    df[[df.a]]       # 取列操作。
    df.select(df.a)  # 显式取列。
    df[df.a]         # a列是boolean列，执行过滤操作。
    df.a             # 取单列。
    df['a']          # 取单列。
    ```

-   同时，您可以使用Pandas中的`query`方法，通过查询语句做数据的筛选，在表达式中直接使用列名如`sepallength`进行操作。在查询语句中，`&`和`and`都表示与操作，`|`和`or`都表示或操作。

    ``` {#codeblock_a57_o21_dpv .language-sql}
    iris.query("(sepallength < 5) and (petallength > 1.5)").head(5)
       sepallength  sepalwidth  petallength  petalwidth             name
    0          4.8         3.4          1.6         0.2      Iris-setosa
    1          4.8         3.4          1.9         0.2      Iris-setosa
    2          4.7         3.2          1.6         0.2      Iris-setosa
    3          4.8         3.1          1.6         0.2      Iris-setosa
    4          4.9         2.4          3.3         1.0  Iris-versicolor
    ```

    当表达式中需要使用到本地变量时，需要在该变量前加一个`@`前缀。

    ``` {#codeblock_fdt_jkg_vs3 .language-sql}
    var = 4
    iris.query("(iris.sepalwidth < 2.5) | (sepalwidth > @var)").head(5)
       sepallength  sepalwidth  petallength  petalwidth             name
    0          5.7         4.4          1.5         0.4      Iris-setosa
    1          5.2         4.1          1.5         0.1      Iris-setosa
    2          5.5         4.2          1.4         0.2      Iris-setosa
    3          4.5         2.3          1.3         0.3      Iris-setosa
    4          5.5         2.3          4.0         1.3  Iris-versicolor
    ```

    目前`query`支持的语法，如下表所示。

    |语法|说明|
    |:-|:-|
    |name|没有`@`前缀的都当做列名处理，有前缀的会获取本地变量。|
    |operator|支持部分运算符：`+`、`-`、`*`、`/`、`//`、`%`、`**`、`==`、`!=`、`<`、`<=`、`>`、`>=`、`in`、`not in`。|
    |bool|与或非操作，其中`&`和`and`表示与，`|`和`or`表示或。|
    |attribute|取对象属性。|
    |index, slice, subscript|切片操作。|


## 并列多行输出 {#section_ndh_rw4_cfb .section}

对于LIST及MAP类型的列，`explode`方法会将该列转换为多行输出。您也可以使用`apply`方法实现多行输出。为了进行聚合等操作，常常需要将这些输出和原表中的列合并。此时可以使用DataFrame提供的并列多行输出功能， 写法为将多行输出函数生成的集合与原集合中的列名一起映射。

并列多行输出的示例如下。

``` {#codeblock_d4n_imx_bit .language-sql}
df
   id         a             b
0   1  [a1, b1]  [a2, b2, c2]
1   2      [c1]      [d2, e2]
df[df.id, df.a.explode(), df.b]
   id   a             b
0   1  a1  [a2, b2, c2]
1   1  b1  [a2, b2, c2]
2   2  c1      [d2, e2]
df[df.id, df.a.explode(), df.b.explode()]
   id   a   b
0   1  a1  a2
1   1  a1  b2
2   1  a1  c2
3   1  b1  a2
4   1  b1  b2
5   1  b1  c2
6   2  c1  d2
7   2  c1  e2
```

如果多行输出方法对某个输入不产生任何输出，默认输入行将不在最终结果中出现。如果需要在结果中出现该行，可以设置`keep_nulls=True`。此时，与该行并列的值将输出为空值。

``` {#codeblock_etu_554_rth .language-sql}
df
   id         a
0   1  [a1, b1]
1   2        []
df[df.id, df.a.explode()]
   id   a
0   1  a1
1   1  b1
df[df.id, df.a.explode(keep_nulls=True)]
   id     a
0   1    a1
1   1    b1
2   2  None
```

使用`explode`方法实现并列多行输出的示例，请参见[集合类型相关操作](cn.zh-CN/开发/PyODPS/DataFrame/列运算.md#)，使用`apply`方法实现并列多行输出的示例，请参见[对一行数据使用自定义函数](cn.zh-CN/开发/PyODPS/DataFrame/排序、去重、采样、数据变换.md#)。

## 限制条数 {#section_n1y_gx4_cfb .section}

``` {#codeblock_xni_g1h_4wd .language-sql}
iris[:3]
   sepallength  sepalwidth  petallength  petalwidth         name
0          5.1         3.5          1.4         0.2  Iris-setosa
1          4.9         3.0          1.4         0.2  Iris-setosa
2          4.7         3.2          1.3         0.2  Iris-setosa
```

目前对于ODPS SQL，后端切片不支持`start`和`step`方法，但支持`limit`方法。

``` {#codeblock_fvl_ea1_16f .language-sql}
iris.limit(3)
   sepallength  sepalwidth  petallength  petalwidth         name
0          5.1         3.5          1.4         0.2  Iris-setosa
1          4.9         3.0          1.4         0.2  Iris-setosa
2          4.7         3.2          1.3         0.2  Iris-setosa
```

**说明：** 切片操作只能作用于Collection，不能作用于Sequence。

