# MaxCompute数据迁移至OSS {#task_1375125 .task}

本文为您介绍如何将MaxCompute数据迁移至Object Storage Service（OSS）。

请提前开通MaxCompute和DataWorks服务。本例使用DataWorks简单模式。

1.  在DataWorks控制台中创建表。 
    1.  新建业务流程，并将业务流程命名为mc2oss。![新建业务流程](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1240529/156518493254426_zh-CN.jpg)


    2.  新建表transition。![新建表](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1240529/156518493254478_zh-CN.jpg)


    3.  单击**添加字段**，创建表字段name、id、gender。![添加字段](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1240529/156518493254476_zh-CN.jpg)


    4.  单击**提交​到生产环境**。![提交到生产环境](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1240529/156518493354477_zh-CN.jpg)


    5.  单击**导入数据**。![导入数据](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1240529/156518493354647_zh-CN.jpg)


    6.  导入表数据。![导入表数据](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1240529/156518493354487_zh-CN.jpg)

 表数据如下。

        ``` {#codeblock_h42_ql7_80a}
        qwe,145,F
        asd,256,F
        xzc,345,M
        rgth,234,F
        ert,456,F
        dfg,12,M
        tyj,4,M
        bfg,245,M
        nrtjeryj,15,F
        rwh,2344,M
        trh,387,F
        srjeyj,67,M
        saerh,567,M
        ```

2.  在对象存储（OSS）控制台中创建表。 
    1.  登录[对象存储（OSS）控制台](https://oss.console.aliyun.com/overview)，创建Bucket。![创建表](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1240529/156518493354569_zh-CN.jpg)


    2.  单击**文件管理** \> **上传文件**。![上传文件](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1240529/156518493454515_zh-CN.jpg)


    3.  在上传文件页签中，传入文件qwee.csv。![传入文件](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1240529/156518493454593_zh-CN.jpg)

 请确保qwee.csv文件中的字段与表transition的字段完全一致。
3.  新增数据源。 
    1.  单击**新增数据源**。![新增数据源](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1095436/156518493453716_zh-CN.jpg)


    2.  新增MaxCompute（ODPS）数据源，并将数据源命名为odps\_first。![新增数据源](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1095436/156518493553761_zh-CN.jpg)


    3.  新增表格存储（OSS）数据源，并将数据源命名为Trans。![新增数据源](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1240529/156518493554542_zh-CN.jpg)


4.  配置MaxCompute（ODPS）Reader和对象存储（OSS）Writer。 
    1.  新建数据同步节点mc2oss。![新建节点](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1240529/156518493554544_zh-CN.jpg)


    2.  在脚本模式中，导入模板。![导入模板](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1240529/156518493554546_zh-CN.jpg)


    3.  修改JSON代码后，单击运行按钮。![运行](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1240529/156518493554550_zh-CN.jpg)

 代码如下。

        ``` {#codeblock_dre_er1_x0u}
        {
            "order":{
                "hops":[
                    {
                        "from":"Reader",
                        "to":"Writer"
                    }
                ]
            },
            "setting":{
                "errorLimit":{
                    "record":"0"
                },
                "speed":{
                    "concurrent":1,
                    "dmu":1,
                    "throttle":false
                }
            },
            "steps":[
                {
                    "category":"reader",
                    "name":"Reader",
                    "parameter":{
                        "column":[
                            "name",
                            "id",
                            "gender"
                        ],
                        "datasource":"odps_first",
                        "partition":[],
                        "table":"Transs"
                    },
                    "stepType":"odps"
                },
                {
                    "category":"writer",
                    "name":"Writer",
                    "parameter":{
                        "datasource":"Trans",
                        "dateFormat":"yyyy-MM-dd HH:mm:ss",
                        "encoding":"UTF-8",
                        "fieldDelimiter":",",
                        "fileFormat":"csv",
                        "nullFormat":"null",
                        "object":"qweee.csv",
                        "writeMode":"truncate"
                    },
                    "stepType":"oss"
                }
            ],
            "type":"job",
            "version":"2.0"
        }
        							
        ```

5.  在对象存储（OSS）控制台中查看新增的表数据。![查看结果](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1240529/156518493654631_zh-CN.jpg)



