# SQL {#concept_hqh_5vf_cfb .concept}

PyODPS支持ODPS SQL的查询和执行结果的读取。

`execute_sql`和`run_sql`方法的返回值是[运行实例](cn.zh-CN/开发/PyODPS/基本操作/任务实例.md#) 。

**说明：** 在ODPS Console中可执行的命令并非都是ODPS接受的SQL语句。 在调用非DDL/DML语句时，请使用其它方法，例如，调用GRANT/REVOKE等语句时，请使用`run_security_query`方法；调用PAI命令时，请使用`run_xflow`或`execute_xflow`方法。

## 执行SQL {#section_ntm_54m_cfb .section}

``` {#codeblock_h2p_y60_6e1 .language-sql}
o.execute_sql('select * from dual')  #  同步的方式执行，会阻塞直到SQL执行完成。
instance = o.run_sql('select * from dual')  # 异步的方式执行。
print(instance.get_logview_address())  # 获取logview地址。
instance.wait_for_success()  # 阻塞直到完成。
```

## 设置运行参数 {#section_jr3_w4m_cfb .section}

您可以通过设置`hints`来设置运行时参数，参数类型是dict。

``` {#codeblock_iss_vxu_h0o .language-sql}
o.execute_sql('select * from pyodps_iris', hints={'odps.sql.mapper.split.size': 16})
```

如果全局配置设置了`sql.settings`，则每次运行时都会添加相关的运行时参数。

``` {#codeblock_c3g_sn1_wbw .language-sql}
from odps import options
options.sql.settings = {'odps.sql.mapper.split.size': 16}
o.execute_sql('select * from pyodps_iris')  # 会根据全局配置添加hints。
```

## 读取SQL执行结果 {#section_uv2_1pm_cfb .section}

运行SQL的Instance能够直接执行`open_reader`的操作，一种情况是SQL返回了结构化的数据。

``` {#codeblock_u2r_9ec_c34 .language-sql}
with o.execute_sql('select * from dual').open_reader() as reader:
    for record in reader:
    # 处理每一个record。
```

另一种情况是SQL可能执行了`desc`命令，这时可以通过`reader.raw`取到原始的SQL执行结果。

``` {#codeblock_lls_oat_5dg .language-sql}
with o.execute_sql('desc dual').open_reader() as reader:
print(reader.raw)
```

如果`options.tunnel.use_instance_tunnel == True`，在调用`open_reader`时，PyODPS会默认调用Instance Tunnel，否则会调用旧的Result接口。如果您使用了版本较低的MaxCompute服务，或者调用Instance Tunnel出现了问题，PyODPS会给出警告并自动降级到旧的Result接口，您可根据警告信息判断导致降级的原因。如果Instance Tunnel的结果不合预期，请将该选项设为False。在调用`open_reader`时，也可以使用 `tunnel`参数来指定使用何种结果接口。

``` {#codeblock_xk5_wfe_8bo .language-sql}
# 使用Instance Tunnel。
with o.execute_sql('select * from dual').open_reader(tunnel=True) as reader:
    for record in reader:
    # 处理每一个record。
# 使用Results接口。
with o.execute_sql('select * from dual').open_reader(tunnel=False) as reader:
    for record in reader:
    # 处理每一个record。
```

PyODPS默认不限制从Instance读取的数据规模。对于受保护的Project，通过Tunnel下载数据将受限。此时，如果未设置`options.tunnel.limit_instance_tunnel`，则数据量限制会被自动打开。此时，可下载的数据条数受到Project配置限制，通常该限制为10000条。如果您需要手动限制下载数据的规模，可以为`open_reader`方法增加limit选项， 或者设置`options.tunnel.limit_instance_tunnel = True` 。

如果您使用的MaxCompute只能支持旧Result接口，并且需要读取所有的数据，您可将SQL结果写入另一张表后用读表接口读取（可能受到Project安全设置的限制）。

从PyODPS v0.7.7.1开始，您可以通过`open_reader`方法使用[Instance tunnel](cn.zh-CN/开发/数据上传下载/批量数据通道SDK介绍/InstanceTunnel.md#)来获取全量数据。

``` {#codeblock_a84_5m6_1s9}
instance = o.execute_sql('select * from movielens_ratings limit 20000')
with instance.open_reader() as reader:
    print(reader.count)
    # for record in reader 遍历这2万条数据，这里通过切片只取10条。
    for record in reader[:10]:  
        print(record)
```

## 设置alias {#section_h2h_qpm_cfb .section}

在运行时，如果某个UDF引用的资源是动态变化的，您可以将旧的资源名alias到新的资源，无需重新删除或创建新的UDF。

``` {#codeblock_fod_57y_uem .language-sql}
from odps.models import Schema
myfunc = '''\
from odps.udf import annotate
from odps.distcache import get_cache_file

@annotate('bigint->bigint')
class Example(object):
    def __init__(self):
        self.n = int(get_cache_file('test_alias_res1').read())

    def evaluate(self, arg):
        return arg + self.n
'''
res1 = o.create_resource('test_alias_res1', 'file', file_obj='1')
o.create_resource('test_alias.py', 'py', file_obj=myfunc)
o.create_function('test_alias_func',
                  class_type='test_alias.Example',
                  resources=['test_alias.py', 'test_alias_res1'])

table = o.create_table(
    'test_table',
    schema=Schema.from_lists(['size'], ['bigint']),
    if_not_exists=True
)

data = [[1, ], ]
# 写入一行数据，只包含一个值1。
o.write_table(table, 0, [table.new_record(it) for it in data])

with o.execute_sql(
    'select test_alias_func(size) from test_table').open_reader() as reader:
    print(reader[0][0])
res2 = o.create_resource('test_alias_res2', 'file', file_obj='2')
# 把内容为1的资源alias成内容为2的资源，您不需要修改UDF或资源。
with o.execute_sql(
    'select test_alias_func(size) from test_table',
    aliases={'test_alias_res1': 'test_alias_res2'}).open_reader() as reader:
    print(reader[0][0])
```

## 在交互式环境执行SQL {#section_xgv_vpm_cfb .section}

在IPython和Jupyter里支持[使用SQL插件的方式运行SQL](cn.zh-CN/开发/PyODPS/交互体验增强/命令行增强.md#)和[参数化查询](cn.zh-CN/开发/PyODPS/交互体验增强/IPython增强.md#)。

## 设置biz\_id {#section_p3z_1sm_cfb .section}

在少数情形下，提交SQL时需要同时提交`biz_id`，否则执行会报错。此时，您可以通过设置全局`options`里的`biz_id`解决此类报错。

``` {#codeblock_p5t_s6o_uet .language-sql}
from odps import options
options.biz_id = 'my_biz_id'
o.execute_sql('select * from pyodps_iris')
```

