---
keyword: [Python 3, UDF, UDAF, UDTF]
---

# Python 3 UDF

Python 2官方即将停止维护，MaxCompute支持Python 3。MaxCompute使用的Python 3版本为CPython-3.7.3。本文为您介绍如何通过MaxCompute Python 3 UDF创建UDF、UDAF和UDTF三种函数。

## 使用限制

Python 3与Python 2不兼容。由于在一个SQL作业中，只能选定Python 3或Python 2版本来执行，所以在您使用Python 3之前，需要考虑兼容性问题，在一个SQL里不允许同时使用Python 3和Python 2。

## 开启Python 3

只有作业级别支持Python 3。您在通过Python 3执行SQL语句时，增加如下命令，即可开启Python 3。

```
set odps.sql.python.version=cp37;
```

## 第三方库

MaxCompute Python UDF支持第三方库，Python 2运行环境中安装了第三方库Numpy，作为标准库的补充。

MaxCompute内置的Python 3运行环境中未安装第三方库Numpy。如果您需要使用Numpy的UDF，请手动上传Numpy的WHEEL包。从PyPI或镜像下载Numpy包时，包的文件名为numpy-<版本号\>-cp37-cp37m-manylinux1\_x86\_64.whl。上传包的操作请参见[资源操作](/cn.zh-CN/开发/常用命令/资源操作.md)或[PyODPS使用第三方包](/cn.zh-CN/开发/PyODPS/示例程序/PyODPS使用第三方包.md)。

## Python 2 UDF迁移

Python 2官方即将停止维护，建议您根据项目类型执行迁移操作：

