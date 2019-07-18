# 使用自定义函数及Python第三方库 {#concept_xhx_rfg_cfb .concept}

本文为您介绍如何使用自定义函数及Python第三方库。

## 使用自定义函数 {#section_q1x_07w_b5w .section}

DataFrame函数支持对Sequence使用`map`，它会对它的每个元素调用自定义函数。

``` {#codeblock_m55_m75_u04}
>>> iris.sepallength.map(lambda x: x + 1).head(5)
   sepallength
0          6.1
1          5.9
2          5.7
3          5.6
4          6.0
```

**说明：** 目前，受限于Python UDF，自定义函数无法支持将List/Dict类型作为输入或输出。

如果`map`前后，Sequence的类型发生了变化，则需要显式指定`map`后的类型。

``` {#codeblock_e4b_eww_44k}
>>> iris.sepallength.map(lambda x: 't'+str(x), 'string').head(5)
   sepallength
0         t5.1
1         t4.9
2         t4.7
3         t4.6
4         t5.0
```

如果在函数中包含闭包，则函数外闭包变量值的变化会引起函数内该变量值的变化。

``` {#codeblock_ffs_8pk_2ub}
>>> dfs = []
>>> for i in range(10):
>>>     dfs.append(df.sepal_length.map(lambda x: x + i))
```

结果为`dfs`中每个`SequenceExpr`均为`df.sepal_length+9`。为解决此问题，可以将函数作为另一函数的返回值，或者使用`partial`。两个示例如下。

``` {#codeblock_pih_y3c_shh}
>>> dfs = []
>>> def get_mapper(i):
>>>     return lambda x: x + i
>>> for i in range(10):
>>>     dfs.append(df.sepal_length.map(get_mapper(i)))
```

``` {#codeblock_35r_5a6_moc}
>>> import functools
>>> dfs = []
>>> for i in range(10):
>>>     dfs.append(df.sepal_length.map(functools.partial(lambda v, x: x + v, i)))
```

