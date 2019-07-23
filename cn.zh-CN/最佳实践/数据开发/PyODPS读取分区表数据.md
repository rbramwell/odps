# PyODPS读取分区表数据 {#task_1253808 .task}

本文为您介绍如何通过PyODPS读取分区表数据。

1.  下载安装Python 2.7版本，并配置环境变量。
2.  配置IntelliJ IDEA与MaxCompute Studio的Python集成环境，详情请参见[Python开发使用须知](../../../../cn.zh-CN/工具及下载/MaxCompute Studio/开发Python程序/Python开发使用须知.md#)。
3.  在命令窗口运行pip install pyodps，安装pyodps库。

1.  准备数据 
    1.  登录Dataworks控制台，以**DDL模式**创建分区表user\_detail。![创建分区表](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1014080/156384872852566_zh-CN.jpg)

 代码如下。

        ``` {#codeblock_gsa_3pz_x5x}
        create table if not exists user_detail
        (
        userid    BIGINT comment '用户id',
        job       STRING comment '工作类型',
        education STRING comment '教育程度'
        ) comment '用户信息表'
        partitioned by (dt STRING comment '日期',region STRING comment '地区');
        ```

    2.  以**DDL模式**创建源数据表user\_detail\_ods。![创建源数据表](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1014080/156384872852567_zh-CN.jpg)

 代码如下。

        ``` {#codeblock_dzt_xfq_8fh}
        create table if not exists user_detail_ods
        (
          userid    BIGINT comment '用户id',
          job       STRING comment '工作类型',
          education STRING comment '教育程度',
          dt STRING comment '日期',
          region STRING comment '地区'
        );
        ```

    3.  将以下测试数据保存为user\_detail.txt文件。 

        ``` {#codeblock_ccu_gl3_jwv}
        0001,互联网,本科,20190715,beijing
        0002,教育,大专,20190716,beijing
        0003,金融,硕士,20190715,shandong
        0004,互联网,硕士,20190715,beijing
        ```

    4.  将user\_detail.txt中的数据导入到表user\_detail\_ods。![数据导入](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1014080/156384872952373_zh-CN.jpg)


    5.  在**业务流程** \> **数据开发**中，新建ODPS SQL节点，并将节点命名为test。![新建SQL节点](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1014080/156384872952375_zh-CN.jpg)

 test.sql代码如下。

        ``` {#codeblock_dg0_mnc_zpk}
        insert overwrite table user_detail partition (dt,region)
        select userid,job,education,dt,region from user_detail_ods;
        ```

    6.  单击运行按钮，将动态分区插入到分区表user\_detail。![数据插入](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1014080/156384873052376_zh-CN.jpg)


2.  通过DataWorks读取分区表数据 
    1.  在**业务流程** \> **数据开发**中，新建PyODPS节点，并将节点命名为读取分区表数据。![新建节点](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1014080/156384873052361_zh-CN.jpg)

 代码如下。

        ``` {#codeblock_fkh_s2f_mba}
        import sys
        from odps import ODPS
        reload(sys)
        print('dt=' + args['dt'])
        #修改系统默认编码。
        sys.setdefaultencoding('utf8')
        #个人信息凭证。
        t = o.get_table('user_detail')
        #获取分区表。
        print t.exist_partition('dt=20190715,region=beijing')
        #查看所有分区。
        for partition in t.partitions:
            print partition.name
        #您可以通过以下三种方式查询分区表数据。
        #第一种方式如下。
        with t.open_reader(partition='dt=20190715,region=beijing') as reader1:
            count = reader1.count
        print("第一种方式查询分区表数据：")
        for record in reader1:
            print record[0],record[1],record[2]
        #第二种方式如下。
        print("第二种方式查询分区表数据：")
        reader2 = t.open_reader(partition='dt=20190715,region=beijing')
        for record in reader2:
            print record["userid"],record["job"],record["education"]
        #第三种方式如下。
        print("第三种方式查询分表数据：")
        for record in o.read_table('user_detail', partition='dt=20190715,region=beijing'):
            print record["userid"],record["job"],record["education"]
        ```

    2.  在调度配置窗口中，将**参数**设置为dt=20190715。![调度配置](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1014080/156384873052362_zh-CN.jpg)


    3.  单击运行按钮。![单击运行](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1014080/156384873152363_zh-CN.jpg)


    4.  在**运行日志**中查看运行结果。![运行日志](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1014080/156384873152364_zh-CN.jpg)


3.  通过MaxCompute Studio读取分区表数据 
    1.  单击**File** \> **New** \> **Project**。![py文件](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1014080/156384873152368_zh-CN.png)


    2.  在弹出的New Project对话框中，勾选Python。单击**Next**。![选择Python](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1014080/156384873252367_zh-CN.jpg)


    3.  将**Project Name**设置为PythonPartition。单击**Finish**。![设置项目名称](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1014080/156384873252366_zh-CN.jpg)


    4.  右键单击**scripts**。再单击**New** \> **MaxCompte Python**创建PythonThree.py文件。![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1014080/156384873252369_zh-CN.png)

 代码如下。

        ``` {#codeblock_hgc_s5f_guo}
        # -*- coding: utf-8 -*-
        from odps import ODPS
        #个人信息凭证。
        o = ODPS('**your-access-id**','**your-secret-access-key**','**your-default-project**',endpoint='**your-end-point**')
        t = o.get_table('user_detail')
        #获取分区表。
        print t.exist_partition('dt=20190715,region=beijing')
        #查看所有分区。
        for partition in t.partitions:
            print partition.name
        #您可以通过以下三种方式查询分区表数据。
        #第一种方式如下。
        with t.open_reader(partition='dt=20190715,region=beijing') as reader1:
            count = reader1.count
        print("第一种方式查询分区表数据：")
        for record in reader1:
            print record[0],record[1],record[2]
        #第二种方式如下。
        print("第二种方式查询分区表数据：")
        reader2 = t.open_reader(partition='dt=20190715,region=beijing')
        for record in reader2:
            print record["userid"],record["job"],record["education"]
        #第三种方式如下。
        print("第三种方式查询分区表数据：")
        for record in o.read_table('user_detail', partition='dt=20190715,region=beijing'):
            print record["userid"],record["job"],record["education"]
        ```

    5.  查看运行结果。![运行结果](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1014080/156384873252365_zh-CN.jpg)