-   全新项目：适用于新项目空间，或者第一次使用Python编写UDF的项目空间。建议所有的Python UDF都直接使用Python 3编写。
-   存量项目：适用于使用了大量Python 2 UDF的项目空间，请您谨慎开启Python 3。因为在一个SQL中不允许同时使用Python 3和Python 2 。如果您计划逐步将所有Python 2 UDF迁移成Python 3 UDF，推荐如下方法：
    -   新作业和新UDF：使用Python 3编写，设置作业级别开启Python 3。
    -   Python 2 UDF：改写Python 2 UDF，使其可以同时兼容Python 2和Python 3。改写方法请参见[将Python 2代码移植到Python 3](https://docs.python.org/3/howto/pyporting.html)。

        **说明：** 如果您需要编写公共UDF，并为多个项目空间授权，建议同时兼容Python 2和Python 3。


## 参数与返回值

参数与返回值的指定方式如下。

```
@odps.udf.annotate(signature)
```

Python UDF支持的MaxCompute SQL数据类型包括BIGINT、STRING、DOUBLE、BOOLEAN、DATETIME、DECIMAL、复杂数据类型（ARRAY、MAP和STRUCT）和复杂数据类型嵌套。在执行SQL语句前，必须确定所有函数的参数类型和返回值类型，因此对于Python这一动态类型语言，需要通过对UDF类加Decorator的方式指定函数签名。

函数签名Signature通过字符串指定，命令格式如下。

```
arg_type_list '->' type_list
arg_type_list: type_list | '*' | '' | 'char(n)' | 'varchar(n)'
type_list: [type_list ','] type
type: 'bigint' | 'string' | 'double' | 'boolean' | 'datetime' | 'float' | 'binary' | 'date' | 'decimal' | 'decimal(precision,scale)'
```

-   箭头左边表示参数类型，右边表示返回值类型。
-   只有UDTF的返回值可以是多列，UDF和UDAF只能返回一列。
-   星号（`*`）代表变长参数。使用变长参数时，UDF、UDTF和UDAF可以匹配任意输入参数。
-   如果项目空间使用2.0数据类型版本，decimal可以设置precision和scale，复杂类型写法请参见[2.0数据类型版本](/cn.zh-CN/开发/数据类型/2.0数据类型版本.md)。
-   返回值暂不支持CHAR和VARCHAR类型。

合法的函数签名Signature示例如下。

```
'bigint,double->string'            # 参数类型为BIGINT、DOUBLE，返回值类型为STRING。
'bigint,boolean->string,datetime'  # UDTF参数类型为BIGINT、BOOLEAN，返回值类型为STRING，DATETIME。
'*->string'                        # 变长参数，输入参数类型任意，返回值类型为STRING。
'->double'                         # 参数为空，返回值类型为DOUBLE。
'array<bigint>->struct<x:string, y:int>' # 参数类型为ARRAY<BIGINT>，返回值类型为STRUCT<x:STRING, y:BIGINT>。
'->map<bigint, string>'            # 参数为空，返回值类型为MAP<INT, STRING>。
```

查询语义解析阶段会检查不符合函数签名的用法，并返回错误，禁止执行此函数。执行时，UDF函数的参数会以函数签名指定的类型传入。返回值类型也要与函数签名指定的类型一致，否则检查到类型不匹配时也会报错。

MaxCompute SQL数据类型对应的Python类型如下。

|MaxCompute SQL Type|Python 3 Type|
|-------------------|-------------|
|BIGINT|INT|
|STRING|UNICODE|
|DOUBLE|FLOAT|
|BOOLEAN|BOOL|
|DATETIME|DATETIME.DATETIME|
|FLOAT|FLOAT|
|CHAR|UNICODE|
|VARCHAR|UNICODE|
|BINARY|BYTES|
|DATE|DATETIME.DATE|
|DECIMAL|DECIMAL.DECIMAL|
|ARRAY|LIST|
|MAP|DICT|
|STRUCT|COLLECTIONS.NAMEDTUPLE|

**说明：**

-   DATETIME类型对应的Python类型是INT，值为Epoch UTC Time起至今的毫秒数。您可以通过Python标准库中的DATETIME模块处理日期时间类型。
-   `odps.udf.int(value,[silent=True])`增加了参数`silent`。当`silent`为True时，如果`value`无法转为INT，则会返回None（不会返回异常）。
-   NULL值对应Python的None。

## UDF

定义一个New-style Class，通过`evaluate`方法，即可实现Python UDF。

```
from odps.udf import annotate
@annotate("bigint,bigint->bigint")
class MyPlus(object):
   def evaluate(self, arg0, arg1):
       if None in (arg0, arg1):
           return None
       return arg0 + arg1
```

**说明：** Python UDF必须通过`annotate`指定函数签名。

Python UDF的使用示例请参考[使用MaxCompute分析IP来源最佳实践](/cn.zh-CN/最佳实践/数据开发/使用MaxCompute分析IP来源最佳实践.md)。

## UDAF

-   `class odps.udf.BaseUDAF`：继承此类实现Python UDAF。
-   `BaseUDAF.new_buffer()`：返回聚合函数的中间值的`buffer`。`buffer`必须是[Marshal](https://docs.python.org/3.3/library/marshal.html#module-marshal)对象（例如LIST、DICT），并且`buffer`的大小不应该随数据量递增。在极限情况下，`buffer`在执行对象序列化后的大小不应该超过2 MB。
-   `BaseUDAF.iterate(buffer[, args, ...])`：将`args`聚合到中间值`buffer`中。
-   `BaseUDAF.merge(buffer, pbuffer)`：将两个中间值`buffer`聚合到一起，即将`pbuffer`合并到`buffer`中。
-   `BaseUDAF.terminate(buffer)`：将中间值`buffer`转换为MaxCompute SQL的基本类型。

使用UDAF计算平均值的示例如下。

```
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

## UDTF

-   `class odps.udf.BaseUDTF`：Python UDTF的基类。您可以继承此类实现`process`或`close`等方法。
-   `BaseUDTF.__init__()`：初始化方法。继承类如果需要实现此方法，必须在一开始调用基类的初始化方法`super(BaseUDTF, self).__init__()`。`init`方法在整个UDTF生命周期中只会被调用一次，即在处理第一条记录之前。当UDTF需要保存内部状态时，可以通过此方法初始化所有状态。
-   `BaseUDTF.process([args, ...])`：此方法由MaxCompute SQL框架调用，SQL中每一条记录都会调用一次`process`，`process`的参数为SQL语句中指定的UDTF输入参数。
-   `BaseUDTF.forward([args, ...])`：UDTF的输出方法。此方法由用户代码调用。每调用一次`forward`，便会输出一条记录。 `forward`的参数为SQL语句中指定的UDTF的输出参数。
-   `BaseUDTF.close()`：UDTF的结束方法。此方法由MaxCompute SQL框架调用，并且只会被调用一次，即在处理完最后一条记录之后。

UDTF的示例如下。

```
#coding:utf-8
# explode.py
from odps.udf import annotate
from odps.udf import BaseUDTF
@annotate('string -> string')
class Explode(BaseUDTF):
   #将string按逗号分隔输出成多条记录。
   def process(self, arg):
       props = arg.split(',')
       for p in props:
           self.forward(p)
```

**说明：** Python UDTF可以不通过`annotate`指定参数类型和返回值类型。这样，在SQL中使用函数时可以匹配任意输入参数，但无法推导返回值类型，所有输出参数都将被视为STRING类型。因此在调用`forward`时，必须将所有输出值转换为STRING类型。

## 引用资源

Python UDF可以通过`odps.distcache`模块引用资源文件，支持引用文件资源和表资源。

-   `odps.distcache.get_cache_file(resource_name)`：返回指定名字的资源内容。

    -   `resource_name`为STRING类型，对应当前项目空间中已存在的资源名。如果资源名非法或者没有相应的资源，则会返回异常。

        **说明：** 使用UDF访问资源，在创建UDF时需要声明引用的资源，否则会报错。

    -   返回值为File-like对象。在使用完此对象后，您需要调用`close`方法释放打开的资源文件。
    示例如下。

    ```
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

-   `odps.distcache.get_cache_table(resource_name)`：返回指定资源表的内容。
    -   `resource_name`支持BIGINT、STRING、DOUBLE、BOOLEAN、DATETIME、FLOAT、CHAR、VARCHAR、BINARY、DATE、DECIMAL、ARRAY、MAP和STRUCT数据类型。对应当前Project中已存在的资源表名。如果资源名非法或者没有相应的资源，会返回异常。
    -   返回值为Generator类型，调用者通过遍历获取表的内容，每次遍历得到的是以数组形式存在的表中的一条记录。

示例如下。

```
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

