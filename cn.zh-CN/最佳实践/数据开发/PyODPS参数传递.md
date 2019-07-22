# PyODPS参数传递 {#task_1253808 .task}

本文为您介绍如何在DataWorks中进行PyODPS参数的传递。

请提前开通MaxCompute和DataWorks服务。

1.  准备数据 
    1.  登录Dataworks控制台，以**DDL模式**创建分区表user\_detail。![创建分区表](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1014079/156375928652332_zh-CN.jpg)

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

    2.  以**DDL模式**创建源数据表user\_detail\_ods。![源数据表](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1014079/156375928752333_zh-CN.jpg)

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

    4.  将user\_detail.txt中的数据导入到表user\_detail\_ods。![数据导入](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1000780/156375928752234_zh-CN.jpg)


    5.  在**业务流程** \> **数据开发**中，新建ODPS SQL节点，并将节点命名为test。![新建SQL节点](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1000780/156375928752241_zh-CN.jpg)

 test.sql代码如下。

        ``` {#codeblock_dg0_mnc_zpk}
        insert overwrite table user_detail partition (dt,region)
        select userid,job,education,dt,region from user_detail_ods;
        ```

    6.  单击运行按钮，将动态分区插入到分区表user\_detail。![新动态分区插入](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1000780/156375928852269_zh-CN.jpg)


2.  传递参数 
    1.  在**业务流程** \> **数据开发**中，新建PyODPS节点，并将节点命名为传递参数示例。![传递参数](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1014079/156375928852334_zh-CN.jpg)

 代码如下。

        ``` {#codeblock_fkh_s2f_mba}
        import sys
        reload(sys)
        print('dt=' + args['dt'])
        #修改系统默认编码。
        sys.setdefaultencoding('utf8')
        #获取表。
        t = o.get_table('user_detail')
        #接受传入的分区参数。
        with t.open_reader(partition='dt=' + args['dt'] + ',region=beijing') as reader1:
            count = reader1.count
        print("查询分区表数据：")
        for record in reader1:
            print record[0],record[1],record[2]
        ```

    2.  在调度配置窗口中，将**参数**设置为undefineddt=20190715。![调度配置](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1014079/156375928952356_zh-CN.jpg)


    3.  单击高级运行（带参数运行）按钮。![高级运行](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1014079/156375928952357_zh-CN.jpg)


    4.  在**运行日志**中查看运行结果。![运行日志](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1014079/156375928952360_zh-CN.jpg)



