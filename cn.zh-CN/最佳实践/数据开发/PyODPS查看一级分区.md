# PyODPS查看一级分区 {#task_1461379 .task}

本文为您介绍如何在PyODPS中查看一级分区。

请提前开通MaxCompute和DataWorks服务。

1.  准备测试数据。 
    1.  单击[此处](http://t.cn/Rf8GeUq)下载鸢尾花数据集iris.csv文件。
    2.  登录Dataworks控制台，以**DDL模式**创建分区表user\_detail。![创建分区表](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1014080/156462470452566_zh-CN.jpg)

 代码如下。

        ``` {#codeblock_514_5le_rqm}
        create table if not exists user_detail
        (
        userid    BIGINT comment '用户id',
        job       STRING comment '工作类型',
        education STRING comment '教育程度'
        ) comment '用户信息表'
        partitioned by (dt STRING comment '日期',region STRING comment '地区');
        ```

    3.  将以下测试数据保存为user\_detail.txt文件。 

        ``` {#codeblock_4eq_ir6_xyx}
        0001,互联网,本科,20190715,beijing
        0002,教育,大专,20190716,beijing
        0003,金融,硕士,20190715,shandong
        0004,互联网,硕士,20190715,beijing
        ```

    4.  将user\_detail.txt中的数据导入到表user\_detail。![数据导入](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1014080/156462470452373_zh-CN.jpg)


2.  在**业务流程** \> **数据开发**中，新建PyODPS节点，并将节点命名为查看一级分区。![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1161778/156462470454007_zh-CN.jpg)

 代码如下。

    ``` {#codeblock_s3o_ksm_bif}
    import sys
    reload(sys)
    #修改系统默认编码。
    sys.setdefaultencoding('utf8')
    
    #异步方式读取一级分区。
    instance = o.run_sql('select * from user_detail WHERE dt=\'20190715\'')
    instance.wait_for_success()
    for record in instance.open_reader():
        print record["userid"],record["job"],record["education"]
    
    #同步方式读取一级分区。
    with o.execute_sql('select * from user_detail WHERE dt=\'20190715\'').open_reader() as reader4:
        print reader4.raw
        for record in reader4:
            print record["userid"],record["job"],record["education"]
    
    #使用ODPS的DataFrame获取一级分区。
    pt_df = DataFrame(o.get_table('user_detail').get_partition('dt=20190715'))
    print pt_df.head(10)
    ```

3.  单击运行按钮。![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1161778/156462470554008_zh-CN.jpg)


4.  在**运行日志**中查看运行结果。![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1161778/156462470554026_zh-CN.jpg)



