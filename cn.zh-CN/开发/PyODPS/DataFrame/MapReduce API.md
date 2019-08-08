# MapReduce API {#concept_cyr_tfg_cfb .concept}

本文为您介绍MapReduce API。

PyODPS DataFrame支持MapReduce API，您可以分别编写`map`和`reduce`函数（`map_reduce`可以只有`mapper`或者`reducer`过程）。

`wordcount`的示例如下。

``` {#codeblock_b77_720_bf3}
>>> #encoding=utf-8
>>> from odps import ODPS
>>> from odps import options
>>> options.verbose = True
>>> o = ODPS('your-access-id', 'your-secret-access-key',project='DMP_UC_dev', endpoint='http://service-corp.odps.aliyun-inc.com/api')
>>> from odps.df import DataFrame

>>> def mapper(row):
>>>     for word in row[0].split():
>>>         yield word.lower(), 1
>>>
>>> def reducer(keys):
>>>     # 这里使用list而不是cnt=0，否则h内的cnt会被认为是局部变量，其中的赋值无法输出。
>>>     cnt = [0]
>>>     def h(row, done):  # done表示这个key已经迭代结束。
>>>         cnt[0] += row[1]
>>>         if done:
>>>             yield keys[0], cnt[0]
>>>     return h
>>> # zx_word_count表只有一列，为STRING类型。
>>> word_count = DataFrame(o.get_table('zx_word_count'))
>>> table = word_count.map_reduce(mapper, reducer, group=['word', ],
                        mapper_output_names=['word', 'cnt'],
                        mapper_output_types=['string', 'int'],
                        reducer_output_names=['word', 'cnt'],
                        reducer_output_types=['string', 'int'])
         word  cnt
0         are    1
1         day    1
2      doing?    1
3   everybody    1
4       first    1
5       hello    2
6         how    1
7          is    1
8          so    1
9         the    1
10       this    1
11      world    1
12        you    1
```

`group`参数用于指定`reduce`按哪些字段做分组，如果不指定，会按全部字段做分组。`reducer`需要接收聚合的`keys`进行初始化，并能继续处理按这些`keys`聚合的每行数据。`done`表示与这些`keys`相关的所有行是否都迭代完成。

为了方便，此处写成了函数闭包的方式，您也可以写成`Callable`的类。

``` {#codeblock_5fp_1f3_qkj}
class reducer(object):
    def __init__(self, keys):
        self.cnt = 0

    def __call__(self, row, done):  # done表示这个key已经迭代结束。
        self.cnt += row.cnt
        if done:
            yield row.word, self.cnt
```

使用`output`进行注释会让代码更简单。

``` {#codeblock_dvu_4va_qbt}
>>> from odps.df import output
>>>
>>> @output(['word', 'cnt'], ['string', 'int'])
>>> def mapper(row):
>>>     for word in row[0].split():
>>>         yield word.lower(), 1
>>>
>>> @output(['word', 'cnt'], ['string', 'int'])
>>> def reducer(keys):
>>>     # 此处使用list而不是cnt=0，否则h内的cnt会被认为是局部变量，其中的赋值无法输出。
>>>     cnt = [0]
>>>     def h(row, done):  # done表示这个key已经迭代结束。
>>>         cnt[0] += row.cnt
>>>         if done:
>>>             yield keys.word, cnt[0]
>>>     return h
>>>
>>> word_count = DataFrame(o.get_table('zx_word_count'))
>>> table = word_count.map_reduce(mapper, reducer, group='word')
         word  cnt
0         are    1
1         day    1
2      doing?    1
3   everybody    1
4       first    1
5       hello    2
6         how    1
7          is    1
8          so    1
9         the    1
10       this    1
11      world    1
12        you    1
```

在迭代的时候，可以使用`sort`参数实现按指定列排序，通过`ascending`参数指定升序降序。`ascending`参数可以是一个BOOL值，表示所有的`sort`字段是相同升序或降序； 也可以是一个列表，长度必须和`sort`字段长度相同。

