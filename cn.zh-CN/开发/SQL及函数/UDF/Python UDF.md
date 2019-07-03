# Python UDF {#concept_pjv_qs2_xdb .concept}

MaxCompute UDF包括UDF、UDAF和UDTF三种函数。本文为您介绍如何通过MaxCompute Python实现这三种函数。

## 受限环境 {#section_gwg_3ff_xdb .section}

MaxCompute UDF的Python版本为2.7，并以沙箱模式执行用户代码，即代码是在一个受限的运行环境中执行的。在这个环境中，以下行为会被禁止：

-   读写本地文件
-   启动子进程
-   启动线程
-   使用Socket通信
-   其他系统调用

基于上述原因，您上传的代码必须都是纯Python实现，C扩展模块是被禁止的。

此外，Python的标准库中也不是所有模块都可用，涉及到上述功能的模块都会被禁止。具体标准库可用模块说明如下：

-   所有纯Python实现（不依赖扩展模块）的模块都可用。
-   C实现的扩展模块中下列模块可用：
    -   array
    -   audioop
    -   binascii
    -   \_bisect
    -   cmath
    -   \_codecs\_cn
    -   \_codecs\_hk
    -   \_codecs\_iso2022
    -   \_codecs\_jp
    -   \_codecs\_kr
    -   \_codecs\_tw
    -   \_collections
    -   cStringIO
    -   datetime
    -   \_functools
    -   future\_builtins
    -   \_hashlib
    -   \_heapq
    -   itertools
    -   \_json
    -   \_locale
    -   \_lsprof
    -   math
    -   \_md5
    -   \_multibytecodec
    -   operator
    -   \_random
    -   \_sha256
    -   \_sha512
    -   \_sha
    -   \_struct
    -   strop
    -   time
    -   unicodedata
    -   \_weakref
    -   cPickle
-   部分模块功能受限。例如，沙箱限制了您的代码最多可往标准输出和标准错误输出写出数据的大小，即`sys.stdout/sys.stderr`最多能写20KB，多余的字符会被忽略。

## 第三方库 {#section_lrw_zff_xdb .section}

运行环境中还安装了除标准库外比较常用的第三方库，做为标准库的补充，例如Numpy。

**说明：** 第三方库的使用同样受到禁止本地、网络IO或其他在受限环境下的限制，因此第三方库中涉及到相关功能的API也是被禁止的。

## 参数与返回值类型 {#section_mrj_cgf_xdb .section}

参数与返回值的指定方式，如下所示。

``` {#codeblock_ulp_3kx_hvz}
@odps.udf.annotate(signature)
```

Python UDF目前支持的MaxCompute SQL数据类型包括BIGINT、STRING、DOUBLE、BOOLEAN和DATETIME。SQL语句在执行之前，必须确定所有函数的参数类型和返回值类型。因此对于Python这一动态类型语言，需要通过对UDF类加Decorator的方式指定函数签名。

函数签名Signature通过字符串指定，命令格式如下。

``` {#codeblock_z5z_a8w_s1s}
arg_type_list '->' type_list
arg_type_list: type_list | '*' | ''
type_list: [type_list ','] type
type: 'bigint' | 'string' | 'double' | 'boolean' | 'datetime'
```

-   箭头左边表示参数类型，右边表示返回值类型。
-   只有UDTF的返回值可以是多列，UDF和UDAF只能返回一列。
-   星号（`*`）代表变长参数。使用变长参数时，UDF、UDTF、UDAF可以匹配任意输入参数。

合法的Signature示例如下。

``` {#codeblock_ae2_7dw_ykd}
'bigint,double->string'            # 参数为bigint、double，返回值为string。
'bigint,boolean->string,datetime'  # UDTF参数为bigint、boolean，返回值为string,datetime。
'*->string'                        # 变长参数，输入参数任意，返回值为string。
'->double'                         # 参数为空，返回值为double。
```

Query语义解析阶段会检查到不符合函数签名的用法，抛出错误禁止执行。执行时，UDF函数的参数会以函数签名指定的类型传给您。返回值类型也要与函数签名指定的类型一致，否则检查到类型不匹配时也会报错。MaxCompute SQL数据类型对应的Python类型如下：

|ODPS SQL Type|Bigint|String|Double|Boolean|Datetime|
|:------------|:-----|:-----|:-----|:------|:-------|
|Python Type|int|str|float|bool|int|

**说明：** 

-   DATETIME类型是以INT的形式传给用户代码的，值为Epoch UTC Time起始至今的毫秒数。您可以通过Python标准库中的DATETIME模块处理日期时间类型。
-   NULL值对应Python里的None。

此外，`odps.udf.int(value,[silent=True])`增加了参数`silent`。当`silent`为True时，如果`value`无法转为INT，则会返回None（不会抛出异常）。