`map`也支持使用现有的UDF函数，传入的参数是str类型（函数名）或者Function对象，详情请参见[函数](cn.zh-CN/开发/PyODPS/基本操作/函数.md#)。

`map`传入Python函数的实现使用了MaxCompute Python UDF。因此，如果您所在的Project不支持Python UDF，则`map`函数无法使用。除此以外，所有Python UDF的限制在此都适用。

目前，默认可使用的第三方库（包含C）只有numpy，第三方库使用详情请参见[使用第三方Python库](#section_jly_c2v_npi)。

除了调用自定义函数，DataFrame还提供了很多内置函数，这些函数中部分使用了`map`函数来实现。因此，如果您所在Project未开通Python UDF，则无法使用这些函数（注意：阿里云公共服务暂不提供对Python UDF的支持）。

**说明：** 由于字节码定义的差异，Python 3下使用新语言特性（例如`yield from`）时，代码在使用Python 2.7的MaxCompute Worker上执行时会发生错误。因此，建议您在Python 3下使用MapReduce API编写生产作业前，先确认相关代码是否能正常执行。

示例程序：使用计数器

``` {#codeblock_5m7_o3w_kox}
from odps.udf import get_execution_context
def h(x):
    ctx = get_execution_context()
    counters = ctx.get_counters()
    counters.get_counter('df', 'add_one').increment(1)
    return x + 1
df.field.map(h)
```

Logview的JSONSummary中即可找到计数器值。

## 对一行数据使用自定义函数 {#section_d3c_1pn_cfb .section}

如果您需要对一行数据使用自定义函数，可以使用`apply`方法。参数`axis`的值必须设为1，表示对行进行操作。`apply`的自定义函数接收一个参数，参数为上一步Collection的一行数据。您可以通过属性或者偏移获得一个字段的数据。

-   `reduce`为True时，表示返回结果为Sequence，否则返回结果为Collection。 `names`和`types`参数分别指定返回的Sequence或Collection的字段名和类型。 如果未指定类型，则会默认为STRING类型。

    ``` {#codeblock_519_13t_x5m .language-sql}
    >>> iris.apply(lambda row: row.sepallength + row.sepalwidth, axis=1, reduce=True, types='float').rename('sepaladd').head(3)
       sepaladd
    0       8.6
    1       7.9
    2       7.9
    ```

-   在`apply`的自定义函数中，`reduce`为False时，您可以使用`yield`关键字返回多行结果。

    ``` {#codeblock_rdz_l1u_08b .language-sql}
    >>> iris.count()
    150
    >>>
    >>> def handle(row):
    >>>     yield row.sepallength - row.sepalwidth, row.sepallength + row.sepalwidth
    >>>     yield row.petallength - row.petalwidth, row.petallength + row.petalwidth
    >>>
    >>> iris.apply(handle, axis=1, names=['iris_add', 'iris_sub'], types=['float', 'float']).count()
    300
    ```

-   您也可以在函数中注释返回的字段和类型，无需在函数调用时再指定它们。

    ``` {#codeblock_yav_wan_xwn .language-sql}
    >>> from odps.df import output
    >>>
    >>> @output(['iris_add', 'iris_sub'], ['float', 'float'])
    >>> def handle(row):
    >>>     yield row.sepallength - row.sepalwidth, row.sepallength + row.sepalwidth
    >>>     yield row.petallength - row.petalwidth, row.petallength + row.petalwidth
    >>>
    >>> iris.apply(handle, axis=1).count()
    300
    ```

-   您也可以使用`map-only`的`map_reduce`，该操作与`axis=1`的`apply`操作是等价的。

    ``` {#codeblock_mf8_vxf_4ca .language-sql}
    >>> iris.map_reduce(mapper=handle).count()
    300
    ```

-   如果您想调用ODPS上已经存在的UDTF，函数指定为函数名即可。

    ``` {#codeblock_t9d_wf3_lsu .language-sql}
    >>> iris['name', 'sepallength'].apply('your_func', axis=1, names=['name2', 'sepallength2'], types=['string', 'float'])
    ```

-   使用`apply`对行操作，且`reduce`为False时，您可以使用并列多行输出与已有的行结合，用于后续聚合等操作。

    ``` {#codeblock_v0m_7qf_yjr .language-sql}
    >>> from odps.df import output
    >>>
    >>> @output(['iris_add', 'iris_sub'], ['float', 'float'])
    >>> def handle(row):
    >>>     yield row.sepallength - row.sepalwidth, row.sepallength + row.sepalwidth
    >>>     yield row.petallength - row.petalwidth, row.petallength + row.petalwidth
    >>>
    >>> iris[iris.category, iris.apply(handle, axis=1)]
    ```


## 对所有列调用自定义聚合 {#section_ryl_1sn_cfb .section}

调用`apply`方法，当不指定`axis`，或者`axis`值为0时，您可以通过传入一个自定义聚合类对所有Sequence进行聚合操作。

``` {#codeblock_9jg_0nj_ur1 .language-sql}
class Agg(object):

    def buffer(self):
        return [0.0, 0]

    def __call__(self, buffer, val):
        buffer[0] += val
        buffer[1] += 1

    def merge(self, buffer, pbuffer):
        buffer[0] += pbuffer[0]
        buffer[1] += pbuffer[1]

    def getvalue(self, buffer):
        if buffer[1] == 0:
            return 0.0
        return buffer[0] / buffer[1]
```

``` {#codeblock_tgf_r2o_z6d .language-sql}
>>> iris.exclude('name').apply(Agg)
   sepallength_aggregation  sepalwidth_aggregation  petallength_aggregation  petalwidth_aggregation
0                 5.843333                   3.054                 3.758667                1.198667
```

**说明：** 目前，受限于Python UDF，自定义函数无法支持将LIST/DICT类型作为初始输入或最终输出结果。

## 引用资源 {#section_esf_jsn_cfb .section}

自定义函数也能读取MaxCompute上的资源（表资源或文件资源），或者引用一个Collection作为资源。此时，自定义函数需要写成函数闭包或Callable的类。两个示例如下。

``` {#codeblock_qcm_ldy_w1o}
>>> file_resource = o.create_resource('pyodps_iris_file', 'file', file_obj='Iris-setosa')
>>>
>>> iris_names_collection = iris.distinct('name')[:2]
>>> iris_names_collection
       sepallength
0      Iris-setosa
1  Iris-versicolor
```

``` {#codeblock_dn0_nen_r4v}
>>> def myfunc(resources):  # resources按调用顺序传入。
>>>     names = set()
>>>     fileobj = resources[0] # 文件资源是一个file-like的object。
>>>     for l in fileobj:
>>>         names.add(l)
>>>     collection = resources[1]
>>>     for r in collection:
>>>         names.add(r.name)  # 这里可以通过字段名或者偏移来取。
>>>     def h(x):
>>>         if x in names:
>>>             return True
>>>         else:
>>>             return False
>>>     return h
>>>
>>> df = iris.distinct('name')
>>> df = df[df.name,
>>>         df.name.map(myfunc, resources=[file_resource, iris_names_collection], rtype='boolean').rename('isin')]
>>>
>>> df
              name   isin
0      Iris-setosa   True
1  Iris-versicolor   True
2   Iris-virginica  False
```

**说明：** 分区表资源在读取时不包含分区字段。

当`axis`值为1，即在行上操作时，您需要写一个函数闭包或者Callable的类。 而对于列上的聚合操作，您只需在 `__init__` 函数里读取资源即可。

``` {#codeblock_aop_oim_g2v .language-sql}
>>> words_df
                     sentence
0                 Hello World
1                Hello Python
2  Life is short I use Python
>>>
>>> import pandas as pd
>>> stop_words = DataFrame(pd.DataFrame({'stops': ['is', 'a', 'I']}))
>>>
>>> @output(['sentence'], ['string'])
>>> def filter_stops(resources):
>>>     stop_words = set([r[0] for r in resources[0]])
>>>     def h(row):
>>>         return ' '.join(w for w in row[0].split() if w not in stop_words),
>>>     return h
>>>
>>> words_df.apply(filter_stops, axis=1, resources=[stop_words])
                sentence
0            Hello World
1           Hello Python
2  Life short use Python
```

**说明：** 这里的`stop_words`存放于本地，但在真正执行时会被上传到ODPS作为资源引用。

## 使用第三方Python库 {#section_jly_c2v_npi .section}

您可以把第三方Python包作为资源上传到MaxCompute，支持的格式有whl、egg、zip以及tar.gz。在全局或者在立即执行的方法时，指定需要使用的包文件，即可以在自定义函数中使用第三方库。第三方库的依赖库，也必须指定，否则依然会导致导入错误。

-   您可以通过PyODPS的资源上传接口来完成资源的上传。

    下面以python-dateutil包进行举例：

    1.  使用`pip download`命令，下载包以及其依赖到某个路径。下载后会出现两个包：six-1.10.0-py2.py3-none-any.whl和python\_dateutil-2.5.3-py2.py3-none-any.whl注意：这里需要下载支持Linux环境的包）。

        ``` {#codeblock_9b3_2lk_krb}
        $ pip download python-dateutil -d /to/path/
        ```

    2.  分别把两个文件上传到MaxCompute资源。

        ``` {#codeblock_ajq_mhr_rgo}
        >>> # 这里要确保资源名的后缀是正确的文件类型。
        >>> odps.create_resource('six.whl', 'file', file_obj=open('six-1.10.0-py2.py3-none-any.whl', 'rb'))
        >>> odps.create_resource('python_dateutil.whl', 'file', file_obj=open('python_dateutil-2.5.3-py2.py3-none-any.whl', 'rb'))
        ```

    3.  现在有个DataFrame，只有一个STRING类型字段。

        ``` {#codeblock_n2p_vbj_ytt}
        >>> df
                       datestr
        0  2016-08-26 14:03:29
        1  2015-08-26 14:03:29
        ```

    4.  全局配置使用到的三方库如下。

        ``` {#codeblock_4lq_t0s_skf}
        >>> from odps import options
        >>>
        >>> def get_year(t):
        >>>     from dateutil.parser import parse
        >>>     return parse(t).strftime('%Y')
        >>>
        >>> options.df.libraries = ['six.whl', 'python_dateutil.whl']
        >>> df.datestr.map(get_year)
           datestr
        0     2016
        1     2015
        ```

        或者通过运行方法的`libraries`参数指定使用到的第三方库。

        ``` {#codeblock_tln_ole_6t0}
        >>> def get_year(t):
        >>>     from dateutil.parser import parse
        >>>     return parse(t).strftime('%Y')
        >>>
        >>> df.datestr.map(get_year).execute(libraries=['six.whl', 'python_dateutil.whl'])
           datestr
        0     2016
        1     2015
        ```

    PyODPS默认支持执行仅包含Python且不含文件操作的第三方库。在较新版本的MaxCompute服务下，PyODPS也支持执行带有二进制代码或带有文件操作的Python库。这些库的后缀必须是cp27-cp27m-manylinux1\_x86\_64，以`archive`方式上传，whl后缀的包需要重命名为zip。同时，作业需要开启`odps.isolation.session.enable`选项，或者在Project级别开启`isolation`。以下示例为您展示如何上传并使用`scipy`中的特殊函数。

    ``` {#codeblock_yov_4so_a0v}
    >>> # 对于含有二进制代码的包，必须使用archive方式上传资源，whl后缀需要改为zip。
    >>> odps.create_resource('scipy.zip', 'archive', file_obj=open('scipy-0.19.0-cp27-cp27m-manylinux1_x86_64.whl', 'rb'))
    >>>
    >>> # 如果Project开启了isolation，下面的选项不是必需的。
    >>> options.sql.settings = { 'odps.isolation.session.enable': True }
    >>>
    >>> def psi(value):
    >>>     # 建议在函数内部import第三方库，以防止不同操作系统下二进制包结构差异造成执行错误。
    >>>     from scipy.special import psi
    >>>     return float(psi(value))
    >>>
    >>> df.float_col.map(psi).execute(libraries=['scipy.zip'])
    ```

    对于只提供源码的二进制包，可以在Linux Shell中打包成Wheel再上传，Mac和Windows中生成的Wheel包无法在MaxCompute中使用。

    ``` {#codeblock_q5y_e2p_xju}
    python setup.py bdist_wheel
    ```

-   您也可以通过MaxCompute Console上传资源。

    1.  现在主流的Python包都提供了whl包，提供了各平台包含二进制文件的包，因此找到可以在MaxCompute上运行的包是第一步。
    2.  其次，要想在MaxCompute上运行，需要包含所有的依赖包，这个是比较繁琐的。各个包的依赖情况如下表所示。

        |包名|依赖|
        |:-|:-|
        |pandas|numpy、python-dateutil、pytz、six|
        |scipy|numpy|
        |scikit-learn|numpy、scipy|

        **说明：** 其中numpy已包含，您只需上传python-dateutil、pytz、pandas、scipy、sklearn、six包，pandas、scipy和scikit-learn即可使用。

    3.  您可进入[python-dateutils](http://mirrors.aliyun.com/pypi/simple/python-dateutil/)找到[python-dateutil-2.6.0.zip](http://mirrors.aliyun.com/pypi/packages/95/8e/71125f3f24771f50e630b5a6fa9fd209a9f167dcbc3aad65a48cb3dd5694/python-dateutil-2.6.0.zip#md5=530f7b56e36fa42ada6c02a17b15660c)进行下载。

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/15204/15634361856659_zh-CN.png)

    4.  重命名为python-dateutil.zip，通过MaxCompute Console上传资源。

        ``` {#codeblock_e1b_l0e_doj}
        add archive python-dateutil.zip;
        ```

        **说明：** pytz和six的上传方式同上，分别找到 [pytz-2017.2.zip](http://mirrors.aliyun.com/pypi/packages/a4/09/c47e57fc9c7062b4e83b075d418800d322caa87ec0ac21e6308bd3a2d519/pytz-2017.2.zip#md5=f89bde8a811c8a1a5bac17eaaa94383c)和[six-1.11.0.tar.gz](http://mirrors.aliyun.com/pypi/packages/16/d8/bc6316cf98419719bd59c91742194c111b6f2e85abac88e496adefaf7afe/six-1.11.0.tar.gz#md5=d12789f9baf7e9fb2524c0c64f1773f8)进行下载和上传资源操作。

    5.  对于Pandas这种包含c的包，需要找到名字中包含cp27-cp27m-manylinux1\_x86\_64的whl包，这样才能在MaxCompute上正确执行。因此，您需要找到[pandas-0.20.2-cp27-cp27m-manylinux1\_x86\_64.whl](http://mirrors.aliyun.com/pypi/packages/44/39/e71009a0ebdbb6206b9fbde0367fc5cb5bb7fdb4521ae785ca7bd63d36aa/pandas-0.20.2-cp27-cp27m-manylinux1_x86_64.whl#md5=31a4d180048f72337d53cc7b87424568)进行下载，然后把后缀改成zip，在MaxCompute Console中执行`add archive pandas.zip;`进行上传。scipy和scikit-learn包的操作同上。
    所有包需要下载的资源如下表所示。

    |包名|文件名|上传资源名|
    |:-|:--|:----|
    |python-dateutil|[python-dateutil-2.6.0.zip](http://mirrors.aliyun.com/pypi/packages/95/8e/71125f3f24771f50e630b5a6fa9fd209a9f167dcbc3aad65a48cb3dd5694/python-dateutil-2.6.0.zip#md5=530f7b56e36fa42ada6c02a17b15660c)|python-dateutil.zip|
    |pytz|[pytz-2017.2.zip](http://mirrors.aliyun.com/pypi/packages/a4/09/c47e57fc9c7062b4e83b075d418800d322caa87ec0ac21e6308bd3a2d519/pytz-2017.2.zip#md5=f89bde8a811c8a1a5bac17eaaa94383c)|pytz.zip|
    |six|[six-1.11.0.tar.gz](http://mirrors.aliyun.com/pypi/packages/16/d8/bc6316cf98419719bd59c91742194c111b6f2e85abac88e496adefaf7afe/six-1.11.0.tar.gz#md5=d12789f9baf7e9fb2524c0c64f1773f8)|six.tar.gz|
    |pandas|[pandas-0.20.2-cp27-cp27m-manylinux1\_x86\_64.zip](http://mirrors.aliyun.com/pypi/packages/44/39/e71009a0ebdbb6206b9fbde0367fc5cb5bb7fdb4521ae785ca7bd63d36aa/pandas-0.20.2-cp27-cp27m-manylinux1_x86_64.whl#md5=31a4d180048f72337d53cc7b87424568)|pandas.zip|
    |scipy|[scipy-0.19.0-cp27-cp27m-manylinux1\_x86\_64.zip](http://mirrors.aliyun.com/pypi/packages/ae/94/28ca6f9311e2351bb68da41ff8c1bc8f82bb82791f2ecd34efa953e60576/scipy-0.19.0-cp27-cp27m-manylinux1_x86_64.whl#md5=0e49f7fc8d31c1c79f0a4d63b29e8a1f)|scipy.zip|
    |scikit-learn|[scikit\_learn-0.18.1-cp27-cp27m-manylinux1\_x86\_64.zip](http://mirrors.aliyun.com/pypi/packages/ca/dd/a18dba8ab879b13b43c3838a25887585a45101f4bffa398e1883e1e3d395/scikit_learn-0.18.1-cp27-cp27m-manylinux1_x86_64.whl#md5=b068bde57f00d285cc89eb0b8615fcae)|sklearn.zip|


## 指定第三方Python库 {#section_hxh_qsn_cfb .section}

-   在全局指定使用的库。

    ``` {#codeblock_9pf_nss_ii1 .language-sql}
    >>> from odps import options
    >>> options.df.libraries = ['six.whl', 'python_dateutil.whl']
    ```

-   在立即执行的方法中，局部指定使用的库。

    ``` {#codeblock_nf1_chf_urm .language-sql}
    >>> df.apply(my_func, axis=1).to_pandas(libraries=['six.whl', 'python_dateutil.whl'])
    ```