## 指定COMBINER {#section_lhr_f5n_cfb .section}

`combiner`表示在`map_reduce` API里表示在`mapper`端，就先对数据进行聚合操作，它的用法和`reducer`是完全一致的，但不能引用资源。 并且，`combiner`的输出的字段名和字段类型必须和`mapper`完全一致。

上面的例子，您可以使用`reducer`作为`combiner`在`mapper`端对数据做初步的聚合，减少Shuffle出去的数据量。

``` {#codeblock_gee_31q_hcl}
>>> words_df.map_reduce(mapper, reducer, combiner=reducer, group='word')
```

## 引用资源 {#section_h22_l5n_cfb .section}

在MapReduce API里，您可以分别指定`mapper`和`reducer`所要引用的资源。

以下示例为对`mapper`里的单词做停词过滤，在`reducer`里对白名单的单词数量加5。

``` {#codeblock_lw5_8zj_o2e}
>>> white_list_file = o.create_resource('pyodps_white_list_words', 'file', file_obj='Python\nWorld')
>>>
>>> @output(['word', 'cnt'], ['string', 'int'])
>>> def mapper(resources):
>>>     stop_words = set(r[0].strip() for r in resources[0])
>>>     def h(row):
>>>         for word in row[0].split():
>>>             if word not in stop_words:
>>>                 yield word, 1
>>>     return h
>>>
>>> @output(['word', 'cnt'], ['string', 'int'])
>>> def reducer(resources):
>>>     d = dict()
>>>     d['white_list'] = set(word.strip() for word in resources[0])
>>>     d['cnt'] = 0
>>>     def inner(keys):
>>>         d['cnt'] = 0
>>>         def h(row, done):
>>>             d['cnt'] += row.cnt
>>>             if done:
>>>                 if row.word in d['white_list']:
>>>                     d['cnt'] += 5
>>>                 yield keys.word, d['cnt']
>>>         return h
>>>     return inner
>>>
>>> words_df.map_reduce(mapper, reducer, group='word',
>>>                     mapper_resources=[stop_words], reducer_resources=[white_list_file])
     word  cnt
0   hello    2
1    life    1
2  python    7
3   world    6
4   short    1
5     use    1
```

## 使用第三方Python库 {#section_np5_n5n_cfb .section}

使用方法类似在MAP中使用第三方Python库。

-   在全局指定使用的库。

    ``` {#codeblock_b24_p7e_e0z}
    >>> from odps import options
    >>> options.df.libraries = ['six.whl', 'python_dateutil.whl']
    ```

-   在立即执行的方法中，局部指定使用的库。

    ``` {#codeblock_d61_hf8_96b}
    >>> df.map_reduce(mapper=my_mapper, reducer=my_reducer, group='key').execute(libraries=['six.whl', 'python_dateutil.whl'])
    ```

    **说明：** 由于字节码定义的差异，Python 3下使用新语言特性（例如`yield from`）时，代码在使用 Python 2.7的ODPS Worker上执行时会发生错误。因此，建议您在Python 3下使用MapReduce API编写生产作业前，先确认相关代码是否能正常执行。


## 重排数据 {#section_oss_2vn_cfb .section}

当数据在集群上分布不均匀时，您可以调用`reshuffle`接口对数据重排。

``` {#codeblock_cnu_o2l_36j}
>>> df1 = df.reshuffle()
```

默认会按随机数做哈希来分布，但您也可以按指定列做分布，并且可以指定重排后的排序顺序。

``` {#codeblock_9te_4t9_edz}
>>> df1.reshuffle('name', sort='id', ascending=False)
```

## 布隆过滤器 {#section_pgn_3vn_cfb .section}

PyODPS DataFrame提供了`bloom_filter`接口进行布隆过滤器的计算。