## UDF {#section_0l2_jsh_vbr .section}

实现Python UDF非常简单，只需要定义一个new-style class，并实现`evaluate`方法。

``` {#codeblock_4kc_pqx_gqi}
from odps.udf import annotate
@annotate("bigint,bigint->bigint")
class MyPlus(object):
   def evaluate(self, arg0, arg1):
       if None in (arg0, arg1):
           return None
       return arg0 + arg1
```

**说明：** Python UDF必须通过`annotate`指定函数签名。

从2018年10月16日开始，MaxCompute公共云环境下的Python UDF使用权限全面开放。

Python UDF的使用示例请参考[使用MaxCompute分析IP来源最佳实践](../../../../cn.zh-CN/最佳实践/数据开发/使用MaxCompute分析IP来源最佳实践.md#)。

## UDAF {#section_ilj_mhf_xdb .section}

-   `class odps.udf.BaseUDAF`：继承此类实现Python UDAF。
-   `BaseUDAF.new_buffer()`：实现此方法返回聚合函数的中间值的`buffer`。`buffer`必须是[marshallable](https://docs.python.org/3.3/library/marshal.html#module-marshal)Object（例如LIST、DICT），并且`buffer`的大小不应该随数据量递增。在极限情况下，`buffer`Marshal过后的大小不应该超过2MB。
-   `BaseUDAF.iterate(buffer[, args, ...])`：实现此方法将`args`聚合到中间值`buffer`中。
-   `BaseUDAF.merge(buffer, pbuffer)`：实现此方法将两个中间值`buffer`聚合到一起，即将`pbuffer`合并到`buffer`中。
-   `BaseUDAF.terminate(buffer)`：实现此方法将中间值`buffer`转换为MaxCompute SQL的基本类型。

UDAF求平均值的示例如下。

``` {#codeblock_ajr_b7t_cw2}
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

## UDTF {#section_ipk_thf_xdb .section}

-   `class odps.udf.BaseUDTF`：Python UDTF的基类。您可以继承此类实现`process`、`close`等方法。
-   `BaseUDTF.__init__()`：初始化方法。继承类如果需要实现这个方法，必须在一开始调用基类的初始化方法`super(BaseUDTF, self).__init__()`。`init`方法在整个UDTF生命周期中只会被调用一次，即在处理第一条记录之前。当UDTF需要保存内部状态时，可以在这个方法中初始化所有状态。
-   `BaseUDTF.process([args, ...])`：这个方法由MaxCompute SQL框架调用，SQL中每一条记录都会对应调用一次`process`，`process`的参数为SQL语句中指定的UDTF输入参数。
-   `BaseUDTF.forward([args, ...])`：UDTF的输出方法。此方法由用户代码调用。每调用一次`forward`，便会输出一条记录。 `forward`的参数为SQL语句中指定的UDTF的输出参数。
-   `BaseUDTF.close()`：UDTF的结束方法。此方法由MaxCompute SQL框架调用，并且只会被调用一次，即在处理完最后一条记录之后。

UDTF的示例如下。

``` {#codeblock_ip7_rom_1ws}
#coding:utf-8
# explode.py
from odps.udf import annotate
from odps.udf import BaseUDTF
@annotate('string -> string')
class Explode(BaseUDTF):
   """将string按逗号分隔输出成多条记录。
   def process(self, arg):
       props = arg.split(',')
       for p in props:
           self.forward(p)
```

**说明：** Python UDTF也可以不加`annotate`指定参数类型和返回值类型。这样，函数在SQL中使用时可以匹配任意输入参数，但返回值类型无法推导，所有输出参数都将被视为STRING类型。因此在调用`forward`时，就必须将所有输出值转成STRING类型。

## 引用资源 {#section_bj3_h3f_xdb .section}

Python UDF可以通过`odps.distcache`模块引用资源文件，目前支持引用文件资源和表资源。

-   `odps.distcache.get_cache_file(resource_name)`使用说明：

    -   返回指定名字的资源内容。`resource_name`为STRING类型，对应当前Project中已存在的资源名。如果资源名非法或者没有相应的资源，会抛出异常。
    -   返回值为ile-like Object。在使用完这Object后，调用者应该调用`close`方法释放打开的资源文件。
    使用`get_cache_file`的示例如下。

    ``` {#codeblock_8r4_yk3_a08}
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

-   `odps.distcache.get_cache_table(resource_name)`使用说明：

    -   返回指定资源表的内容。`resource_name`为STRING类型，对应当前Project中已存在的资源表名。如果资源名非法或者没有相应的资源，会抛出异常。
    -   返回值为Generator类型，调用者通过遍历获取表的内容，每次遍历得到的是以Tuple形式存在的表中的一条记录。
    使用`get_cache_table`的示例如下。

    ``` {#codeblock_55j_e4h_czy}
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


