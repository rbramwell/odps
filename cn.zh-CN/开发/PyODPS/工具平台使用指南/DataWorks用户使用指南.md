# DataWorks用户使用指南 {#concept_ch1_lwf_cfb .concept}

本文为您介绍如何在DataWorks上使用PyODPS和使用限制。

## 新建工作流节点 {#section_kbb_lkg_cfb .section}

在工作流节点中会包含PYODPS节点，新建即可。如图所示。

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/21169/156016538711645_zh-CN.png)

## ODPS入口 {#section_rfp_skg_cfb .section}

DataWorks的PyODPS节点中，将会包含一个全局的变量`odps`或者`o` ，即ODPS入口。用户不需要手动定义ODPS入口。

```language-sql
print(o.exist_table('pyodps_iris'))
```

## 执行SQL {#section_h21_vkg_cfb .section}

可以参考[执行SQL文档](cn.zh-CN/开发/PyODPS/基本操作/SQL.md#) 。

**说明：** Dataworks上默认没有打开instance tunnel，即instance.open\_reader默认走result 接口（最多一万条）。 打开instance tunnel，通过reader.count能取到记录数，如果要迭代获取全部数据，则需要关闭limit限制。

要想全局打开，则：

```language-sql
options.tunnel.use_instance_tunnel = True
options.tunnel.limit_instance_tunnel = False  # 关闭limit读取全部数据

with instance.open_reader() as reader:
    # 能通过instance tunnel读取全部数据
```

或者通过在open\_reader上添加`tunnel=True`，来仅对这次open\_reader 打开instance tunnel。添加 `limit=False`，来关闭limit限制从而能下载全部数据。

```language-sql
with instance.open_reader(tunnel=True, limit=False) as reader:
    # 这次 open_reader 会走 instance tunnel 接口，且能读取全部数据
```

**说明：** 若未打开Instance tunnel导致获取数据格式错误，解决方法请参见[PyODPS常见问题](https://help.aliyun.com/knowledge_detail/88457.html)最后一个小节。

## DataFrame {#section_hlv_plg_cfb .section}

-   执行

    在DataWorks的环境里，[DataFrame概述](cn.zh-CN/开发/PyODPS/DataFrame/快速开始.md#) 的执行需要显式调用[立即执行的方法（如execute，head等）](cn.zh-CN/开发/PyODPS/DataFrame/执行.md#) 。

    ```language-sql
    from odps.df import DataFrame
    
    iris = DataFrame(o.get_table('pyodps_iris'))
    for record in iris[iris.sepal_width < 3].execute():  # 调用立即执行的方法
        # 处理每条record
    ```

    如果用户想在print的时候调用立即执行，需要打开 `options.interactive` 。

    ```language-sql
    from odps import options
    from odps.df import DataFrame
    
    options.interactive = True  # 在开始打开开关
    
    iris = DataFrame(o.get_table('pyodps_iris'))
    print(iris.sepal_width.sum())  # 这里print的时候会立即执行
    ```

-   打印详细信息

    通过设置`options.verbose`选项。在DataWorks上，默认已经处于打开状态，运行过程会打印logview等详细过程。


## 获取调度参数 {#section_f5s_lmg_cfb .section}

与DataWorks中的SQL节点不同，为了避免侵入代码，PyODPS节点不会在代码中替换 $\{param\_name\}这样的字符串，而是在执行代码前，在全局变量中增加一个名为`args`的dict，调度参数可以在此获取。例如， 在**节点基本属性** \> **参数**中设置`ds=${yyyymmdd}`，则可以通过下面的方式在代码中获取此参数：

```language-sql
print('ds=' + args['ds'])
```

```language-sql
ds=20161116
```

**说明：** 

如果要获取名为`ds=${yyyymmdd}`的分区，则可以使用：

```language-sql
o.get_table('table_name').get_partition('ds=' + args['ds'])
```

## 受限功能 {#section_rtk_wpg_cfb .section}

由于缺少matplotlib等包，所以如下功能可能会受限。

-   DataFrame的plot函数

DataFrame自定义函数需要提交到MaxCompute执行。由于Python沙箱的原因，第三方库只支持所有的纯Python库以及Numpy， 因此不能直接使用Pandas，可参考 第三方库支持<third\_party\_library\>上传和使用所需的库。DataWorks中执行的非自定义函数代码可以使用平台预装的Numpy和Pandas。其他带有二进制代码的三方包不被支持。

由于兼容性的原因，在DataWorks中，options.tunnel.use\_instance\_tunnel默认设置为False。如果需要全局开启Instance Tunnel， 需要手动将该值设置为True。

由于实现的原因，Python的atexit包不被支持，请使用try - finally结构实现相关功能。

## 使用限制 {#section_ixn_ypg_cfb .section}

在DataWorks上使用PyODPS，为了防止对DataWorks的gateway造成压力，对内存和CPU都有限制。这个限制由DataWorks统一管理。

如果看到**Got killed**，即内存使用超限，进程被kill。因此，尽量避免本地的数据操作。

通过PyODPS起的SQL和DataFrame任务（除to\_pandas\) 不受此限制。

