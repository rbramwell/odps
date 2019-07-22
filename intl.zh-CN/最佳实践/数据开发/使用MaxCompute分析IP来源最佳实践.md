# 使用MaxCompute分析IP来源最佳实践 {#concept_q25_ygl_5fb .concept}

本文将为您介绍如何在MaxCompute上分析IP来源，包括下载、上传IP地址库数据及编写UDF函数、编写SQL四个步骤。

## 背景介绍 {#section_rfz_jz1_pgb .section}

[淘宝IP地址库](http://ip.taobao.com/)的查询接口为[IP地址字串](http://ip.taobao.com/service/getIpInfo.php?ip=[ip%E5%9C%B0%E5%9D%80%E5%AD%97%E4%B8%B2])，使用示例如下。

![查询接口](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/63437/156379529331905_zh-CN.png)

由于在MaxCompute中禁止使用HTTP请求，目前可以通过以下三种方式，实现在MaxCompute中查询IP。

-   用SQL将数据查询到本地，再发起HTTP请求查询。

    **说明：** 效率低下，且淘宝IP库查询频率需小于10QPS，否则拒绝请求。

-   下载IP地址库到本地，进行查询。

    **说明：** 同样效率低，且不利于数仓等分析使用。

-   将IP地址库定期维护上传至MaxCompute，进行连接查询。本文重点为您介绍该方式。

    **说明：** 比较高效，但是IP地址库需自己定期维护。


## 下载IP地址库 {#section_xwc_y3l_5fb .section}

1.  首先您需要获取地址库数据。地址库您可以自行获取，本文仅提供一个[UTF8格式的不完整的地址库demo](http://docs-aliyun.cn-hangzhou.oss.aliyun-inc.com/assets/attach/102762/cn_zh/1547530733280/ipdata.txt.utf8)。
2.  下载UTF-8地址库数据到本地后，检查数据格式，示例如下。

    ![检查格式](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/63437/156379529331907_zh-CN.png)

    前四个数据是IP地址的起始地址与结束地址：前两个是十进制整数形式，后两个是点分形式。这里我们使用整数形式，以便计算IP是否属于这个网段。


**说明：** 如果您需要使用真实IP地址，请自行下载IP地址库，具体的下载地址和使用方式请参见[云栖社区](https://yq.aliyun.com/articles/68432)。

## 上传IP地址库数据 {#section_eqh_pjl_5fb .section}

1.  创建表DDL，您可以使用[MaxCompute客户端](../../../../intl.zh-CN/工具及下载/客户端.md#)进行操作，也可以使用DataWorks进行[图形化建表](../../../../intl.zh-CN/使用指南/数据开发/表管理.md#)。

    ``` {#codeblock_ndw_ic6_png .language-sql}
    DROP TABLE IF EXISTS ipresource ;
    
    CREATE TABLE IF NOT EXISTS ipresource 
    (
        start_ip BIGINT
        ,end_ip BIGINT
        ,start_ip_arg string
        ,end_ip_arg string
        ,country STRING
        ,area STRING
        ,city STRING
        ,county STRING
        ,isp STRING
    );
    ```

2.  使用[Tunnel上传下载命令](../../../../intl.zh-CN/开发/数据上传下载/Tunnel上传下载命令.md#)上传您的文件，本例中ipdata.txt.utf8文件存放在D盘。

    ``` {#codeblock_lx6_w29_6b4 .language-sql}
    odps@ workshop_demo>tunnel upload D:/ipdata.txt.utf8 ipresource;
    ```

    可以通过SQL语句`select count(*) from ipresource;`查看到表中上传的数据条数（通常地址库由于有人更新维护，条目数会不断增长）。

3.  使用SQL语句`select * from ipresource limit 10;`查看ipresource表前10条的样本数据，示例如下。

    ![查看样本](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/63437/156379529331909_zh-CN.png)


## 编写UDF函数 {#section_uht_3kl_5fb .section}

1.  右键单击相应业务流程下的**资源**，选择**新建资源** \> **Python**。

    ![确定](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/63437/156379529431910_zh-CN.png)

2.  在**新建资源**对话框中，填写**资源名称**，并勾选**上传为ODPS资源**，单击**确定**。
3.  在新建的Python资源内，编写Python资源代码，示例如下。

    ``` {#codeblock_ucw_v89_mra .language-sql}
    from odps.udf import annotate
    @annotate("string->bigint")
    class ipint(object):
        def evaluate(self, ip):
            try:
                return reduce(lambda x, y: (x << 8) + y, map(int, ip.split('.')))
            except:
                return 0
    ```

    单击**提交并解锁**。

    ![提交并解锁](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/63437/156379529431911_zh-CN.png)

4.  右键单击相应业务流程下的**函数**，选择**新建函数**。
5.  在新建函数对话框中，填写**函数名称**，单击**提交**。
6.  编辑函数配置，单击**提交并解锁**。

    ![提交并解锁](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/63437/156379529431913_zh-CN.png)

    本示例中，函数的类名为`ipint.ipint`，资源列表填写上文提交的资源的名称。

7.  验证ipint函数是否生效并满足预期，您可以在DataWorks上新建一个ODPS SQL类型节点，执行SQL语句进行查询，示例如下。

    ![执行SQL](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/63437/156379529431914_zh-CN.png)


您也可以在本地创建ipint.py文件，使用[MaxCompute客户端](../../../../intl.zh-CN/工具及下载/客户端.md#)上传资源。

``` {#codeblock_ara_p7n_zpi .language-sql}
odps@ MaxCompute_DOC>add py D:/ipint.py;
OK: Resource 'ipint.py' have been created.
```

完成上传后，使用客户端[注册函数](../../../../intl.zh-CN/开发/常用命令/函数操作.md#)。

``` {#codeblock_u0j_21j_gnc .language-sql}
odps@ MaxCompute_DOC>create function ipint as ipint.ipint using ipint.py;
Success: Function 'ipint' have been created.
```

完成注册后，即可使用该函数。您可以在客户端运行`select ipint('1.2.24.2');`进行测试。

**说明：** 如果同一主账号下其他项目需要使用这个UDF，您可以进行[跨项目授权](../../../../intl.zh-CN/管理/安全功能详解/跨项目空间的资源分享/基于Package的跨项目空间的资源分享.md#)。

1.  创建名为ipint的package。

    ``` {#codeblock_jad_4wq_evf .language-sql}
    odps@ MaxCompute_DOC>create package ipint;
    OK
    ```

2.  将已经创建好的UDF函数加入package。

    ``` {#codeblock_r6e_xyx_349 .language-sql}
    odps@ MaxCompute_DOC>add function ipint to package ipint;
    OK
    ```

3.  允许另外一个项目bigdata\_DOC安装这个package。

    ``` {#codeblock_7fp_exx_d5q .language-sql}
    odps@ MaxCompute_DOC> allow project bigdata_DOC to install package ipint;
    OK
    ```

4.  切换到另一个需要使用UDF的项目bigdata\_DOC，安装package。

    ``` {#codeblock_p9i_tad_6gd}
    odps@ MaxCompute_DOC>use bigdata_DOC;
    odps@ bigdata_DOC>install package MaxCompute_DOC.ipint;
    OK
    ```

5.  现在您就可以使用这个UDF函数了， 如果项目空间bigdata\_DOC的用户Bob需要访问这些资源，那么管理员可以通过ACL给Bob自主授权。

    ``` {#codeblock_lss_8c5_2pa .language-sql}
    odps@ bigdata_DOC>grant Read on package MaxCompute_DOC.ipint to user aliyun$bob@aliyun.com; --通过ACL授权Bob使用package
    ```


## 在SQL中使用 {#section_rmm_bml_5fb .section}

**说明：** 本文以一个随机的IP 1.2.24.2地址为例，您在使用时可以用具体表的字段来读入。

测试使用的SQL代码如下，单击**运行**即可查看结果。

``` {#codeblock_1ez_k3q_2fb .language-sql}
select * from ipresource
WHERE ipint('1.2.24.2') >= start_ip
AND ipint('1.2.24.2') <= end_ip
```

![执行](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/63437/156379529431915_zh-CN.png)

通过为保证数据准确性，您可以定期从淘宝IP库获取数据来维护ipresource表。