给定某个Collection和它的某个列计算的`sequence1`，对另外一个`sequence2`进行布隆过滤时，`sequence1`不存在于`sequence2`中的数据一定会被过滤掉， 但可能无法完全过滤。所以这种过滤方式是一种近似的过滤方法。这样的好处是对Collection进行快速过滤一些无用数据。这在一边数据量远大过另一边数据量（大部分数据并不会参与`join`运算）时的大规模`join`场景很有用。例如，在`join`用户的浏览数据和交易数据时，用户的浏览数据量远大于交易数据量，可以利用交易数据先对浏览数据进行布隆过滤， 然后再`join`可以很好地提升性能。

``` {#codeblock_7r5_qjz_9nd}
>>> df1 = DataFrame(pd.DataFrame({'a': ['name1', 'name2', 'name3', 'name1'], 'b': [1, 2, 3, 4]}))
>>> df1
       a  b
0  name1  1
1  name2  2
2  name3  3
3  name1  4
>>> df2 = DataFrame(pd.DataFrame({'a': ['name1']}))
>>> df2
       a
0  name1
>>> df1.bloom_filter('a', df2.a) # 这里第0个参数可以是个计算表达式如: df1.a + '1'。
       a  b
0  name1  1
1  name1  4
```

示例说明：

-   由于数据量很小，`df1`中的`a`为`name2`和`name3`的行都被正确过滤掉了，当数据量很大的时候，可能无法过滤全部数据。
-   之前提`join`场景中，少量数据不能被过滤的情况并不会影响数据的正确性，但可以较大地提升`join`的性能。
-   您可以传入`capacity`和`error_rate`来设置数据的量以及错误率，默认值是`3000`和`0.01`。

**说明：** 要注意，调大`capacity`或者减小`error_rate`会增加内存的使用，所以应当根据实际情况选择一个合理的值。

Collection对象操作请参见[DataFrame执行](cn.zh-CN/开发/PyODPS/DataFrame/执行.md#)。

## 透视表（PIVOT\_TABLE） {#section_gmz_jwn_cfb .section}

PyODPS DataFrame提供了透视表的功能。示例的表数据如下。

``` {#codeblock_zc4_dhp_qz4}
>>> df
     A    B      C  D  E
0  foo  one  small  1  3
1  foo  one  large  2  4
2  foo  one  large  2  5
3  foo  two  small  3  6
4  foo  two  small  3  4
5  bar  one  large  4  5
6  bar  one  small  5  3
7  bar  two  small  6  2
8  bar  two  large  7  1
```

-   使用透视表功能时，`rows`参数为必选参数，表示按一个或者多个字段做取平均值的操作。

    ``` {#codeblock_vig_5ak_e5b}
    >>> df['A', 'D', 'E'].pivot_table(rows='A')
         A  D_mean  E_mean
    0  bar     5.5    2.75
    1  foo     2.2    4.40
    ```

-   `rows`可以提供多个字段，表示按多个字段做聚合。

    ``` {#codeblock_tex_ppm_pzx}
    >>> df.pivot_table(rows=['A', 'B', 'C'])
         A    B      C  D_mean  E_mean
    0  bar  one  large     4.0     5.0
    1  bar  one  small     5.0     3.0
    2  bar  two  large     7.0     1.0
    3  bar  two  small     6.0     2.0
    4  foo  one  large     2.0     4.5
    5  foo  one  small     1.0     3.0
    6  foo  two  small     3.0     5.0
    ```

-   指定`values`显示指定要计算的列。

    ``` {#codeblock_995_hso_838}
    >>> df.pivot_table(rows=['A', 'B'], values='D')
         A    B    D_mean
    0  bar  one  4.500000
    1  bar  two  6.500000
    2  foo  one  1.666667
    3  foo  two  3.000000
    ```

-   计算值列时，默认会计算平均值。通过`aggfunc`指定一个或者多个聚合函数。

    ``` {#codeblock_sgv_pn7_8mb}
    >>> df.pivot_table(rows=['A', 'B'], values=['D'], aggfunc=['mean', 'count', 'sum'])
         A    B    D_mean  D_count  D_sum
    0  bar  one  4.500000        2      9
    1  bar  two  6.500000        2     13
    2  foo  one  1.666667        3      5
    3  foo  two  3.000000        2      6
    ```

-   将原始数据的某一列的值，作为新的Collection的列。

    ``` {#codeblock_low_nv5_qzf}
    >>> df.pivot_table(rows=['A', 'B'], values='D', columns='C')
         A    B  large_D_mean  small_D_mean
    0  bar  one           4.0           5.0
    1  bar  two           7.0           6.0
    2  foo  one           2.0           1.0
    3  foo  two           NaN           3.0
    ```

-   使用`fill_value`填充空值。

    ``` {#codeblock_9lj_6i6_cci}
    >>> df.pivot_table(rows=['A', 'B'], values='D', columns='C', fill_value=0)
         A    B  large_D_mean  small_D_mean
    0  bar  one             4             5
    1  bar  two             7             6
    2  foo  one             2             1
    3  foo  two             0             3
    ```


## KEY-VALUE字符串转换 {#section_nq4_ywn_cfb .section}

DataFrame提供了将Key-Value对展开为列，以及将普通列转换为Key-Value列的功能。

-   将Key-Value对展开为列，示例数据如下。

    ``` {#codeblock_b7m_z4l_3uf}
    >>> df
        name               kv
    0  name1  k1=1,k2=3,k5=10
    1  name1    k1=7.1,k7=8.2
    2  name2    k2=1.2,k3=1.5
    3  name2      k9=1.1,k2=1
    ```

    通过`extract_kv`方法将Key-Value字段展开。

    ``` {#codeblock_9w2_x3l_3f9}
    >>> df.extract_kv(columns=['kv'], kv_delim='=', item_delim=',')
       name   kv_k1  kv_k2  kv_k3  kv_k5  kv_k7  kv_k9
    0  name1    1.0    3.0    NaN   10.0    NaN    NaN
    1  name1    7.0    NaN    NaN    NaN    8.2    NaN
    2  name2    NaN    1.2    1.5    NaN    NaN    NaN
    3  name2    NaN    1.0    NaN    NaN    NaN    1.1
    ```

    其中，需要展开的字段名由`columns`指定，Key和Value之间的分隔符，以及Key-Value对之间的分隔符分别由`kv_delim`和`item_delim`这两个参数指定，默认分别为半角冒号和半角逗号。输出的字段名为原字段名和Key值的组合，通过`_`相连。缺失值默认为NONE，可通过`fill_value`选择需要填充的值。

    ``` {#codeblock_glc_fsg_d3h}
    >>> df.extract_kv(columns=['kv'], kv_delim='=', fill_value=0)
       name   kv_k1  kv_k2  kv_k3  kv_k5  kv_k7  kv_k9
    0  name1    1.0    3.0    0.0   10.0    0.0    0.0
    1  name1    7.0    0.0    0.0    0.0    8.2    0.0
    2  name2    0.0    1.2    1.5    0.0    0.0    0.0
    3  name2    0.0    1.0    0.0    0.0    0.0    1.1
    ```

-   将多列数据转换为一个Key-Value列，示例数据如下。

    ``` {#codeblock_ukx_8qr_o20}
    >>> df
       name    k1   k2   k3    k5   k7   k9
    0  name1  1.0  3.0  NaN  10.0  NaN  NaN
    1  name1  7.0  NaN  NaN   NaN  8.2  NaN
    2  name2  NaN  1.2  1.5   NaN  NaN  NaN
    3  name2  NaN  1.0  NaN   NaN  NaN  1.1
    ```

    通过`to_kv`方法转换为Key-Value表示的格式。

    ``` {#codeblock_smd_epp_znc}
    >>> df.to_kv(columns=['k1', 'k2', 'k3', 'k5', 'k7', 'k9'], kv_delim='=')
        name               kv
    0  name1  k1=1,k2=3,k5=10
    1  name1    k1=7.1,k7=8.2
    2  name2    k2=1.2,k3=1.5
    3  name2      k9=1.1,k2=1
    ```


-   
DataFrame创建及其对象操作请参见[创建DataFrame](cn.zh-CN/开发/PyODPS/DataFrame/创建DataFrame.md#)。

